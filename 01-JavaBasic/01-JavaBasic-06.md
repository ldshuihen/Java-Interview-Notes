## 本地缓存

本地缓存（Local Cache）是把热点数据存在**当前应用进程内存**里的缓存，核心优势是**纳秒/微秒级访问、无网络IO、低延迟**，常用作**一级缓存（L1）** 配合 Redis 组成多级缓存。

---

### 一、核心特点（一眼看懂）

- ✅ 速度极快：内存直接读写，**μs/ns 级**；Redis 约 **ms 级**，DB 更慢。
- ✅ 无网络开销：进程内访问，不依赖外部服务。
- ✅ 容量有限：受 **JVM 堆/堆外内存**限制，不适合大数据量。
- ✅ 单机隔离：多实例间**不共享**，存在数据不一致风险。
- ✅ 重启丢失：默认**不持久化**（Ehcache 可配置磁盘）。

### 二、适用场景
- 读多写少、更新低频：系统配置、字典表、城市列表、基础资料。
- 极致性能链路：高并发热点（如商品详情、用户信息）。
- 多级缓存 L1：Caffeine → Redis → DB，**减轻 Redis 压力、降低延迟**。
- 防击穿/防雪崩：热点 Key 用本地缓存挡流量。

### 三、Java 主流实现对比
#### 1）ConcurrentHashMap（最简自研）
- 优点：JDK 原生、零依赖、线程安全。
- 缺点：**无过期、无淘汰、无统计**，需自行封装。
- 适用：简单小缓存、临时数据。

#### 2）Guava Cache（经典稳定）
- 特性：LRU 淘汰、写入/访问过期、最大容量、移除监听、命中率统计。
- 缺点：性能弱于 Caffeine，维护放缓。

#### 3）Caffeine（当前最优，推荐）

- 核心：**W-TinyLFU 淘汰算法**，命中率比 LRU 高 **10%–20%**。
- 特性：
  - 自动加载（LoadingCache）、异步刷新
  - 写入/访问过期、最大容量、权重控制
  - 堆内/堆外内存、统计监控
- 现状：Spring Boot 默认集成，高并发首选。

#### 4）Ehcache（全功能老牌）
- 特性：内存+磁盘持久化、堆外内存、集群、事务、过期/淘汰。
- 缺点：较重、配置复杂、性能低于 Caffeine。

| 方案              | 性能  | 过期 | 淘汰      | 持久化 | 推荐指数 |
| ----------------- | ----- | ---- | --------- | ------ | -------- |
| ConcurrentHashMap | ★★★★★ | ❌    | ❌         | ❌      | ★★☆      |
| Guava Cache       | ★★★★  | ✅    | LRU       | ❌      | ★★★☆     |
| Caffeine          | ★★★★★ | ✅    | W-TinyLFU | ❌      | ★★★★★    |
| Ehcache           | ★★★   | ✅    | 多种      | ✅      | ★★★      |

### 四、Caffeine 快速示例（Java）
#### 1）依赖（Maven）
```xml
<dependency>
    <groupId>com.github.benmanes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>
```

#### 2）基础用法（手动 put/get）
```java
Cache<Long, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000) // 最大1万条
    .expireAfterWrite(5, TimeUnit.MINUTES) // 写入5分钟过期
    .expireAfterAccess(2, TimeUnit.MINUTES) // 访问后2分钟过期
    .recordStats() // 开启统计（命中率等）
    .build();

// 存
cache.put(1L, "张三");
// 取（不存在返回 null）
String name = cache.getIfPresent(1L);
```

#### 3）自动加载（LoadingCache，常用）
```java
LoadingCache<Long, String> loadingCache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(5, TimeUnit.MINUTES)
    .refreshAfterWrite(1, TimeUnit.MINUTES) // 写入1分钟后异步刷新
    .build(id -> loadFromDb(id)); // 缓存未命中时执行

// 调用（自动加载）
String name = loadingCache.get(1L);

// 模拟DB查询
private static String loadFromDb(Long id) {
    return "用户" + id;
}
```

### 五、多级缓存架构（推荐）
**请求 → Caffeine（L1）→ Redis（L2）→ DB（L3）**
- L1：本地缓存，**μs 级**，命中率约 **80%**。
- L2：Redis，**ms 级**，分布式共享，命中率约 **15%**。
- L3：DB，持久化，命中率约 **5%**。

### 六、常见问题与解决
1. **数据不一致（多实例）**
   - 方案：本地缓存**短 TTL（1–5min）**；更新时**主动删除本地缓存+Redis**；或用消息队列广播失效。

2. **内存溢出（OOM）**
   - 方案：**严格设置 maximumSize**；用**软/弱引用**；监控内存与缓存大小。

3. **缓存穿透**
   - 方案：本地缓存**空值/默认值**；**布隆过滤器**前置拦截。

### 七、总结
- 本地缓存是**高并发高性能的利器**，优先选 **Caffeine**。
- 生产环境必做：**多级缓存（Caffeine+Redis）、合理 TTL、内存保护、一致性策略、监控告警**。

需要我给你一份可直接复制的 **Caffeine+Redis 多级缓存工具类（含过期删除、异步刷新、空值防穿透）** 吗？

### **Caffeine** 

**Caffeine** 是目前 **Java 生态性能最强、命中率最高**的本地缓存库，是 Guava Cache 的全面升级版，也是 Spring Boot 2.x+ 默认本地缓存实现。

---

#### 一、核心定位与优势

- **定位**：高性能、低延迟、高命中率的**进程内内存缓存**（JVM 本地缓存）
- **核心优势**
  1. **性能极致**：读性能接近 ConcurrentHashMap，写性能远超 Guava Cache（8~10 倍）
  2. **命中率最高**：自研 **W-TinyLFU** 淘汰算法，比 LRU/LFU 命中率高 10%~15%
  3. **功能全面**：过期、刷新、异步、引用回收、统计、监听器、权重控制
  4. **轻量无依赖**：核心包小、无第三方依赖，兼容 Java 8+

---

#### 二、核心原理：W-TinyLFU 算法（关键）

Caffeine 性能与命中率的核心是 **Window-TinyLFU** 双层淘汰机制：

1. **Window Cache（窗口区，约 1% 容量）**
   - 纯 LRU 策略
   - 接纳**新数据**，解决 LFU 冷启动问题（新热点不会刚进就被淘汰）

2. **Main Cache（主缓存区，99% 容量）**
   - TinyLFU 策略
   - 用 **Count-Min Sketch** 极低内存开销统计访问频率
   - 保留**长期高频**数据，淘汰低频数据

**解决两大经典问题**：
- **LRU 缺点**：避免冷数据扫描污染缓存、挤掉热点
- **LFU 缺点**：解决新热点冷启动、历史高频数据僵化

---

#### 三、Maven 依赖

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>
```

---

#### 四、三种核心使用方式

##### 1. 手动缓存（Manual Cache）

显式 put/get，适合主动控制缓存：
```java
Cache<String, String> cache = Caffeine.newBuilder()
    .initialCapacity(100)       // 初始容量
    .maximumSize(10_000)       // 最大条目数
    .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后过期
    .expireAfterAccess(5, TimeUnit.MINUTES) // 最后访问后过期
    .recordStats()             // 开启统计
    .build();

// 存
cache.put("key", "value");

// 取（不存在返回 null）
String val = cache.getIfPresent("key");

// 取（不存在则自动加载）
String val2 = cache.get("key2", k -> loadFromDB(k));

// 删
cache.invalidate("key");
```

##### 2. 自动加载缓存（LoadingCache）

缓存未命中时**自动加载**，适合 DB/接口查询缓存：
```java
LoadingCache<String, User> userCache = Caffeine.newBuilder()
    .maximumSize(1000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> userDao.findById(key)); // 加载函数

// 自动加载：不存在 → 调用 load → 缓存
User user = userCache.get("123");
```

##### 3. 异步加载缓存（AsyncLoadingCache）

I/O 密集场景，**异步加载不阻塞主线程**：
```java
AsyncLoadingCache<String, User> asyncCache = Caffeine.newBuilder()
    .buildAsync(key -> CompletableFuture.supplyAsync(() -> 
        userDao.findById(key)
    ));

// 返回 CompletableFuture
CompletableFuture<User> future = asyncCache.get("123");
User user = future.get();
```

---

#### 五、核心配置详解

##### 1. 容量控制

- `maximumSize(long)`：最大条目数
- `maximumWeight(long)` + `weigher((k,v)->weight)`：按**权重**（如内存大小）限制

##### 2. 过期策略

- `expireAfterWrite(duration)`：写入后固定过期
- `expireAfterAccess(duration)`：最后访问后过期
- `expireAfter(new Expiry<>())`：**自定义过期**（每个 key 独立过期时间）

##### 3. 自动刷新

```java
.refreshAfterWrite(5, TimeUnit.MINUTES)
```
- 后台异步刷新，**不阻塞读请求**
- 旧值可用时继续返回旧值

##### 4. 引用回收（防 OOM）

- `weakKeys()`：键用弱引用（GC 时自动回收）
- `weakValues()`：值弱引用
- `softValues()`：值软引用（内存不足时 GC 回收）

##### 5. 统计与监控

```java
.recordStats() // 开启
CacheStats stats = cache.stats();
stats.hitRate();   // 命中率
stats.missRate();  // 未命中率
stats.evictionCount(); // 淘汰次数
```

---

#### 六、与 Guava Cache / ConcurrentHashMap 对比

| 特性        | Caffeine      | Guava Cache | ConcurrentHashMap |
| ----------- | ------------- | ----------- | ----------------- |
| 淘汰算法    | **W-TinyLFU** | LRU         | 无淘汰            |
| 命中率      | **最高**      | 中          | 无                |
| 过期策略    | 全支持        | 有限        | 无                |
| 异步加载    | ✅             | ❌           | ❌                 |
| 性能        | **最高**      | 中          | 高（无淘汰）      |
| Spring 默认 | ✅ 2.x+        | ❌           | ❌                 |

---

#### 七、Spring Boot 集成（最常用）

1. 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```
2. 开启缓存
```java
@EnableCaching
@SpringBootApplication
public class App {}
```
3. 注解使用
```java
@Cacheable(value = "userCache", key = "#id")
public User getUserById(Long id) {
    return userDao.findById(id);
}

@CacheEvict(value = "userCache", key = "#user.id")
public void updateUser(User user) {}
```

---

#### 八、最佳实践

- **读多写少、热点数据**：用 Caffeine
- **容量控制**：必设 `maximumSize`/`maximumWeight`
- **过期**：写多读少用 `expireAfterWrite`；读多写少用 `expireAfterAccess`
- **防雪崩**：配合 `refreshAfterWrite` + 随机过期
- **监控**：生产必开 `recordStats()` 监控命中率

---

#### 九、一句话总结

**Caffeine = 高性能 + 最高命中率 + 全功能**，是 Java 本地缓存**首选方案**。

要不要我给你一份可直接复制的 **Caffeine 生产级配置模板**（含过期、刷新、权重、统计、防 OOM）？

## 文件路径

### 1.三种路径

非常好的问题！理解 **当前工作目录（Current Working Directory）**、**项目根目录（Project Root）** 和 **类路径（Classpath）** 的区别，是 Java 开发中避免“文件找不到”错误的关键。下面为你详细解析三者的定义、区别、获取方式及使用场景。

------

#### 一、三者定义与本质区别

| 概念                    | 英文                      | 本质                                                         | 作用                                         |
| ----------------------- | ------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| **当前工作目录**        | Current Working Directory | **操作系统概念**：JVM 启动时所在的文件系统目录               | 相对路径（如 `new File("a.txt")`）的基准目录 |
| **类路径（Classpath）** | Classpath                 | **JVM 概念**：JVM 查找 `.class` 文件和资源（resources）的路径集合 | 加载类、读取 `src/main/resources` 下的资源   |
| **项目根目录**          | Project Root Directory    | **开发约定**：包含 `src/`、`pom.xml` 等的源码顶层目录        | 仅在开发阶段有意义，运行时无直接对应         |

> ✅ **关键区别**：
>
> - **工作目录 = 文件系统的起点**
>
> - **类路径 = class 和资源的查找路径**
>
> - **项目根目录 ≠ 工作目录 ≠ 类路径**（尽管开发时可能重合）
>
> - **“项目根目录”是你写代码时的家，“当前工作目录”是程序运行时站的地方——在家时两者一样，出门后就不一定了。**
>
>   如果你只是本地开发测试，放心用；如果要上线，请做好路径解耦 😊



#### **“当前工作目录”和“项目根目录”的区别**

**在概念上是两回事，但在某些开发场景下它们恰好相同。**

------

##### ✅ 简短回答：

> **不是一回事。**  
>
> - **当前工作目录（Current Working Directory）**：是 **操作系统层面** 的概念，指 JVM 启动时所在的目录。  
> - **项目根目录（Project Root Directory）**：是 **开发者/构建工具约定** 的概念，指包含 `src/`、`pom.xml` 等的源码顶层目录。

它们**可能重合，但不一定总是相等**。

------

##### 🔍 详细解释

###### 1. **当前工作目录（`user.dir`）**

- 由 **启动 Java 进程的位置** 决定。

- 可通过 `System.getProperty("user.dir")` 获取。

- 示例：

  ```bash
  # 在 /home/user 下运行
  cd /home/user
  java -jar /opt/myapp/app.jar
  ```

  → 此时 

  ```
  user.dir = /home/user
  ```

  ，

  即使 JAR 文件在 `/opt/myapp/`

###### 2. **项目根目录**

- 是你写代码时的工程结构顶层，比如：

  ```
  ~/projects/my-spring-app/   ← 这就是项目根目录
    ├── src/
    ├── pom.xml
    └── 中二知识笔记.pdf
  ```

- 它只存在于 **开发阶段** 或 **源码仓库中**。

- 一旦项目打包成 JAR/WAR，**“项目根目录”这个概念在运行时就消失了**（只剩文件系统路径或 classpath）。

------

##### 🆚 对比示例

| 场景                                           | 当前工作目录 (`user.dir`)       | 项目根目录                 | 是否相同？ |
| ---------------------------------------------- | ------------------------------- | -------------------------- | ---------- |
| 在 IDEA 中运行 main 方法                       | `~/projects/my-spring-app`      | `~/projects/my-spring-app` | ✅ 是       |
| 在项目根目录执行 `mvn spring-boot:run`         | `~/projects/my-spring-app`      | `~/projects/my-spring-app` | ✅ 是       |
| 打包后，在任意位置运行： `java -jar myapp.jar` | `/home/ops`（你执行命令的地方） | （已不存在）               | ❌ 否       |
| 用 systemd 服务启动 Java 应用                  | `/` 或 `/root`（默认）          | —                          | ❌ 否       |
| Docker 容器中运行（未指定 WORKDIR）            | `/`                             | —                          | ❌ 否       |

------

##### 💡 关键结论

- **开发阶段（IDE / 命令行在项目目录下运行）**：
  两者通常一致，所以 `new FileSystemResource("xxx.pdf")` 能工作。
- **部署/生产环境**：
  “项目根目录”不再存在，只有“当前工作目录”，而它可能是任意位置。此时硬编码相对路径会失败。

------

##### ✅ 最佳实践建议

1. **不要假设 `user.dir` 就是项目根目录**，尤其在生产代码中。

2. 如果需要访问外部文件：

   - 通过配置指定绝对路径（如 `app.document.path=/data/docs/xxx.pdf`）

   - 或明确基于 

     ```
     user.dir
     ```

      构建路径并加校验：

     ```java
     Path file = Paths.get(System.getProperty("user.dir")).resolve("中二知识笔记.pdf");
     if (!Files.exists(file)) {
         // 报错或使用默认路径
     }
     ```

3. **资源文件（如配置、小图片）应放 `src/main/resources`，用 `ClassPathResource` 加载**，与工作目录无关。

------

##### 🎯 一句话总结：

> **“项目根目录”是你写代码时的家，“当前工作目录”是程序运行时站的地方——在家时两者一样，出门后就不一定了。**

如果你只是本地开发测试，放心用；如果要上线，请做好路径解耦 😊

#### 二、如何获取它们？

##### 1. 获取 **当前工作目录**

```java
String workingDir = System.getProperty("user.dir");
System.out.println("当前工作目录: " + workingDir);
```

- 返回 JVM 启动时的目录。
- 示例输出（Linux/macOS）：`/home/user/my-project`
  （Windows）：`C:\Users\user\my-project`

> 📌 所有 **相对路径文件操作**（如 `new File("log.txt")`）都基于此目录。

------

##### 2. 获取 **类路径（Classpath）**

类路径是一个**路径列表**，可通过以下方式查看：

###### 方法 A：打印整个 classpath

```java
String classpath = System.getProperty("java.class.path");
System.out.println("类路径: " + classpath);
```

- 输出示例（Maven 项目）：

  ```
  /home/user/my-project/target/classes:
  /home/user/.m2/repository/org/springframework/spring-core/5.3.0/spring-core-5.3.0.jar:...
  ```

###### 方法 B：获取某个类或资源的实际位置

```java
// 获取当前类的 .class 文件位置
URL classUrl = MyClass.class.getProtectionDomain().getCodeSource().getLocation();
System.out.println("类所在位置: " + classUrl);

// 获取 classpath 下某资源的位置
URL resourceUrl = MyClass.class.getClassLoader().getResource("application.properties");
System.out.println("资源位置: " + resourceUrl);
```

> ⚠️ 注意：如果代码被打包进 JAR，`classUrl` 可能是 `file:/app.jar`，而资源 URL 可能是 `jar:file:/app.jar!/application.properties`。

------

##### 3. “获取项目根目录”？—— 它没有直接 API！

因为 **项目根目录是开发概念**，JVM 运行时并不知道它。

但你可以**间接推测**（仅限开发阶段）：

```java
// 假设你在 target/classes 下运行（Maven 默认）
URL classesUrl = Thread.currentThread().getContextClassLoader().getResource(".");
if (classesUrl != null) {
    try {
        Path classesDir = Paths.get(classesUrl.toURI()); // .../target/classes
        Path projectRoot = classesDir.getParent().getParent(); // .../my-project
        System.out.println("推测的项目根目录: " + projectRoot);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

> ❗ 此方法在 JAR 中失效（因为 `getResource(".")` 在 JAR 内返回 `jar:...!/.`，无法转为文件路径）。

✅ **更可靠的做法**：通过配置或 `user.dir`（见下文）。

------

#### 三、典型使用场景对比

| 需求                                           | 应该用                     | 示例                                              |
| ---------------------------------------------- | -------------------------- | ------------------------------------------------- |
| 读取 `src/main/resources/config.json`          | **类路径**                 | `new ClassPathResource("config.json")`            |
| 读取项目根目录下的 `data/input.csv`            | **当前工作目录**（开发时） | `new FileSystemResource("data/input.csv")`        |
| 写日志到 `logs/app.log`（与启动脚本同目录）    | **当前工作目录**           | `new FileOutputStream("logs/app.log")`            |
| 加载外部配置文件（如 `/etc/myapp/config.yml`） | **绝对路径**               | `new FileSystemResource("/etc/myapp/config.yml")` |

------

#### 四、常见误区与陷阱

##### ❌ 误区 1：认为 `getClass().getResource("/")` 是项目根目录

```java
// 错误！这返回的是 classpath 根（如 target/classes），不是项目根目录
URL url = getClass().getResource("/");
```

- 在 Maven 项目中，它指向 `target/classes/`，比项目根目录深两层。

##### ❌ 误区 2：用 `ClassPathResource` 读项目根目录的文件

```java
// 如果 "readme.txt" 在项目根目录（不在 resources/），这会失败！
new ClassPathResource("readme.txt"); // 找不到
```

→ 只有放在 `src/main/resources/` 的文件才会进入 classpath。

##### ❌ 误区 3：假设打包后还能访问项目结构

- JAR/WAR 包只包含编译后的 class 和 resources，**不包含 `pom.xml`、`src/`、项目根目录下的普通文件**。

------

#### 五、最佳实践建议

| 场景                         | 推荐做法                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| **开发阶段读项目根目录文件** | 确保文件存在 + 使用 `user.dir` + 加存在校验                  |
| **生产环境读外部文件**       | 通过配置指定**绝对路径**（如 `-Dconfig.path=/opt/app/data.cfg`） |
| **读应用配置/模板等资源**    | 放入 `src/main/resources`，用 `ClassPathResource`            |
| **不确定路径？**             | 打印 `user.dir` 和 `java.class.path` 调试                    |

------

#### 六、总结表

| 特性               | 当前工作目录                     | 类路径（Classpath）                     | 项目根目录                 |
| ------------------ | -------------------------------- | --------------------------------------- | -------------------------- |
| **本质**           | OS 概念                          | JVM 概念                                | 开发约定                   |
| **获取方式**       | `System.getProperty("user.dir")` | `System.getProperty("java.class.path")` | 无直接方式（需推测或配置） |
| **相对路径基准**   | 是                               | 否（classpath 资源用自己规则）          | 否                         |
| **打包后是否有效** | 取决于启动位置                   | 有效（JAR 内部）                        | 无效                       |
| **典型用途**       | 读写外部文件、日志               | 加载类、读 resources 资源               | 仅用于开发组织代码         |

------

如果你告诉我你的具体需求（比如：“我想在 Spring Boot 里读一个放在项目根目录的 PDF 文件”），我可以给出最合适的代码方案！

### 2.绝对路径和相对路径

在 Java（以及所有操作系统和编程语言）中，**绝对路径**和**相对路径**是文件系统操作的基础概念。理解它们对正确读写文件、避免 `FileNotFoundException` 至关重要。

------

#### 一、定义

##### ✅ 绝对路径（Absolute Path）

- **从文件系统的根目录开始**，完整描述文件或目录的位置。

- **不依赖当前工作目录**，在任何地方运行都指向同一个位置。

- 示例

  ：

  - Windows:  

    ```text
    C:\Users\Alice\Documents\notes.txt
    D:\myapp\config\app.conf
    ```

  - Linux / macOS:  

    ```text
    /home/alice/documents/notes.txt
    /opt/myapp/logs/app.log
    ```

> 🔑 特点：以盘符（Windows）或 `/`（Unix-like）开头。

------

##### ✅ 相对路径（Relative Path）

- **相对于“当前工作目录”（Current Working Directory）** 的路径。

- **会随着程序启动位置不同而指向不同文件**。

- 示例

  （假设当前工作目录是 

  ```
  /home/alice/myproject
  ```

  ）：

  ```text
  notes.txt           → /home/alice/myproject/notes.txt
  data/input.csv      → /home/alice/myproject/data/input.csv
  ../shared/utils.jar → /home/alice/shared/utils.jar
  ./config/app.conf   → /home/alice/myproject/config/app.conf
  ```

> 🔑 特点：不以 `/` 或盘符开头；常用 `.`（当前目录）、`..`（上级目录）。

------

#### 二、在 Java 中如何使用？

##### 1. 使用 `java.io.File`

```java
// 相对路径：相对于 user.dir
File relative = new File("data.txt");

// 绝对路径
File absolute = new File("/home/alice/data.txt");        // Linux/macOS
File absoluteWin = new File("C:\\Users\\Alice\\data.txt"); // Windows
```

##### 2. 使用 `java.nio.file.Path`（推荐）

```java
Path relative = Paths.get("data.txt");
Path absolute = Paths.get("/home/alice/data.txt");

// 将相对路径转为绝对路径（基于当前工作目录）
Path resolved = Paths.get(System.getProperty("user.dir")).resolve("data.txt");
```

##### 3. 在 Spring 中使用 `Resource`

```java
// 相对路径（基于工作目录）
Resource rel = new FileSystemResource("config/app.yml");

// 绝对路径
Resource abs = new FileSystemResource("/etc/myapp/app.yml");
```

------

#### 三、关键区别总结

| 特性                 | 绝对路径                          | 相对路径                               |
| -------------------- | --------------------------------- | -------------------------------------- |
| **起点**             | 文件系统根目录（如 `/` 或 `C:\`） | 当前工作目录（`user.dir`）             |
| **可移植性**         | ❌ 差（硬编码路径）                | ✅ 好（但依赖运行位置）                 |
| **是否受 `cd` 影响** | 否                                | 是                                     |
| **Java 中判断方法**  | `path.isAbsolute()`               | `!path.isAbsolute()`                   |
| **适用场景**         | 配置文件、日志目录（固定位置）    | 项目内资源、临时文件（与启动位置相关） |

------

#### 四、如何查看当前工作目录？

```java
String workingDir = System.getProperty("user.dir");
System.out.println("当前工作目录: " + workingDir);
```

- 这就是**所有相对路径的基准**！

------

#### 五、常见陷阱与最佳实践

##### ⚠️ 陷阱 1：在 IDE 能跑，打包后找不到文件

- **原因**：IDE 中工作目录 = 项目根目录，但生产环境不是。
- **解决**：用绝对路径，或通过配置指定路径。

##### ⚠️ 陷阱 2：混用 `/` 和 `\`

- 错误（Windows）

  ：

  ```java
  new File("C:\my folder\file.txt"); // \m 被当作转义字符！
  ```

- 正确做法

  ：

  ```java
  new File("C:\\my folder\\file.txt");     // 转义
  new File("C:/my folder/file.txt");       // Java 允许用 / 即使在 Windows
  Paths.get("C:", "my folder", "file.txt"); // 最安全
  ```

##### ✅ 最佳实践

1. **开发阶段**：可用相对路径（确保文件在工作目录下）。

2. 生产部署

   ：

   - 外部文件用**绝对路径**；
   - 或通过 `-Dconfig.path=...` 等 JVM 参数传入路径。

3. **资源文件**（如模板、配置）：放 `src/main/resources`，用 **classpath** 加载（与路径无关）。

4. 永远校验文件是否存在

   ：

   ```java
   if (!Files.exists(path)) {
       throw new FileNotFoundException("文件不存在: " + path.toAbsolutePath());
   }
   ```

------

#### 六、示例：相对路径 vs 绝对路径行为对比

假设项目结构：

```
/home/user/myapp/
├── app.jar
├── config/
│   └── settings.properties
└── data.txt
```

##### 场景 1：在 `/home/user/myapp` 下运行

```bash
cd /home/user/myapp
java -jar app.jar
```

- `new File("data.txt")` → ✅ 找到
- `new File("config/settings.properties")` → ✅ 找到

##### 场景 2：在 `/tmp` 下运行

```bash
cd /tmp
java -jar /home/user/myapp/app.jar
```

- `new File("data.txt")` → ❌ 找不到（会去 `/tmp/data.txt`）
- `new File("/home/user/myapp/data.txt")` → ✅ 找到

------

#### 七、一句话总结

> **绝对路径 = “身份证号”，全球唯一；
> 相对路径 = “我家楼下”，取决于你站在哪。**

在 Java 中：

- 用 **绝对路径** 确保稳定（适合生产）；
- 用 **相对路径** 方便开发（需控制运行目录）；
- 用 **classpath** 管理应用内资源（最可靠）。

如有具体场景（比如 Spring Boot 读外部文件、Web 上传保存等），欢迎告诉我，我可以给出针对性代码！

