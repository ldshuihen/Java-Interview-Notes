在 **Java 实际开发 / 面试 / 线上运维** 中，常见问题一般可以按 **系统层、代码层、架构层、数据层、性能层、稳定性层、安全层** 等分类。下面给你整理一套 **真实生产环境常见问题分类 + 排查方向**（基本覆盖互联网 Java 后端 90% 场景）。

------

## 一、JVM 类问题

典型表现：**CPU高、内存溢出、GC频繁**

### 1 内存问题

- **OOM (OutOfMemoryError)**
  - Heap OOM
  - Metaspace OOM
  - DirectMemory OOM
  - GC overhead limit exceeded

排查工具

- `jmap -dump`
- `jhat`
- `MAT`
- `jvisualvm`
- `arthas`

常见原因

- 大对象
- 集合无限增长
- 缓存未清理
- ThreadLocal未释放
- 线程池任务堆积

------

### 2 GC问题

表现

- GC频繁
- Full GC时间长
- STW停顿

排查

```
jstat -gc
jstat -gcutil
```

查看

```
GC日志
```

常见原因

- 内存分配不合理
- 新生代过小
- 对象创建过多
- 大对象直接进入老年代

------

### 3 CPU 100%

排查流程

```
top
top -Hp pid
jstack pid
```

常见原因

- 死循环
- 锁竞争
- 大量JSON序列化
- 正则表达式
- 线程空转

------

### 4 线程问题

- 死锁
- 线程泄漏
- 线程池耗尽

排查

```
jstack
```

------

## 二、数据库问题

### 1 慢SQL

表现

- 接口响应慢
- DB CPU高

排查

```
慢查询日志
explain
```

常见原因

- 没有索引
- 索引失效
- 回表
- 大量扫描
- join过多

------

### 2 数据库连接池耗尽

表现

```
GetConnectionTimeoutException
```

原因

- SQL慢
- 连接未释放
- 连接池过小

常见连接池

- HikariCP
- Druid

------

### 3 数据一致性问题

场景

- 分布式事务
- 消息重复
- 幂等失败

解决

- 本地消息表
- TCC
- Saga
- 最终一致性

------

## 三、Redis问题

### 1 缓存穿透

原因

```
查询不存在数据
```

解决

- 布隆过滤器
- 缓存空值

------

### 2 缓存击穿

原因

```
热点key过期
```

解决

- 互斥锁
- 逻辑过期

------

### 3 缓存雪崩

原因

```
大量key同时过期
```

解决

- 过期时间随机
- 多级缓存
- 限流

------

### 4 大Key

问题

- 阻塞Redis
- 网络传输慢

解决

- 拆Key
- 分片

------

## 四、消息队列问题（MQ）

常见 MQ

- Kafka
- RabbitMQ
- RocketMQ

------

### 1 消息重复消费

原因

- MQ至少一次投递

解决

```
幂等设计
```

方法

- 唯一ID
- Redis去重
- 数据库唯一索引

------

### 2 消息积压

表现

- consumer消费慢

原因

- 下游系统慢
- 单线程消费

解决

- 扩容consumer
- 批量消费

------

### 3 消息丢失

可能发生

1 生产者
2 MQ
3 消费者

解决

- producer confirm
- 持久化
- ack机制

------

## 五、接口性能问题

### 1 接口RT高

原因

- 慢SQL
- 外部接口慢
- 锁竞争
- GC

排查工具

- APM
- trace

常见工具

- Apache SkyWalking
- Zipkin
- Jaeger



接口 **RT（Response Time）高** 是线上最常见的问题之一。你列出的原因和工具是对的，但在真实生产中通常会形成一套 **完整排查链路**。我给你整理成 **更贴近真实生产排查的体系**。

------

#### 一、接口 RT 高的常见原因分类

##### 1 数据库问题

最常见，占线上问题 **60%+**

典型原因

- 慢 SQL
- SQL 未命中索引
- 大表扫描
- 锁等待
- 数据库 CPU 高
- 连接池耗尽

排查方法

```sql
show processlist;
explain select ...
```

看

- type
- rows
- key
- extra

------

##### 2 外部依赖慢

典型依赖

- 第三方支付
- 其他微服务
- RPC调用
- HTTP调用

问题表现

```
A服务接口RT 3s
其中：
DB 20ms
远程调用 2800ms
```

------

##### 3 锁竞争

代码示例

```java
synchronized(lock){
    doSomething();
}
```

问题表现

- QPS低
- RT高
- CPU不高

排查

```bash
jstack pid
```

看

```
BLOCKED
WAITING
```

------

##### 4 GC停顿

GC Stop The World

表现

- RT突然升高
- TPS下降

排查

```bash
jstat -gcutil pid 1000
```

看

```
FGC
YGC
```

------

##### 5 线程池阻塞

常见代码

```java
ThreadPoolExecutor
```

问题

- 线程池满
- 队列堆积

日志

```
RejectedExecutionException
```

------

##### 6 网络问题

例如

- DNS慢
- TCP重试
- 网络丢包

排查

```bash
ping
netstat
```

------

#### 二、生产环境最重要排查工具

##### 1 APM链路追踪

APM 可以看到 **接口完整调用链**。

常见 APM 工具：

- Apache SkyWalking
- Zipkin
- Jaeger

功能

- traceId
- span
- 调用链
- 方法耗时
- SQL耗时

调用链示例

```
/order/create
 ├─ OrderService
 │   ├─ MySQL 20ms
 │   └─ Redis 5ms
 └─ PaymentService 2.5s
```

可以快速定位慢点。

------

##### 2 JVM排查工具

###### 查看线程

```bash
jstack pid
```

###### 查看GC

```bash
jstat -gcutil pid
```

###### Dump内存

```bash
jmap -dump
```

------

##### 3 实时诊断工具

非常强的线上工具：

- Arthas

常用命令

查看方法耗时

```bash
trace com.xxx.OrderService createOrder
```

查看慢方法

```bash
monitor
```

查看线程

```bash
thread
```

------

#### 三、真实生产排查步骤（标准流程）

遇到 **接口 RT 高**，一般这样排查：

##### 第一步：看监控

看

- RT
- QPS
- 错误率

监控系统

- Prometheus
- Grafana

------

##### 第二步：看调用链

使用

- Apache SkyWalking

看

```
哪个服务慢
哪个SQL慢
哪个方法慢
```

------

##### 第三步：看日志

查看

```
error log
slow log
```

------

##### 第四步：看JVM

```bash
top
```

看

- CPU
- 内存

然后

```bash
jstack
jstat
```

------

##### 第五步：看数据库

检查

- 慢SQL
- 锁等待
- 索引

------

#### 四、真实生产案例

案例：

用户反馈

```
订单接口RT 3s
```

通过 APM 看到

```
/createOrder
 ├─ Redis 5ms
 ├─ MySQL 20ms
 └─ inventory-service 2800ms
```

继续看 inventory-service

发现

```
调用第三方库存系统
```

第三方接口慢导致。

最终解决

```
增加缓存
+ 异步调用
```

RT 从

```
3s → 120ms
```

------

#### 五、RT问题的终极经验

生产中 **接口RT高的概率分布**

```
慢SQL           60%
外部依赖慢      20%
锁竞争          10%
GC              5%
网络问题        5%
```

所以排查优先级：

```
1 SQL
2 RPC
3 线程
4 GC
```

------

如果你愿意，我可以继续给你整理一个 **非常实用的内容**：

**《Java线上问题排查命令大全（30个命令）》**

比如：

```
top
htop
jstack
jmap
jstat
arthas
tcpdump
lsof
netstat
```

以及 **每个命令在什么情况下使用**，这在生产排查和面试里非常加分。

------

### 2 QPS突然下降

原因

- CPU打满
- 线程池耗尽
- DB慢



**QPS（Queries Per Second）突然下降**通常说明系统 **吞吐能力变差**。你列的几个原因是最常见的，但在真实生产中一般从 **系统资源 → JVM → 线程 → 依赖服务 → 数据库** 逐层排查。下面给你整理一套 **生产级排查思路**。

------

#### 一、QPS突然下降的常见原因

##### 1 CPU打满

表现

- CPU 100%
- 系统负载升高
- QPS下降
- RT升高

排查步骤

###### 查看CPU

```bash
top
```

查看 Java 进程。

###### 查看线程

```bash
top -Hp pid
```

找到 CPU 占用最高线程。

###### 查看线程栈

```bash
jstack pid
```

常见原因

- 死循环
- 正则回溯
- JSON序列化过多
- 加密算法
- 大对象处理

------

##### 2 线程池耗尽

表现

- 请求排队
- QPS下降
- RT升高

日志

```
RejectedExecutionException
```

典型代码

```java
ThreadPoolExecutor executor
```

原因

- 外部接口慢
- 任务执行时间长
- 线程池参数不合理

排查

查看

```
activeCount
queueSize
```

解决

- 扩大线程池
- 限流
- 拆分线程池

------

##### 3 数据库变慢

表现

- DB CPU高
- SQL执行慢
- 连接池等待

排查

查看慢查询

```sql
show processlist;
```

分析 SQL

```sql
explain select ...
```

常见原因

- 索引失效
- 大表扫描
- 锁等待
- join过多

------

#### 二、其他常见原因（真实生产很常见）

##### 4 GC频繁

表现

- QPS下降
- RT波动

排查

```bash
jstat -gcutil pid 1000
```

看

```
FGC
YGC
```

------

##### 5 外部依赖变慢

例如

- 微服务调用
- 第三方接口
- RPC

排查工具

- Apache SkyWalking
- Zipkin
- Jaeger

查看 **调用链耗时**。

------

##### 6 Redis问题

例如

- Redis阻塞
- Redis大Key
- 网络慢

排查

```bash
redis-cli info
```

查看

```
latency
blocked_clients
```

------

##### 7 网络问题

例如

- DNS慢
- TCP重传
- 网络拥堵

排查

```bash
netstat -an
```

------

#### 三、真实生产排查流程

QPS下降一般按这个顺序排查：

##### 第一步：看监控

查看

- QPS
- RT
- 错误率

监控系统常见组合：

- Prometheus
- Grafana

------

##### 第二步：看机器资源

```bash
top
```

看

- CPU
- load
- memory

------

##### 第三步：看线程

```bash
jstack pid
```

看

```
BLOCKED
WAITING
```

------

##### 第四步：看GC

```bash
jstat -gcutil pid
```

------

##### 第五步：看依赖服务

通过调用链工具：

- Apache SkyWalking

找到慢服务。

------

##### 第六步：看数据库

检查

- 慢SQL
- 锁等待
- 连接池

------

#### 四、真实生产案例

案例：某电商系统

现象

```
QPS: 2000 → 200
```

监控发现

```
CPU 95%
```

排查

```
top -Hp pid
jstack
```

发现代码

```java
while(true){
   retry();
}
```

无限重试导致 CPU 打满。

解决

```
增加重试间隔
限制重试次数
```

QPS恢复。

------

#### 五、经验总结

线上 **QPS下降的概率分布**

```
慢SQL              40%
CPU打满            20%
外部依赖慢         20%
线程池耗尽         10%
GC问题              5%
网络问题            5%
```

排查优先顺序

```
1 CPU
2 SQL
3 RPC
4 线程池
5 GC
```

------

如果你需要，我可以给你整理一份 **《Java线上问题排查思维图（完整体系）》**，里面会包含：

- 20类线上事故
- 50个排查命令
- JVM / DB / Redis / MQ / 网络 故障定位路径

### 接口 **RT（Response Time）** 

> **一次请求从客户端发起，到服务器处理完成并返回响应所花费的总时间。**

简单理解：

```
用户请求 → 服务处理 → 返回结果
        ↑-------------↑
              RT
```

------

#### 一、RT包含哪些时间

一个接口的 RT 一般由以下几部分组成：

```
RT =
网络传输时间
+ 服务处理时间
+ 数据库时间
+ 远程调用时间
+ 序列化/反序列化时间
```

更细一点：

```
RT =
客户端网络时间
+ 网关时间
+ 服务业务逻辑时间
+ 数据库查询时间
+ Redis时间
+ RPC调用时间
+ 返回数据时间
```

例如：

```
接口RT = 520ms

其中：
业务逻辑       20ms
MySQL查询     200ms
Redis查询      5ms
RPC调用      250ms
网络          45ms
```

------

#### 二、RT与QPS的关系

RT 和 **QPS（吞吐量）** 是系统性能的两个核心指标。

关系公式：

```
QPS ≈ 并发数 / RT
```

例如：

```
并发 = 100
RT   = 100ms

QPS = 100 / 0.1 = 1000
```

因此：

**RT越高 → 系统吞吐能力越低。**

------

#### 三、RT常见指标范围（互联网系统经验值）

| RT时间    | 评价             |
| --------- | ---------------- |
| < 50ms    | 非常快           |
| 50–200ms  | 正常             |
| 200–500ms | 偏慢             |
| 500ms–1s  | 慢               |
| >1s       | 用户明显感觉卡顿 |

例如：

电商系统目标

```
下单接口 RT < 200ms
```

------

#### 四、RT高的常见原因

最常见的几个原因：

##### 1 慢SQL

占线上问题 **60%**

例如

```
SELECT * FROM order WHERE user_id = ?
```

没有索引。

------

##### 2 外部接口慢

例如

```
订单服务 → 库存服务
订单服务 → 支付服务
```

远程服务慢导致 RT 高。

------

##### 3 锁竞争

例如

```java
synchronized(lock){
   doSomething();
}
```

线程等待锁。

------

##### 4 GC停顿

JVM **Stop The World**

例如

```
Full GC 2s
```

所有请求卡住。

------

##### 5 线程池阻塞

线程池满了

```
请求排队
```

RT变高。

------

#### 五、RT在监控系统中的体现

一般通过 **APM链路追踪** 查看。

常见工具：

- Apache SkyWalking
- Zipkin
- Jaeger

调用链示例：

```
/createOrder

├── OrderService           10ms
├── MySQL                  40ms
├── Redis                   3ms
└── InventoryService      300ms
```

可以直接看到 **哪个环节慢**。

------

#### 六、RT、TP99、TP95

生产环境不仅看平均 RT，还看 **分位数**。

例如：

```
RT avg  = 120ms
RT p95  = 300ms
RT p99  = 800ms
```

含义：

- **P95**：95%请求小于这个时间
- **P99**：99%请求小于这个时间

如果：

```
avg 很低
但 p99 很高
```

说明：

**系统存在长尾请求。**

------

#### 七、一句话总结

**接口 RT = 一个请求从发送到返回所消耗的总时间。**

```
RT = 网络 + 业务逻辑 + DB + Redis + RPC
```

它是 **衡量系统响应速度的核心指标**。

------

如果你愿意，我可以再给你整理一份 **面试/生产非常常见的一组性能指标解释**（很多人其实分不清）：

- **RT**
- **QPS**
- **TPS**
- **并发数**
- **吞吐量**
- **TP99 / TP95**
- **SLA / SLO / SLI**

我可以用 **一张完整性能指标关系图** 帮你彻底讲清楚，这在系统设计和面试里非常加分。

------

## 六、微服务问题

### 1 服务调用失败

原因

- 网络问题
- 服务实例挂掉

解决

- 熔断
- 降级
- 重试

常见框架

- Spring Cloud
- Alibaba Sentinel
- Netflix Hystrix

------

### 2 服务雪崩

表现

```
A调用B
B调用C
C挂掉
A全部阻塞
```

解决

- 限流
- 熔断
- 隔离

------

## 七、并发问题

### 1 锁竞争

表现

```
接口RT变慢
```

原因

- synchronized
- ReentrantLock

解决

- 减少锁粒度
- CAS

------

### 2 并发安全

问题

- 重复下单
- 超卖

解决

- 分布式锁
- CAS
- 数据库锁

------

## 八、系统稳定性问题

### 1 流量突增

解决

- 限流
- 预热
- 扩容

常见工具

- Nginx
- Envoy

------

### 2 服务不可用

原因

- 内存泄漏
- CPU打满
- 依赖服务挂掉

解决

- 健康检查
- 自动重启

------

## 九、安全类问题

### 1 DDoS攻击

解决

- CDN
- WAF

------

### 2 暴力登录

解决

- 登录限流
- 验证码

------

### 3 Token滥用

解决

- Token过期
- IP绑定

------

## 十、日志与排查工具

线上问题排查常用工具

日志系统

- ELK Stack
- Grafana
- Prometheus

Java诊断

- Arthas
- JVisualVM

------

## 十一、真实线上排查标准流程（重要）

生产问题一般按这个顺序排查：

### 第一步 看监控

- CPU
- 内存
- QPS
- RT
- GC

------

### 第二步 看日志

```
error.log
```

------

### 第三步 看线程

```
jstack
```

------

### 第四步 看GC

```
jstat
```

------

### 第五步 看SQL

```
slow log
```

------

## 十二、最经典的 20 个 Java 线上问题

1 CPU 100%
2 Full GC频繁
3 OOM
4 线程死锁
5 慢SQL
6 Redis雪崩
7 MQ积压
8 MQ重复消费
9 接口RT高
10 服务雪崩
11 分布式事务问题
12 数据不一致
13 并发超卖
14 线程池耗尽
15 DB连接池耗尽
16 大Key问题
17 JVM内存泄漏
18 网络调用超时
19 缓存击穿
20 DDoS攻击

------

如果你愿意，我可以再给你整理一份 **Java面试最喜欢问的：**

**《100个真实生产事故案例 + 排查思路》**

比如：

- **CPU 100% 真实排查案例**
- **Full GC 每秒一次事故**
- **Redis 雪崩导致系统崩溃**
- **Kafka 积压 2000 万消息**
- **订单重复支付事故**

这类基本是 **阿里 / 字节 / 美团面试高频题**。

## 真实生产事故案例 + 排查思路

下面给你整理 **10 个典型 Java 真实生产事故案例 + 完整排查思路**。这些基本都是互联网公司真实遇到的场景，也是很多面试非常喜欢问的。

------

### 一、CPU 100% 事故

#### 事故现象

线上某服务 **CPU 100%**，接口响应极慢。

#### 排查流程

##### 1 查看CPU

```bash
top
```

找到 CPU 占用最高的进程。

------

##### 2 找到线程

```bash
top -Hp pid
```

找到 **CPU最高线程ID**

------

##### 3 转换线程ID

```bash
printf "%x\n" 线程ID
```

转为 **16进制**

------

##### 4 查看线程堆栈

```bash
jstack pid | grep 16进制线程号 -A 30
```

------

#### 真实原因案例

1️⃣ **死循环**

```java
while(true){
}
```

2️⃣ **错误重试**

```java
while(true){
   retry();
}
```

3️⃣ **正则灾难回溯**

复杂正则导致 CPU 爆炸

------

### 二、Full GC 每秒一次事故

#### 事故现象

- 服务 **RT突然变高**
- CPU不高
- GC日志大量Full GC

------

#### 排查

查看 GC

```bash
jstat -gcutil pid 1000
```

观察

```
FGC 1000+
```

------

#### Dump 内存

```bash
jmap -dump:live,format=b,file=heap.hprof pid
```

使用 **MAT** 分析。

------

#### 真实原因

缓存设计错误

```java
Map<String,Object> cache = new HashMap<>();
```

缓存一直增长。

------

### 三、线程池耗尽事故

#### 事故现象

系统请求 **全部阻塞**

日志

```
RejectedExecutionException
```

------

#### 排查

查看线程池

```java
ThreadPoolExecutor
```

查看

```
activeCount
queueSize
```

------

#### 真实原因

线程池执行 **同步HTTP调用**

```java
httpClient.call();
```

外部接口慢 → 线程全部阻塞。

------

#### 解决

- 设置超时
- 限流
- 隔离线程池

------

### 四、数据库连接池耗尽

#### 事故现象

日志

```
GetConnectionTimeoutException
```

------

#### 排查

查看连接池

```
active
idle
max
```

------

#### 真实原因

代码忘记关闭连接

```java
Connection conn = dataSource.getConnection();
```

没有

```java
close()
```

------

#### 解决

使用

```java
try-with-resource
```

------

### 五、Redis雪崩事故

#### 事故现象

系统突然

- QPS暴涨
- DB被打爆

------

#### 原因

缓存同时过期

```
TTL = 1h
```

------

#### 解决

过期时间随机

```java
ttl = 3600 + random(300)
```

------

### 六、Redis大Key事故

#### 事故现象

Redis **延迟突然升高**

------

#### 排查

```bash
redis-cli --bigkeys
```

------

#### 真实原因

用户列表缓存

```
user:list:all
```

大小

```
200MB
```

------

#### 解决

拆Key

```
user:list:1
user:list:2
```

------

### 七、Kafka消息积压事故

#### 事故现象

消息积压

```
lag = 10,000,000
```

------

#### 排查

查看消费速率

```
consumer TPS
```

------

#### 真实原因

消费逻辑

```java
for(message){
   callHttp();
}
```

外部接口慢。

------

#### 解决

- 批量消费
- 多线程消费
- 降级

常见消息系统如
Apache Kafka
RabbitMQ

------

### 八、订单重复提交事故

#### 事故现象

用户出现 **重复订单**

------

#### 原因

接口没有幂等

```java
createOrder();
```

用户多次点击。

------

#### 解决

幂等设计

方案

1️⃣ Redis token
2️⃣ 数据库唯一索引
3️⃣ 分布式锁

------

### 九、服务雪崩事故

#### 事故现象

一个服务挂掉

所有服务崩溃

------

调用链

```
A → B → C → DB
```

C慢

B线程阻塞

A全部阻塞

------

#### 解决

- 限流
- 熔断
- 隔离

常见组件

Alibaba Sentinel
Netflix Hystrix

------

### 十、日志爆盘事故

#### 事故现象

服务器 **磁盘满**

系统崩溃。

------

#### 原因

日志级别

```
DEBUG
```

日志无限增长。

------

#### 解决

日志滚动

```xml
logback rollingPolicy
```

日志平台

ELK Stack

------

### 十一、真实生产排查万能流程（非常重要）

线上问题 **99%按这个流程排查**

```
1 看监控
2 看日志
3 看CPU
4 看线程
5 看GC
6 看数据库
7 看Redis
8 看MQ
```

监控工具

Prometheus
Grafana
Apache SkyWalking

------

### 十二、面试最喜欢问的 8 个生产事故

1️⃣ CPU 100% 怎么排查
2️⃣ Full GC频繁怎么排查
3️⃣ Redis雪崩怎么解决
4️⃣ MQ积压怎么解决
5️⃣ DB连接池耗尽
6️⃣ 线程池耗尽
7️⃣ 订单重复提交
8️⃣ 服务雪崩

------

如果你愿意，我可以再给你整理 **一份更硬核的内容**：

### 《20个Java最经典生产事故（大厂级）》

例如：

- **一次Full GC导致全站卡顿**
- **Redis击穿导致DB崩溃**
- **线程池参数配置错误引发雪崩**
- **MySQL索引失效导致接口慢10倍**
- **Kafka消息堆积2亿条**

并且我可以给你整理 **完整排查命令图谱（超实用）**：

```
top
htop
jstack
jmap
jstat
arthas
tcpdump
netstat
```

很多都是 **阿里 / 字节 / 美团一线面试题级别**。