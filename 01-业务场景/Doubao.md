# Doubao

以下是Java大厂面试中高频的业务场景题分类列举，覆盖核心业务场景、技术落地、问题排查等维度，贴合大厂实际业务诉求：

## 一、核心业务逻辑设计类

1. 订单系统设计（如订单状态流转、幂等处理、超时未支付自动取消）
2. 库存管理场景（如秒杀/高并发下的库存扣减、防超卖、库存回滚）
3. 支付流程设计（如支付回调处理、支付状态同步、多支付渠道适配）
4. 用户体系设计（如用户注册/登录/鉴权、Token设计、第三方登录整合）
5. 消息通知场景（如短信/推送/邮件的批量发送、异步处理、重试机制）
6. 购物车场景（如购物车数据存储、合并、价格实时计算）

你列出的这些都是后端开发中非常核心且高频的业务场景，每个场景都涉及到高并发、数据一致性、异常处理等关键技术点。下面我会针对每个场景，提供**核心设计思路 + 关键代码实现 + 最佳实践**，帮助你理解并落地这些业务逻辑。

---

### 1. 订单系统设计

#### 核心设计思路
- **订单状态流转**：定义清晰的状态枚举（待支付、已支付、已取消、已发货、已完成等），状态变更需记录日志且只能单向流转。
- **幂等处理**：通过订单号 + 业务操作类型生成唯一幂等号，防止重复创建/支付订单。
- **超时未支付自动取消**：结合定时任务（XXL-Job/Quartz）+ 延迟队列（Redis ZSet/RocketMQ延迟消息）实现。

#### 关键代码实现
```java
// 1. 订单状态枚举
public enum OrderStatus {
    PENDING_PAYMENT(1, "待支付"),
    PAID(2, "已支付"),
    CANCELLED(3, "已取消"),
    DELIVERED(4, "已发货"),
    COMPLETED(5, "已完成");

    private final int code;
    private final String desc;
    // 构造器、getter省略
}

// 2. 幂等处理（基于Redis）
@Service
public class OrderIdempotentService {
    @Autowired
    private StringRedisTemplate redisTemplate;

    // 生成幂等号：订单号+操作类型（如CREATE_ORDER/PAY_ORDER）
    public String generateIdempotentKey(String orderNo, String operationType) {
        return "order:idempotent:" + orderNo + ":" + operationType;
    }

    // 验证幂等（SET NX EX：不存在则设置，带过期时间）
    public boolean checkIdempotent(String key, long expireSeconds) {
        return Boolean.TRUE.equals(redisTemplate.opsForValue()
                .setIfAbsent(key, "1", expireSeconds, TimeUnit.SECONDS));
    }
}

// 3. 超时取消订单（Redis延迟队列示例）
@Service
public class OrderTimeoutService {
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Autowired
    private OrderService orderService;

    // 下单时将订单加入延迟队列（30分钟后取消）
    public void addToDelayQueue(String orderNo) {
        long delayTime = System.currentTimeMillis() + 30 * 60 * 1000;
        redisTemplate.opsForZSet().add("order:delay:queue", orderNo, delayTime);
    }

    // 定时任务消费延迟队列
    @Scheduled(fixedRate = 60 * 1000) // 每分钟执行一次
    public void handleTimeoutOrder() {
        long now = System.currentTimeMillis();
        // 查询所有超时的订单
        Set<String> timeoutOrderNos = redisTemplate.opsForZSet()
                .rangeByScore("order:delay:queue", 0, now);
        if (CollectionUtils.isEmpty(timeoutOrderNos)) {
            return;
        }
        // 处理每个超时订单
        for (String orderNo : timeoutOrderNos) {
            // 加分布式锁，防止并发处理
            String lockKey = "order:lock:" + orderNo;
            boolean locked = Boolean.TRUE.equals(redisTemplate.opsForValue()
                    .setIfAbsent(lockKey, "1", 5, TimeUnit.MINUTES));
            if (locked) {
                try {
                    // 检查订单状态，仍为待支付则取消
                    Order order = orderService.getByOrderNo(orderNo);
                    if (order != null && OrderStatus.PENDING_PAYMENT.equals(order.getStatus())) {
                        orderService.cancelOrder(orderNo, "超时未支付自动取消");
                    }
                    // 从延迟队列移除
                    redisTemplate.opsForZSet().remove("order:delay:queue", orderNo);
                } finally {
                    // 释放锁
                    redisTemplate.delete(lockKey);
                }
            }
        }
    }
}
```

#### 关键说明
- 状态流转：通过枚举限制状态值，业务代码中校验状态变更的合法性（如已取消的订单不能再支付）。
- 幂等性：所有写操作（创建订单、支付、取消）都必须校验幂等号，防止重复请求。
- 超时取消：延迟队列 + 定时任务是轻量级方案，高并发场景建议用RocketMQ/RabbitMQ的延迟消息。

---

### 2. 库存管理场景
#### 核心设计思路
- **高并发库存扣减**：基于Redis预扣减 + 数据库最终一致性（异步同步），或直接使用数据库乐观锁/悲观锁。
- **防超卖**：扣减前校验库存，扣减操作保证原子性（Redis Lua脚本/数据库事务）。
- **库存回滚**：订单取消/支付失败时，回滚Redis和数据库库存（需处理回滚失败的重试机制）。

#### 关键代码实现
```java
@Service
public class StockService {
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Autowired
    private StockMapper stockMapper;
    @Autowired
    private TransactionTemplate transactionTemplate;

    // 1. 初始化库存到Redis（启动时执行）
    @PostConstruct
    public void initStockToRedis() {
        List<Stock> stockList = stockMapper.selectList(null);
        for (Stock stock : stockList) {
            redisTemplate.opsForValue().set("stock:count:" + stock.getSkuId(), stock.getStockCount().toString());
        }
    }

    // 2. 扣减库存（Redis Lua脚本保证原子性，防超卖）
    public boolean deductStock(String skuId, int num) {
        String script = """
                local stock = tonumber(redis.call('get', KEYS[1]))
                if not stock or stock < tonumber(ARGV[1]) then
                    return 0
                end
                redis.call('decrby', KEYS[1], ARGV[1])
                return 1
                """;
        // 执行Lua脚本
        Long result = redisTemplate.execute(
                new DefaultRedisScript<>(script, Long.class),
                Collections.singletonList("stock:count:" + skuId),
                String.valueOf(num)
        );
        // 脚本返回1表示扣减成功，0表示库存不足
        if (result != null && result == 1) {
            // 异步同步到数据库（高并发下避免直接操作DB）
            asyncSyncStockToDb(skuId);
            return true;
        }
        return false;
    }

    // 3. 异步同步库存到数据库
    @Async
    public void asyncSyncStockToDb(String skuId) {
        // 加分布式锁，防止重复同步
        String lockKey = "stock:sync:lock:" + skuId;
        boolean locked = Boolean.TRUE.equals(redisTemplate.opsForValue()
                .setIfAbsent(lockKey, "1", 10, TimeUnit.SECONDS));
        if (!locked) {
            return;
        }
        try {
            // 获取Redis中的库存
            String redisStock = redisTemplate.opsForValue().get("stock:count:" + skuId);
            if (redisStock == null) {
                return;
            }
            // 数据库更新（乐观锁防止超卖）
            int rows = stockMapper.updateStockByOptimisticLock(skuId, Integer.parseInt(redisStock));
            // 乐观锁更新失败则重试（最多3次）
            int retry = 0;
            while (rows == 0 && retry < 3) {
                retry++;
                // 重新查询最新库存
                Stock stock = stockMapper.selectBySkuId(skuId);
                if (stock == null) {
                    break;
                }
                // 重新计算并更新
                rows = stockMapper.updateStockByOptimisticLock(skuId, Integer.parseInt(redisStock), stock.getVersion());
            }
        } finally {
            redisTemplate.delete(lockKey);
        }
    }

    // 4. 库存回滚
    public void rollbackStock(String skuId, int num) {
        // Redis回滚（原子操作）
        redisTemplate.opsForValue().increment("stock:count:" + skuId, num);
        // 异步同步到数据库
        asyncSyncStockToDb(skuId);
    }
}

// Mapper层乐观锁更新库存
public interface StockMapper extends BaseMapper<Stock> {
    @Update("UPDATE stock SET stock_count = #{stockCount}, version = version + 1 WHERE sku_id = #{skuId} AND version = #{version}")
    int updateStockByOptimisticLock(@Param("skuId") String skuId, @Param("stockCount") int stockCount, @Param("version") int version);

    @Update("UPDATE stock SET stock_count = #{stockCount} WHERE sku_id = #{skuId}")
    int updateStockBySkuId(@Param("skuId") String skuId, @Param("stockCount") int stockCount);
}
```

#### 关键说明
- 防超卖：Lua脚本将“查库存 + 扣库存”做成原子操作，避免并发下库存扣成负数。
- 高并发优化：Redis预扣减 + 异步同步DB，秒杀场景可再结合库存预热、限流。
- 库存回滚：订单取消时必须回滚库存，建议加重试机制（如失败后放入消息队列重试）。

---

### 3. 支付流程设计
#### 核心设计思路
- **支付回调处理**：回调接口需幂等、防篡改（验签）、异步处理，回调结果即时返回给支付渠道。
- **支付状态同步**：主动查询 + 被动回调结合，保证支付状态最终一致。
- **多支付渠道适配**：基于策略模式封装不同支付渠道的接口，统一调用入口。

#### 关键代码实现
```java
// 1. 支付渠道枚举
public enum PayChannel {
    ALIPAY("alipay", "支付宝"),
    WECHAT_PAY("wechat_pay", "微信支付"),
    UNION_PAY("union_pay", "银联支付");

    private final String code;
    private final String desc;
    // 构造器、getter省略
}

// 2. 支付策略接口
public interface PayStrategy {
    // 发起支付
    String createPayOrder(PayRequest request);
    // 处理回调
    String handleCallback(Map<String, String> params);
}

// 3. 支付宝支付实现
@Component
public class AlipayStrategy implements PayStrategy {
    @Override
    public String createPayOrder(PayRequest request) {
        // 调用支付宝SDK生成支付链接/参数
        return "https://openapi.alipay.com/gateway.do?xxx";
    }

    @Override
    public String handleCallback(Map<String, String> params) {
        // 1. 验签（防止伪造回调）
        boolean verify = AlipaySignature.rsaCheckV1(params, ALIPAY_PUBLIC_KEY, "UTF-8", "RSA2");
        if (!verify) {
            return "fail";
        }
        // 2. 幂等校验（根据out_trade_no）
        String outTradeNo = params.get("out_trade_no");
        String idempotentKey = "pay:callback:" + outTradeNo;
        if (!Boolean.TRUE.equals(redisTemplate.opsForValue().setIfAbsent(idempotentKey, "1", 24, TimeUnit.HOURS))) {
            return "success"; // 已处理过，直接返回成功
        }
        // 3. 处理回调逻辑（更新订单状态）
        try {
            String tradeStatus = params.get("trade_status");
            if ("TRADE_SUCCESS".equals(tradeStatus)) {
                orderService.paySuccess(outTradeNo, params.get("trade_no"), PayChannel.ALIPAY);
            }
            return "success"; // 必须返回success，否则支付渠道会重试
        } catch (Exception e) {
            log.error("支付宝回调处理失败", e);
            return "fail"; // 返回失败，支付渠道会重试
        }
    }
}

// 4. 支付策略工厂
@Component
public class PayStrategyFactory {
    @Autowired
    private Map<String, PayStrategy> payStrategyMap; // Spring自动注入所有PayStrategy实现

    public PayStrategy getStrategy(PayChannel channel) {
        PayStrategy strategy = payStrategyMap.get(channel.getCode());
        if (strategy == null) {
            throw new IllegalArgumentException("不支持的支付渠道：" + channel.getCode());
        }
        return strategy;
    }
}

// 5. 支付回调控制器
@RestController
@RequestMapping("/pay/callback")
public class PayCallbackController {
    @Autowired
    private PayStrategyFactory payStrategyFactory;

    @PostMapping("/{channel}")
    public String callback(@PathVariable String channel, @RequestParam Map<String, String> params) {
        // 异步处理回调，立即返回结果
        CompletableFuture.runAsync(() -> {
            try {
                PayChannel payChannel = PayChannel.valueOf(channel.toUpperCase());
                PayStrategy strategy = payStrategyFactory.getStrategy(payChannel);
                strategy.handleCallback(params);
            } catch (Exception e) {
                log.error("支付回调处理异常", e);
            }
        });
        // 必须即时返回支付渠道要求的成功标识（如支付宝返回success）
        return "success";
    }
}

// 6. 支付状态主动查询（定时任务）
@Service
public class PayQueryService {
    @Autowired
    private PayStrategyFactory payStrategyFactory;
    @Autowired
    private OrderService orderService;

    @Scheduled(fixedRate = 5 * 60 * 1000) // 每5分钟查询一次
    public void queryPayStatus() {
        // 查询所有待支付且超过5分钟的订单
        List<Order> pendingOrders = orderService.listPendingPayOrders(5);
        for (Order order : pendingOrders) {
            try {
                PayStrategy strategy = payStrategyFactory.getStrategy(order.getPayChannel());
                // 调用支付渠道查询接口
                String payStatus = strategy.queryPayStatus(order.getOrderNo());
                if ("SUCCESS".equals(payStatus)) {
                    orderService.paySuccess(order.getOrderNo(), "", order.getPayChannel());
                }
            } catch (Exception e) {
                log.error("查询订单{}支付状态失败", order.getOrderNo(), e);
            }
        }
    }
}
```

#### 关键说明
- 回调处理：必须**先返回成功再异步处理业务**，防止支付渠道重复回调；验签 + 幂等是核心，避免伪造回调和重复处理。
- 多渠道适配：策略模式隔离不同支付渠道的实现，新增渠道只需实现PayStrategy接口，符合开闭原则。
- 状态同步：被动回调为主，主动查询为辅，解决回调丢失的问题。

---

### 4. 用户体系设计
#### 核心设计思路
- **用户鉴权**：基于JWT生成Token，结合Redis存储黑名单（登出/过期Token）。
- **第三方登录**：基于OAuth2.0协议，统一用户ID生成规则，绑定第三方账号与本地账号。
- **密码安全**：密码加盐哈希存储（BCrypt），禁止明文/简单MD5存储。

#### 关键代码实现
```java
// 1. JWT工具类
@Component
public class JwtUtil {
    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.expire}")
    private long expire; // 过期时间，单位秒

    // 生成Token
    public String generateToken(Long userId) {
        Date now = new Date();
        Date expireDate = new Date(now.getTime() + expire * 1000);
        return Jwts.builder()
                .setSubject(userId.toString()) // 存储用户ID
                .setIssuedAt(now)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();
    }

    // 解析Token获取用户ID
    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody();
        return Long.parseLong(claims.getSubject());
    }

    // 验证Token是否有效
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}

// 2. 登录鉴权拦截器
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Autowired
    private JwtUtil jwtUtil;
    @Autowired
    private StringRedisTemplate redisTemplate;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 获取Token
        String token = request.getHeader("Authorization");
        if (token == null || !token.startsWith("Bearer ")) {
            response.setStatus(401);
            response.getWriter().write("未登录");
            return false;
        }
        token = token.substring(7);

        // 2. 检查Token是否在黑名单（登出）
        if (Boolean.TRUE.equals(redisTemplate.hasKey("token:blacklist:" + token))) {
            response.setStatus(401);
            response.getWriter().write("Token已失效");
            return false;
        }

        // 3. 验证Token有效性
        if (!jwtUtil.validateToken(token)) {
            response.setStatus(401);
            response.getWriter().write("Token无效或过期");
            return false;
        }

        // 4. 将用户ID存入请求上下文
        Long userId = jwtUtil.getUserIdFromToken(token);
        request.setAttribute("userId", userId);
        return true;
    }
}

// 3. 第三方登录（微信登录示例）
@Service
public class WxLoginService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private JwtUtil jwtUtil;

    public String wxLogin(String code) {
        // 1. 调用微信接口获取openid
        String openid = getWxOpenid(code);
        // 2. 查询是否已绑定本地用户
        User user = userMapper.selectByOpenid(openid);
        if (user != null) {
            // 已绑定，直接生成Token
            return jwtUtil.generateToken(user.getId());
        }
        // 3. 未绑定，创建新用户
        User newUser = new User();
        newUser.setOpenid(openid);
        newUser.setNickname("微信用户" + RandomStringUtils.randomNumeric(6));
        newUser.setCreateTime(new Date());
        userMapper.insert(newUser);
        // 4. 生成Token
        return jwtUtil.generateToken(newUser.getId());
    }

    // 调用微信接口获取openid（简化版）
    private String getWxOpenid(String code) {
        String url = "https://api.weixin.qq.com/sns/jscode2session?appid=xxx&secret=xxx&js_code=" + code + "&grant_type=authorization_code";
        // 发送HTTP请求并解析返回结果，获取openid
        return "wx_openid_123456";
    }
}

// 4. 密码加密
@Service
public class UserService {
    // 密码加密（BCrypt自带盐值，无需手动加盐）
    public String encryptPassword(String password) {
        return BCrypt.hashpw(password, BCrypt.gensalt());
    }

    // 密码验证
    public boolean verifyPassword(String rawPassword, String encryptedPassword) {
        return BCrypt.checkpw(rawPassword, encryptedPassword);
    }
}
```

#### 关键说明
- Token设计：JWT存储非敏感信息（用户ID），敏感信息需从数据库查询；Redis黑名单解决JWT无法主动失效的问题。
- 第三方登录：统一用户标识，第三方账号可绑定/解绑本地账号，保证用户体系一致性。
- 密码安全：BCrypt哈希不可逆，且自带盐值，避免彩虹表破解。

---

### 5. 消息通知场景
#### 核心设计思路
- **异步处理**：消息发送放入消息队列（RocketMQ/RabbitMQ），主线程不阻塞。
- **批量发送**：积累一定数量/时间后批量发送，减少接口调用次数。
- **重试机制**：发送失败后放入重试队列，设置最大重试次数，失败后记录异常日志。

#### 关键代码实现
```java
// 1. 消息通知枚举
public enum NotifyType {
    SMS("sms", "短信"),
    PUSH("push", "推送"),
    EMAIL("email", "邮件");

    private final String code;
    private final String desc;
    // 构造器、getter省略
}

// 2. 消息发送接口
public interface NotifyService {
    // 单条发送（异步）
    void sendAsync(NotifyRequest request);
    // 批量发送（异步）
    void batchSendAsync(List<NotifyRequest> requests);
}

// 3. 短信发送实现
@Service
public class SmsNotifyService implements NotifyService {
    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    @Override
    public void sendAsync(NotifyRequest request) {
        // 发送到消息队列
        rocketMQTemplate.send("notify-sms-topic", MessageBuilder.withPayload(request).build());
    }

    @Override
    public void batchSendAsync(List<NotifyRequest> requests) {
        // 批量发送到消息队列
        List<Message<NotifyRequest>> messages = requests.stream()
                .map(request -> MessageBuilder.withPayload(request).build())
                .collect(Collectors.toList());
        rocketMQTemplate.sendBatch("notify-sms-topic", messages);
    }

    // 消费消息队列，发送短信
    @RocketMQMessageListener(
            topic = "notify-sms-topic",
            consumerGroup = "notify-sms-consumer",
            consumeMode = ConsumeMode.CONCURRENTLY,
            maxReconsumeTimes = 3 // 最大重试次数3次
    )
    @Component
    public class SmsConsumer implements RocketMQListener<NotifyRequest> {
        @Autowired
        private SmsApi smsApi;

        @Override
        public void onMessage(NotifyRequest request) {
            try {
                // 调用短信接口发送
                smsApi.sendSms(request.getPhone(), request.getContent(), request.getTemplateId());
            } catch (Exception e) {
                log.error("发送短信失败，手机号：{}，内容：{}", request.getPhone(), request.getContent(), e);
                // 超过重试次数后，记录到异常表
                if (request.getRetryCount() >= 3) {
                    saveFailNotify(request, e.getMessage());
                    throw new RuntimeException("短信发送失败，已达最大重试次数");
                }
                // 重试次数+1，重新发送
                request.setRetryCount(request.getRetryCount() + 1);
                sendAsync(request);
            }
        }
    }

    // 保存发送失败的消息
    private void saveFailNotify(NotifyRequest request, String errorMsg) {
        // 插入数据库异常表，人工排查
    }
}

// 4. 批量发送管理器（定时批量）
@Service
public class NotifyBatchManager {
    @Autowired
    private SmsNotifyService smsNotifyService;
    // 本地缓存待发送的短信（线程安全）
    private final BlockingQueue<NotifyRequest> smsQueue = new LinkedBlockingQueue<>(10000);

    // 添加短信到批量队列
    public void addToBatchQueue(NotifyRequest request) {
        try {
            smsQueue.put(request);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            log.error("添加短信到批量队列失败", e);
        }
    }

    // 定时任务批量发送（每30秒执行一次）
    @Scheduled(fixedRate = 30 * 1000)
    public void batchSendSms() {
        List<NotifyRequest> requests = new ArrayList<>();
        // 从队列中取出最多100条
        smsQueue.drainTo(requests, 100);
        if (!requests.isEmpty()) {
            smsNotifyService.batchSendAsync(requests);
        }
    }
}
```

#### 关键说明
- 异步处理：消息队列解耦发送方和消费方，主线程快速响应，消费方异步处理发送逻辑。
- 批量发送：定时+定量批量发送，减少短信/邮件接口的调用次数，降低成本和延迟。
- 重试机制：基于消息队列的重试机制，设置最大重试次数，避免无限重试；失败后记录异常，便于人工干预。

---

### 6. 购物车场景
#### 核心设计思路
- **数据存储**：未登录用户购物车存在本地（LocalStorage），登录后合并到服务端（Redis + 数据库）；登录用户直接存在服务端。
- **购物车合并**：登录时将本地购物车与服务端购物车合并，相同商品数量累加，价格以服务端为准。
- **价格实时计算**：展示购物车时，实时查询商品最新价格，避免价格过期。

#### 关键代码实现
```java
@Service
public class CartService {
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Autowired
    private ProductService productService;

    // Redis Key前缀：用户购物车 -> cart:user:{userId}
    private String getCartKey(Long userId) {
        return "cart:user:" + userId;
    }

    // 1. 添加商品到购物车
    public void addToCart(Long userId, Long skuId, int num) {
        String cartKey = getCartKey(userId);
        // Redis Hash存储：key=skuId，value=数量
        redisTemplate.opsForHash().increment(cartKey, skuId.toString(), num);
        // 可选：同步到数据库（异步）
        asyncSyncToDb(userId);
    }

    // 2. 获取购物车列表（实时计算价格）
    public List<CartItemVO> getCartList(Long userId) {
        String cartKey = getCartKey(userId);
        // 获取购物车所有商品
        Map<Object, Object> cartMap = redisTemplate.opsForHash().entries(cartKey);
        if (cartMap.isEmpty()) {
            return Collections.emptyList();
        }
        // 转换为VO并实时查询价格
        List<CartItemVO> cartItems = new ArrayList<>();
        for (Map.Entry<Object, Object> entry : cartMap.entrySet()) {
            Long skuId = Long.parseLong(entry.getKey().toString());
            int num = Integer.parseInt(entry.getValue().toString());
            // 查询商品最新信息
            ProductVO product = productService.getProductBySkuId(skuId);
            if (product == null) {
                // 商品已下架，从购物车移除
                redisTemplate.opsForHash().delete(cartKey, skuId.toString());
                continue;
            }
            // 构建购物车项
            CartItemVO cartItem = new CartItemVO();
            cartItem.setSkuId(skuId);
            cartItem.setProductName(product.getName());
            cartItem.setPrice(product.getPrice()); // 实时价格
            cartItem.setNum(num);
            cartItem.setTotalPrice(product.getPrice().multiply(new BigDecimal(num)));
            cartItems.add(cartItem);
        }
        return cartItems;
    }

    // 3. 合并购物车（登录时合并本地购物车）
    public void mergeCart(Long userId, List<CartItemDTO> localCartItems) {
        if (CollectionUtils.isEmpty(localCartItems)) {
            return;
        }
        String cartKey = getCartKey(userId);
        // 批量合并：相同商品数量累加
        Map<String, Integer> mergeMap = new HashMap<>();
        for (CartItemDTO item : localCartItems) {
            mergeMap.put(item.getSkuId().toString(), item.getNum());
        }
        // Redis批量操作
        redisTemplate.opsForHash().increment(cartKey, mergeMap);
        // 同步到数据库
        asyncSyncToDb(userId);
    }

    // 4. 异步同步到数据库
    @Async
    public void asyncSyncToDb(Long userId) {
        String cartKey = getCartKey(userId);
        Map<Object, Object> cartMap = redisTemplate.opsForHash().entries(cartKey);
        // 清空数据库原有记录
        cartMapper.deleteByUserId(userId);
        // 批量插入
        List<Cart> cartList = cartMap.entrySet().stream()
                .map(entry -> {
                    Cart cart = new Cart();
                    cart.setUserId(userId);
                    cart.setSkuId(Long.parseLong(entry.getKey().toString()));
                    cart.setNum(Integer.parseInt(entry.getValue().toString()));
                    cart.setUpdateTime(new Date());
                    return cart;
                })
                .collect(Collectors.toList());
        if (!cartList.isEmpty()) {
            cartMapper.batchInsert(cartList);
        }
    }
}
```

#### 关键说明
- 数据存储：Redis Hash存储购物车（key=用户ID，field=SKU ID，value=数量），读写性能高；数据库做持久化备份。
- 购物车合并：登录时将本地购物车数据提交到服务端，批量累加数量，保证数据不丢失。
- 价格计算：展示购物车时实时查询商品最新价格，避免价格过期导致的用户纠纷。

---

### 总结
1. **高并发场景核心**：Redis预扣减/缓存 + 数据库最终一致性，结合分布式锁/乐观锁保证数据安全，消息队列解耦异步逻辑。
2. **数据一致性**：状态流转单向化、幂等处理防重复、重试机制保障失败恢复，核心操作（扣库存、改订单状态）需原子性。
3. **性能优化**：异步处理（消息队列）、批量操作、缓存预热、限流降级，避免单点瓶颈，提升系统吞吐量。
4. **异常处理**：超时取消、库存回滚、回调重试、失败日志记录，保证系统鲁棒性，异常可追溯、可恢复。

## 二、高并发/高可用场景类

1. 秒杀/抢购场景（如限流、削峰、防重复下单、分布式锁应用）
2. 接口高并发优化（如接口性能瓶颈分析、缓存策略、异步化改造）
3. 分布式系统一致性（如分布式事务、最终一致性方案选型）
4. 服务熔断/降级/限流（如雪崩防护、降级策略设计、限流算法选型）
5. 热点数据处理（如热点商品缓存、缓存击穿/穿透/雪崩解决方案）
6. 大流量下的数据库优化（如分库分表、读写分离、索引设计）

这6个是**面试必问、架构必考**的高并发&高可用核心场景，我给你整理成**可直接背、可直接写在简历、可直接讲给面试官**的标准答案，结构统一：
**核心问题 → 解决方案 → 关键实现 → 避坑要点**

---

### 1. 秒杀/抢购场景

#### 核心痛点

超卖、重复下单、瞬间流量打垮服务、数据库被冲垮

#### 标准解决方案（分层防护）

1. **前端限流**
   - 按钮置灰、防抖、验证码
   - 防止用户疯狂点击

2. **网关层限流**
   - 单机限流 + 分布式限流（Redis + Lua）
   - 限制同一用户/IP访问频率

3. **服务层削峰**
   - **消息队列异步下单**（RabbitMQ/RocketMQ）
   - 前端只做“提交申请”，不直接写库

4. **库存防超卖（最核心）**
   - **Redis Lua 原子扣减**（判断+扣减一步完成）
   - 数据库用**乐观锁/版本号**最终一致
   - 禁止直接数据库扣减

5. **防重复下单**
   - 用户ID + 活动ID 做**分布式锁**（Redisson）
   - 幂等键：`seckill:user:{userId}:{activityId}`

6. **隔离**
   - 秒杀服务独立集群、独立数据库、独立缓存
   - 不影响主站正常业务

#### 关键技术

- Redis Lua、Redisson 分布式锁、消息队列削峰、库存预热、乐观锁

---

### 2. 接口高并发优化

#### 通用优化路线（面试官最爱听）

**先定位瓶颈 → 再分层优化**

1. **接口瓶颈定位**
   - 监控：RT、QPS、TP99/TP999
   - 链路追踪：SkyWalking/Pinpoint
   - 慢查询、慢接口、循环调用、大事务

2. **缓存策略（第一优化手段）**
   - 本地缓存：Caffeine
   - 分布式缓存：Redis
   - 缓存结构：**本地缓存 → Redis → DB**
   - 过期策略：主动更新+过期淘汰

3. **异步化改造**
   - 同步逻辑拆分为：**核心流程同步 + 非核心异步**
   - 日志、通知、统计、消息推送全部丢MQ
   - 使用 CompletableFuture 并行调用

4. **减少RPC/DB交互**
   - 批量接口替代循环调用
   - 避免 N+1 查询
   - 避免大事务、长事务

5. **代码层优化**
   - 减少锁粒度
   - 减少反射、序列化
   - 使用连接池、线程池

#### 结论

**缓存 + 异步 + 批量**是接口性能三驾马车。

---

### 3. 分布式系统一致性

#### 面试官必问：分布式事务怎么选？

#### 方案对比（直接背）

1. **2PC / XA**
   - 强一致，性能差，不适合高并发
2. **TCC**
   - Try-Confirm-Cancel
   - 侵入性高，开发复杂，适合核心交易
3. **SAGA**
   - 长事务，补偿机制
   - 适合跨多服务、流程长的业务
4. **本地消息表（最终一致）**
   - 业务表 + 消息表 本地事务
   - MQ 确保投递，简单可靠
5. **RocketMQ 事务消息**
   - 半消息机制，最优最终一致方案
   - 无业务侵入，高并发首选

#### 项目中通用选型

- **高并发非核心：最终一致性（事务消息/本地消息表）**
- **核心交易（订单+库存+支付）：TCC 或 Seata**
- **绝不强推 2PC**

---

### 4. 服务熔断/降级/限流（高可用三大利器）

#### 1）限流（拒绝多余流量）

常用算法：
- **计数器**：简单粗暴
- **滑动窗口**：更精准
- **漏桶**：平滑流量
- **令牌桶**：支持突发流量（Guava RateLimiter、Sentinel）

分布式限流：
- Redis + Lua 实现

#### 2）熔断（防止雪崩）

- 失败率达到阈值 → 打开熔断 → 拒绝请求 → 半开试探
- 框架：Sentinel、Resilience4j、Hystrix

#### 3）降级（保核心弃非核心）

策略：
- 关闭非核心接口（推荐、统计、日志）
- 返回兜底数据（缓存、默认值）
- 读降级、写限流
- 开关统一配置（Nacos/Apollo）

#### 一句话总结

**限流防流量激增，熔断防连锁崩溃，降级保核心可用。**

---

### 5. 热点数据处理

#### 三大问题 + 标准解决方案

1. **缓存穿透**
   - 原因：查询不存在的数据
   - 解决：**布隆过滤器** + 缓存空值 + 接口校验

2. **缓存击穿**
   - 原因：热点Key过期，大量请求打DB
   - 解决：**互斥锁（Redisson）** + 热点Key永不过期

3. **缓存雪崩**
   - 原因：大量Key同时过期/Redis宕机
   - 解决：
     - 过期时间加随机值
     - 集群、主从、哨兵
     - 多级缓存
     - 熔断降级

#### 热点Key特殊处理

- 本地缓存Caffeine扛量
- 拆分Key（如`hot_item_1` → `hot_item_1_1`…`hot_item_1_5`）
- 独立缓存集群

---

### 6. 大流量下数据库优化

#### 从易到难（架构演进路线）

1. **索引优化**
   - 最左前缀、避免索引失效
   - 联合索引、覆盖索引
   - 禁止`select *`

2. **读写分离**
   - 主库写，从库读
   - Sharding-JDBC / MyCat
   - 容忍主从延迟

3. **分库分表（水平切分）**
   - 分库：降低单库连接压力
   - 分表：降低单表数据量
   - 分片键：用户ID、订单ID
   - 避免跨库分页、跨库事务

4. **其他优化**
   - 大字段拆表
   - 禁止长事务、大事务
   - 批量插入、分批处理
   - 归档历史数据

#### 高并发DB铁律

**能缓存绝不查库，能异步不实时，能分表不单表，能读写分离不单点。**

---

#### 整体一句话总结（面试收尾金句）

高并发架构核心就是三件事：
**用缓存扛读、用消息队列削峰写、用熔断限流保可用，最终一致性优先，强一致只用于核心交易。**

需要我把这6类整理成**一页纸面试背诵版**，或者给你生成**模拟面试问答（你问我答）**吗？

## 三、数据处理/存储场景类

1. 海量数据分页查询（如大数据量下的高效分页、游标分页实现）
2. 数据同步场景（如跨库/跨系统数据同步、CDC方案、定时同步策略）
3. 大文件处理（如大文件上传/下载、分片传输、断点续传）
4. 缓存设计与优化（如缓存更新策略、缓存与数据库一致性、多级缓存）
5. 非结构化数据存储（如文件/图片/视频的存储方案、OSS/MinIO应用）
6. 数据脱敏/加密（如敏感字段（手机号/身份证）脱敏、传输/存储加密）

这一类是**后端数据存储、数据治理、性能优化**的高频面试题，我给你整理成**面试官最爱听、简历可直接写、项目可直接落地**的标准答案，结构统一：
**核心痛点 → 方案选型 → 关键实现 → 避坑要点**

---

### 1. 海量数据分页查询

#### 痛点

- 普通 `limit 1000000,10` 性能极差：需要扫描前面所有行
- 跳页 + 海量数据 = DB 雪崩

#### 方案

1. **深分页优化（推荐）**
   - **主键游标分页（Keyset Pagination）**
   - 条件：`id > #{lastId} limit 10`
   - 优点：O(1) 定位，无论多少页速度一致
   - 缺点：**不支持随意跳页**，适合瀑布流、滚动加载

2. **延迟关联（优化普通Limit）**
   ```sql
   SELECT t.* FROM 表 t 
   JOIN (SELECT id FROM 表 WHERE 条件 ORDER BY id LIMIT 1000000,10) tmp 
   ON t.id = tmp.id
   ```
   只先查ID，再回表，减少IO

3. **业务层面限制**
   - 禁止查看1000页以后数据
   - 后台用**ES**做分页，不分担MySQL压力

#### 结论

**海量数据优先用游标分页，严禁深分页直接Limit。**

---

### 2. 数据同步场景（跨库/跨系统）

#### 主流方案（按企业使用率排序）

1. **CDC 实时同步（首选）**
   - 基于 Binlog：Canal、Debezium、Flink CDC
   - 准实时，无侵入，不影响业务
   - 适用：MySQL → ES、Redis、数仓、其他DB

2. **定时任务批同步**
   - XXL-Job/Quartz
   - 按 `update_time` 增量同步
   - 适用：非实时、对账、统计

3. **消息队列最终一致**
   - 业务写完DB，发MQ，下游消费同步
   - 适用：跨微服务数据同步

4. **ETL 工具**
   - DataX、Kettle
   - 离线批量同步、数据清洗

#### 选型口诀

**实时用CDC，准实时用MQ，离线批量用DataX，简单小数据用定时任务。**

---

### 3. 大文件处理（上传/下载/分片/断点续传）

#### 核心方案

1. **大文件上传**
   - **前端分片 + 后端合并**
   - 分片大小：2MB～10MB
   - 流程：
     1. 前端计算文件MD5（唯一标识）
     2. 查询已上传分片（断点续传）
     3. 上传未完成分片
     4. 全部上传完发送**合并请求**

2. **断点续传**
   - 用 **MD5** 作为唯一标识
   - Redis 记录已上传分片索引
   - 重传只需传缺失分片

3. **秒传**
   - 上传前先查MD5是否存在
   - 存在则直接返回成功，引用已有文件

4. **下载**
   - 分块下载（Range 请求）
   - 大文件用**流式输出**，避免一次性加载内存

#### 存储

- 小文件：MinIO/OSS
- 大文件：对象存储 + CDN

---

### 4. 缓存设计与优化

#### 4.1 缓存更新策略（面试必考）

1. **Cache Aside 旁路缓存（最常用）**
   - 读：命中返回，未命中查DB → 写缓存
   - 写：**更DB → 删缓存**
   - 禁止：更DB → 更缓存（容易不一致）

2. **Write Through**
   - 同步写DB+缓存，强一致，性能差

3. **Write Back**
   - 写缓存，异步刷DB，性能高，有丢数据风险

#### 4.2 缓存与DB一致性

- 方案：**先更DB，再删缓存**
- 最终一致，允许极短时间脏数据
- 并发问题：通过**过期时间 + 分布式锁**兜底

#### 4.3 多级缓存

- 层级：**Caffeine（本地）→ Redis（分布式）→ DB**
- 本地缓存扛热点，减少Redis压力
- 注意本地缓存**一致性问题**：用MQ广播失效

#### 4.4 三大问题（穿透/击穿/雪崩）

- 穿透：布隆过滤器、缓存空对象
- 击穿：互斥锁、热点永不过期
- 雪崩：随机过期、集群、熔断、多级缓存

---

### 5. 非结构化数据存储（文件/图片/视频）

#### 标准架构

1. **存储选型**
   - 企业级：阿里云OSS、腾讯COS
   - 自建：**MinIO**（兼容S3，部署简单）
   - 禁止：存在数据库（Blob）、存在应用服务器

2. **上传流程**
   - 后端生成**临时上传签名**（前端直传OSS）
   - 前端直传，不经过应用服务
   - 回调通知后端入库

3. **访问**
   - CDN 加速
   - 私有文件：加签名、时效URL

4. **治理**
   - 缩略图、水印、格式转换
   - 分桶存储、权限控制
   - 生命周期管理（自动归档/删除）

#### 结论

**小文件用MinIO/OSS，前端直传+回调，绝不存DB。**

---

### 6. 数据脱敏/加密

#### 6.1 脱敏（展示/日志）

- 手机号：138****1234
- 身份证：110***********1234
- 邮箱：a***b@qq.com
- 实现：**AOP切面 + 自定义注解** 自动脱敏

#### 6.2 存储加密

- 敏感字段：**不可逆加密 + 可逆加密**结合
  - 密码：**BCrypt**（不可逆）
  - 身份证/银行卡：**AES**（可逆，密钥严格管理）

#### 6.3 传输加密

- HTTPS（TLS）
- 接口敏感字段可额外 RSA 非对称加密

#### 6.4 最佳实践

- 日志**严禁打印明文**
- 密钥放配置中心/密钥服务，不硬编码
- 脱敏、加密统一封装Starter，全局生效

---

### 整体一句话总结（可直接背）

海量分页用**游标**，数据同步用**CDC**，大文件**分片上传**，缓存**先更DB再删缓存**，文件存**MinIO/OSS**，敏感数据**脱敏+加密**。

需要我把这三类（业务+高并发+数据存储）合成**一份3页PDF级别的面试背诵笔记**，或者给你出**模拟面试题+逐题讲解**吗？

## 四、业务异常处理/问题排查类

1. 生产环境接口超时排查（如慢接口定位、性能瓶颈分析、优化思路）
2. 数据不一致问题排查（如数据库与缓存数据不一致、分布式事务失败）
3. 接口幂等性问题（如重复提交导致的数据错误、幂等方案落地）
4. 分布式锁失效问题（如锁超时、死锁、锁竞争导致的业务异常）
5. 第三方接口调用异常（如超时、重试、降级、熔断处理）
6. 内存溢出/泄漏排查（如OOM场景定位、内存分析、代码优化）

这一类是**生产环境高频故障排查**的核心场景，也是面试官考察**问题分析能力、落地解决能力**的重点。我按「故障现象 → 排查步骤 → 解决方案 → 避坑要点」的结构，给你整理成**可直接落地、可直接讲给面试官**的标准答案：

---

### 1. 生产环境接口超时排查

#### 核心排查流程（按优先级）

#### 步骤1：快速定位超时接口

- 监控平台（Prometheus/Grafana）：看接口RT、QPS、TP99/TP999
- 链路追踪（SkyWalking/Pinpoint）：看接口调用链路，定位**耗时最长的节点**（如DB/Redis/第三方接口）
- 日志：搜索超时接口的`traceId`，查看完整调用日志

#### 步骤2：分层定位瓶颈

| 瓶颈类型       | 排查手段                                                     |
| -------------- | ------------------------------------------------------------ |
| DB 慢查询      | 1. 开启慢查询日志（MySQL slow log）<br>2. explain 分析SQL执行计划<br>3. 检查索引是否失效 |
| Redis 慢查询   | 1. `slowlog get` 查看Redis慢命令<br>2. 检查大Key、复杂命令（如hgetall、keys *） |
| 第三方接口超时 | 1. 查看调用第三方接口的超时日志<br>2. 核对第三方接口SLA，是否对方服务异常 |
| 代码逻辑问题   | 1. 检查是否有循环调用、大循环<br>2. 检查是否有同步阻塞（如未异步化的非核心逻辑） |
| 资源瓶颈       | 1. 服务器监控（CPU、内存、磁盘IO、网络）<br>2. 线程池/连接池是否打满 |

#### 步骤3：临时止损 + 长期优化

| 阶段     | 解决方案                                                     |
| -------- | ------------------------------------------------------------ |
| 临时止损 | 1. 熔断/降级非核心接口<br>2. 扩容（实例/容器）<br>3. 临时禁用超时接口的非核心逻辑 |
| 长期优化 | 1. DB：优化SQL/索引、读写分离、分库分表<br>2. 缓存：加缓存、优化缓存策略<br>3. 代码：异步化、批量处理<br>4. 资源：扩容线程池/连接池 |

#### 避坑要点

- 禁止直接在生产环境改代码/重启服务，先止损再排查
- 优先看监控/链路，再看日志，最后查代码
- 超时问题**90%是DB/Redis慢查询或第三方接口**，不是代码逻辑

---

### 2. 数据不一致问题排查

#### 核心场景：DB与缓存不一致、分布式事务失败

#### 场景1：DB与缓存数据不一致

##### 排查步骤
1. 确认缓存更新策略：是否是「先更DB，再删缓存」（正确策略）
2. 检查日志：是否有缓存删除失败、DB更新成功但缓存未删的情况
3. 检查并发：是否有高并发更新+查询，导致脏数据写入缓存
4. 检查缓存过期时间：是否过期时间设置过长

##### 解决方案
- 临时：手动删除不一致的缓存Key，强制刷新
- 长期：
  1. 严格遵循「更DB → 删缓存」策略，禁止「更DB → 更缓存」
  2. 缓存添加合理过期时间（兜底）
  3. 高并发场景加分布式锁，保证更新+删缓存原子性
  4. 定时任务校验DB与缓存数据，发现不一致自动修复

#### 场景2：分布式事务失败导致数据不一致

##### 排查步骤
1. 查看事务日志：TCC/Seata日志、MQ消息投递日志
2. 检查补偿逻辑：是否有未执行的补偿操作
3. 检查数据库：是否有单边更新（如扣库存成功、创建订单失败）
4. 检查网络：是否有MQ消息丢失、服务间调用超时

##### 解决方案
- 临时：人工核对数据，手动回滚/补全
- 长期：
  1. 核心场景用TCC/Seata，非核心用「本地消息表+MQ」最终一致
  2. 所有分布式操作加日志，便于问题追溯
  3. 增加定时对账任务，发现不一致自动触发补偿

#### 避坑要点

- 数据不一致**优先止损**（如冻结异常数据），再排查根因
- 禁止忽略小量不一致，小问题会演变成大故障
- 最终一致性方案必须加「对账+补偿」，不能只依赖MQ

---

### 3. 接口幂等性问题

#### 故障现象

- 重复提交导致：重复创建订单、重复扣库存、重复发送短信
- 日志特征：同一请求ID/业务ID多次执行写操作

#### 排查步骤

1. 确认幂等方案：是否实现了幂等校验（如幂等Key、分布式锁）
2. 检查幂等Key：是否唯一（如用户ID+业务ID+操作类型）、是否过期
3. 检查接口：是否是POST接口、是否未做防重提交
4. 检查前端：是否有重复点击、网络重试导致的重复请求

#### 解决方案

| 场景           | 幂等方案                           |
| -------------- | ---------------------------------- |
| 提交订单/支付  | 分布式锁 + 幂等Key（Redis SET NX） |
| MQ消费         | 消费位点记录 + 业务唯一键          |
| 接口防重复提交 | 前端Token + 后端Redis校验          |
| 批量操作       | 分段幂等，按批次生成幂等Key        |

##### 落地示例（Redis幂等）
```java
// 生成幂等Key：用户ID + 业务ID + 操作类型
public String generateIdempotentKey(Long userId, String bizId, String opType) {
    return "idempotent:" + opType + ":" + userId + ":" + bizId;
}

// 幂等校验
public boolean checkIdempotent(String key, long expireSeconds) {
    // SET NX EX：不存在则设置，原子操作
    Boolean success = redisTemplate.opsForValue()
            .setIfAbsent(key, "1", expireSeconds, TimeUnit.SECONDS);
    return Boolean.TRUE.equals(success);
}

// 业务调用
public void createOrder(OrderRequest request) {
    String idempotentKey = generateIdempotentKey(request.getUserId(), request.getBizId(), "CREATE_ORDER");
    if (!checkIdempotent(idempotentKey, 300)) {
        throw new BusinessException("请勿重复提交");
    }
    // 执行业务逻辑
    orderMapper.insert(request);
}
```

#### 避坑要点

- 幂等Key必须**全局唯一**，且过期时间要覆盖业务最大处理时长
- 幂等校验必须放在**业务逻辑前**，且是原子操作
- 禁止用「查询+判断」做幂等（高并发下会失效）

---

### 4. 分布式锁失效问题

#### 常见失效场景 + 排查&解决

| 失效场景           | 排查要点                                         | 解决方案                                               |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------ |
| 锁超时释放         | 1. 检查锁过期时间是否过短<br>2. 检查业务执行时长 | 1. 锁自动续期（Redisson看门狗）<br>2. 合理设置过期时间 |
| 死锁               | 1. 检查是否未释放锁<br>2. 检查是否有异常未捕获   | 1. 加try-finally确保释放<br>2. 锁必须设置过期时间      |
| 锁竞争导致性能差   | 1. 检查锁粒度是否过大<br>2. 检查并发量           | 1. 减小锁粒度（如按SKU锁，不是按商品锁）<br>2. 分段锁  |
| 主从切换导致锁丢失 | 1. 检查Redis主从状态<br>2. 检查锁是否写入主库    | 1. 用RedLock/Redisson MultiLock<br>2. 或用ZooKeeper锁  |
| 误删别人的锁       | 1. 检查锁Key是否唯一<br>2. 检查释放逻辑          | 1. 加锁时存入唯一标识（如UUID）<br>2. 释放前校验标识   |

##### 正确的分布式锁使用示例（Redisson）
```java
@Service
public class StockService {
    @Autowired
    private RedissonClient redissonClient;

    public void deductStock(String skuId, int num) {
        // 锁粒度：按SKU锁，减小竞争
        String lockKey = "lock:stock:" + skuId;
        RLock lock = redissonClient.getLock(lockKey);
        try {
            // 加锁：自动续期（看门狗），最多等10秒，锁30秒
            boolean locked = lock.tryLock(10, 30, TimeUnit.SECONDS);
            if (!locked) {
                throw new BusinessException("系统繁忙，请稍后再试");
            }
            // 执行业务逻辑：扣减库存
            deductStockDb(skuId, num);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new BusinessException("扣减库存失败");
        } finally {
            // 释放锁：只释放自己加的锁
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

#### 避坑要点

- 分布式锁**必须设置过期时间**，防止死锁
- 锁粒度越小越好，禁止用全局锁（如lock:stock）
- 释放锁前必须校验「是否是当前线程持有」，防止误删

---

### 5. 第三方接口调用异常

#### 核心问题：超时、重试、降级、熔断

#### 排查步骤

1. 检查第三方接口文档：确认参数、签名、请求方式是否正确
2. 检查网络：是否能通第三方接口、是否有防火墙/代理拦截
3. 检查日志：查看请求/响应报文、超时时间、错误码
4. 核对第三方状态：是否对方服务故障、限流、维护

#### 解决方案（分级处理）

| 异常类型       | 处理策略                                                     |
| -------------- | ------------------------------------------------------------ |
| 超时           | 1. 设置合理超时时间（3-5秒）<br>2. 异步调用（非核心）<br>3. 熔断（多次超时后） |
| 调用失败       | 1. 幂等重试（最多3次，指数退避）<br>2. 失败后记录日志，定时补偿 |
| 第三方服务降级 | 1. 返回兜底数据（如缓存的历史数据）<br>2. 关闭非核心功能     |
| 数据不一致     | 1. 定时对账任务<br>2. 手动补偿接口                           |

##### 落地示例（重试+降级）
```java
@Service
public class ThirdPayService {
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private HystrixCommandFactory hystrixCommandFactory;

    // 调用第三方支付接口（带重试+熔断降级）
    public String callPayApi(PayRequest request) {
        // 熔断降级配置：失败率>50%触发熔断，熔断窗口5秒
        HystrixCommand<String> command = hystrixCommandFactory.createHystrixCommand(
            "pay-api", 
            () -> callPayApiWithRetry(request),  // 核心逻辑
            () -> "default_pay_result"           // 降级逻辑
        );
        return command.execute();
    }

    // 重试逻辑（指数退避）
    private String callPayApiWithRetry(PayRequest request) {
        RetryTemplate retryTemplate = RetryTemplate.builder()
                .maxAttempts(3)  // 最多重试3次
                .fixedBackoff(1000) // 每次重试间隔1秒
                .retryOn(IOException.class, TimeoutException.class) // 重试异常类型
                .build();
        
        return retryTemplate.execute(context -> {
            // 调用第三方接口
            ResponseEntity<String> response = restTemplate.postForEntity(
                "https://third-pay.com/api/pay",
                request,
                String.class
            );
            if (!response.getStatusCode().is2xxSuccessful()) {
                throw new BusinessException("第三方接口返回非200");
            }
            return response.getBody();
        });
    }
}
```

#### 避坑要点

- 第三方接口调用**必须加超时时间**，禁止无限等待
- 重试必须**幂等**，防止重复操作
- 核心接口必须有**降级方案**，不能因为第三方故障导致自身服务不可用

---

### 6. 内存溢出/泄漏排查

#### 核心场景：OOM（OutOfMemoryError）、内存泄漏

#### 排查步骤（生产环境）

1. 抓取堆快照：`jmap -dump:format=b,file=heap.hprof <pid>`
2. 分析堆快照：用MAT（Memory Analyzer Tool）/JProfiler
   - 查看大对象：哪些对象占用内存最多（如大List、大字符串）
   - 查看对象引用链：是否有内存泄漏（如静态集合未释放）
3. 查看GC日志：`jstat -gc <pid> 1000`，看YGC/FGC频率、内存回收情况
4. 检查代码：是否有大对象创建、静态集合、未关闭的资源（如IO、连接）

#### 常见OOM场景 + 解决方案
| OOM类型            | 场景                                                | 解决方案                                                     |
| ------------------ | --------------------------------------------------- | ------------------------------------------------------------ |
| Heap Space         | 1. 创建大对象（如100万条数据的List）<br>2. 内存泄漏 | 1. 分批处理数据，用流式输出<br>2. 释放无用对象引用<br>3. 调大堆内存（-Xmx） |
| Metaspace          | 动态生成类过多（如反射、CGLIB）                     | 1. 减少动态类生成<br>2. 调大Metaspace（-XX:MaxMetaspaceSize） |
| StackOverflowError | 递归调用过深、方法栈过深                            | 1. 优化递归为循环<br>2. 调大栈内存（-Xss）                   |
| Direct Memory      | NIO直接内存使用过多                                 | 1. 释放DirectBuffer<br>2. 调大直接内存（-XX:MaxDirectMemorySize） |

#### 内存泄漏排查要点

- 静态集合（如static Map）是最常见的内存泄漏原因
- 未关闭的资源（如IO流、数据库连接、线程池）会导致内存泄漏
- 内部类引用外部类，导致外部类无法被GC回收

#### 避坑要点

- 禁止直接调大JVM内存「治标不治本」，必须找到根因
- 生产环境必须开启GC日志，便于问题排查
- 大文件/大数据处理必须用「流式处理」，禁止一次性加载到内存

---

#### 整体总结（面试金句）

生产故障排查核心：**先止损，再定位，最后解决**；异常处理核心：**超时必有兜底，失败必有重试，核心必有降级**；内存问题核心：**拒绝大对象，及时释放引用，流式处理数据**。

如果需要，我可以把这四类（业务+高并发+数据存储+故障排查）整理成**一份完整的后端核心场景面试手册**，或者针对某一个场景出**模拟面试题+逐题详解**。

## 五、中间件应用场景类

1. MQ应用场景（如异步解耦、削峰填谷、消息积压处理、消息重试）
2. Redis业务落地（如分布式锁、限流器、计数器、延迟队列）
3. Elasticsearch应用（如全文检索、日志检索、聚合分析、性能优化）
4. 定时任务场景（如分布式定时任务、任务幂等、任务失败重试）
5. 微服务通信场景（如Feign/RPC调用、服务注册发现、负载均衡）

这一类是**中间件实战+选型+调优**的**面试重灾区**，我给你整理成**面试官最爱、简历可直接写、生产可直接落地**的标准答案，结构统一：
**核心作用 → 典型场景 → 关键实现 → 坑点&优化**

---

### 1. MQ 应用场景（RabbitMQ / RocketMQ / Kafka）

#### 核心作用

**异步、解耦、削峰、最终一致性**

#### 1）经典业务场景

- 订单创建后：发短信、push、通知物流、更新统计
- 秒杀：前端请求入队，后端异步消费，防止打崩DB
- 分布式事务：RocketMQ 事务消息
- 广播通知：缓存失效、配置刷新
- 流量削峰：秒杀、红包、大促

#### 2）消息积压（高频问题）

**排查&解决**
1. 消费能力不足：**增加消费者线程数/扩容节点**
2. 生产者流量暴增：限流、降级
3. 消费阻塞：慢SQL、第三方调用卡死
4. 堆积过大：**跳过过期数据/批量处理**，先恢复再对账

#### 3）消息可靠性（必问）

- 生产者：**事务消息 / confirm 机制**
- 存储：**落盘+主从**
- 消费者：**手动ACK + 幂等 + 重试**
- 死信：死信队列+人工排查

#### 4）消息重复（必问）

- 消费端必须**幂等**：唯一ID去重表 / Redis 幂等表

---

### 2. Redis 业务落地（高频到爆炸）

#### 1）分布式锁（Redisson 标准方案）

- 核心：**SET NX EX** + 唯一值 + 看门狗自动续期
- 避免：锁超时、误删锁、主从切换丢锁
- 正确用法：
  ```java
  RLock lock = redisson.getLock(key);
  lock.tryLock(5, 30, TimeUnit.SECONDS);
  ```

#### 2）限流器

- 简单：**INCR + EXPIRE** 固定窗口
- 精准：**Lua + ZSet** 滑动窗口
- 高并发：**令牌桶（RedissonRateLimiter）**

#### 3）计数器

- 点赞、收藏、阅读量、UV/PV
- 高并发用**INCR**，异步刷DB

#### 4）延迟队列

- 订单超时取消、支付超时
- 实现：**ZSet 分数=时间戳**，定时轮询

#### 5）三大问题（穿透/击穿/雪崩）

- 穿透：布隆过滤器 / 缓存空对象
- 击穿：互斥锁 / 热点永不过期
- 雪崩：随机过期 + 集群 + 多级缓存

---

### 3. Elasticsearch 应用

#### 1）核心场景

- **全文检索**：商品搜索、订单搜索
- **日志检索**：ELK（Elasticsearch+Logstash+Kibana）
- **聚合分析**：统计、报表、多维分析
- **高亮、纠错、拼音搜索**

#### 2）性能优化（面试必背）

- 避免`*`前缀模糊：用 **ngram/pinyin**
- 分页：**search_after** 替代深分页
- 写入优化：batch bulk、refresh_interval 调大
- 查询优化：**filter 替代 query**、善用 routing
- Mapping：keyword 不分词，text 分词

#### 3）与 MySQL 如何同步？

- **Canal / Debezium** 监听 Binlog
- 准实时同步，无业务侵入

---

### 4. 定时任务场景

#### 1）选型

- 单机：Timer、Scheduled
- 分布式：**XXL-Job / Quartz / PowerJob**（企业首选 XXL-Job）

#### 2）核心问题

- **重复执行**：分布式锁 / 任务唯一ID 幂等
- **失败处理**：重试策略、告警、死信任务
- **阻塞**：任务线程池隔离，避免长任务阻塞

#### 3）标准落地

- 任务执行前：**加分布式锁**
- 执行中：记录执行日志
- 执行后：释放锁，更新状态
- 超时时长：合理设置，避免重复执行

---

### 5. 微服务通信场景

#### 1）服务注册发现

- Nacos / Eureka / Consul
- 健康检查、自动剔除、动态上下线

#### 2）负载均衡

- Ribbon / SpringCloud LoadBalancer
- 策略：轮询、随机、权重、**同区域优先**、预热权重

#### 3）Feign 调用

- 统一：**超时、重试、熔断、日志**
- 异常：降级、fallback、异常码封装
- 避坑：**Feign 重试必须配合幂等**

#### 4）RPC vs HTTP

- **Dubbo/gRPC**：高性能、长连接、序列化高效
- **Feign/HTTP**：简单通用、跨语言
- 内部微服务：**优先 Dubbo/gRPC**

#### 5）高可用保障

- 超时控制
- 熔断降级（Sentinel）
- 重试（只幂等接口）
- 链路追踪（SkyWalking）

---

#### 一句话总结（可直接背）

**MQ 削峰解耦保最终一致，Redis 做锁限流计数器，ES 全文检索与日志，定时任务用 XXL-Job 加锁防重，微服务通信 Feign/RPC 配合熔断限流。**

需要我把**五类全部合成一份「后端面试100场景速记版」**，或者给你做**模拟面试：我问你答+逐题打分**吗？

## 六、业务架构设计类

1. 微服务拆分场景（如按业务域拆分、服务边界定义、接口设计）
2. 接口版本管理（如接口兼容、版本升级、灰度发布）
3. 多租户系统设计（如数据隔离、权限隔离、配置隔离）
4. 容灾/备份场景（如数据备份策略、服务异地多活、故障恢复）
5. 系统监控告警（如关键指标监控、异常告警规则、链路追踪）

这一类是**架构师/高级后端必考题**，考察**系统设计能力、边界把控、架构思想**。我给你整理成**面试可直接口述、简历可直接写、方案可直接落地**的标准答案，结构统一：
**核心原则 → 设计方案 → 关键要点 → 避坑**

---

### 1. 微服务拆分场景

#### 核心原则

**DDD 领域驱动 + 高内聚低耦合 + 单一职责**

#### 拆分步骤（标准方法论）

1. **业务域划分**
   - 订单域、用户域、商品域、支付域、库存域、物流域
2. **限界上下文界定**
   - 每个上下文独立数据、独立服务、独立团队
3. **服务粒度控制**
   - 不过度拆分（避免服务过多、调用链过长）
   - 不过度聚合（避免巨石服务）
4. **接口设计规范**
   - 统一请求/响应结构
   - 统一异常码
   - 接口必须**幂等、可重试、可兼容**

#### 拆分黄金标准

- 数据独立：**一个服务一张主表**，不跨库强关联
- 业务闭环：能独立完成一个业务能力
- 变更频率：相同变更频率放一起
- 团队结构：康威定律——**组织沟通结构 = 系统架构**

#### 避坑

- 严禁为了微服务而微服务
- 严禁跨服务事务滥用
- 严禁循环依赖（A调B、B调A）

---

### 2. 接口版本管理

#### 核心目标

**升级不中断业务、向前兼容、灰度可控**

#### 主流方案

1. **URL 路径版本（最常用）**
   - `/api/v1/order`、`/api/v2/order`
   - 优点：清晰、隔离彻底
2. **请求头版本**
   - Header：`Api-Version: 1`
   - 优点：URL 不变，适合网关统一管控
3. **参数版本**
   - 不推荐，混乱难维护

#### 兼容与升级策略

- **只增不减**：不删除字段、不改字段含义
- **新字段默认值**：保证老客户端可用
- **灰度发布**：按用户/IP/流量比例切流
- **双写兼容**：新旧接口并行一段时间，再下线旧接口

#### 最佳实践

- 对外接口：**必须永久兼容**
- 内部接口：可迭代，但要**文档+通知**
- 大版本升级：走v2，不强行兼容

---

### 3. 多租户系统设计（SaaS 必备）

#### 三种数据隔离方案（必背）

1. **独立数据库（隔离级最高）**
   - 一个租户一个库
   - 优点：安全、隔离好、可单独扩容
   - 缺点：成本高、运维复杂
2. **共享数据库、独立 Schema**
   - 同一库，不同Schema
   - 平衡成本与隔离
3. **共享数据库、共享Schema（最常用）**
   - 所有租户同一张表，通过 `tenant_id` 区分
   - 优点：成本最低、易运维
   - 缺点：开发需处处带租户ID，隔离弱

#### 架构设计要点

- 全局拦截器自动注入 `tenant_id`
- 统一租户上下文工具类
- 权限严格按租户隔离
- 索引必须包含 `tenant_id`
- 定时任务、批量任务必须按租户分片

#### 一句话选型

**中小SaaS：共享库+tenant_id；金融/政务：独立库。**

---

### 4. 容灾/备份场景

#### 4.1 数据备份策略

- **全量备份**：每天一次
- **增量备份**：每小时
- **Binlog 实时备份**：用于Point-in-Time恢复
- 备份校验：**自动恢复演练**，防止备份不可用

#### 4.2 高可用架构

- **主从 + 哨兵**：MySQL/Redis
- **集群分片**：Elasticsearch、Kafka
- **多实例部署**：无状态服务水平扩展

#### 4.3 异地多活（高级）

- **单元化架构**：按用户/区域分片
- **同城双活 + 异地灾备**
- 故障自动切换：DNS/网关层切流
- 核心原则：**数据单向同步、避免双写冲突**

#### 4.4 故障恢复RTO/RPO

- RTO：恢复时间目标（分钟级）
- RPO：数据丢失目标（秒级）
- 核心：**预案 + 演练 + 自动切换**

---

### 5. 系统监控告警

#### 黄金监控体系（**四维一体**）

1. **基础监控**
   - CPU、内存、磁盘、网络、连接数
2. **应用监控**
   - QPS、RT、错误率、线程池、GC
3. **业务监控（最重要）**
   - 下单量、支付成功率、库存扣减、退款率
4. **链路监控**
   - SkyWalking/Pinpoint：全链路Trace、依赖分析

#### 告警规则（标准）

- **分级告警**：P0（宕机）、P1（异常）、P2（预警）
- **告警收敛**：抑制风暴，按故障聚合
- **告警升级**：5分钟未处理自动升级电话/短信
- **无监控不上线**

#### 告警三大原则

- 不误报
- 不漏报
- 不骚扰（夜间降噪）

---

### 六类全部总结（面试终极金句）

微服务按**DDD业务域拆分**，接口**向前兼容+灰度**，多租户**共享库+tenant_id**，容灾**备份+多活+自动切换**，监控**基础+应用+业务+链路四维一体**。

---

如果你需要，我可以立刻把**一～六大类全部合成一份「后端面试100场景终极背诵版（PDF结构）」**，直接拿去背、面试稳过。

### 总结

1. Java大厂业务场景题核心围绕**高并发、高可用、数据一致性、业务逻辑合理性**四大核心诉求展开；
2. 题目类型覆盖**设计类、问题排查类、中间件落地类、架构类**，侧重考察技术落地能力而非单纯的语法；
3. 所有场景题均贴合大厂实际业务（如电商、支付、微服务），注重解决方案的可行性和性价比。

