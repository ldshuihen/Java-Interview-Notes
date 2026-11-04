## Spring Web

### 一、什么是 Spring Web

Spring Web 是 **Spring 生态下统一的 Web 开发基础模块**，是所有 Spring Web 开发的**底层公共依赖**，**SpringMVC就是Spring Web的核心实现**，SpringBoot的`spring‑boot‑starter‑web`底层封装Spring Web(MVC)，用来开发Java后端Web接口。为 Web 应用提供：
HTTP 通信、请求响应封装、报文转换、异常处理、Web 通用工具、客户端请求等基础能力。

它不是完整框架，而是**基础能力底座**，在此之上衍生两大主流 Web 编程模型：

1. **Spring MVC（Servlet 同步阻塞）**：传统主流、企业开发首选，基于Servlet，阻塞IO（普通SpringBoot web应用）
2. **Spring WebFlux（Reactor 异步非阻塞）**：高并发、长连接、响应式编程，响应式非阻塞，Gateway底层使用，不依赖Servlet

#### 核心概念

1. **Servlet**：JavaEE原生Web组件，接收Http请求；SpringMVC底层基于Servlet（DispatcherServlet）。
2. **DispatcherServlet（前端控制器）**：SpringMVC核心总调度，唯一的Servlet。
   所有http请求先进它，负责请求分发、参数解析、调用Controller、视图解析、返回响应。
3. 组件：`Controller、HandlerMapping、HandlerAdapter、ViewResolver、Interceptor、ExceptionResolver`

---

### 二、核心依赖组成

#### 1. 核心包划分

- `spring-web`：**通用 Web 基础包**
  提供 HTTP 工具、请求头/响应工具、RestTemplate、文件上传、跨域基础、报文解析通用能力。
- `spring-webmvc`：**Spring MVC 核心**
  基于 Servlet 规范，实现 MVC 架构、控制器、视图解析、拦截器、数据绑定。
- `spring-webflux`：**响应式 Web 核心**
  基于 Reactor + Netty，异步非阻塞，不依赖 Servlet。

#### 2. SpringBoot 引入方式

```xml
<!-- 传统 MVC 开发（包含 spring-web + spring-webmvc） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 响应式 WebFlux 开发 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

> 注意：`spring-boot-starter-web` 自动内置：Tomcat 容器、jackson 序列化、web 基础配置。

---

### 三、Spring Web 底层运行基础

#### 1. Spring MVC 底层依赖（Servlet阻塞，spring‑boot‑starter‑web）

- 遵循 **Java Servlet 规范**
- 内置容器：默认 **Tomcat**（也可切换 Jetty、Undertow）
- 通信底层：Tomcat 封装 **Socket + TCP + HTTP 协议解析**
- 模型：**同步阻塞 BIO / 线程池模型**
  一个请求占用一个业务线程，请求阻塞等待（数据库、接口调用）。
- 底层Tomcat容器，Servlet API，每一个请求占用一个线程。
- 我们写普通后端接口、Controller‑Service‑Mapper 用这个。> 

> 依赖：`spring‑boot‑starter‑web

#### 2. Spring WebFlux 底层依赖（响应式非阻塞）

- **不依赖 Servlet**
- 底层容器：**Netty（NIO 多路复用）**
- 模型：**异步非阻塞、事件驱动**
  少量线程处理大量请求，适合高并发、网关、WebSocket。
- 不基于Servlet，Netty作为服务器；Mono/Flux响应式编程。
- Spring Cloud Gateway底层就是WebFlux。> 

> ❗不能同时引入web和webflux，会冲突。
> 不使用：`@RequestParam、@RequestBody`等MVC注解；使用ServerWebExchange。



---

### 四、Spring MVC 完整核心架构（重点）

#### 1. 核心执行流程（必背）

```
前端HTTP请求
↓
Tomcat 容器（Socket监听、HTTP解析）
↓
Filter 过滤器（全局拦截、编码、跨域）
↓
前端控制器 DispatcherServlet（MVC核心入口）
↓
1.HandlerMapping：匹配URL→找到对应Controller
↓
2.HandlerAdapter：调用Controller方法、参数绑定
↓
3.Controller：接收参数、调用业务Service
↓
4.结果封装 + 全局异常处理
↓
5.ViewResolver / 消息转换器：渲染JSON/页面
↓
响应结果返回前端
```

#### 2. 九大核心组件

1. **DispatcherServlet**：中央调度器，整个 MVC 的入口
2. **HandlerMapping**：URL 与控制器映射
3. **HandlerAdapter**：适配不同控制器执行方式
4. **Controller**：业务请求处理器（开发者编写）
5. **Service/DAO**：业务逻辑、数据访问
6. **ViewResolver**：视图解析（前后端分离基本不用）
7. **MessageConverter**：报文转换（JSON、String、文件）
8. **HandlerInterceptor**：拦截器（登录校验、日志、权限）
9. **GlobalExceptionHandler**：全局异常统一处理

#### Spring Web MVC完整请求执行流程（面试必背）

1. 浏览器发送HTTP请求，Tomcat接收请求，交给 **DispatcherServlet**
2. **HandlerMapping（处理器映射器）**：根据URL找到对应的Controller方法（Handler）
3. **HandlerAdapter（处理器适配器）**：适配调用Controller方法；做参数解析绑定（`@RequestBody @RequestParam @PathVariable`）
4. 执行Controller业务方法：Controller调用Service→Mapper拿到数据，返回数据对象
5. 返回数据经过**消息转换器 HttpMessageConverter**，把对象序列化为JSON
6. 响应返回给DispatcherServlet，最终Tomcat把JSON返回前端

> 简单记：请求→Tomcat→DispatcherServlet→HandlerMapping→HandlerAdapter→Controller→序列化JSON→响应返回

#### 关键组件作用

1. **DispatcherServlet**：总调度，整个MVC的入口，拦截所有请求。
2. **HandlerMapping**：URL → Controller方法映射，找要执行哪个方法。
3. **HandlerAdapter**：适配执行Controller方法，处理参数绑定。
4. **HttpMessageConverter**：Java对象 ↔ JSON，`@RequestBody`解析请求体；`@ResponseBody`序列化返回JSON。
5. **Interceptor拦截器**：在Controller执行**前后**做处理（登录校验、日志）；**不能捕获Controller异常**。
6. **@RestControllerAdvice全局异常处理器**：统一捕获Controller抛出异常，包装返回JSON。

> ✔拦截器 Interceptor vs Filter过滤器

- Filter：Servlet层，在DispatcherServlet**之前执行**，属于Servlet规范；可以修改request原始请求。
- Interceptor：SpringMVC内部，DispatcherServlet之后，Controller前后执行；能拿到Spring上下文。



#### 分层架构（Spring Web应用）

```
前端HTTP请求
    ↓
Filter(Servlet过滤器)
    ↓
DispatcherServlet
    ↓
Interceptor拦截器
    ↓
Controller（Web层：接收请求、参数校验、调用Service、封装返回）
    ↓
Service（业务层：业务校验、业务编排、事务、调用Mapper）
    ↓
Mapper/DAO（数据访问层，执行SQL操作数据库）
    ↓
MySQL
```

### 各层约束回顾

1. **Controller**：只做web相关；不写业务逻辑；统一返回`Result<T>`；格式校验@Valid。
2. **Service**：业务逻辑、事务`@Transactional`；对象转换DTO/DO/VO；抛业务异常；不感知http。
3. **Mapper**：只做数据库CRUD；只返回DO；不写业务逻辑。

---

### 五、Spring Web 核心功能详解

#### 1. 控制器与请求接收

注解驱动、零配置，快速开发接口

```java
@RestController
@RequestMapping("/user")
public class UserController {

    // 普通GET
    @GetMapping("/get")
    public String getUser() {
        return "success";
    }

    // 路径变量
    @GetMapping("/{id}")
    public String getById(@PathVariable Long id){}

    // 请求参数
    @GetMapping("/list")
    public String list(@RequestParam String name){}

    // JSON请求体
    @PostMapping("/add")
    public Result add(@RequestBody User user){}
}
```

#### 2. 数据绑定 & 类型转换

Spring Web 自动完成：

- 前端字符串 → 后端实体类
- 日期格式化、枚举转换、自动封装
- 支持 `@RequestParam`、`@PathVariable`、`@RequestBody`、`@RequestHeader`

#### 3. 全局跨域支持

内置 CORS 处理，两种方式：

1. 注解跨域：`@CrossOrigin`
2. 全局配置：实现 `WebMvcConfigurer#addCorsMappings`

#### 4. 拦截器 Interceptor

区别于 Filter：

- Filter：Servlet 规范，在 Controller **之前**
- Interceptor：Spring Web 内部，只拦截 Controller 请求
  用途：登录校验、接口限流、请求日志、签名校验

#### 5. 统一异常处理

通过 `@RestControllerAdvice + @ExceptionHandler`
全局捕获所有 Controller 异常，统一返回标准 JSON 格式，避免前端报错白屏。

#### 6. 文件上传下载

`spring-web` 内置文件解析器，直接使用 `MultipartFile` 即可实现上传。

#### 7. HTTP 客户端工具

内置：

- `RestTemplate`：传统同步调用第三方接口
- `WebClient`：WebFlux 提供，异步非阻塞 HTTP 调用

#### 8. 序列化与反序列化

默认整合 Jackson：

- 自动将对象转为 JSON 返回
- 自动解析前端 JSON 为实体类
  可全局配置：日期格式、空值处理、字段忽略等。

---

### 六、Spring MVC VS Spring WebFlux 对比

| 对比项   | Spring MVC             | Spring WebFlux                 |
| -------- | ---------------------- | ------------------------------ |
| 底层容器 | Tomcat（Servlet）      | Netty                          |
| 编程模型 | 同步、阻塞             | 异步、非阻塞、事件驱动         |
| 依赖规范 | Servlet API            | 无Servlet依赖                  |
| 线程模型 | 一线程一请求           | 少量线程处理万级请求           |
| 适用场景 | 常规CRUD、企业业务系统 | 网关、高并发、IM、长连接、秒杀 |
| 学习成本 | 低、上手快             | 高、需掌握响应式编程           |

---

### 七、WebMvcConfigurer 扩展接口

Spring Web 提供**自定义 Web 配置入口**，是开发常用扩展点：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    // 注册拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry){}

    // 跨域配置
    @Override
    public void addCorsMappings(CorsRegistry registry){}

    // 静态资源映射
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry){}

    // 自定义消息转换器
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters){}
}
```

---

### 八、Spring Web 优势总结

1. **高度封装底层网络**
   无需手动写 Socket、TCP、HTTP 解析，专注业务代码。
2. **注解驱动、开发高效**
   零 xml 配置，快速开发 RESTful 接口。
3. **生态完善**
   拦截器、全局异常、跨域、文件、参数校验、定时任务无缝整合。
4. **微服务标准基石**
   SpringCloud 全家桶全部基于 Spring Web 构建。
5. **扩展性极强**
   通过 WebMvcConfigurer、Bean 自定义，灵活改造底层行为。

### 常见功能

1. **文件上传下载**：`MultipartFile`接收上传文件；设置spring.servlet.multipart配置大小。
2. **CORS跨域**：`@CrossOrigin`局部注解，或全局Cors配置类。
3. **拦截器Interceptor**：登录校验、请求日志。
4. **全局异常处理 @RestControllerAdvice**。
5. **参数校验 JSR‑303**。

### Spring Web 测试方式

1. **Controller单元测试**：`@WebMvcTest + MockMvc`，模拟http请求，Mock Service，不启动完整tomcat。
2. **完整集成测试**：`@SpringBootTest`，启动Tomcat，使用`TestRestTemplate`，完整走Controller‑Service‑Mapper‑DB。

---

### 九、面试高频考点（精简背诵版）

1. Spring Web 包含 spring-web、spring-webmvc、spring-webflux。
2. Spring MVC 核心入口：**DispatcherServlet**。
3. 完整请求流程：Tomcat→Filter→DispatcherServlet→HandlerMapping→Controller。
4. Filter 是 Servlet 级别，Interceptor 是 Spring Web 级别。
5. @RequestBody 依靠 **HttpMessageConverter** 实现 JSON 解析。
6. MVC 同步阻塞，WebFlux 异步非阻塞、底层 Netty。
7. 全局异常使用 @RestControllerAdvice 统一处理。

#### 面试背诵精简版

> Spring Web（SpringMVC）是Spring框架的Web模块，底层基于Servlet，核心组件是DispatcherServlet前端控制器。
> 请求流程：http请求经过Tomcat到达DispatcherServlet，HandlerMapping匹配Controller，HandlerAdapter完成参数解析，执行Controller方法；方法返回对象由HttpMessageConverter序列化为JSON返回前端。
> 主要分层：Controller接收http请求做格式校验调用Service；Service执行业务逻辑与事务；Mapper操作数据库。支持拦截器、全局异常、参数校验、文件上传、跨域等能力。
> 区分WebMVC（Servlet阻塞，普通业务接口）和WebFlux（Netty响应式，Gateway使用）。

#### 高频面试点

1. DispatcherServlet作用？

> MVC的前端控制器，所有请求统一入口，负责请求分发、调用处理器、结果处理。

2. Filter和Interceptor区别？

> Filter属于Servlet规范，执行在DispatcherServlet之前；Interceptor属于SpringMVC，在Controller前后执行。

3. @RequestBody底层？

> 依靠HttpMessageConverter读取请求体JSON反序列化对象。

4. SpringMVC和WebFlux区别？

> MVC基于Servlet阻塞，一个请求一个线程；WebFlux基于Netty响应式非阻塞，适合高IO场景，Gateway使用。





## Spring Web Controller层代码规范

> 
> 面向后端面试+实际开发，覆盖命名、入参、返回值、异常、注解、业务边界、最佳实践、反例。Controller职责：**接收http请求、参数校验、调用Service、组装返回结果，不写业务逻辑、不写数据库操作**。

### 一、类层面规范

1. **注解使用**

- 普通REST接口：`@RestController`（等价 `@Controller + @ResponseBody`），不要混用。
- 不要在类上加`@ResponseBody`，优先直接`@RestController`。
- 不要写`@Controller`只用于接口，`@Controller`多用于页面模板跳转。
- `@RequestMapping`写在类上用于统一前缀，版本控制。

```java
// 推荐
@RestController
@RequestMapping("/api/v1/user")
public class UserController {}
```

2. **命名规范**

- 类名：`业务+Controller`，例如 `UserController`、`OrderController`
- 禁止：`UserApi`、`UserAction`、`UserHandler`（项目统一除外）
- 一个Controller尽量对应一个业务模块，不要大杂烩，不要一个Controller几百行。

3. **组件扫描**
`@RestController`被`@Component`元注解标记，会自动注册Bean，**不要额外加@Component**。

### 二、接口方法规范

#### 1. 请求方式严格匹配REST语义

| 注解 | 用途 |
| --- | --- |
| `@GetMapping` | 查询，只读，无副作用 |
| `@PostMapping` | 新增、提交操作 |
| `@PutMapping` | 全量更新 |
| `@PatchMapping` | 局部更新 |
| `@DeleteMapping` | 删除 |

> 
> ❌ 禁止：所有接口统一写`@PostMapping`；不要用GET做删除/修改。

路径命名：

- 小写，横杠分隔，不用下划线、不用驼峰
- 资源名词复数 `/api/v1/orders`，不要动词 `/api/v1/getOrder`

```java
// 好
@GetMapping("/{id}")
public Result<UserVO> getById(@PathVariable Long id){}

// 坏：路径带动词
@GetMapping("/getUserById")
```

#### 2. 方法命名

- 动词+名词：`getUser`、`createOrder`、`deleteById`
- 返回void极少，REST接口尽量返回统一包装对象。

### 三、入参处理规范（高频面试点）

#### 1. 参数注解区分

1. `@RequestBody`：json请求体，POST/PUT，**只能有一个**，接收对象DTO
2. `@RequestParam`：url查询参数 `?name=xxx`，普通简单参数
3. `@PathVariable`：路径变量 `/user/{id}`
4. `@RequestPart`：文件上传，multipart/form-data

> 
> 注意：`@RequestBody`不能接收form表单；`@RequestParam`不能接收json。

#### 2. DTO入参对象

- 请求对象后缀 **DTO**：`UserCreateDTO`、`UserQueryDTO`
- **禁止直接使用Entity实体作为入参**，数据库实体不能直接暴露给前端。
- 入参DTO只包含前端传过来字段，不要数据库多余字段。

#### 3. 参数校验（JSR‑303 Validation）

- Controller层做基础参数校验，不把校验全部丢给Service
- 类上加 `@Valid` / `@Validated`

```java
@PostMapping
public Result<Void> create(@Valid @RequestBody UserCreateDTO dto) {
    userService.create(dto);
    return Result.success();
}
```

常用校验注解：`@NotBlank`、`@NotNull`、`@Size`、`@Min`、`@Max`

> 
> 坑：@RequestBody 必须加@Valid才会触发校验；不加校验注解，校验不生效。

#### 4. 分页参数

不要自己手写pageNum,pageSize重复代码，统一使用PageDTO。

```java
@GetMapping("/list")
public Result<PageVO<UserVO>> page(UserQueryDTO dto){
    PageInfo<UserVO> page = userService.pageList(dto);
    return Result.success(page);
}
```

#### 5. 文件上传

`MultipartFile` 使用`@RequestPart`，限制文件大小在yml配置。

> 
> ❌ 禁止：

1. Controller方法直接写大量参数（超过5个建议封装DTO）
2. 直接接收Entity实体类作为入参
3. 不做参数校验直接传给service

### 四、返回值规范

#### 1. 统一返回包装对象（强制规范）

所有接口统一返回`Result<T>`，禁止返回原始对象、String、void。

```java
@Data
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;

    public static <T> Result<T> success(T data){
        Result<T> r = new Result<>();
        r.setCode(200);
        r.setMsg("success");
        r.setData(data);
        return r;
    }
    public static <T> Result<T> fail(Integer code,String msg){
        Result<T> r = new Result<>();
        r.setCode(code);
        r.setMsg(msg);
        return r;
    }
}
```

- 成功：`Result.success(data)`
- 无返回数据：`Result.success()`
- 失败：`Result.fail(500,"操作失败")`

#### 2. 返回VO视图对象

返回给前端对象后缀 **VO**：`UserVO`、`OrderVO`

> 
> ❌ 严禁直接返回数据库Entity实体，防止数据库敏感字段（密码）泄露。

#### 3. 不要在Controller做对象转换

Controller只调用service，**DO <-> DTO <-> VO 的转换交给Service层**。

```java
// 不推荐，controller写大量转换逻辑
UserVO vo = new UserVO();
vo.setId(entity.getId());

// 推荐
UserVO vo = userService.getUserVo(id);
```

### 五、异常处理规范

1. **Controller中尽量不要写try‑catch**
统一使用 `@RestControllerAdvice + @ExceptionHandler` 全局异常处理器。

- 参数校验异常、业务异常、系统异常全部交给全局捕获。
- Controller只抛出异常，不捕获业务异常。

```java
// 反例
@PostMapping
public Result<?> create(@RequestBody UserCreateDTO dto){
    try {
        userService.create(dto);
    }catch (Exception e){
        return Result.fail(500,e.getMessage());
    }
    return Result.success();
}
```

> 
> 只有极少数特殊场景才允许在Controller捕获异常。

2. 自定义业务异常，Service抛出，全局Advice捕获包装成Result返回。

### 六、Controller职责边界（非常重要，面试常问）

✅ Controller允许做：

1. 接收http请求
2. 参数接收、参数校验
3. 调用Service业务方法
4. 组装返回Result对象
5. 获取request、response、header信息

❌ Controller禁止做：

1. 写业务逻辑
2. 直接操作Mapper/DAO，写SQL
3. 复杂的对象转换
4. 事务注解 `@Transactional` **不要写在Controller**，事务放在Service层
5. 复杂if‑else业务判断
6. 读取数据库、缓存操作

> 
> 原则：Controller越薄越好，只是一个“转发层”。

### 七、常用注解最佳实践

1. `@Slf4j`：lombok日志注解，打印请求日志，记录异常信息

```java
@RestController
@RequestMapping("/api/v1/user")
@Slf4j
public class UserController {
}
```

打印日志：`log.info("创建用户，dto:{}",dto);`

2. 不要在Controller字段注入，推荐构造器注入

```java
// 推荐
private final UserService userService;
public UserController(UserService userService){
    this.userService = userService;
}

// 不推荐字段注入
@Autowired
private UserService userService;
```

### 八、yml配置相关

- 全局日期格式化、jackson序列化配置统一写配置类，不要在Controller每个字段上加`@JsonFormat`。

### 九、接口安全规范

1. 不要直接把path参数、请求参数直接拼接SQL，防止注入。
2. 敏感接口权限校验，使用拦截器/AOP，不要每个controller方法写if判断token。
3. 返回数据过滤密码、手机号等敏感字段，不要返回明文。

### 十、完整标准示例

```java
@RestController
@RequestMapping("/api/v1/user")
@Slf4j
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable @NotNull Long id) {
        UserVO vo = userService.getUserById(id);
        return Result.success(vo);
    }

    @PostMapping
    public Result<Void> create(@Valid @RequestBody UserCreateDTO dto) {
        log.info("新增用户，dto={}", dto);
        userService.createUser(dto);
        return Result.success();
    }

    @GetMapping("/page")
    public Result<PageVO<UserVO>> queryPage(UserQueryDTO dto) {
        PageVO<UserVO> page = userService.queryPage(dto);
        return Result.success(page);
    }
}
```

### 十一、面试高频问题总结

1. **Controller为什么不写业务逻辑？**

> 
> 分层解耦，Controller是web层，只处理http；业务放Service，Service可以被其他地方调用，不依赖web。
2. **为什么不能直接返回Entity？**

> 
> 实体包含数据库全部字段，容易泄露密码等敏感信息；数据库表变更，接口返回会被破坏。
3. **@Transactional为什么不写Controller？**

> 
> Controller是web层，职责不匹配；Controller会被AOP拦截，事务容易失效；Service可以被复用。
4. 入参DTO、返回VO、数据库DO三者区别？

- DTO：前端传给后端
- VO：后端返回给前端
- DO/Entity：数据库映射实体

如果你需要，我可以给你整理一份**面试口述精简版**，或者继续讲Service层代码规范。



### Controller层作用（SpringMVC / Spring‑Web）

**Controller属于Web层，是整个后端程序对外的入口，接收HTTP请求。**

#### 核心职责

1. **接收前端HTTP请求**
接收浏览器/前端发过来的http，解析url、请求头、请求参数、请求体。
处理各类参数：`@RequestBody、@RequestParam、@PathVariable、@RequestPart`。
2. **参数接收与基础参数校验**
通过`@Valid`做**格式校验**：非空、长度、数值范围。

> 
> 只做格式校验；业务校验交给Service层。
3. **调用Service业务层**
本身**不实现业务逻辑**，把解析好的参数传给Service，委托Service完成业务处理。
4. **封装HTTP响应返回给前端**
接收Service返回的数据，包装统一返回对象`Result<T>`，序列化为JSON返回前端。
5. **Web相关能力处理**
获取Request、Response、Header、Cookie；文件上传下载；跨域相关。

#### ✅ Controller能干什么

- 接收http请求，解析参数
- 参数格式校验
- 调用Service
- 组装返回结果
- 获取web请求上下文(request/response)

#### ❌ Controller绝对不应该做什么

1. 不写业务逻辑、复杂if‑else业务判断
2. 不直接调用Mapper操作数据库
3. **不要加@Transactional事务注解**，事务放到Service
4. 不要写复杂对象转换（DO/DTO/VO转换交给Service）
5. 不要捕获大量业务异常，交给全局异常处理器`@RestControllerAdvice`
6. 不能感知数据库，不处理业务规则

> 
> 一句话概括：**Controller只负责“Web相关”，业务逻辑全部下沉Service；Controller越薄越好，只是转发层。**

#### 分层流转

`前端http请求 → Controller(web层) → Service(业务层) → Mapper(数据层) → MySQL`

#### 面试简答背诵版

> 
> Controller是Web控制层，作为接口入口，接收HTTP请求，完成参数接收和格式校验，调用Service执行业务，将结果封装返回给前端；**不编写业务逻辑，业务逻辑交给Service层处理**。

需要我顺带把 Service、Mapper 的简短面试版总结给你吗？

## Spring Web Service层代码规范

> 
> Service层：**业务逻辑层**，核心处理业务规则、事务、数据组装、调用DAO/Mapper、第三方调用；Controller薄，Service厚。
> 分层：Controller(Web) → Service(业务) → Mapper/DAO(数据访问)

### 一、类层面规范

#### 1. 接口 + 实现类模式（主流企业规范）

- 接口：`UserService`，放抽象业务方法
- 实现类：`UserServiceImpl`，实现接口，写业务逻辑> 
> 也有部分项目直接写实现类不加接口；**大厂/传统项目优先接口+Impl**，便于AOP、代理、mock测试。

```java
// 业务接口
public interface UserService {
    UserVO getUserById(Long id);
    void createUser(UserCreateDTO dto);
}

// 实现类
@Service
@Slf4j
public class UserServiceImpl implements UserService {
}
```

注解：

- 实现类上加 `@Service`，把实现交给Spring管理
- **接口上不要加@Service**

命名：

- 接口：`XxxService`
- 实现：`XxxServiceImpl`，禁止乱命名 `UserManager、UserBiz`（项目统一除外）
- 按业务模块拆分，不要一个Service几千行，大业务拆分成多个Service。

#### 2. 依赖注入规范

✅ **构造器注入（强制推荐）**，禁止字段`@Autowired`注入

```java
@Service
@Slf4j
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;
    private final OrderService orderService;

    // 构造器注入，Spring4.3+ 单个构造器可以省略@Autowired
    public UserServiceImpl(UserMapper userMapper, OrderService orderService) {
        this.userMapper = userMapper;
        this.orderService = orderService;
    }
}
```

❌ 反例

```java
@Service
public class UserServiceImpl{
    @Autowired
    private UserMapper userMapper; // 不推荐字段注入
}
```

> 
> 构造器注入好处：循环依赖提前暴露、依赖不可变、方便单元测试。

### 二、方法规范

#### 1. 方法命名

动词+名词，见名知意

- 查询：`getUserById`、`listUser`、`pageQueryUser`
- 新增：`createUser`、`addUser`
- 修改：`updateUser`
- 删除：`deleteUserById`
- 校验：`validateUserXXX`

> 
> 禁止：`processData`、`handleBiz` 语义模糊命名。

#### 2. 返回值规范

1. 查询：返回 **VO / DO / PageVO**，不要返回DTO（DTO是前端入参）
2. 新增/修改/删除：`void` 或者返回主键id
3. **Service禁止直接返回Result包装类**！`Result`是Controller层web返回对象，Service属于业务层，不感知HTTP。

❌错误

```java
Result<UserVO> getUser(Long id); // Service不要返回Result
```

✅正确

```java
UserVO getUserById(Long id);
void createUser(UserCreateDTO dto);
```

#### 3. 入参规范

1. 入参可以接收 **Controller传过来的DTO**；
2. Service内部之间调用，尽量传DO/基本类型，不建议到处传DTO；
3. 参数超过5个，封装对象；
4. Service层**建议二次校验业务规则**（业务校验，不是简单参数非空；简单格式校验放Controller）。

> 
> 区分两层校验：

- Controller(@Valid)：格式校验，非空、长度、数字范围
- Service：**业务校验**，例如：用户是否存在、账户余额是否充足、状态是否允许操作。

### 三、事务 @Transactional（重点面试）

1. **`@Transactional`写在Service层实现类 / Service方法上，绝对不要写Controller**
2. 优先加在**public方法**，private、final、static不会生效（AOP代理限制）
3. 只读查询方法：`@Transactional(readOnly = true)`，优化数据库性能
4. 指定回滚异常：默认只回滚RuntimeException，受检异常不会回滚。

```java
// 推荐写法
@Service
public class OrderServiceImpl implements OrderService {

    // 写操作
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(OrderCreateDTO dto){
        // 新增订单、扣库存、保存记录
    }

    // 查询只读
    @Transactional(readOnly = true)
    public OrderVO getOrder(Long id){
    }
}
```

⚠️事务失效高频坑（面试常考）

1. 方法不是public
2. 同类内部调用 `this.xxx()`，不走代理，事务失效
3. try‑catch捕获异常，异常没有抛出，事务不会回滚
4. 数据库引擎不支持事务（MyISAM）

> 
> 注意：不要类上全部加`@Transactional`，粒度放到方法；查询方法不要开启写事务。

### 四、异常处理规范

1. Service层**禁止捕获业务异常后返回null或者布尔值**；业务不满足直接抛**自定义业务异常**。
2. 不要在Service写大量try‑catch，只有IO、第三方调用等特殊场景捕获。
3. Service抛出异常，向上抛到Controller，交给`@RestControllerAdvice`全局异常处理器统一封装返回。

✅推荐

```java
public void createUser(UserCreateDTO dto){
    UserDO exist = userMapper.selectByUsername(dto.getUsername());
    if(exist != null){
        // 业务不满足，直接抛自定义异常，不要返回false
        throw new BizException(5001,"用户名已存在");
    }
}
```

❌反例

```java
// 不推荐，业务用返回boolean，上层要不停判断，代码臃肿
public boolean createUser(UserCreateDTO dto){
    UserDO exist = userMapper.selectByUsername(dto.getUsername());
    if(exist != null){
        return false;
    }
}
```

> 
> 原则：业务异常用异常驱动，不要靠返回码/布尔值做业务判断。

### 五、职责边界（非常重要）

✅ Service允许做

1. 业务逻辑、业务规则判断
2. 事务控制
3. DTO → DO转换；DO → VO转换（对象拷贝）
4. 调用Mapper操作数据库
5. 调用其他Service、调用第三方接口（http、rpc）
6. 缓存操作（Redis读写）
7. 业务数据组装、聚合多表数据

❌ Service禁止做

1. 不要操作HttpServletRequest、HttpServletResponse，这是Web层职责
2. **不要返回Result，不要感知http状态码**
3. 不要写Servlet相关API
4. 不要写页面跳转、文件response输出
5. 不要写复杂参数格式校验（交给Controller @Valid）
6. 禁止直接写SQL，SQL交给Mapper

> 
> 分层核心：Service不依赖Web环境，可以脱离Controller单独单元测试。

### 六、对象转换规范 DTO / DO / VO

- **DTO**：Controller接收前端入参，传给Service
- **DO/Entity**：数据库实体，Mapper返回
- **VO**：Service组装，返回给Controller，给到前端

流转：
`Controller(DTO) → Service` → DTO转DO交给Mapper；查询DO → Service组装成VO返回Controller。

工具：MapStruct / BeanUtil，不要手写大量set/get。

```java
// service内部
UserDO userDO = UserConvert.INSTANCE.toDo(dto);
userMapper.insert(userDO);

UserDO dbUser = userMapper.selectById(id);
UserVO vo = UserConvert.INSTANCE.toVo(dbUser);
return vo;
```

> 
> 禁止直接把DO传给Controller返回前端，防止敏感字段泄露。

### 七、日志规范

1. `@Slf4j`打在Service实现类上
2. 关键业务节点打印info日志；异常不要e.printStackTrace()，使用log.error打印异常堆栈

```java
log.info("创建用户，username={}",dto.getUsername());

try {
    //第三方调用
}catch (IOException e){
    log.error("调用第三方接口失败",e);
    throw new BizException("调用外部服务失败");
}
```

- 不要打印敏感信息（密码、身份证明文）

### 八、Service之间调用规范

1. ServiceA调用ServiceB，直接注入ServiceB接口，不要new对象
2. **同类中不要this调用本类带事务的方法，事务失效**；需要调用本类事务方法：拿到代理对象调用。

### 九、单元测试

Service层最适合单元测试；因为不依赖web容器，可以mock Mapper做单元测试。

### 十、标准完整示例

```java
// 接口
public interface UserService {
    UserVO getUserById(Long id);
    void createUser(UserCreateDTO dto);
}

//实现类
@Service
@Slf4j
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;

    public UserServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public UserVO getUserById(Long id) {
        if(id == null){
            throw new BizException("id不能为空");
        }
        UserDO userDO = userMapper.selectById(id);
        if(userDO == null){
            throw new BizException("用户不存在");
        }
        return UserConvert.INSTANCE.toVo(userDO);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void createUser(UserCreateDTO dto) {
        log.info("新增用户 username:{}",dto.getUsername());
        UserDO exist = userMapper.selectByUsername(dto.getUsername());
        if(exist != null){
            throw new BizException("用户名已存在");
        }
        UserDO userDO = UserConvert.INSTANCE.toDo(dto);
        userMapper.insert(userDO);
    }
}
```

### 十一、面试高频问答

1. **Service为什么要接口+Impl？**

> 
> 面向接口编程，便于AOP代理、单元测试mock、后期替换实现类。简单项目也可以不用接口，直接写Service实现。
2. **@Transactional为什么放在Service？**

> 
> 事务属于业务操作；Controller是web层，职责不匹配；Service可以脱离web复用。
3. Service和Controller校验如何分工？

> 
> Controller：格式校验（非空、长度）；Service：业务校验（是否存在、状态判断）。
4. Service为什么不能返回Result？

> 
> Result是HTTP响应包装对象，Service是纯业务层，要做到不依赖web，方便单元测试、RPC调用。

需要的话我继续讲 **Mapper/DAO层代码规范**，或者整理面试背诵精简版。



### Service层作用

Service属于**业务逻辑层**，位于Controller和Mapper中间，核心负责处理业务规则。

#### 核心职责

1. **接收Controller传递过来的入参DTO**
   Controller把参数传给Service，Service接手进行业务处理。

2. **业务校验（业务规则校验）**

> Controller做格式校验（@Valid：非空、长度）
> Service做**业务校验**：用户是否存在、余额是否足够、状态是否允许操作等。
> 校验不通过直接抛出自定义业务异常。

3. **编排业务逻辑**
   组合调用多个Mapper、或者调用其他Service，完成完整业务流程。
   比如下单：校验用户 → 保存订单 → 扣减库存 → 生成流水。

4. **事务控制**
   `@Transactional`写在Service方法上，保证一组数据库操作要么全部成功，要么全部回滚。

5. **对象转换**
   DTO → DO（传给Mapper存库）；DO → VO（返回Controller给前端）。

6. **调用外部资源**
   操作Redis缓存、调用第三方接口、RPC调用其他微服务。

7. **返回业务数据**
   返回DO/VO，**不返回Result**，Result是Web层包装对象，Service不感知HTTP。

#### ✅ Service可以做

- 业务逻辑、业务规则判断
- 事务控制
- 编排多个Mapper、其他Service
- 缓存、第三方接口调用
- DTO/DO/VO对象转换

#### ❌ Service不应该做

1. 不处理HttpServletRequest、Response等Web对象，不感知web环境
2. 不返回Result统一返回体
3. 不写SQL，SQL交给Mapper
4. 不做简单格式参数校验（交给Controller @Valid）

> 核心思想：Service是纯业务层，可以脱离Web容器，直接单元测试。

#### 流转链路

`Controller → Service → Mapper → MySQL`

#### 面试背诵简短版

> Service是业务逻辑层，接收Controller传入参数，完成业务校验、业务逻辑编排、事务控制，调用Mapper完成数据库操作，也可以操作缓存、调用第三方服务；返回业务数据给Controller，**不感知HTTP，不处理web相关逻辑**。

需要我把Mapper层也整理成这种面试简答版吗？

## DAO / Mapper层代码规范（MyBatis / MyBatis‑Plus）

> 
> Mapper层职责：**只负责数据访问，和数据库交互**；只做CRUD，**不写业务逻辑**。
> 分层链路：`Controller → Service → Mapper → Mysql`

> 说明：DAO是传统叫法，现在项目基本叫 Mapper；MyBatis接口叫Mapper，对应xml映射文件。







### 一、类/接口规范

#### 1. 命名规范

- Mapper接口：`实体名 + Mapper`，例如 `UserMapper`、`OrderMapper`
- Mapper xml文件：`UserMapper.xml`，和接口名完全一致
- DO数据库实体：`UserDO` / `User`，对应数据库表，字段和表尽量一一对应

```java
// Mapper接口，不需要加@Service/@Component
@Mapper
public interface UserMapper {
}
```

注解：

1. `@Mapper`：MyBatis注解，标记为Mapper接口，交给MyBatis扫描。
2. 也可以启动类加`@MapperScan("com.xxx.mapper")`批量扫描，接口上就可以省略`@Mapper`。> 
> ❌不要给Mapper接口加`@Service、@Component`。

#### 2. 文件存放位置

- Java Mapper接口：`com.xxx.mapper`
- xml映射文件：
  - 方式1：resources/mapper 下面，`UserMapper.xml`
  - 方式2：和Mapper接口同包（不推荐，打包容易漏xml）> 
  > 企业主流：resources/mapper 存放xml文件。

xml文件namespace **必须和Mapper接口全限定类名完全一致**

```
<mapper namespace="com.xxx.mapper.UserMapper">
```

### 二、Mapper接口方法规范

#### 1. 方法命名

动词+业务，见名知意，统一风格

- 查询：`selectById`、`selectList`、`selectPage`、`selectByUsername`
- 新增：`insert`、`insertBatch`
- 更新：`updateById`、`updateByCondition`
- 删除：`deleteById`、`deleteBatchIds`

> 
> ❌禁止命名：`getUser`、`queryAll`、`procXXX`，团队统一，不要混用多种动词。

#### 2. 返回值规范

1. 查询单条：`UserDO selectById(Long id);`
2. 查询多条：`List<UserDO> selectList(QueryWrapper wrapper);`
3. 分页：`Page<UserDO> selectPage(Page<UserDO> page, @Param("query") UserQueryDTO dto);`
4. 新增、修改、删除：**返回int**，代表受影响行数。

```
// 返回int，数据库影响行数
int insert(UserDO userDO);
int updateById(UserDO userDO);
int deleteById(Long id);
```

> 
> 注意：Mapper返回永远是 **DO实体**，**绝对不能返回DTO、VO**，DTO/VO属于上层业务对象。

#### 3. 入参规范

1. 单个参数可以直接传；多个参数必须使用`@Param("xxx")`注解

```java
// 多参数，必须@Param
List<UserDO> selectByCondition(@Param("name") String name, @Param("status") Integer status);

// 对象参数
List<UserDO> selectByQuery(@Param("query") UserQueryDTO dto);
```

> 
> 坑：多个参数不加`@Param`，xml里面拿不到参数，会报`Parameter 'xxx' not found`。

2. 批量操作：集合参数用`@Param("list") List<UserDO> list`

✅批量插入

```java
int insertBatch(@Param("list") List<UserDO> userList);
```

#### 4. MyBatis‑Plus注意点

继承`BaseMapper<T>`，自带基础CRUD，不要重复手写已经有的方法。

```java
@Mapper
public interface UserMapper extends BaseMapper<UserDO> {
    // 只写BaseMapper没有的复杂sql
}
```

> 
> 不要滥用BaseMapper，复杂多表查询自己写xml。

### 三、xml映射文件规范

1. 标签顺序：`resultMap → sql片段 → select → insert → update → delete`
2. select id 必须和Mapper接口方法名一一对应。
3. 优先使用`resultMap`，不要直接写`resultType`做复杂映射；简单单表可以resultType。
4. 字段和实体属性，尽量开启驼峰自动转换 `map‑underscore‑to‑camel‑case: true`，避免重复写列映射。

```xml
<resultMap id="UserResultMap" type="com.xxx.entity.UserDO">
    <id column="id" property="id"/>
    <result column="user_name" property="userName"/>
</resultMap>
```

5. sql片段：公共查询字段抽`<sql>`复用，减少重复代码

```xml
<sql id="user_columns">
    id,user_name,password,phone,create_time
</sql>
```

6. 动态sql优先使用`<if> <where> <set> <foreach>`

- `<where>`自动去掉多余and/or；不要自己手写where 1=1（旧写法不推荐）

```xml
<select id="selectByQuery" resultMap="UserResultMap">
    select <include refid="user_columns"/>
    from t_user
    <where>
        <if test="query.name != null and query.name != ''">
            and user_name like concat('%',#{query.name},'%')
        </if>
        <if test="query.status != null">
            and status = #{query.status}
        </if>
    </where>
</select>
```

update使用`<set>`标签，自动去掉末尾逗号

```xml
<update id="updateUser">
    update t_user
    <set>
        <if test="userName != null">user_name = #{userName},</if>
        <if test="phone != null">phone = #{phone},</if>
    </set>
    where id = #{id}
</update>
```

foreach批量插入

```xml
<insert id="insertBatch">
    insert into t_user(user_name,phone)
    values
    <foreach collection="list" item="item" separator=",">
        (#{item.userName},#{item.phone})
    </foreach>
</insert>
```

#### ⚠️xml禁止事项

1. ❌禁止直接拼接 `${}`接收前端传入参数，防止SQL注入；**普通传参一律用`#{}`预编译**。> 
> `${}`仅用于动态表名、动态列。
2. ❌不要写select *；明确写需要查询的字段。
3. ❌Mapper/xml中不要写业务逻辑，不要做if‑else业务判断，业务逻辑放到Service层。
4. ❌不要在xml写大段复杂计算，数据库尽量只做存取。

### 四、DO数据库实体规范

```java
@Data
@TableName("t_user") // MyBatis‑Plus指定表名
public class UserDO {
    private Long id;
    private String userName;
    private String phone;
    private String password;
    private Integer status;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

1. 数据库下划线，实体驼峰命名；开启驼峰转换。
2. 基础字段统一：`id、create_time、update_time、create_by、update_by、is_deleted`
3. 逻辑删除字段 `is_deleted`，MyBatis‑Plus配置全局逻辑删除。
4. 时间类型推荐 `LocalDateTime`，不要使用Date。
5. **DO只映射数据库表，不要加业务字段**，业务扩展字段放到VO/DTO。
6. 禁止DO里面写业务方法，DO就是简单POJO。

### 五、职责边界（面试高频）

✅ Mapper允许：

1. 数据库CRUD操作
2. 简单多表关联查询（join）
3. 分页查询
4. 批量新增、批量更新
5. 简单统计count、sum

❌ Mapper禁止：

1. ❌写业务逻辑、if‑else业务判断
2. ❌事务，事务在Service层`@Transactional`
3. ❌处理字符串、日期转换等业务计算
4. ❌返回VO、DTO，只能返回DO
5. ❌调用其他Mapper；Mapper之间禁止互相调用，多个Mapper调用交给Service层编排。

> 
> 核心原则：Mapper只关心“怎么拿数据”，**不关心“拿数据用来干什么”**。

### 六、分页规范

MyBatis‑Plus使用`Page`对象，分页插件`MybatisPlusInterceptor`；
Mapper方法第一个参数传`Page<T>`，返回`Page<T>`，自动分页。

```
// mapper接口
Page<UserDO> selectPage(Page<UserDO> page, @Param("query") UserQueryDTO dto);
```

> 
> 不要自己手写limit分页，统一分页插件。

### 七、批量操作规范

1. 批量插入不要循环调用mapper.insert()，会多次交互数据库，性能差；用`insertBatch`批量。
2. 批量数据量大，注意分批，一次性不要上万条，避免数据包过大。

### 八、常见坑总结

1. 多参数忘记`@Param` → 参数找不到报错
2. xml namespace和Mapper接口不一致
3. 使用`${}`接收普通参数，产生SQL注入风险
4. Mapper互相调用；Mapper不能注入Mapper，Service层编排多个Mapper
5. DO里面放业务字段
6. select * 查询多余字段，性能差，表变更容易出问题
7. 业务逻辑写在xml里面，sql写的很重，后续维护困难

### 九、完整示例

Mapper接口

```java
@Mapper
public interface UserMapper extends BaseMapper<UserDO> {

    UserDO selectByUsername(@Param("username") String username);

    List<UserDO> selectList(@Param("query") UserQueryDTO dto);

    Page<UserDO> selectPage(Page<UserDO> page, @Param("query") UserQueryDTO dto);

    int insertBatch(@Param("list") List<UserDO> list);
}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xxx.mapper.UserMapper">

    <resultMap id="UserResultMap" type="com.xxx.entity.UserDO">
        <id column="id" property="id"/>
    </resultMap>

    <sql id="user_cols">
        id,user_name,phone,status,create_time,update_time
    </sql>

    <select id="selectByUsername" resultMap="UserResultMap">
        select <include refid="user_cols"/>
        from t_user
        where user_name = #{username}
    </select>

</mapper>
```

### 十、面试高频问答

1. **Mapper为什么不能互相调用？**

> 
> Mapper只负责单表/查询操作；多个Mapper组合属于业务编排，交给Service层。Mapper互相调用会耦合，不方便维护、事务不好控制。
2. `#{}` 和 `${}`区别？

> 
> `#{}`预编译，防止SQL注入；`${}`直接字符串替换，有注入风险，只用于动态表名、列名。
3. DO、DTO、VO三者区别？

- DO：数据库映射实体，Mapper输出；
- DTO：前端入参，Controller传给Service；
- VO：返回前端，Service组装输出。

4. 为什么不建议写select * ?> 
> 取出无用字段消耗IO；表增加删除字段，上层代码会受影响，可控性差。

需要的话，我可以给你整理一份**全套分层（Controller‑Service‑Mapper）完整面试背诵版**。



### DAO / Mapper层作用

Mapper（DAO）属于**数据访问层**，负责和数据库交互，位于Service下层。

#### 核心职责

1. **负责数据库CRUD操作**
完成新增、删除、修改、查询，向数据库发送SQL，获取数据库返回的数据。

2. **数据对象映射**
把数据库查询出来的结果集，封装为DO（数据库实体）返回给Service。

3. **支持批量操作、分页、多表查询**
提供批量增删改、分页查询、join关联查询能力。

> Mapper只关心**数据存取**，不关心业务含义。

#### ✅ Mapper可以做

- 执行SQL，和数据库交互
- 单表、多表关联查询
- 批量插入、更新、删除
- count、sum等简单统计查询

#### ❌ Mapper不应该做

1. **不写业务逻辑、业务判断**，业务逻辑放到Service
2. **不处理事务**，事务由Service层`@Transactional`控制
3. Mapper之间不能互相调用，多Mapper编排交给Service
4. 只返回DO实体，**不能返回DTO、VO**
5. 不做复杂业务计算，计算逻辑上移Service

#### 流转链路

`Controller → Service → Mapper → MySQL`

#### 面试背诵简短版

> Mapper/DAO是数据访问层，专门负责与数据库交互，执行CRUD SQL，把数据库结果封装为DO返回给Service；**只做数据存取，不编写业务逻辑，事务控制在Service层**。

---

#### 三层完整速记（面试直接背）

1. **Controller(Web层)**：接收http请求，参数格式校验，调用Service，封装返回结果；**不写业务**。
2. **Service(业务层)**：业务校验、业务编排、事务控制，调用Mapper；**不感知HTTP**。
3. **Mapper(数据访问层)**：执行SQL操作数据库；**只存取数据，无业务**。

要不要我把这三层整理一张对比表格方便复习？



## Controller‑Service‑Mapper 三层

### Controller‑Service‑Mapper 三层交互流程

#### 完整执行流程

> 
> 请求流向：**前端HTTP请求 → Controller → Service → Mapper → MySQL**
> 结果回流：**MySQL → Mapper → Service → Controller → 前端**

##### 1）正向调用（请求往下走）

1. **前端发起HTTP请求**（POST/GET等），携带请求参数。
2. **Controller接收请求**
   - 使用`@RequestBody / @RequestParam / @PathVariable`接收参数，封装成**DTO**。
   - 通过`@Valid`做**格式校验**（非空、长度、范围）。校验失败直接全局异常返回。
   - Controller**不做业务逻辑**，把DTO传给Service的方法。
3. **Service接收DTO**
   - 做**业务校验**（用户是否存在、状态是否允许操作等），不满足抛自定义业务异常。
   - 做对象转换：**DTO → DO**，DO是数据库实体。
   - 编排业务逻辑，控制事务`@Transactional`，调用Mapper。
4. **Mapper接收DO/参数**
   - 执行SQL，访问MySQL数据库。
5. **MySQL执行SQL，返回查询结果**

##### 2）结果回流（数据往上返回）

1. MySQL返回数据集 → **Mapper封装成DO对象/List 返回给Service**> 
> Mapper只返回DO，不返回VO、DTO。
2. Service拿到DO：
   - 将DO转换为**VO**（面向前端的数据对象）。
   - 返回VO / List / 主键 / void 给 Controller。> 
   > Service**不返回Result**，Result是Web层包装类。
3. Controller拿到Service返回的VO：
   - 使用统一包装类`Result<T>`包装VO。
   - 序列化为JSON，通过HTTP响应返回前端。

---

#### 对象流转非常重要（面试高频）

```
前端JSON → DTO(Controller入参)
    ↓
Service：DTO → DO 传给Mapper
    ↓
Mapper ↔ MySQL（只使用DO）
    ↓
Service：DO → VO 返回Controller
    ↓
Controller：VO包装为Result → JSON返回前端
```

- **DTO**：前端 → 后端（入参）
- **DO**：Service ↔ Mapper ↔ MySQL（数据库实体）
- **VO**：后端 → 前端（出参）
- **Result**：Controller层专用http返回包装对象

> 
> ❌禁忌：
> 
> 
> 1. Controller直接把DO返回前端（泄露敏感字段）
> 2. Service返回Result（业务层不能感知HTTP）
> 3. Mapper返回VO/DTO

#### 异常流转

1. Controller @Valid校验失败 → 抛出参数异常 → `@RestControllerAdvice`捕获，封装Result返回。
2. Service业务校验不通过 → 抛出BizException业务异常 → 向上抛到Controller，全局异常处理器捕获。
3. Mapper数据库异常 → 抛出SQL异常，向上抛到Service；如果加了`@Transactional`触发事务回滚，继续向上抛，全局异常处理。

> 
> 原则：**异常向上抛出，不在每层到处写try‑catch，统一全局处理**。

#### 代码极简伪代码，直观看懂交互

```
// Controller
@PostMapping
public Result<UserVO> create(@Valid @RequestBody UserCreateDTO dto){
    // 只调用service，不写业务
    UserVO vo = userService.create(dto);
    return Result.success(vo);
}

// Service
@Override
@Transactional(rollbackFor = Exception.class)
public UserVO create(UserCreateDTO dto){
    // 业务校验
    if(userMapper.selectByName(dto.getName()) != null){
        throw new BizException("用户名已存在");
    }
    // DTO转DO
    UserDO userDO = convert.toDo(dto);
    userMapper.insert(userDO);
    // DO转VO返回
    return convert.toVo(userDO);
}

// Mapper
int insert(UserDO userDO);
```

#### 面试背诵总结

> 
> 前端发送http请求到达Controller，Controller接收参数封装DTO，完成格式校验，调用Service；Service进行业务校验、业务编排、事务控制，DTO转DO调用Mapper；Mapper执行SQL操作MySQL，返回DO给Service；Service把DO转为VO返回Controller；Controller将VO包装Result，以JSON返回前端。异常逐层上抛，由全局异常处理器统一处理。

需要我把 **Controller、Service、Mapper三层的允许/禁止行为** 合并成一张速记对比表吗？方便面试快速复习。





### 如何测试 Controller / Service / Mapper 三层交互是否正常

分三种测试：**单元测试 UnitTest、集成测试、接口测试**

> 
> 核心区别：

- 单元测试：只测当前层，**依赖全部Mock**，不启动完整容器、不连真实数据库
- 集成测试：启动Spring容器，真实注入Bean，**可以连真实/测试数据库**，测多层交互
- 接口测试：直接发HTTP请求，走完整链路，模拟前端调用

#### 1、Mapper层（DAO）测试

Mapper直接和数据库打交道，**不能mock数据库，需要测试库**。

> 
> 目标：验证SQL是否正确，查询、新增、修改、删除结果是否符合预期。

##### 方式：Spring集成测试 + H2内存库 / 独立测试MySQL库

注解：`@SpringBootTest`

```
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testSelectById() {
        UserDO user = userMapper.selectById(1L);
        assertNotNull(user);
        assertEquals("张三",user.getUserName());
    }
}
```

✅要点

1. 使用**专门的测试数据库**，不要操作生产库；可以用H2内存数据库，测试结束自动销毁数据。
2. 测试前后清理数据，避免脏数据互相影响。
3. 验证返回DO字段、受影响行数是否正确。

❌不要Mock Mapper，Mock就测不到SQL逻辑。

> 
> 测试重点：SQL语句、字段映射、条件查询、批量操作。

#### 2、Service层测试

Service依赖Mapper，还可能依赖其他Service。两种方案。

##### 方案A：单元测试（Mock Mapper，推荐，速度快）

使用 `@ExtendWith(MockitoExtension.class)`，**不启动Spring容器，Mock掉Mapper**，只测Service业务逻辑。

> 
> 关注点：业务逻辑、业务校验、异常抛出、对象转换、事务逻辑。

```
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    // mock依赖的Mapper
    @Mock
    private UserMapper userMapper;
    // 注入mock到service
    @InjectMocks
    private UserServiceImpl userService;

    @Test
    void testGetUserById(){
        // 模拟mapper返回数据
        UserDO mockDo = new UserDO();
        mockDo.setId(1L);
        mockDo.setUserName("测试");
        when(userMapper.selectById(1L)).thenReturn(mockDo);

        // 执行service方法
        UserVO vo = userService.getUserById(1L);

        // 断言结果
        assertNotNull(vo);
        assertEquals("测试",vo.getUserName());
        // 校验mapper方法确实被调用1次
        verify(userMapper,times(1)).selectById(1L);
    }

    @Test
    void testUserNotExistThrowException(){
        // mock返回null
        when(userMapper.selectById(999L)).thenReturn(null);
        // 断言抛出业务异常
        Assertions.assertThrows(BizException.class,()->{
            userService.getUserById(999L);
        });
    }
}
```

> 
> 单元测试Service：**不跑真实SQL，只验证业务代码逻辑是否正确**。

##### 方案B：集成测试（@SpringBootTest，真实Mapper，连测试库）

启动完整Spring，真实注入Mapper，操作测试数据库，测完整业务+数据库交互。
适合测试**事务是否生效**，Mock方式测不到事务。

```
@SpringBootTest
class UserServiceIntegrationTest {
    @Autowired
    private UserService userService;

    @Test
    void testCreateUser(){
        UserCreateDTO dto = new UserCreateDTO();
        dto.setUserName("test001");
        userService.createUser(dto);
    }
}
```

> 
> Service测试重点：业务校验是否抛异常；对象转换；事务回滚；调用Mapper次数是否正确。

#### 3、Controller层测试

Controller依赖Service，接收http请求。两种测试方式：

##### 方案A：MockMvc单元测试（最常用，不启动web服务端口）

`@WebMvcTest(UserController.class)`，只加载web层组件，**Mock Service**，模拟发送http请求，测试接口参数校验、返回结果、状态码。

> 
> 不会启动完整Spring容器，速度快。

```
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void testGetUser() throws Exception {
        UserVO vo = new UserVO();
        vo.setId(1L);
        vo.setUserName("test");
        when(userService.getUserById(1L)).thenReturn(vo);

        //模拟GET请求
        mockMvc.perform(MockMvcRequestBuilders.get("/api/v1/user/1")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                //断言返回json
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data.userName").value("test"));
    }
}
```

测试点：参数校验@Valid、http状态码、统一返回Result结构、路径、请求方式。

##### 方案B：完整集成测试 `@SpringBootTest + TestRestTemplate / WebTestClient`

**启动真实web端口**，真正发http请求，加载全部Spring上下文，Service、Mapper全部真实Bean，连接测试数据库。

> 
> 真正完整三层交互：Http → Controller → Service → Mapper → DB，完整链路跑通。适合做整体集成。

```
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserApiIntegrationTest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void testApi(){
        String url = "http://localhost:" + port + "/api/v1/user/1";
        ResponseEntity<Result> resp = restTemplate.getForEntity(url, Result.class);
        Assertions.assertEquals(200,resp.getBody().getCode());
    }
}
```

##### 方案C：Postman / Apifox（接口黑盒测试）

部署应用，直接http调用接口，属于手工测试，不写代码。

#### 📌三层交互整体怎么测（面试高频）

1. **单元测试（分层隔离）**

- Mapper：`@SpringBootTest` + 测试库，**不mock mapper**，验证SQL。
- Service：`Mockito`，Mock Mapper，测试业务逻辑；事务场景用集成测试。
- Controller：`@WebMvcTest + MockMvc`，Mock Service，测试http接口、参数校验。

> 
> 单元测试：**每层单独测，把下层依赖Mock掉，定位问题精准**。

2. **集成测试（测多层交互）**`@SpringBootTest`完整Spring容器，真实Bean，连接测试数据库。
**Controller → Service → Mapper → DB全部真实执行，验证三层交互链路是否正常。**
3. **接口测试（黑盒）**
Apifox/Postman，模拟前端请求，验证完整业务链路。

#### 不同测试的使用场景对比表

| 测试类型 | 注解 | 是否启动web | 是否mock依赖 | 是否连真实数据库 | 用途 |
| --- | --- | --- | --- | --- | --- |
| Mapper集成测试 | @SpringBootTest | 否 | 不mock Mapper | 测试库 | 验证SQL正确性 |
| Service单元测试 | @ExtendWith(MockitoExtension.class) | 否 | Mock Mapper | 不连库 | 业务逻辑、校验、异常 |
| Service集成测试 | @SpringBootTest | 否 | 不mock | 测试库 | 测试事务、真实数据库交互 |
| Controller单元测试 | @WebMvcTest + MockMvc | 不启端口 | Mock Service | 不连库 | 接口参数校验、返回JSON |
| Controller完整集成 | @SpringBootTest(RANDOM_PORT) | 随机端口 | 不mock | 测试库 | **完整三层交互测试** |

#### 面试背诵精简版

> 
> 1. Mapper层：使用`@SpringBootTest`集成测试，连接测试数据库，不能mock Mapper，校验SQL执行结果。
> 2. Service层：单元测试用Mockito Mock Mapper，只测试业务逻辑；事务场景使用集成测试连接测试库。
> 3. Controller层：使用`@WebMvcTest + MockMvc`模拟http请求，Mock Service，测试接口参数校验、返回值；
> 4. 如果要完整测试**Controller‑Service‑Mapper三层交互**，使用`@SpringBootTest`完整集成测试，加载全部Spring Bean，连接测试数据库，走完完整调用链路；也可以使用Postman/Apifox做接口黑盒测试。

> 
> 补充考点：
> 
> 
> - 单元测试：隔离依赖，测试速度快，定位单一层bug；
> - 集成测试：测多层之间协作交互，但是速度慢，依赖数据库环境。

如果你需要，我可以整理Mockito常见注解速记。



## Spring Cloud Gateway 代码&配置规范

> Gateway定位：**API网关，请求入口**，负责路由转发、鉴权、限流、跨域、日志、熔断、请求改写；不写业务逻辑。
> 核心组件：`Route(路由)`、`Predicate(断言)`、`Filter(过滤器)`
> 两种开发模式：**yml配置方式、Java代码RouteBuilder方式**；企业主流：yaml配置路由 + Java自定义全局Filter。

### 一、项目模块与依赖规范

#### 1. Maven依赖

> 
> ⚠️Gateway**不能引入spring‑web starter**，底层是Reactor‑Netty响应式WebFlux，冲突会启动报错。

```xml
<!-- spring cloud gateway 核心依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!-- nacos注册发现 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!-- 限流、redis限流需要 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

❌禁止引入：`spring‑boot‑starter‑web`，会造成启动异常。

#### 2. 包结构规范

```
com.xxx.gateway
├── GatewayApplication.java        //启动类
├── config                         //配置类
│   ├── GatewayRouteConfig.java    //Java代码方式路由（可选）
│   └── CorsConfig.java            //全局跨域配置
├── filter                         //自定义过滤器
│   ├── global                     //全局过滤器 GlobalFilter
│   │   ├── AuthGlobalFilter.java  //token鉴权
│   │   ├── LogGlobalFilter.java   //请求日志打印
│   │   └── RateLimitGlobalFilter.java //限流
│   └── factory                    //局部过滤器工厂 GatewayFilterFactory
├── util                           //工具类
└── exception                      //网关自定义异常、全局错误处理
```

### 二、路由配置规范（yml优先）

> 
> 生产环境优先使用 **application.yml配置路由**；Java代码路由适合动态测试，不建议大量业务路由写死在代码。

#### yml路由标准模板

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service-route       #路由id，全局唯一，业务名+route，不要随便写uuid
          uri: lb://user-service       #lb代表负载均衡，指向nacos服务名；不要写死ip:port
          predicates:                  #断言，匹配请求条件
            - Path=/api/v1/user/**     #路径匹配，匹配规则明确
            - Method=GET,POST,PUT,DELETE
          filters:
            - StripPrefix=2            #去掉路径前2段 /api/v1 → 转发到后端去掉前缀
            - name: RequestRateLimiter #内置限流过滤器
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
      globalcors: #全局跨域，不要每个route重复写cors
        cors-configurations:
          '[/**]':
            allowed-origins: "*"
            allowed-methods: "*"
            allowed-headers: "*"
            allow-credentials: false
```

#### 路由编写约束

1. `id`命名规范：`{服务名}-route`，保证唯一，便于日志排查。
2. uri：微服务必须使用 `lb://服务名`，不要硬编码`http://127.0.0.1:8080`。
3. predicates：尽量用`Path`+`Method`组合，断言不要写的过于复杂。
4. filters：内置过滤器写在对应route下；**全局逻辑（鉴权、日志）写GlobalFilter，不要每个路由重复配置**。
5. 跨域：统一`globalcors`全局配置，禁止每个路由重复配置跨域。
6. 路由顺序：数字越小优先级越高，`order`，公共通用路由order设小一点。

> 
> 动态路由：生产推荐使用Nacos配置中心存放gateway路由配置，热更新，不用重启网关。

#### ❌不推荐做法

- 大量业务路由写死Java代码；
- 每个路由重复写鉴权、跨域；
- uri硬编码ip端口；
- Path写模糊 `/**` 全部匹配，容易路由冲突。

### 三、过滤器规范（重点面试）

Gateway过滤器分两类：

1. **GlobalFilter 全局过滤器**：对**所有路由全部生效**，鉴权、日志、限流、请求头改写；开发写的最多。
2. **GatewayFilter 局部过滤器**：只对某一个route生效，yml配置内置过滤器。

> 
> Gateway是**响应式WebFlux**，全部使用Mono/Flux，**不能使用Servlet Api(HttpServletRequest)**。

#### 1. GlobalFilter开发规范

实现 `GlobalFilter, Ordered` 两个接口

- `getOrder()`：设置执行顺序，数字越小越先执行。> 
> 顺序参考：
> 
> 
> - -100：鉴权过滤器
> - -50：请求日志
> - 0：网关内置过滤器
> - 100：响应处理过滤器

示例：鉴权全局过滤器

```java
@Component
@Slf4j
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        //1. 获取token
        String token = request.getHeaders().getFirst("token");
        //2. 校验token；如果不通过返回401
        if (!checkToken(token)) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            byte[] bytes = JSON.toJSONBytes(Result.fail(401,"token无效"));
            DataBuffer buffer = response.bufferFactory().wrap(bytes);
            return response.writeWith(Mono.just(buffer));
        }
        //token合法，放行，往下传递过滤器链
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -100; //优先级高，优先执行鉴权
    }
}
```

##### 过滤器编码强制约束

1. 必须加`@Component`交给Spring管理；
2. 实现`Ordered`，明确order执行顺序，避免过滤器执行顺序混乱；
3. **禁止使用Servlet相关API：HttpServletRequest、HttpServletResponse**，全部使用`ServerWebExchange、ServerHttpRequest、ServerHttpResponse`；
4. 校验失败直接返回响应，**不要继续调用chain.filter(exchange)**；校验通过必须放行；
5. 异常捕获：响应式要捕获Mono异常，不要用普通try-catch直接捕获异步逻辑；
6. 不要在过滤器写复杂业务逻辑；只做网关层面通用处理。

✅允许写在GlobalFilter：

- token鉴权、token解析
- 请求日志、响应日志
- 请求头增加、修改
- IP限流、Redis限流
- 黑白名单

❌禁止写在Filter：

- 业务逻辑、数据库查询；如果必须查库，使用Reactive响应式客户端，**禁止同步阻塞JDBC**；
- 复杂业务判断；网关尽量轻量化。

### 四、全局异常处理规范

Gateway是WebFlux，**@RestControllerAdvice无效，不能捕获网关层异常**。

> 
> 必须实现 `ErrorWebExceptionHandler` 做网关全局异常处理。

```java
@Component
@Slf4j
public class GatewayGlobalExceptionHandler implements ErrorWebExceptionHandler {
    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        //处理路由失败、服务不可用、超时、鉴权异常，统一返回JSON
    }
}
```

常见网关异常：

1. 服务找不到：503；
2. 转发超时；
3. token非法401；
4. 请求限流429。

> 
> 不要让网关返回默认html错误页面，统一返回和业务一致的Result JSON格式。

### 五、限流规范

1. 官方内置`RequestRateLimiter`基于Redis令牌桶，优先使用；
2. 限流key：推荐使用IP、用户token；
3. 限流逻辑优先yml内置过滤器；复杂自定义限流写GlobalFilter。> 
> ⚠️必须使用 `spring‑boot‑starter‑data‑redis‑reactive` 响应式redis，不要普通redis starter，会阻塞。

### 六、跨域规范

1. **优先yml globalcors全局配置**；
2. 不建议写自定义filter处理跨域，容易出现options预检请求放行问题；
3. 如果自定义Filter处理跨域，要处理OPTIONS请求直接返回200，不往下转发。

### 七、日志规范

1. GlobalFilter打印日志：打印请求url、请求方式、客户端ip、响应码、耗时；
2. 不要打印token明文，日志脱敏；
3. 开启gateway访问日志，配置logback输出网关访问日志。

### 八、职责边界（面试高频）

✅网关Gateway允许做：

1. 请求路由转发、负载均衡(lb)
2. 全局鉴权、token校验
3. 限流、熔断、黑白名单
4. 请求头、响应头修改，路径重写、剥离前缀
5. 跨域处理、统一异常响应
6. 请求日志、监控埋点

❌Gateway禁止做：

1. ❌写业务逻辑，不要操作数据库（禁止同步JDBC）；
2. ❌不要做复杂参数解析、业务校验；业务交给后端微服务；
3. ❌引入spring‑web starter，会冲突；
4. ❌网关过滤器中做阻塞操作（Thread.sleep、同步http调用），Gateway是响应式非阻塞，阻塞会耗尽netty线程。

> 
> 核心原则：网关轻量化，只做横切通用能力，业务逻辑下沉到各个微服务。

### 九、常见坑点总结

1. 引入spring‑web starter，网关启动报错；
2. GlobalFilter忘记设置order，过滤器执行顺序错乱；
3. 使用Servlet的HttpServletRequest，代码报错；
4. @RestControllerAdvice不能捕获网关异常，需要实现ErrorWebExceptionHandler；
5. 使用普通非响应式Redis，造成阻塞；
6. 跨域没有处理OPTIONS预检请求；
7. 过滤器校验失败还执行chain.filter(exchange)，没有直接返回响应；
8. 路由id重复，路由覆盖失效；
9. uri写死ip，不使用lb负载均衡。

### 十、面试高频问题

1. Gateway和Zuul区别？

> 
> Gateway基于WebFlux响应式非阻塞；Zuul1.x是Servlet阻塞；Gateway内置断言过滤器，性能更好。
2. GlobalFilter的order作用？

> 
> 控制过滤器执行顺序，数字越小，优先执行；鉴权一般设置负数优先执行。
3. Gateway能不能用@RestControllerAdvice捕获异常？

> 
> 不能，Gateway运行在WebFlux环境，不走SpringMVC，需要实现ErrorWebExceptionHandler。
4. Gateway过滤器中能不能写数据库查询？

> 
> 不建议；网关要轻量化；如果必须查询，要使用响应式驱动，禁止同步阻塞JDBC。
5. lb:// 是什么？

> 
> lb代表开启SpringCloud负载均衡，从注册中心获取服务实例做转发。

如果你需要，我可以整理一套Gateway面试精简背诵版，或者继续讲微服务统一返回、全局异常、Feign规范。

## Spring Cloud微服务

### Spring Cloud微服务之间的交互

> 
> 单体：Controller → Service → Mapper；所有代码在一个进程内，方法直接调用。
> 微服务：拆成多个独立进程（user服务、order服务、pay服务），**进程与进程之间网络通信**，不能直接new对象调用方法。

#### 核心组件总览

1. **Nacos/Eureka**：注册中心。各个微服务启动把自己的ip、端口、服务名注册上去；服务消费方从注册中心拿到服务实例地址。
2. **OpenFeign**：声明式HTTP客户端，**最常用，做服务之间调用**。
3. **Spring Cloud Gateway**：API网关，前端请求统一入口，转发到后端微服务。
4. **Ribbon(LoadBalancer)**：客户端负载均衡，从多个服务实例选一个节点发起调用。
5. **Sentinel / Resilience4j**：熔断、限流，防止服务雪崩。
6. **Nacos Config**：配置中心，统一管理配置。

#### 两种交互场景

##### 场景1：前端 → 网关 → 微服务（外部请求流入）

```
前端浏览器/APP
        ↓ http
Spring Cloud Gateway（网关）
        ↓ lb负载均衡
        ├→ user-service实例1
        └→ user-service实例2
```

流程：

1. 前端请求访问网关地址；
2. Gateway根据路由断言匹配路由；
3. 网关从Nacos获取`user‑service`的实例列表；
4. 负载均衡挑选一个实例，转发http请求；
5. 请求到达微服务的Controller，执行业务逻辑返回，原路返回响应。

> 
> 网关只处理外部流量；**微服务和微服务内部互相调用不走网关！**

##### 场景2：微服务 ↔ 微服务（内部服务调用，OpenFeign）

> 
> 例子：下单服务order‑service，需要查询用户信息，调用user‑service。

```
order‑service(消费者)
     ↓ OpenFeign http调用
user‑service(提供者)
```

完整调用流程：

1. order‑service里面定义Feign接口，标注`@FeignClient("user‑service")`；
2. OpenFeign底层通过LoadBalancer去Nacos注册中心拉取`user‑service`全部实例；
3. 负载均衡算法选出一个实例ip:port；
4. 组装http请求，发送http调用user‑service的Controller接口；
5. user‑service接收http请求，Controller → Service → Mapper处理，返回JSON；
6. Feign把返回JSON自动反序列化为Java对象，回到order‑service业务代码。

> 
> ⚠️重点：**微服务之间调用本质还是HTTP请求，跨网络，有网络超时、失败风险，和本地方法调用完全不一样。**

##### OpenFeign简单代码示例

```
// 调用方 order-service
@FeignClient("user-service") //服务名，和nacos注册的服务名一致
public interface UserFeignClient {
    @GetMapping("/api/v1/user/{id}")
    Result<UserVO> getUserById(@PathVariable("id") Long id);
}
```

order‑service的Service中直接注入使用：

```
@Service
public class OrderServiceImpl {
    private final UserFeignClient userFeignClient;
    public OrderServiceImpl(UserFeignClient userFeignClient){
        this.userFeignClient = userFeignClient;
    }

    public void createOrder(Long userId){
        //远程调用用户服务
        Result<UserVO> resp = userFeignClient.getUserById(userId);
        UserVO userVO = resp.getData();
    }
}
```

被调用方 user‑service 就是普通SpringMVC Controller，没有特殊代码。

#### 微服务交互常见问题（面试高频）

##### 1. 负载均衡在哪里？

LoadBalancer（原Ribbon）是**客户端负载均衡**。

> 
> 调用方（order‑service）本地获取实例列表，本地算法选节点，不是网关做负载均衡。

##### 2. 为什么不用直接http工具（RestTemplate）调用，要用Feign？

- Feign声明式接口，写接口即可，不用手动拼接url、解析json；
- 整合负载均衡；
- 整合熔断Sentinel；
- 可配置超时、日志、拦截器。

##### 3. 服务雪崩，熔断降级

A调用B，B挂了，大量请求卡住A线程，导致A也卡死 → **服务雪崩**。
Sentinel：

- **熔断**：B异常比例高，直接切断调用，不再发请求给B；
- **降级**：调用失败，返回兜底数据，而不是一直等待超时。

##### 4. 注册中心作用

各个微服务启动向Nacos注册自己；调用方不需要写死ip端口，通过**服务名**调用；服务实例上下线，Nacos实时感知推送。

###### 同步调用缺点（Feign是同步）

1. 网络开销，有超时风险；
2. 强依赖被调用服务，如果对方不可用，当前业务受影响；
3. 同步等待，阻塞线程。

> 
> 解决：重要场景引入消息队列RocketMQ/Kafka做**异步通信**。

#### 场景3：微服务异步交互：消息队列MQ

适合：解耦、削峰，不需要立刻拿到返回结果。

> 
> 示例：下单成功，发送消息，积分服务、短信服务消费消息。

```
order-service 发送消息到MQ
        ↓
MQ消息队列
    ├→ point‑service消费消息，增加积分
    └→ msg‑service消费消息，发送短信
```

特点：异步，不阻塞主流程；服务之间不直接依赖。

#### 完整整体链路（面试常画）

> 
> 用户前端 → Gateway网关 → order‑service
>  order‑service内部：
>   1. Feign同步调用 user‑service 获取用户信息
>   2. Feign同步调用 stock‑service扣库存
>   3. 发送MQ消息，异步通知积分、短信服务
>  order‑service自己操作数据库保存订单

#### 区分：3种通信方式对比

| 方式 | 使用场景 | 说明 |
| --- | --- | --- |
| Gateway转发http | 前端访问后端微服务 | 外部流量入口，前端只访问网关 |
| OpenFeign(http同步) | 微服务之间同步调用，需要拿到返回结果 | 进程间http调用，有网络超时风险 |
| MQ消息队列异步 | 不需要立即返回，解耦削峰 | 生产者发消息，消费者异步处理 |

#### 面试背诵精简版

> 
> SpringCloud微服务各个服务独立部署运行，进程之间通过网络交互。
> 
> 
> 1. 所有前端请求统一经过Gateway网关，网关从Nacos注册中心获取服务实例，负载均衡转发请求到对应微服务。
> 2. **微服务内部互相调用使用OpenFeign声明式客户端**，调用方通过服务名，结合LoadBalancer客户端负载均衡，从Nacos拿到提供者实例，发送HTTP请求完成远程调用；可以配合Sentinel做熔断降级，防止服务雪崩。
> 3. 对于解耦、异步场景，使用消息队列MQ实现异步交互。
> 注册中心Nacos负责服务注册与发现，保存各个微服务实例地址。

> 
> 补充考点：
> 
> 
> - Feign调用本质是HTTP；不是本地方法调用，会有网络超时；
> - 微服务之间调用不走网关；网关只处理前端外部请求。

如果你需要，我可以对比：**单体三层交互 vs 微服务交互**，帮你巩固记忆。



### 如何测试微服务之间交互是否正常

> 微服务之间主要交互方式：**OpenFeign同步远程调用、MQ异步消息、Gateway网关转发**。
> 难点：跨进程、网络、注册中心Nacos、负载均衡、超时熔断，不仅仅是本地代码。
> 测试分为：**单元测试、集成测试、契约测试、接口测试、全链路测试**。

#### 1、OpenFeign远程调用测试（微服务A调用微服务B）

##### ① 单元测试（本地，不启动Nacos、不启动被调用微服务）

目标：只测调用方A的业务代码，**Mock Feign客户端**，不发起真实网络请求。
> 不会真正访问B服务，验证调用方A的逻辑：是否调用Feign、处理返回值、处理异常。

```java
@SpringBootTest
class OrderServiceTest {
    // Mock Feign接口，模拟远程服务返回，不走网络
    @MockBean
    private UserFeignClient userFeignClient;

    @Autowired
    private OrderService orderService;

    @Test
    void testCreateOrder(){
        // 模拟远程服务正常返回
        UserVO mockVo = new UserVO();
        mockVo.setUserId(1L);
        when(userFeignClient.getUserById(1L))
                .thenReturn(Result.success(mockVo));

        orderService.createOrder(1L);

        // 校验Feign接口确实被调用一次
        verify(userFeignClient,times(1)).getUserById(1L);
    }

    @Test
    void testFeignCallFail(){
        //模拟远程调用异常（超时、服务不可用）
        when(userFeignClient.getUserById(99L))
                .thenThrow(new FeignException.ServiceUnavailable("服务不可用"));
        //断言触发熔断/抛出业务异常
        Assertions.assertThrows(BizException.class,()->{
            orderService.createOrder(99L);
        });
    }
}
```
✅适合：测试调用方的业务逻辑、异常处理、Sentinel熔断降级逻辑。
❌局限：**没有真实网络，测不到网络、Nacos、Feign配置、提供者B接口是否正常**。

---

##### ② 集成测试（启动完整环境，真实Feign调用）

需要环境：**Nacos注册中心启动，被调用微服务B也启动，测试数据库**。
调用方A完整Spring上下文，Feign真实从Nacos拉取服务实例，真实发HTTP请求访问B服务。
> 验证：Nacos服务发现、负载均衡、Feign超时配置、A‑B之间真实交互。

> 缺点：环境依赖重，启动慢；一般放在测试环境做，很少本地单元测试写。

##### ③ 契约测试（Spring‑Cloud‑Contract）

> 解决问题：服务提供者B接口改了，调用方A不知道，线上出bug。
- **提供者B编写契约（json/groovy）定义接口入参出参**；
- B服务测试时生成契约；上传到Nacos/Maven仓库；
- 调用方A下载契约，自动生成Mock，按照契约校验自己调用逻辑。
> 保证提供者和消费者接口契约一致。企业微服务常用。

#### 2、Gateway网关交互测试

验证：路由转发、断言、过滤器、鉴权、转发到后端微服务是否正常。

1. **单元测试**：`@WebFluxTest`，Mock下游微服务响应，测试Filter、路由逻辑，不启动真实微服务。
2. **集成测试**：启动Gateway、Nacos、后端微服务，使用`WebTestClient`发真实请求，走完整网关转发链路。
3. 手工测试：Apifox/postman直接请求网关地址，看是否正确转发、返回结果。

#### 3、MQ异步交互测试（RocketMQ/Kafka）

场景：A发送消息，B消费消息。
1. **单元测试**：Mock MQ生产者，不真实发消息，校验是否执行send方法；Mock消费者测试消费逻辑。
2. **集成测试**：启动测试MQ服务，A发送真实消息，断言B消费成功，数据库数据变更。
> 需要清理测试topic脏消息，避免旧消息干扰用例。

#### 4、黑盒接口测试（手工/自动化）

把整套微服务部署到**测试环境**：Nacos、Gateway、所有微服务、MQ、测试数据库全部就绪。
使用Apifox / Postman / JMeter：
前端请求 → Gateway → 微服务A → Feign调用微服务B。
观察最终返回结果，观察各个服务日志，看Feign调用日志、MQ消息流转。
> 这是最贴近线上真实场景。

#### 5、全链路压测

不仅测功能，同时测超时、高并发、熔断场景。
压测工具模拟请求，验证：
- 提供者B宕机，A是否触发Sentinel熔断降级；
- 网络延迟下Feign超时是否生效；
- 多实例负载均衡是否轮询。

#### 📊各种测试对比

|测试类型|是否启动Nacos|真实网络调用|用途|局限|
|---|---|---|---|---|
|Feign单元测试(MockBean)|❌|❌|测试调用方业务、异常、熔断|测不到提供者、网络、注册中心|
|契约测试|可选|❌|保证消费者和提供者接口契约一致|不测试运行时网络问题|
|微服务集成测试|✅|✅|完整测试Feign调用、服务发现、负载均衡|环境重，启动慢|
|Gateway集成测试|✅|✅|测试网关路由转发过滤器|依赖整套环境|
|MQ集成测试|✅|✅|测试消息生产消费异步交互|需要MQ测试实例|
|接口黑盒测试（Apifox）|✅|✅|模拟真实用户完整链路|属于手工/自动化UI接口，不能做断言代码回归|

#### 面试背诵版（重点）

> 微服务交互包含Feign远程调用、Gateway网关转发、MQ异步消息三种方式。
1. **单元测试**：使用`@MockBean`Mock Feign客户端，不发起真实网络请求，测试调用方业务逻辑、异常和熔断逻辑；无法验证提供者接口、注册中心、网络。
2. **集成测试**：搭建完整测试环境，启动Nacos、提供者微服务，Feign真实进行远程调用，验证服务发现、负载均衡、跨服务真实交互；MQ场景启动测试MQ，验证消息生产消费。
3. **契约测试**：使用Spring Cloud Contract，维护服务提供者与消费者接口契约，提前发现接口不兼容问题。
4. **黑盒接口测试**：整套微服务部署测试环境，通过Apifox/Postman从网关发起请求，跑完整链路，验证端到端交互。
5. 还可以做全链路压测，模拟超时、服务不可用，验证熔断降级策略。

> 高频面试坑：
> 单元测试Mock Feign，**不会真正调用被调用微服务**，只能测调用方代码；想要验证A和B真正网络交互，必须集成测试或者测试环境接口测试。

需要我帮你整理一份：**单体三层测试 vs 微服务交互测试对比总结**吗？方便你整体复习。