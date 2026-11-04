# Apache Tomcat 核心架构、类加载机制与性能调优指南

> **核心速记**：
> - **两大核心模块**：**Coyote（连接器 `Connector`）** 负责监听端口、网络 IO 与 HTTP 协议解析；**Catalina（容器 `Container`）** 采用四层父子容器（`Engine → Host → Context → Wrapper`）+ **`Pipeline-Valve` 责任链模式**管理 Servlet 生命周期并执行业务。
> - **NIO 线程模型**：默认采用 **NIO（同步非阻塞网络 IO + 同步阻塞业务处理）**——`Acceptor` 接收 TCP 连接 → `Poller`（基于 Selector）轮询可读事件 → 交给 **`Worker` 线程池**执行 Filter 链与 `Servlet.service()`。
> - **打破双亲委派**：每个 Web 应用（`Context`）独享一个 `WebappClassLoader`，重写 `loadClass()` 在加载**非 JDK 核心类**时**优先加载本应用 `WEB-INF/classes` 和 `WEB-INF/lib`**，本地找不到才向上委托，从而实现多 War 包**依赖版本隔离与单应用热部署**。
> - **Spring Boot 内嵌原理**：在 Spring 容器 `refresh()` 的 `onRefresh()` 阶段，通过 `TomcatServletWebServerFactory` 编程式 `new Tomcat()` 启动，并将 `DispatcherServlet` 注册为 `Wrapper`。

---

## 目录

- [一、Tomcat 核心定位、版本演进与目录配置](#一tomcat-核心定位版本演进与目录配置)
  - [1.1 Tomcat 是什么与核心身份](#11-tomcat-是什么与核心身份)
  - [1.2 版本演进关键节点（Jakarta EE 变迁）](#12-版本演进关键节点jakarta-ee-变迁)
  - [1.3 目录结构与四大核心配置文件](#13-目录结构与四大核心配置文件)
  - [1.4 Web 应用部署方式与 Nginx 动静分离配合](#14-web-应用部署方式与-nginx-动静分离配合)
- [二、Tomcat 核心架构：Coyote 连接器与 Catalina 容器](#二tomcat-核心架构coyote-连接器与-catalina-容器)
  - [2.1 整体组件层级全景（Server → Wrapper）](#21-整体组件层级全景server--wrapper)
  - [2.2 Connector（Coyote 连接器）内部三大组件](#22-connectorcoyote-连接器内部三大组件)
  - [2.3 Container（Catalina 四层父子容器）与 Pipeline-Valve 责任链](#23-containercatalina-四层父子容器与-pipeline-valve-责任链)
  - [2.4 Lifecycle 生命周期机制（组合模式 + 观察者模式）](#24-lifecycle-生命周期机制组合模式--观察者模式)
  - [2.5 一次 HTTP 请求在 Tomcat 中的完整 9 步流转流程](#25-一次-http-请求在-tomcat-中的完整-9-步流转流程)
- [三、IO 模型与 NIO 线程模型深度解析](#三io-模型与-nio-线程模型深度解析)
  - [3.1 四种 IO 模型演进与对比](#31-四种-io-模型演进与对比)
  - [3.2 Tomcat NIO 三类核心线程（Acceptor / Poller / Worker）](#32-tomcat-nio-三类核心线程acceptor--poller--worker)
  - [3.3 核心并发参数辨析（maxConnections vs acceptCount vs maxThreads）](#33-核心并发参数辨析maxconnections-vs-acceptcount-vs-maxthreads)
- [四、Tomcat 类加载机制与打破双亲委派（面试最高频）](#四tomcat-类加载机制与打破双亲委派面试最高频)
  - [4.1 为什么要打破双亲委派？（类隔离三大核心价值）](#41-为什么要打破双亲委派类隔离三大核心价值)
  - [4.2 Tomcat 类加载器层级树](#42-tomcat-类加载器层级树)
  - [4.3 WebappClassLoader 完整加载顺序（先子后父 + 安全底线）](#43-webappclassloader-完整加载顺序先子后父--安全底线)
  - [4.4 破坏双亲委派的风险与延伸场景](#44-破坏双亲委派的风险与延伸场景)
- [五、Spring Boot 内嵌 Tomcat 原理与生产调优配置](#五spring-boot-内嵌-tomcat-原理与生产调优配置)
  - [5.1 自动装配与 onRefresh() 启动全流程](#51-自动装配与-onrefresh-启动全流程)
  - [5.2 Spring Boot 内嵌 Tomcat vs 独立外部 Tomcat 深度对比](#52-spring-boot-内嵌-tomcat-vs-独立外部-tomcat-深度对比)
  - [5.3 application.yml 生产级最佳参数配置模板与详解](#53-applicationyml-生产级最佳参数配置模板与详解)
  - [5.4 编程式深度定制：WebServerFactoryCustomizer](#54-编程式深度定制webserverfactorycustomizer)
- [六、Tomcat 高频面试真题与线上故障排查](#六tomcat-高频面试真题与线上故障排查)
  - [6.1 1 分钟面试背诵版（架构篇 + 内嵌篇）](#61-1-分钟面试背诵版架构篇--内嵌篇)
  - [6.2 线上 Tomcat 接口响应变慢 / 线程池打满 5 步排查法](#62-线上-tomcat-接口响应变慢--线程池打满-5-步排查法)
  - [6.3 高频易错追问 Q&A](#63-高频易错追问-qa)

---

## 一、Tomcat 核心定位、版本演进与目录配置

### 1.1 Tomcat 是什么与核心身份

**Apache Tomcat** 是 Apache 基金会旗下开源、轻量级的 **Jakarta Servlet / JSP 容器**（旧称 Java EE），纯 Java 编写：
1. **HTTP Web 服务器**：监听端口（默认 `8080`），直接接收并解析 HTTP/HTTPS 请求；
2. **Servlet 容器**：严格实现 Servlet、Filter、Listener、Session、WebSocket 规范，**管理 Servlet 全生命周期（加载 → `init()` → `service()` → `destroy()`）**；
3. **轻量级应用服务器**：不包含重量级 EJB/JTA 组件，占用资源少、启动快，是 Spring Boot 默认内嵌的 Web 容器。

---

### 1.2 版本演进关键节点（Jakarta EE 变迁）

| 版本 | 最低 Java | Servlet 规范 | 核心里程碑特性 |
| :--- | :---: | :---: | :--- |
| **Tomcat 7.x** | Java 6 | 3.0 | 引入异步 Servlet、WebSocket 初始支持 |
| **Tomcat 8.x / 8.5** | Java 7 | 3.1 | 默认 IO 模型切为 **NIO**，支持 NIO2（AIO）与 HTTP/2 |
| **Tomcat 9.x** | Java 8 | 4.0 | 支持 TLS 1.3、HTTP/2 服务端推送（**Java 8 + Spring Boot 2.x 生产主力版本**） |
| **Tomcat 10.x** | Java 8 / 11 | 5.0 / 6.0 | **重大包名迁移**：所有 API 包名从 `javax.servlet.*` 改为 **`jakarta.servlet.*`**（**Spring Boot 3.x 默认内置 Tomcat 10.1.x**） |
| **Tomcat 11.x** | Java 17 / 21 | 6.1 | Jakarta EE 11，深度适配 JDK 21 **虚拟线程（Virtual Threads）** |

---

### 1.3 目录结构与四大核心配置文件

```text
tomcat/
├── bin/         # 启动与停止脚本（startup.sh、shutdown.sh、catalina.sh）
├── conf/        # 核心配置文件目录（server.xml、web.xml、context.xml、tomcat-users.xml）
├── lib/         # 公共依赖 jar 包（servlet-api.jar 等，由 CommonClassLoader 加载，所有应用共享）
├── logs/        # 日志目录（catalina.out 控制台日志、localhost.log、access_log 访问日志）
├── temp/        # 运行时临时文件目录
├── webapps/     # Web 应用默认自动部署目录（放入 war 包或文件夹即可部署）
└── work/        # JSP 编译生成的 .java 与 .class 缓存目录
```

- **四大核心配置文件（`conf/` 目录）**：
  1. **`server.xml`（最核心）**：配置 Tomcat 顶层架构组件（`Server` 关闭端口 `8005`、`Service`、`Connector` 监听端口 `8080`、`Engine`、`Host` 虚拟主机、`Context` 应用映射）；
  2. **`web.xml`**：全局 Web 应用默认配置（定义处理静态资源的 `DefaultServlet`、处理 JSP 的 `JspServlet`、会话超时时间与 MIME 类型映射）；
  3. **`context.xml`**：全局应用上下文资源配置（如 JNDI 数据源、Session 持久化管理器）；
  4. **`tomcat-users.xml`**：配置 Tomcat 内置管理后台（`manager-gui`）的管理员账号、密码与角色权限。

---

### 1.4 Web 应用部署方式与 Nginx 动静分离配合

- **部署 Web 应用的 3 种方式**（独立 Tomcat）：
  1. **直接放入 `webapps/` 目录**：Tomcat 启动或运行时自动解压 War 包部署（`autoDeploy="true"`）；
  2. **在 `conf/server.xml` 的 `<Host>` 标签内添加 `<Context path="/demo" docBase="/opt/app/demo" />`**（修改需重启 Tomcat，不推荐）；
  3. **在 `conf/Catalina/localhost/` 下新建 `demo.xml`**：写入 `<Context docBase="/opt/app/demo" />`，支持热加载无需重启。
- **Tomcat 与 Nginx 的区别及配合（动静分离最佳实践）**：
  - **Nginx**：C 语言编写的高性能 **反向代理与静态 Web 服务器**，基于异步非阻塞事件驱动（epoll），极度擅长**百万高并发连接、静态资源响应、SSL 卸载与负载均衡转发**，但不能执行 Java 代码；
  - **Tomcat**：Java 编写的 **Servlet 容器**，擅长**运行动态 Java 业务逻辑（Spring MVC）**，直接处理静态文件与超高并发连接的性能远不如 Nginx；
  - **生产标准组合**：`浏览器 ──(80/443 HTTPS)──► Nginx（静态资源直返 + 负载均衡） ──(8080 HTTP)──► 多台后端 Tomcat 集群（只处理 /api 动态请求）`。

---

## 二、Tomcat 核心架构：Coyote 连接器与 Catalina 容器

![Tomcat架构与请求处理流程图](Tomcat.assets/Tomcat架构与请求处理流程图.png)

### 2.1 整体组件层级全景（Server → Wrapper）

Tomcat 顶层设计围绕两大核心职责展开：**“怎么收发解析网络请求（`Connector`）”** 与 **“怎么找到对应 Servlet 执行业务（`Container`）”**，在 `server.xml` 中呈现严格的父子嵌套结构：

```text
Server (代表整个 Tomcat JVM 实例，监听 8005 关闭端口)
 └── Service (默认名 Catalina：将多个 Connector 与唯一一个 Engine 绑定成一组服务)
      ├── Connector 1 (HTTP/1.1 监听 8080)  ┐
      ├── Connector 2 (HTTP/2 或 AJP)       ├─► 【Coyote 网络连接器：负责网络 IO 与协议解析】
      └── Engine (顶层容器，默认名 Catalina) ┐
           └── Host (虚拟主机，如 localhost)  │
                └── Context (单个 Web 应用)   ├─► 【Catalina 四层父子容器：负责路由与执行 Servlet】
                     └── Wrapper (单个 Servlet)┘
```

---

### 2.2 Connector（Coyote 连接器）内部三大组件

`Connector` 对外屏蔽了底层复杂的 IO 模型与协议差异，其内部由三个核心子组件顺序协作：

| 内部组件 | 核心职责 | 产出结果 |
| :--- | :--- | :--- |
| **1. `Endpoint`（通信端点）** | 最底层网络 IO 实现（BIO / NIO / NIO2 / APR），管理 TCP 端口监听、Socket 读写与 Worker 线程池 | 将就绪的 Socket 交给 `Processor` |
| **2. `Processor`（协议处理器）** | 读取 Socket 字节流，按 **HTTP/1.1、HTTP/2 或 AJP 协议**解析请求行、请求头与请求体 | 封装成 Tomcat 内部的 `org.apache.coyote.Request / Response` |
| **3. `Adapter`（`CoyoteAdapter`）** | **适配器模式（经典设计模式考点）**：连接 `Connector` 与 `Container` 的桥梁 | 将 `coyote.Request` 转换为标准 **`HttpServletRequest / HttpServletResponse`**，调用 `Engine` 容器入口 |

---

### 2.3 Container（Catalina 四层父子容器）与 Pipeline-Valve 责任链

`Container` 采用 **组合模式（Composite Pattern）** 组织四层父子容器，并在容器间采用 **`Pipeline（管道） + Valve（阀门）` 责任链模式** 逐层传递请求：

| 容器层级 | 对应粒度 | 核心职责与基础阀门（`BasicValve`） |
| :--- | :--- | :--- |
| **1. `Engine`（引擎）** | 全局唯一请求总入口 | 接收来自 `CoyoteAdapter` 的请求，根据请求头中的**域名（Host）**匹配对应的 `Host` 虚拟主机（`StandardEngineValve`） |
| **2. `Host`（虚拟主机）** | 对应一个域名（如 `localhost`） | 管理该域名下的多个 Web 应用，根据 **URL 上下文路径（Context Path）**匹配对应的 `Context`（`StandardHostValve`） |
| **3. `Context`（应用上下文）** | **对应 1 个独立 Web 应用（War 包）** | 持有独立的 **`WebappClassLoader`** 与 `Session Manager`，根据 **Servlet 映射路径**匹配对应的 `Wrapper`（`StandardContextValve`） |
| **4. `Wrapper`（包装器）** | **最小容器，对应 1 个 Servlet 实例** | 负责 Servlet 实例的加载、初始化与销毁；其末端阀门 `StandardWrapperValve` 负责**创建 `ApplicationFilterChain`（过滤器链），依次执行所有匹配的 `Filter`，最后调用 `Servlet.service()`**（Spring MVC 中即 `DispatcherServlet`） |

---

### 2.4 Lifecycle 生命周期机制（组合模式 + 观察者模式）

Tomcat 中所有核心组件（`Server`、`Service`、`Connector`、`Engine`、`Host`、`Context`、`Wrapper`）均实现了统一的 **`Lifecycle` 接口**：
- **核心生命周期方法**：`init()`（初始化） → `start()`（启动） → `stop()`（停止） → `destroy()`（销毁）；
- **组合模式递归控制**：父容器持有子容器引用，只需调用顶层 `Server.init()` 和 `Server.start()`，就会自顶向下**递归初始化并启动所有子组件**；
- **观察者模式事件通知**：组件状态变更时会向注册的 `LifecycleListener` 广播生命周期事件，支持灵活扩展。

---

### 2.5 一次 HTTP 请求在 Tomcat 中的完整 9 步流转流程

1. **TCP 建连**：客户端浏览器发起 HTTP 请求，完成 TCP 三次握手后到达 Tomcat `8080` 端口；
2. **`Acceptor` 接收连接**：`Endpoint` 中的 `Acceptor` 线程 `accept()` 接收该 Socket 连接，将其封装并注册到 `Poller` 的 NIO `Selector` 上；
3. **`Poller` 监听可读事件**：当客户端发送完 HTTP 报文触发读就绪事件时，`Poller` 线程从 `Worker` 线程池中分配一个工作线程处理该连接；
4. **`Processor` 协议解析**：`Worker` 线程调用 `Http11Processor` 读取 Socket 字节流，解析出 HTTP 请求行与 Header，生成 `coyote.Request`；
5. **`CoyoteAdapter` 协议适配**：通过 `CoyoteAdapter.service()` 将 `coyote.Request` 适配包装为标准的 `ServletRequest` 和 `ServletResponse`；
6. **四层容器 `Pipeline-Valve` 责任链路由**：请求依次流经 `Engine Pipeline`（按域名找 Host） → `Host Pipeline`（按前缀找 Context） → `Context Pipeline`（按 URL 找 Wrapper） → `Wrapper Pipeline`；
7. **执行 `FilterChain` 过滤器链**：`StandardWrapperValve` 组装匹配的 `Filter` 链并依次调用 `doFilter()`；
8. **执行 `Servlet.service()`**：过滤器链走完后，调用目标 Servlet（如 Spring MVC 的 `DispatcherServlet` → `Controller`）执行业务逻辑并写入 `Response`；
9. **原路响应写回**：响应沿责任链逆向返回至 `Connector`，经 Socket 将 HTTP 响应报文写回客户端（若开启 `keep-alive` 则将连接归还 `Poller` 继续监听复用，否则关闭连接）。

---

## 三、IO 模型与 NIO 线程模型深度解析

### 3.1 四种 IO 模型演进与对比

| IO 模型 | 全称与协议配置类 | 核心原理与特点 | 版本与生产现状 |
| :--- | :--- | :--- | :--- |
| **BIO** | 阻塞 IO（`Http11Protocol`） | **一连接一线程**，连接建立后哪怕没发数据（如 `keep-alive` 空闲期）也会死死占住一个 Worker 线程，高并发下迅速线程耗尽 | Tomcat 7 及以前默认；**Tomcat 8.5+ 已彻底移除** |
| **NIO（默认 ✅）** | 同步非阻塞 IO（`Http11NioProtocol`） | 基于 JDK NIO `Selector` 多路复用：用极少数 `Poller` 线程管理上万条长连接，**只有真正有数据可读时才分配 Worker 线程处理业务** | **Tomcat 8+ 及 Spring Boot 默认模式，生产绝对主流** |
| **NIO2（AIO）** | 异步非阻塞 IO（`Http11Nio2Protocol`） | 基于 JDK 7 AIO，由操作系统完成读写后触发回调；在 Windows（IOCP）下表现好，但在 Linux 下底层仍由 epoll 模拟，性能与 NIO 持平 | 极少使用 |
| **APR** | Apache 原生库（`Http11AprProtocol`） | 通过 JNI 调用 C 语言编写的 Apache Portable Runtime 与 OpenSSL 库，支持 OS 级零拷贝与极速 SSL 握手，但需额外部署 `.so/.dll` 本地库且排查 Crash 较难 | Tomcat 10.1+ 已废弃，由 NIO + OpenSSL FFM 替代 |

---

### 3.2 Tomcat NIO 三类核心线程（Acceptor / Poller / Worker）

在 Tomcat 的 NIO `Endpoint`（`NioEndpoint`）中，存在三类各司其职的线程：

```text
客户端连接 ──► 【1. Acceptor 线程 (默认1个)】
                    │ accept() 拿到 SocketChannel，设置为非阻塞
                    ▼
              【2. Poller 线程 (Selector 轮询线程, 默认1~2个)】
                    │ 监听 OP_READ 可读事件；空闲连接挂在 Selector 上不占业务线程！
                    │ 一旦检测到有 HTTP 数据到达，封装为 SocketProcessor 任务
                    ▼
              【3. Worker 业务线程池 (http-nio-8080-exec-*, 默认最大200)】
                    │ 同步阻塞执行：HTTP 解析 -> Filter -> Controller -> 查 DB/Redis -> 写回响应
```

> ⚠️ **高频核心认知（Tomcat NIO vs Netty）**：
> Tomcat 的 NIO **仅在“连接管理与等待数据到达（`Poller` 层）”是非阻塞的**；一旦进入 `Worker` 线程读取请求体并执行 `Servlet` / `Controller` 业务代码，**全程依然是同步阻塞的**！如果某个 Controller 接口慢 SQL 卡住 5 秒，这个 `Worker` 线程就会被死死占住 5 秒；若并发有 200 个慢请求，整个 Tomcat 的 200 个 Worker 线程瞬间耗尽，导致所有其他正常接口全部排队超时！

---

### 3.3 核心并发参数辨析（maxConnections vs acceptCount vs maxThreads）

这 4 个参数决定了 Tomcat 的并发承载漏斗（面试极高频追问）：

| 参数名（Spring Boot 配置项） | 默认值 | 作用层级与核心含义 |
| :--- | :---: | :--- |
| **`max-connections`** | `8192`（NIO） | **Tomcat 应用层最大同时持有的 TCP 连接数**。由 `Poller` 的 `LimitLatch` 控制；超过 `8192` 后，`Acceptor` 线程暂停调用 `accept()` |
| **`accept-count`** | `100` | **操作系统内核层 TCP 全连接队列长度（`backlog`）**。当连接数已达 `max-connections`、`Acceptor` 不再取连接时，新完成三次握手的连接在 OS 内核队列排队；若超过 `100`，客户端直接收到 `Connection refused` 或超时 |
| **`threads.max`（`maxThreads`）** | `200` | **`Worker` 业务线程池的最大线程数**（即**同一时刻最多能并行执行多少个请求**，生产通常调优为 `200 ~ 500`） |
| **`threads.min-spare`** | `10` | `Worker` 线程池常驻核心最小空闲线程数（生产建议设为 `20 ~ 50` 避免冷启动频繁建线程） |

- **Tomcat 专用线程池（`org.apache.tomcat.util.threads.ThreadPoolExecutor`）与 JDK 原生线程池的重大区别（加分考点！）**：
  - **JDK 原生 `ThreadPoolExecutor`（CPU 密集型倾向）**：核心线程满 → **先塞满阻塞队列** → 队列满了才创建非核心线程直到 `maximumPoolSize`（因为原生 Tomcat 若用无界/大队列，永远不会扩充到 `maxThreads`）；
  - **Tomcat 定制 `ThreadPoolExecutor`（IO 密集型倾向）**：核心线程（`minSpareThreads`）满 → **优先直接创建新线程直到达到 `maxThreads`（200）** → 只有当工作线程已达上限 `200` 时，才将后续任务放入等待队列 `TaskQueue`！

---

## 四、Tomcat 类加载机制与打破双亲委派（面试最高频）

### 4.1 为什么要打破双亲委派？（类隔离三大核心价值）

JDK 原生的**双亲委派模型（向上委托，父优先）**保证了同一个全限定类名在整個 JVM 中只会被加载一次（全局唯一）。但当一台独立 Tomcat 部署了多个不同的 `.war` 包应用（如 `App-A.war` 和 `App-B.war`）时，原生双亲委派无法满足以下三大需求：
1. **依赖库多版本并存隔离（最核心）**：`App-A` 依赖 Spring 4.x 和 MyBatis 3.4，而 `App-B` 依赖 Spring 5.x 和 MyBatis 3.5。如果由同一个父加载器加载，同名类只能存在一份，必然引发版本冲突报错；
2. **多应用间业务类安全隔离**：`App-A` 与 `App-B` 各自编写的同名业务类应当彼此不可见，防止相互覆盖或越权访问；
3. **支持单个 Web 应用热部署与卸载回收**：在 JVM 中，一个 `Class` 对象只有在其所属的 `ClassLoader` 实例被回收时才能被卸载。为每个 Web 应用配备独立的 `WebappClassLoader`，重新部署单个应用时只需销毁该应用的 `WebappClassLoader` 并新建一个即可，**无需重启整个 Tomcat**。

---

### 4.2 Tomcat 类加载器层级树

```text
          Bootstrap ClassLoader (加载 JRE/lib 核心类库 java.*)
                    ▲
                    │
          Extension ClassLoader (加载 JRE/lib/ext 扩展类库)
                    ▲
                    │
         Application ClassLoader (加载 CLASSPATH / tomcat bin/bootstrap.jar)
                    ▲
                    │
           Common ClassLoader (加载 tomcat/lib 公共目录，所有 Web 应用与 Tomcat 共享)
             ▲             ▲
             │             └───────────────────────┐
   Catalina ClassLoader                   Shared ClassLoader
(加载 Tomcat 自身私有类，对 Web 应用不可见)   (加载所有 Web 应用共享但对 Tomcat 隔离的类)
                                            ▲             ▲
                                            │             │
                                 WebappClassLoader A   WebappClassLoader B
                              (加载 App-A/WEB-INF/*)  (加载 App-B/WEB-INF/*)
                                     【打破双亲委派：本地优先加载！】
```

> 注：在 Tomcat 6+ 默认配置中，`Catalina` 与 `Shared` 目录默认合并到 `CommonClassLoader`（即 `tomcat/lib`），简化为 `Bootstrap → Ext → App → CommonClassLoader → WebappClassLoader`。

---

### 4.3 WebappClassLoader 完整加载顺序（先子后父 + 安全底线）

`WebappClassLoader` 重写了 `loadClass()` 方法，将非核心类的加载顺序改为 **“先子后父（本地优先）”**，但严守 JDK 核心类与 Servlet 规范类的安全底线：

1. **查本地缓存**：先检查当前 `WebappClassLoader` 的本地缓存中是否已经加载过该类，若已加载则直接返回；
2. **查 JVM 缓存**：调用 `findLoadedClass()` 检查 JVM 是否已加载过该类；
3. **🔒 安全底线（委托 `Ext / Bootstrap` 加载 JDK 核心类与 Servlet 规范类）**：
   - 使用 `JavaseClassLoader`（即 `ExtClassLoader`）尝试加载 **`java.*`、`javax.servlet.*` / `jakarta.servlet.*` 等 JRE 核心类与 Web 容器规范类**；
   - **目的**：防止 Web 应用在自己的 `WEB-INF/classes` 下伪造一个恶意的 `java.lang.String` 篡改核心类库，或私自打包 `servlet-api.jar` 引发类型转换异常（`ClassCastException`）！
4. **🔥 打破双亲委派点（优先自行加载本应用下的类）**：
   - 若不是 JDK 核心类（且未开启强制委托 `delegate=true`），**直接调用自身的 `findClass()`，优先扫描并加载当前应用 `/WEB-INF/classes` 目录和 `/WEB-INF/lib/*.jar` 中的类**！
5. **兜底向上委托父加载器**：
   - 如果本应用的 `/WEB-INF` 下找不到该类（说明它可能是 `tomcat/lib` 下的公共类），才向上委托给父加载器 `CommonClassLoader → AppClassLoader` 加载；
6. **加载失败**：若父加载器也找不到，最终抛出 `ClassNotFoundException`。

---

### 4.4 破坏双亲委派的风险与延伸场景

- **破坏双亲委派的潜在风险**：
  1. 若两个不同加载器加载了同一个接口/类（如未排掉公共 `servlet-api.jar`），在跨加载器传递对象时会抛出诡异的 `ClassCastException: com.xxx.Foo cannot be cast to com.xxx.Foo`（因为两个 `Class` 对象的类加载器不同，JVM 视为两个完全不相关的类型）；
  2. 若不加步骤 3 的安全拦截，会导致核心类库被恶意替换。
- **Java 生态中还有哪些破坏双亲委派的经典案例？（连环必问）**：
  1. **JDK SPI 机制（如 JDBC `DriverManager`）**：`java.sql.DriverManager` 位于 `rt.jar`（由顶层 `BootstrapClassLoader` 加载），但它需要加载第三方厂商的 `com.mysql.cj.jdbc.Driver`（位于 `classpath` 下，Bootstrap 加载不到），因此通过 **`Thread.currentThread().getContextClassLoader()`（线程上下文类加载器）** 逆向委托子类加载器 `AppClassLoader` 完成加载；
  2. **OSGi 模块化框架与热部署工具（JRebel、Spring Boot DevTools `RestartClassLoader`）**。

---

## 五、Spring Boot 内嵌 Tomcat 原理与生产调优配置

### 5.1 自动装配与 onRefresh() 启动全流程

引入 `spring-boot-starter-web` 后，会自动传递引入 `spring-boot-starter-tomcat`：
1. **自动配置工厂 Bean**：`ServletWebServerFactoryAutoConfiguration` 检测到类路径下存在 `Tomcat.class`，向容器注册 `TomcatServletWebServerFactory`；
2. **`refresh() -> onRefresh()` 触发创建**：`SpringApplication.run()` 执行到 `ServletWebServerApplicationContext.onRefresh()` 时，调用 `createWebServer()`；
3. **编程式构建 Tomcat 组件**：`TomcatServletWebServerFactory.getWebServer()` 内部直接 `new Tomcat()`，通过 Java 代码创建 `Connector`、`Engine`、`Host` 与 `TomcatEmbeddedContext`（替代了外部 Tomcat 解析 `server.xml` 的过程）；
4. **注册 `DispatcherServlet`**：通过 `DispatcherServletRegistrationBean`（实现 `ServletContextInitializer`）将 Spring MVC 的核心前端控制器 `DispatcherServlet` 及所有 `Filter` 编程式注册到 Tomcat 的 `Context`（包装为 `Wrapper`）；
5. **启动监听**：调用 `tomcat.start()` 开启 8080 端口监听。

---

### 5.2 Spring Boot 内嵌 Tomcat vs 独立外部 Tomcat 深度对比

| 对比维度 | Spring Boot 内嵌 Tomcat | 独立外部 Tomcat |
| :--- | :--- | :--- |
| **打包与运行方式** | 打成可执行 Fat **`jar` 包**，直接 `java -jar` 启动 | 打成 **`war` 包**，放入外部 Tomcat 的 `webapps/` 目录启动 |
| **主导控制权** | **Spring 容器启动并管理 Tomcat**（Tomcat 是 Spring 管理的对象） | **Tomcat 启动并加载 Spring**（通过 `ServletContainerInitializer` 引导） |
| **配置方式** | 在 `application.yml` 中配置 `server.tomcat.*` 或编写 `WebServerFactoryCustomizer` | 修改外部 `conf/server.xml` 配置文件 |
| **应用数量（Context）** | 单进程单应用，默认只有 **1 个 `Context`** | 单进程可部署多个 `.war` 包（对应**多个 `Context`**） |
| **类加载器机制（纠偏考点！）** | 创建 `TomcatEmbeddedWebappClassLoader`，但因单 Jar 部署无需多应用隔离，**默认直接向上委托给 Spring Boot 的 `LaunchedURLClassLoader` / `AppClassLoader` 加载** | 每个 `Context` 拥有独立的 `ParallelWebappClassLoader`，开启**本地 `WEB-INF` 优先加载**以实现多 War 包类隔离 |

> 💡 **补充**：若需将 Spring Boot 项目打成 `war` 包部署到外部独立 Tomcat，需：① 将 `pom.xml` 的 `<packaging>` 改为 `war`；② 将 `spring-boot-starter-tomcat` 的 `<scope>` 设为 `provided`；③ 启动类继承 `SpringBootServletInitializer` 并重写 `configure()` 方法。

---

### 5.3 application.yml 生产级最佳参数配置模板与详解

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
    # 1. Worker 业务线程池配置（核心并发瓶颈点）
    threads:
      max: 300                  # 最大工作线程数（默认 200；IO 密集型生产建议 200~500，切勿超过 800）
      min-spare: 50             # 常驻最小空闲核心线程数（默认 10；调大至 20~50 避免突发流量建线程延迟）
    max-queue-capacity: 200     # 线程达到 max 后的任务等待队列容量（默认 Integer.MAX_VALUE 易堆积，建议设有限值）

    # 2. 网络连接与超时配置
    max-connections: 10000      # NIO 最大并发持有 TCP 连接数（默认 8192）
    accept-count: 200           # 连接数达上限后，OS 内核 TCP 全连接等待队列长度（默认 100）
    connection-timeout: 20000   # 连接建立后等待客户端发送数据的超时时间（20s）
    keep-alive-timeout: 60000   # HTTP Keep-Alive 长连接空闲保持时间（60s）
    max-keep-alive-requests: 100 # 单条长连接最多复用的 HTTP 请求次数（达到后主动断开重连）

    # 3. 请求体与上传限制
    max-http-form-post-size: 50MB # 表单 POST 请求最大字节数（默认 2MB，上传大文件需调大）
    max-swallow-size: 2MB       # 服务端拒绝请求时最多吞吐丢弃的请求体大小

    # 4. 字符集与访问日志
    uri-encoding: UTF-8         # URL 路径与查询参数编码（解决 GET 中文乱码）
    accesslog:
      enabled: true             # 开启 AccessLog 记录接口耗时与状态码
      directory: logs
      prefix: access_log
      suffix: .log
      pattern: "%h %l %u %t \"%r\" %s %b %D" # %D 记录请求处理毫秒耗时，排查慢接口神器！

  # 5. 开启响应 GZIP 压缩（大幅降低 JSON / 文本网络传输耗时）
  compression:
    enabled: true
    min-response-size: 1024     # 响应体超过 1KB 才启用压缩
    mime-types: application/json,application/xml,text/html,text/xml,text/plain,text/css,application/javascript
```

---

### 5.4 编程式深度定制：WebServerFactoryCustomizer

当 `application.yml` 无法覆盖某些底层 `Connector` 参数（如关闭 DNS 反查、开启虚拟线程或添加额外端口）时，可注入 `WebServerFactoryCustomizer`：

```java
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TomcatCustomConfig {

    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> tomcatCustomizer() {
        return factory -> {
            factory.setProtocol("org.apache.coyote.http11.Http11NioProtocol");
            factory.addConnectorCustomizers(connector -> {
                // 关闭 DNS 反向查询，提升高并发性能
                connector.setEnableLookups(false);
                connector.setURIEncoding("UTF-8");
            });
        };
    }
}
```

---

## 六、Tomcat 高频面试真题与线上故障排查

### 6.1 1 分钟面试背诵版（架构篇 + 内嵌篇）

> **【架构与类加载串讲】**：
> “Tomcat 是轻量级 Servlet 容器，整体分为 **Coyote 连接器（`Connector`）** 和 **Catalina 容器（`Container`）** 两大部分。
> `Connector` 内部通过 `Endpoint`（默认 NIO 模型：`Acceptor` 接收连接、`Poller` 基于 Selector 监听可读事件、`Worker` 线程池执行业务）读取字节流，经 `Processor` 解析 HTTP 协议，再由 `CoyoteAdapter` 转换为标准 `ServletRequest` 交给容器。
> `Container` 由 `Engine`（引擎）、`Host`（虚拟主机）、`Context`（Web 应用）、`Wrapper`（单个 Servlet）四层父子容器组成，内部采用 **`Pipeline + Valve` 责任链模式**逐层传递请求，最终由 `StandardWrapperValve` 执行 `FilterChain` 和 `DispatcherServlet.service()`。
> 此外，为了支持单台 Tomcat 部署多个 War 包时的依赖版本隔离与热部署，每个 `Context` 都拥有独立的 `WebappClassLoader`，通过重写 `loadClass()` 在加载非 JDK 核心类时**优先加载本地 `WEB-INF` 下的类**，从而打破了双亲委派。”

---

### 6.2 线上 Tomcat 接口响应变慢 / 线程池打满 5 步排查法

当线上出现“接口大面积超时、但服务器 CPU 利用率却不高”时，90% 是 **Tomcat `Worker` 线程池被慢 IO 阻塞打满**，标准排查步骤如下：

1. **看监控指标（确认是否线程池耗尽）**：
   - 查看 Prometheus / Actuator 指标 `tomcat_threads_busy_threads` 是否已经顶满达到 `tomcat_threads_config_max_threads`（如 `200/200`）；
2. **抓线程栈定位卡点（`jstack`）**：
   - 执行 `jstack <pid> | grep -A 30 "http-nio-8080-exec-"`，查看这 200 个 Tomcat 工作线程绝大部分阻塞在哪个业务方法栈上（常见元凶：① **数据库慢 SQL 或死锁等待连接池**；② **同步调用第三方 HTTP / Feign 接口未设超时或下游卡死**；③ **Redis 大 Key 阻塞或分布式锁等待**）；
3. **查 `access_log` 定位罪魁祸首接口**：
   - 由于 Tomcat 全局共享同一个 `Worker` 线程池（无接口级隔离），只需开启 `access_log` 中的 `%D`（处理耗时 ms），即可一眼找出是哪一个慢接口把全部 200 个线程耗尽并拖垮了其他正常接口；
4. **看 GC 与内存状态（`jstat -gcutil`）**：
   - 排查是否因大对象查询导致频繁 Full GC 引发 STW（Stop-The-World）卡顿；
5. **应急止血与根治方案**：
   - 为所有外部 Feign / HTTP 调用强制设置严格的 `connectTimeout` 与 `readTimeout`，并接入 Sentinel 对慢接口做熔断降级或舱壁线程隔离；优化慢 SQL；将非核心耗时操作改为 MQ 或异步线程池处理。

---

### 6.3 高频易错追问 Q&A

#### Q1：Tomcat 是多线程的吗？Servlet 和 Spring MVC 的 Controller 是线程安全的吗？
- Tomcat 处理请求是**多线程**的（每个请求由 `Worker` 线程池中的一个 `http-nio-exec` 线程执行）；
- 但 **Servlet 和 Spring MVC 的 `@Controller` 默认都是单例（Singleton）的，不是线程安全的！** 多个请求线程会同时并发调用同一个 Controller 实例的方法。因此，**严禁在 Servlet 或 Controller 中定义可读写的非线程安全成员变量**（若需线程隔离上下文，必须使用局部变量或 `ThreadLocal`）。

#### Q2：多台 Tomcat 集群部署时，如何解决 Session 共享问题？
1. **无状态 Token 方案（现代微服务首选 ✅）**：弃用服务端 Session，登录后签发 **JWT** 或将认证信息存入 **Redis**，前端通过 Header 携带 Token；
2. **Spring Session + Redis 集中式存储（传统系统改造首选 ✅）**：通过 Filter 拦截 `HttpServletRequest`，将 `getSession()` 透明替换为从 Redis 读写 Session；
3. **Nginx `ip_hash` 粘性会话（Sticky Session）**：同一 IP 固定路由到同一台 Tomcat，但节点宕机仍会丢失会话；
4. **Tomcat 原生广播复制（DeltaManager）**：节点间全量同步 Session，节点数超过 4 台时网络广播风暴严重，生产不推荐。

#### Q3：Tomcat 处理 GET 和 POST 请求在底层编码与参数解析上有什么区别？
- **请求体读取时机**：GET 请求参数在 URL 查询串（QueryString）中，解析请求头时即读取完毕；POST 表单参数在请求体（Body）中，Tomcat 默认采用**延迟解析**——只有当业务首次调用 `request.getParameter()` 时才会真正从 Socket 输入流读取并解析请求体（且输入流只能读取一次）。
- **字符编码控制**：GET URL 参数编码由 `server.tomcat.uri-encoding=UTF-8` 控制（Tomcat 8+ 默认已为 `UTF-8`）；POST 请求体编码由 `request.setCharacterEncoding("UTF-8")`（即 Spring Boot 的 `CharacterEncodingFilter`）控制。
