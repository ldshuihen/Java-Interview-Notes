# Gradle 

Gradle 是**基于 Groovy/Kotlin DSL**、面向 Java/Kotlin/Android/C++ 等多语言的**现代化自动化构建工具**，它结合了 Ant 的灵活性和 Maven 的依赖管理、约定优于配置优势，是目前 Android、Spring Boot 等主流项目的**默认构建工具**。

一句话总结：**Gradle 就是帮你自动编译代码、管理依赖、打包、测试、发布的「项目管家」**。

---

## 一、先搞懂：为什么要用 Gradle？

对比传统构建工具，Gradle 的核心优势：
1. **强大的依赖管理**：自动下载第三方库（如 Spring、OkHttp）、处理版本冲突、传递依赖
2. **灵活可扩展**：支持自定义任务、插件，能适配任何项目结构
3. **高性能**：增量构建、构建缓存、并行执行，比 Maven 快 10~100 倍
4. **多语言支持**：Java、Kotlin、Android、C++、Python 等都能用
5. **生态成熟**：Android、Spring、Spring Cloud 全栈标配

---

## 二、核心基础概念（必须掌握）
### 1. 项目（Project）
每一个用 Gradle 构建的模块/工程都是一个 `Project`，比如一个 Android app、一个 Java 库。

### 2. 任务（Task）
Gradle 构建的**最小执行单元**，所有操作都是 Task：
- 编译代码：`compileJava`
- 运行测试：`test`
- 打包：`jar`/`assembleRelease`
- 清理：`clean`

你也可以**自定义 Task**（比如自动生成文件、上传包）。

### 3. 构建脚本
Gradle 靠配置文件驱动，核心文件：
| 文件                                      | 作用                                    |
| ----------------------------------------- | --------------------------------------- |
| `build.gradle` / `build.gradle.kts`       | 模块/项目的核心配置（依赖、插件、任务） |
| `settings.gradle` / `settings.gradle.kts` | 管理多模块项目，声明包含哪些模块        |
| `gradle/wrapper/`                         | Gradle 包装器，保证所有人用同一版本构建 |
| `gradle.properties`                       | 全局配置（JVM 参数、版本号等）          |

### 4. 依赖管理（核心）
Gradle 会从**仓库**下载依赖，分为三类：
1. **本地依赖**：项目内的 jar 包
2. **模块依赖**：多模块项目互相引用（如 `implementation project(':library')`）
3. **远程依赖**：Maven 中央仓库、阿里镜像、Google 仓库等

### 5. 插件（Plugin）
Gradle 本身是「空壳」，靠插件实现具体功能：
- `java`：Java 项目构建
- `org.springframework.boot`：Spring Boot 项目
- `com.android.application`：Android 应用
- `kotlin-android`：Kotlin 安卓开发

---

## 三、Gradle 安装与基础使用
### 1. 推荐方式：使用 Gradle Wrapper（无需手动安装）
**几乎所有正式项目都用这个**，它会自动下载对应版本的 Gradle：
```bash
# Linux/Mac
./gradlew [任务名]

# Windows
gradlew.bat [任务名]
```

### 2. 手动安装（可选）
1. 官网下载：https://gradle.org/install/
2. 配置环境变量
3. 验证：`gradle -v`

### 3. 常用命令（必背）
```bash
# 查看所有可用任务
gradlew tasks

# 清理编译产物
gradlew clean

# 编译项目
gradlew build

# 跳过测试编译
gradlew build -x test

# 运行项目（Spring Boot）
gradlew bootRun

# 查看依赖树（排查版本冲突）
gradlew dependencies
```

---

## 四、核心配置文件详解（build.gradle）
以**Java 项目 + Spring Boot**为例，逐行解释：
```groovy
// 1. 声明插件（赋予项目构建能力）
plugins {
    id 'java' // Java 插件
    id 'org.springframework.boot' version '3.2.0' // Spring Boot 插件
    id 'io.spring.dependency-management' version '1.1.4'
}

// 2. 项目基本信息
group = 'com.example' // 组织名
version = '1.0.0' // 版本号
sourceCompatibility = '17' // Java 版本

// 3. 仓库（依赖下载地址）
repositories {
    mavenCentral() // Maven 中央仓库
    // maven { url 'https://maven.aliyun.com/repository/public/' } // 阿里镜像（加速）
}

// 4. 依赖管理（核心！）
dependencies {
    // 编译时依赖（打包进项目）
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // 测试依赖（仅测试时生效）
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// 5. 自定义任务（示例：打印 Hello Gradle）
task helloGradle {
    doLast {
        println 'Hello Gradle!'
    }
}
```

### 依赖关键字区别（高频面试/易错点）
| 关键字               | 作用                                          |
| -------------------- | --------------------------------------------- |
| `implementation`     | 编译+运行时依赖，**不传递给下游模块**（推荐） |
| `api`                | 编译+运行时依赖，**传递给下游模块**           |
| `compileOnly`        | 仅编译时（如 Lombok）                         |
| `runtimeOnly`        | 仅运行时                                      |
| `testImplementation` | 仅测试时                                      |

---

## 五、多模块项目构建（企业级必备）
大型项目都会拆分成多个模块（如 `user-service`、`order-service`），Gradle 完美支持。

### 1. 结构示例
```
root-project/
├── build.gradle // 根项目配置（统一版本）
├── settings.gradle // 声明子模块
├── user-module/
│   └── build.gradle
└── order-module/
    └── build.gradle
```

### 2. settings.gradle（声明模块）
```groovy
rootProject.name = 'demo-project'
// 包含子模块
include 'user-module', 'order-module'
```

### 3. 模块间依赖
`order-module` 依赖 `user-module`：
```groovy
dependencies {
    implementation project(':user-module')
}
```

---

## 六、进阶核心特性
### 1. 增量构建
Gradle 会**缓存已执行的任务**，只重新编译修改过的代码，大幅提速。

### 2. 构建缓存
跨设备/CI 缓存构建产物，团队共享缓存，第二次构建秒级完成。

### 3. 依赖版本统一管理
根项目定义版本，子模块直接引用，避免版本混乱：
```groovy
// 根 build.gradle
ext {
    springBootVersion = '3.2.0'
}

// 子模块引用
implementation "org.springframework.boot:spring-boot-starter-web:$springBootVersion"
```

### 4. 自定义 Task
自动化脚本神器，比如自动备份、上传文件：
```groovy
task backupFiles(type: Copy) {
    from 'src'
    into 'backup'
}
```

---

## 七、常见问题解决方案
### 1. 依赖下载慢
添加**阿里 Maven 镜像**：
```groovy
repositories {
    maven { url 'https://maven.aliyun.com/repository/public/' }
    mavenCentral()
}
```

### 2. 依赖版本冲突
用 `gradlew dependencies` 查看冲突，强制指定版本：
```groovy
configurations.all {
    resolutionStrategy.force 'com.fasterxml.jackson.core:jackson-databind:2.15.2'
}
```

### 3. 构建报错：编码/内存
在 `gradle.properties` 中配置：
```properties
# 编码
org.gradle.jvmargs=-Dfile.encoding=UTF-8
# 增大内存
org.gradle.jvmargs=-Xmx2048m
# 开启并行构建
org.gradle.parallel=true
```

---

## 八、Gradle vs Maven（核心对比）
| 特性     | Gradle                       | Maven          |
| -------- | ---------------------------- | -------------- |
| 配置文件 | Groovy/Kotlin（简洁灵活）    | XML（冗长）    |
| 构建速度 | 极快（增量/缓存）            | 慢             |
| 灵活性   | 极高（自定义任务）           | 低             |
| 依赖管理 | 优秀                         | 良好           |
| 主流场景 | Android、Spring Boot、微服务 | 传统 Java 项目 |

---

### 总结
1. **Gradle 是现代化构建工具**，核心是**依赖管理 + 任务自动化**
2. 核心文件：`build.gradle`（配置）、`settings.gradle`（模块）、`gradlew`（构建命令）
3. 必用命令：`clean`、`build`、`dependencies`、`bootRun`
4. 企业必备：**多模块构建、依赖统一管理、增量构建、构建缓存**
5. 完全替代 Maven，是 Java/Android 开发的**必备技能**

我可以为你提供**Gradle 速记手册**（打印版）、**Spring Boot 完整 Gradle 模板**，或者手把手教你**搭建多模块微服务项目**，需要吗？

**当然可以！而且非常简单！**

# Gradle 添加依赖

完全支持添加 MyBatis、MyBatis-Plus、MySQL驱动等所有Java生态的依赖，用法和 Maven 几乎一样，只是写法更简洁。

我直接给你**最常用、可直接复制**的配置模板。

---

## 一、Gradle 中添加 MyBatis 核心依赖

适用于 **Spring Boot + MyBatis** 项目

### 1. 最简依赖配置（build.gradle）

```groovy
dependencies {
    // Spring Boot Web（必选）
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // MyBatis 整合 Spring Boot（核心）
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'

    // MySQL 驱动（必选，根据你的数据库选择）
    runtimeOnly 'com.mysql:mysql-connector-j'

    // Druid 连接池（可选，推荐）
    implementation 'com.alibaba:druid-spring-boot-3-starter:1.2.21'

    // 数据库测试依赖
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### 2. 如果你用 MyBatis-Plus（国内更常用）

把 mybatis 依赖换成这个：
```groovy
// MyBatis-Plus 3.5.3 + Spring Boot 3
implementation 'com.baomidou:mybatis-plus-spring-boot3-starter:3.5.3.1'
```

---

## 二、完整可运行的 build.gradle 示例（Spring Boot 3 + MyBatis）

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '1.0.0'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

dependencies {
    // web
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // mybatis
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'

    // mysql
    runtimeOnly 'com.mysql:mysql-connector-j'

    // test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## 三、Gradle 依赖小知识（一看就懂）

1. **implementation**：编译+运行都生效，会打进包里
2. **runtimeOnly**：只在运行时生效（如数据库驱动）
3. **Maven 坐标 → Gradle 写法**
   - Maven：`groupId:artifactId:version`
   - Gradle：`implementation 'groupId:artifactId:version'`

**所以所有 Maven 上有的依赖，Gradle 都能直接用！**

#  Gradle + Spring Cloud

下面给你一套**可直接复制、开箱即用**的 Gradle + Spring Cloud（含 Alibaba）完整配置，从版本匹配、仓库、BOM 管理到常用组件（Nacos/Feign/Gateway/MyBatis）全覆盖，适合微服务多模块项目。

---

## 一、版本匹配（必须严格对应）
先定好“Spring Boot ↔ Spring Cloud ↔ Spring Cloud Alibaba”版本，避免诡异报错。
推荐稳定组合（2026 常用）：
- **Spring Boot**：3.2.x（如 3.2.5）
- **Spring Cloud**：2023.0.x（代号 Leyton，如 2023.0.3）
- **Spring Cloud Alibaba**：2023.0.1.2（适配 Spring Cloud Leyton）

---

## 二、项目结构（多模块微服务）
```
cloud-demo/
├── build.gradle          # 根项目（统一版本、BOM）
├── settings.gradle        # 声明子模块
├── common/                # 公共模块（工具、实体、异常）
│   └── build.gradle
├── order-service/         # 订单微服务
│   └── build.gradle
└── gateway/               # 网关模块
    └── build.gradle
```

---

## 三、第一步：根项目 build.gradle（核心！统一管理）
```groovy
// plugins 块（全局插件）
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.5' apply false // 不打包，仅版本管理
    id 'io.spring.dependency-management' version '1.1.4' apply false
}

// 全局变量：统一版本号（子模块直接引用）
ext {
    springBootVersion = '3.2.5'
    springCloudVersion = '2023.0.3'
    springCloudAlibabaVersion = '2023.0.1.2'
    mybatisVersion = '3.0.3'
    mysqlVersion = '8.0.33'
    druidVersion = '1.2.21'
}

// 所有子模块共享配置
subprojects {
    apply plugin: 'java'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    group = 'com.example.cloud'
    version = '1.0.0'
    sourceCompatibility = '17' // 统一 JDK 版本

    // 仓库：优先阿里云（国内加速）
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        mavenCentral()
    }

    // 🔴 关键：引入 Spring Cloud 与 Alibaba BOM（子模块无需写版本）
    dependencyManagement {
        imports {
            // Spring Cloud 官方 BOM
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
            // Spring Cloud Alibaba BOM（Nacos、Sentinel 等）
            mavenBom "com.alibaba.cloud:spring-cloud-alibaba-dependencies:${springCloudAlibabaVersion}"
        }
    }

    // 子模块通用依赖（每个微服务都要）
    dependencies {
        // Spring Boot Web
        implementation 'org.springframework.boot:spring-boot-starter-web'
        // 测试
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
}
```

---

## 四、第二步：settings.gradle（声明子模块）
```groovy
rootProject.name = 'cloud-demo'
// 包含所有子模块
include 'common', 'order-service', 'gateway'
```

---

## 五、第三步：子模块配置（以 order-service 为例）
`order-service/build.gradle`
```groovy
dependencies {
    // 1. 依赖公共模块
    implementation project(':common')

    // 2. Spring Cloud Alibaba 核心（Nacos 注册+配置中心）
    implementation 'com.alibaba.cloud:spring-cloud-starter-alibaba-nacos-discovery'
    implementation 'com.alibaba.cloud:spring-cloud-starter-alibaba-nacos-config'

    // 3. 微服务常用组件
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign' // 远程调用
    implementation 'org.springframework.cloud:spring-cloud-starter-loadbalancer' // 负载均衡（替代 Ribbon）

    // 4. 数据库：MyBatis + MySQL + Druid
    implementation "org.mybatis.spring.boot:mybatis-spring-boot-starter:${mybatisVersion}"
    runtimeOnly "com.mysql:mysql-connector-j:${mysqlVersion}"
    implementation "com.alibaba:druid-spring-boot-3-starter:${druidVersion}"

    // 5. 可选：Sentinel 熔断降级
    // implementation 'com.alibaba.cloud:spring-cloud-starter-alibaba-sentinel'
}
```

---

## 六、第四步：网关模块（gateway/build.gradle）
```groovy
dependencies {
    // 网关核心
    implementation 'org.springframework.cloud:spring-cloud-starter-gateway'
    // 注册到 Nacos
    implementation 'com.alibaba.cloud:spring-cloud-starter-alibaba-nacos-discovery'
    // 负载均衡
    implementation 'org.springframework.cloud:spring-cloud-starter-loadbalancer'

    // 网关不用 web（会冲突）
    exclude group: 'org.springframework.boot', module: 'spring-boot-starter-web'
}
```

---

## 七、第五步：配置文件（application.yml）
### 1. order-service 的 application.yml
```yaml
spring:
  application:
    name: order-service # 服务名（Nacos 注册用）
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 地址
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml # 配置文件格式

# 数据库
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cloud_order?useSSL=false&serverTimezone=UTC
    username: root
    password: 123456

# MyBatis
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.cloud.order.entity
```

### 2. 网关 gateway 的 application.yml
```yaml
spring:
  application:
    name: gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes:
        - id: order-route
          uri: lb://order-service # 负载均衡到订单服务
          predicates:
            - Path=/order/** # 拦截 /order/ 开头请求
```

---

## 八、第六步：启动类注解
### 1. 普通微服务（order-service）
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient // 注册到 Nacos
@EnableFeignClients    // 开启 Feign 远程调用
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

### 2. 网关（gateway）
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

---

## 九、常用命令（构建/启动）
```bash
# 根目录执行：构建所有模块
gradlew build

# 启动指定模块（如 order-service）
gradlew :order-service:bootRun

# 查看依赖树（排查冲突）
gradlew :order-service:dependencies
```

---

## 十、避坑要点

1. **版本严格匹配**：Spring Cloud ↔ Spring Boot ↔ Alibaba 必须对应，否则启动报错。
2. **网关排除 web**：Gateway 用 WebFlux，和 spring-boot-starter-web 冲突。
3. **Nacos 配置**：`spring.cloud.nacos.config.file-extension` 必须指定 yaml 或 properties。
4. **依赖关键字**：微服务间调用用 `implementation`，API 模块用 `api`。

---

要不要我把这套配置整理成一个**可直接解压运行的压缩包**，包含完整目录结构、所有配置文件和启动类，你直接导入 IDEA 就能用？