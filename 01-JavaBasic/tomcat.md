### Tomcat

#### 一、Tomcat 是什么

**Apache Tomcat**（简称 Tomcat）是 Apache 基金会旗下**开源、轻量**的 **Jakarta Servlet / JSP 容器**（旧称 Java EE），用 Java 编写，可独立作为 HTTP Web 服务器运行，也常与 Nginx/Apache 配合部署。


- **核心身份**：
  - **Web 服务器**：监听 8080 端口，处理 HTTP/HTTPS 请求，可直接返回静态资源（HTML/CSS/JS/图片）。
  - **Servlet 容器**：实现 Servlet、JSP、EL、WebSocket 规范，**管理 Servlet 生命周期**（加载→初始化→服务→销毁）。
  - **JSP 引擎**：自动把 JSP 编译成 Servlet 并执行。
  - **轻量应用服务器**：非完整 Java EE 服务器（无 EJB、JTA 等），**占用资源少、启动快、易部署**。

---

#### 二、核心特性

- ✅ **开源免费**：Apache 2.0 许可证，商业项目可用。
- ✅ **跨平台**：纯 Java，支持 Windows/Linux/macOS。
- ✅ **规范兼容**：
  - Tomcat 9：Servlet 4.0、JSP 2.3、EL 3.0、WebSocket 1.1（Java 8+）。
  - Tomcat 10：Jakarta EE 9（包名从 `javax.servlet` 改为 `jakarta.servlet`）。
  - Tomcat 11：Jakarta EE 10，最低 Java 11。
- ✅ **高并发**：内置线程池、支持 NIO/NIO2/APR 网络模型。
- ✅ **集群与负载均衡**：支持会话复制、粘性会话，可与 Nginx 做负载均衡。
- ✅ **安全**：内置 Realm 认证、SSL/TLS、访问控制、防 CSRF。
- ✅ **可扩展**：组件化架构，可自定义 Connector、Valve、Listener。

---

#### 三、版本演进（关键节点）

| 版本 | 发布 | 最低 Java | Servlet | JSP  | 重要特性                         |
| ---- | ---- | --------- | ------- | ---- | -------------------------------- |
| 7.x  | 2011 | 6         | 3.0     | 2.2  | 异步 Servlet、WebSocket 初始支持 |
| 8.x  | 2014 | 7         | 3.1     | 2.3  | NIO2、HTTP/1.1 优化              |
| 9.x  | 2017 | 8         | 4.0     | 2.3  | HTTP/2、TLS 1.3、Jakarta EE 过渡 |
| 10.x | 2021 | 8         | 5.0     | 3.0  | 包名迁移到 `jakarta.*`、EE9      |
| 11.x | 2024 | 11        | 6.0     | 4.0  | EE10、虚拟线程（预览）、性能提升 |

> 生产环境建议用 **9.x（稳定兼容）** 或 **10.x（新项目）**，避免已终止维护的旧版（如 7.x 及更早）。

---

#### 四、目录结构（安装后）

```
tomcat/
├── bin/         # 启动/停止脚本（startup.sh、shutdown.sh、catalina.sh）
├── conf/        # 核心配置（server.xml、web.xml、context.xml、tomcat-users.xml）
├── lib/         # 依赖 jar（servlet-api、jsp-api、el-api 等）
├── logs/        # 日志（catalina.out、localhost.log、access.log）
├── temp/        # 临时文件
├── webapps/     # 应用部署目录（ROOT、examples、docs、manager、host-manager）
└── work/        # JSP 编译后的 Servlet 缓存
```

---

#### 五、核心架构（组件层级）

Tomcat 采用**分层容器架构**，核心组件：**Server → Service → Connector → Engine → Host → Context → Wrapper**。
##### 1. Server（顶层容器）

- 代表**整个 Tomcat 实例**，一个 JVM 中只有一个 Server。
- 管理生命周期（启动/关闭），包含多个 Service（默认一个）。
- 配置：`conf/server.xml` 中 `<Server port="8005" shutdown="SHUTDOWN">`（8005 是关闭端口，不对外）。

##### 2. Service（服务）

- 绑定**一个 Engine + 多个 Connector**，负责请求传递与组件生命周期管理。
- 默认 Service 名：**Catalina**。
- 配置：`<Service name="Catalina">`。

##### 3. Connector（连接器，入口）

- **监听端口、处理网络连接、解析协议**（HTTP/1.1、AJP、HTTP/2）。
- 核心内部组件：
  - **Endpoint**：底层 IO（NIO/NIO2/APR），监听 Socket 端口（默认 8080）。
  - **Processor**：解析 HTTP 字节流 → Tomcat Request/Response 对象。
  - **Adapter（CoyoteAdapter）**：把 Request/Response 转为 Servlet 规范对象，调用 Engine。
- 配置（HTTP 8080）：
  ```xml
  <Connector port="8080" protocol="HTTP/1.1"
             connectionTimeout="20000" redirectPort="8443" />
  ```

##### 4. Engine（引擎，顶层容器）

- **请求处理总入口**，接收所有 Connector 请求，分发给对应 Host。
- 一个 Service 只有一个 Engine，默认引擎名：**Catalina**。
- 配置：`<Engine name="Catalina" defaultHost="localhost">`。

##### 5. Host（虚拟主机）

- 对应**一个域名**（如 localhost、www.example.com），可配置多个 Host 实现多域名部署。
- 管理多个 Context（Web 应用）。
- 配置：`<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">`。

##### 6. Context（Web 应用）

- 代表**一个独立 Web 应用**（如 ROOT、/demo），每个应用隔离运行。
- 管理 Servlet、Filter、Listener、Session、JSP 等。
- 配置：`<Context path="/demo" docBase="demo" reloadable="true" />`。

##### 7. Wrapper（包装器）

- **最小容器单元**，代表一个 Servlet 实例。
- 管理 Servlet 生命周期：加载→初始化（init()）→服务（service()）→销毁（destroy()）。

---

#### 六、请求处理全流程（一次 HTTP 请求）

1. **用户**发请求：`http://localhost:8080/demo/index.jsp` → 到达服务器 8080 端口。
2. **Connector（Endpoint）** 接收 TCP 连接，线程池取线程处理。
3. **Processor** 解析 HTTP 协议，生成 Tomcat Request/Response。
4. **Adapter** 转为 Servlet 规范对象，调用 Engine。
5. **Engine** 根据域名（localhost）匹配 Host。
6. **Host** 根据路径（/demo）匹配 Context。
7. **Context** 找到对应 Servlet（JSP 编译后的 Servlet），交由 Wrapper 调用 `service()`。
8. **Servlet** 处理业务（查数据库、生成动态 HTML），写入 Response。
9. 响应沿原路返回 → Connector → 用户浏览器。

---

#### 七、核心配置文件详解

##### 1. server.xml（主配置，核心）

- 位置：`conf/server.xml`
- 作用：配置 Server、Service、Connector、Engine、Host 等核心组件。
- 关键配置项：
  - 关闭端口：`<Server port="8005" shutdown="SHUTDOWN">`
  - HTTP 端口：`<Connector port="8080" ...>`
  - 虚拟主机：`<Host name="localhost" appBase="webapps">`

##### 2. web.xml（全局默认配置）

- 位置：`conf/web.xml`
- 作用：定义所有 Web 应用默认配置（Servlet、Filter、MIME 类型、错误页面）。
- 常用配置：默认 Servlet（处理静态资源）、JSP Servlet、字符编码（UTF-8）。

##### 3. context.xml（数据源/资源配置）

- 位置：`conf/context.xml` 或 应用 `META-INF/context.xml`
- 作用：配置 JDBC 数据源、连接池、JNDI、资源引用。

##### 4. tomcat-users.xml（用户/角色/权限）

- 位置：`conf/tomcat-users.xml`
- 作用：配置 Tomcat 内置用户（如 manager/host-manager 控制台账号）。
- 示例：
  ```xml
  <user username="admin" password="admin" roles="manager-gui,admin-gui"/>
  ```

---

#### 八、部署 Web 应用的 3 种方式

1. **直接放 webapps**：将 WAR 包或解压文件夹放入 `webapps`，Tomcat 启动时**自动部署**（`autoDeploy="true"`）。
2. **配置 Context 标签**：在 `server.xml` 的 Host 内添加 `<Context path="/demo" docBase="/path/to/demo" />`。
3. **配置 Host 子目录**：在 `conf/Catalina/localhost/` 下创建 `demo.xml`，内容同上 Context 配置。

---

#### 九、IO 模型（性能关键）

- **BIO（阻塞 IO，Tomcat 7 前默认）**：一个请求一个线程，高并发下线程阻塞、资源耗尽、性能差。
- **NIO（非阻塞 IO，Tomcat 7+ 默认）**：基于 Java NIO，**单线程处理多个连接**，非阻塞读写，并发能力强。
- **NIO2（异步 IO，Tomcat 8+ 支持）**：Java 7 AIO，操作系统异步通知，**高并发、低延迟**。
- **APR（Apache Portable Runtime，原生 IO）**：调用本地库（C 实现），**高性能、高吞吐**，适合生产环境（需安装 APR 库）。

> 配置 NIO：`<Connector ... protocol="org.apache.coyote.http11.Http11NioProtocol" />`。

---

#### 十、常见使用场景

- ✅ **Java Web 应用部署**：Spring Boot、SSM、Struts、JSP/Servlet 项目。
- ✅ **前后端分离**：后端 API（Spring Boot）部署在 Tomcat，Nginx 反向代理静态资源。
- ✅ **企业内部系统**：OA、ERP、CRM、报表系统。
- ✅ **小型网站/商城**：个人博客、小型电商、毕业设计。
- ✅ **集群高可用**：多台 Tomcat 做集群，Nginx 负载均衡，会话复制保证一致性。

---

#### 十一、优缺点

##### 优点

- ✅ **轻量高效**：启动快、内存占用小、部署简单。
- ✅ **稳定成熟**：20+ 年历史，社区活跃，Bug 少，企业广泛使用。
- ✅ **规范兼容**：严格实现 Servlet/JSP 规范，迁移方便。
- ✅ **免费开源**：无授权费用，商业项目可用。

##### 缺点

- ❌ **不支持 EJB/JTA**：非完整 Java EE 服务器，复杂企业级应用需用 WebLogic、JBoss。
- ❌ **默认配置性能一般**：需调优（线程池、IO 模型、JVM）才能支撑高并发。
- ❌ **静态资源性能不如 Nginx**：高并发静态请求建议 Nginx 反向代理。

---

#### 十二、与 Nginx 的区别与配合

- **Nginx**：高性能 **HTTP 服务器/反向代理**，擅长**静态资源、高并发、负载均衡、SSL 终端**，不支持 Servlet/JSP。
- **Tomcat**：**Servlet 容器**，擅长**动态 Java 应用**，静态性能弱。
- **最佳实践**：**Nginx 前端 + Tomcat 后端**
  - Nginx：监听 80/443，处理静态资源、SSL、负载均衡、反向代理。
  - Tomcat：监听 8080，只处理动态请求（/api、/jsp）。

---

#### 十三、总结

Tomcat 是 **Java Web 生态的基石**，轻量、稳定、开源，完美适配绝大多数 Java Web 应用。理解其**分层架构、请求流程、IO 模型、配置调优**，是 Java 后端开发与运维的必备技能。

要不要我把以上内容浓缩成一份可直接复制的 **Tomcat 核心配置模板（含 Nginx 反向代理示例）**？#

### SpringBoot 集成 Tomcat 参数配置

SpringBoot **默认内置 Tomcat**，无需单独安装，所有 Tomcat 参数都通过 **application.yml / application.properties** 统一配置，比原生 Tomcat 更简单、更规范。

---

#### 一、最核心：application.yml 完整配置（推荐）

```yaml
server:
  # 1. 基础端口与路径
  port: 8080                 # 服务端口
  servlet:
    context-path: /demo      # 项目访问根路径（默认空）
    encoding:
      charset: UTF-8
      enabled: true
      force: true

  # 2. Tomcat 核心配置（最重要！）
  tomcat:
    # ==================== 线程池（并发核心）====================
    threads:
      max: 200                # 最大工作线程（生产 200~500）
      min-spare: 20           # 最小空闲线程（常驻保活）
      max-queue-capacity: 100 # 等待队列大小（线程满后排队）

    # ==================== 连接参数 ====================
    max-connections: 8192     # 最大并发连接数（NIO 模式）
    accept-count: 100         # 连接等待队列（TCP层）
    connection-timeout: 20000 # 连接超时 ms
    keep-alive-timeout: 60000  # 长连接超时
    max-keep-alive-requests: 100 # 长连接最大请求数

    # ==================== 请求大小限制（上传文件必配）====================
    max-http-post-size: 52428800  # POST 请求最大大小 50MB
    max-swallow-size: 2MB         # 最大缓冲大小

    # ==================== 性能优化 ====================
    compression:
      enabled: true            # 开启 GZIP 压缩
      mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
      min-response-size: 1024 # 超过 1KB 才压缩

    # ==================== 连接协议（IO模型）====================
    protocol: org.apache.coyote.http11.Http11NioProtocol  # NIO 模式（最优）

    # ==================== URI编码（解决中文乱码）====================
    uri-encoding: UTF-8

    # ==================== 其他优化 ====================
    accesslog:
      enabled: true           # 开启访问日志
      directory: logs
      prefix: access_log
      suffix: .log
      rotate: true
      pattern: common         # 日志格式

    # 关闭DNS解析，提升性能
    enable-lookups: false
```

---

#### 二、等价 application.properties 版本（如果你用 properties）

```properties
# 基础
server.port=8080
server.servlet.context-path=/demo
server.servlet.encoding.charset=UTF-8
server.servlet.encoding.enabled=true
server.servlet.encoding.force=true

# 线程池
server.tomcat.threads.max=200
server.tomcat.threads.min-spare=20
server.tomcat.threads.max-queue-capacity=100

# 连接数
server.tomcat.max-connections=8192
server.tomcat.accept-count=100
server.tomcat.connection-timeout=20000
server.tomcat.keep-alive-timeout=60000

# 请求大小
server.tomcat.max-http-post-size=52428800
server.tomcat.max-swallow-size=2097152

# 压缩
server.tomcat.compression.enabled=true
server.tomcat.compression.mime-types=text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
server.tomcat.compression.min-response-size=1024

# IO模型
server.tomcat.protocol=org.apache.coyote.http11.Http11NioProtocol

# 编码
server.tomcat.uri-encoding=UTF-8

# 访问日志
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.directory=logs
server.tomcat.accesslog.prefix=access_log
```

---

#### 三、每个参数 **详细解释**（面试 + 生产必看）

##### 1. 线程池（决定并发能力，最重要）

```yaml
server.tomcat.threads.max: 200          # 最大能同时处理多少请求
server.tomcat.threads.min-spare: 20     # 常驻线程，不用反复创建销毁
server.tomcat.threads.max-queue-capacity: 100 # 请求等待队列
```
###### server.tomcat.threads.max

最小值：**10**（强制下限，低于 10 自动变 10）

推荐最大值：**500~800**（超过 1000 必翻车）

默认值：**200**

生产最佳值：**200~500**

**并发规则：**

- 请求进来 → 先用空闲线程
- 线程不够 → 新建线程，直到 `max`
- 线程满了 → 进入等待队列 `max-queue-capacity`
- 队列也满 → **直接拒绝请求（503错误）**

---

##### 2. 连接数（TCP 连接）

```yaml
server.tomcat.max-connections: 8192    # 最大 TCP 连接数
server.tomcat.accept-count: 100        # 内核层连接等待队列
```
`max-connections` 是**同时能建立多少连接**，不是同时处理多少请求。

---

##### 3. 请求大小限制（上传文件必配）

```yaml
server.tomcat.max-http-post-size: 52428800 # 50MB
```
如果不配，**上传大文件会报错 400 / 413**。

---

##### 4. GZIP 压缩（大幅提升前端速度）

```yaml
server.tomcat.compression.enabled: true
```
压缩 html/js/css/json，**节省 50%~70% 流量**。

---

##### 5. IO 模型（高性能关键）

```yaml
server.tomcat.protocol: org.apache.coyote.http11.Http11NioProtocol
```
- NIO 模式：**高并发、低资源**（SpringBoot 默认）
- 不要用 BIO（老款阻塞，性能差）

---

##### 6. 中文乱码终极解决

```yaml
server.tomcat.uri-encoding: UTF-8
server.servlet.encoding.charset: UTF-8
server.servlet.encoding.force: true
```

---

#### 四、高级配置：代码方式自定义 Tomcat（更强大）

如果你需要**更深度定制**（如自定义连接器、优化HTTP协议头），可以用 `WebServerFactoryCustomizer`：

```java
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TomcatConfig {

    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> tomcatCustomizer() {
        return factory -> {
            factory.setProtocol("org.apache.coyote.http11.Http11NioProtocol");
            factory.addConnectorCustomizers(connector -> {
                // 这里可以写任意原生 Tomcat Connector 配置
                connector.setEnableLookups(false);
                connector.setURIEncoding("UTF-8");
            });
        };
    }
}
```

---

#### 五、生产环境最佳实践（直接用这套）

```yaml
server:
  port: 8080
  servlet:
    context-path: /api
    encoding:
      charset: UTF-8
      enabled: true
      force: true
  tomcat:
    threads:
      max: 300
      min-spare: 50
      max-queue-capacity: 200
    max-connections: 10000
    accept-count: 200
    connection-timeout: 30000
    max-http-post-size: 104857600 # 100MB
    uri-encoding: UTF-8
    enable-lookups: false
    compression:
      enabled: true
      min-response-size: 1024
    accesslog:
      enabled: true
```

---

#### 六、最常见面试题（SpringBoot + Tomcat）

1. **SpringBoot 怎么修改 Tomcat 线程数？**
   `server.tomcat.threads.max`
2. **SpringBoot 怎么限制上传文件大小？**
   `server.tomcat.max-http-post-size`
3. **SpringBoot 怎么开启 GZIP？**
   `server.tomcat.compression.enabled=true`
4. **maxThreads、max-queue-capacity、maxConnections 区别？**
   - maxThreads：处理请求的线程
   - queue：等待队列
   - maxConnections：TCP连接数+
