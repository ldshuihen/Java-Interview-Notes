## 技能

Java 开发需要的技能取决于**职业阶段**和**业务领域**，但核心能力可以分层来看：

---

### 一、基础必备（任何阶段都绕不开）

| 技能               | 说明                                                       |
| ------------------ | ---------------------------------------------------------- |
| **Java 核心语法**  | 集合、多线程、IO、反射、泛型、JVM 内存模型                 |
| **面向对象与设计** | SOLID 原则、设计模式（不是背 23 种，而是理解常用 5-10 种） |
| **数据结构与算法** | LeetCode 中等难度能独立解决，理解时间/空间复杂度           |
| **数据库基础**     | SQL 编写、索引原理、事务 ACID、简单调优                    |
| **版本控制**       | Git 分支管理、冲突解决、rebase/cherry-pick                 |
| **构建工具**       | Maven/Gradle 依赖管理、生命周期、插件机制                  |
| **Linux 基础**     | 常用命令、日志排查、简单 Shell 脚本                        |

---

### 二、工程化能力（从"能写"到"能交付"）

| 技能            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| **Spring 生态** | Spring Boot（自动配置、Starter）、Spring MVC、Spring Data JPA/MyBatis |
| **Web 开发**    | RESTful API 设计、HTTP 协议、前后端交互、JSON/XML 处理       |
| **中间件**      | Redis（缓存策略、分布式锁）、Kafka/RabbitMQ（消息可靠性）、Elasticsearch（倒排索引） |
| **微服务**      | Spring Cloud（Gateway、Nacos/Consul、Feign、熔断限流）或 Dubbo |
| **容器化**      | Docker 镜像构建、Docker Compose、K8s 基础概念                |
| **CI/CD**       | Jenkins/GitLab CI、自动化测试、灰度发布                      |

---

### 三、高阶与系统思维（3-5 年以上）

| 技能           | 说明                                                       |
| -------------- | ---------------------------------------------------------- |
| **JVM 调优**   | GC 算法、内存泄漏排查、Arthas 等诊断工具                   |
| **高并发架构** | 分库分表、读写分离、分布式事务（Seata）、幂等性设计        |
| **性能优化**   | SQL 慢查询优化、缓存穿透/击穿/雪崩防护、接口压测（JMeter） |
| **安全**       | JWT/OAuth2、XSS/SQL 注入防护、接口幂等与防重放             |
| **云原生**     | K8s 深入、Service Mesh（Istio）、Serverless（可选）        |

---

### 四、容易被忽视但关键的"软技能"

- **代码审查（Code Review）**：能发现逻辑漏洞、性能隐患、可读性问题
- **文档能力**：写清楚技术方案、接口文档、故障复盘
- **业务理解**：技术方案要匹配业务场景，避免过度设计
- **问题排查**：从日志、监控、链路追踪（SkyWalking/Zipkin）定位根因

---

### 实用建议

如果你是**初学者**，先死磕 **Java 核心 + Spring Boot + MySQL + Redis**，能独立做完一个 CRUD 项目再谈其他。  
如果是**进阶开发者**，重点补**分布式系统原理**和**性能调优**，这是面试和实际工作中最能拉开差距的地方。

你目前处于哪个阶段？我可以针对你的情况给更具体的学习路径。

## 调度

这是一个非常切中要害的架构问题。

**答案是：不应该由一个“Skill”来统一调用，而应该由“Agent的大脑（LLM配合System Prompt / 状态机）”来统一调度。**

在AI Agent的架构设计中，必须严格区分**“技能”**和**“调度器”**。如果把这9个Skills封装成一个大Skill，会陷入典型的“反模式”。以下是具体的分析和正确的架构设计：

### 一、 为什么不能用一个“超级Skill”统一调用？

假设你设计了一个名为 `develop_java_feature` 的超级Skill，内部按顺序调用那9个小Skill。
1. **丧失了灵活性（死板）**：
   Java开发不是线性的流水线。如果Agent执行到 Skill 4（编译）发现报错，它需要退回到 Skill 2（读取）和 Skill 3（修改）。如果是一个超级Skill，内部的循环和跳转逻辑会极其复杂，甚至陷入死循环。
2. **LLM失去了思考能力（黑盒）**：
   我们使用Agent的目的是让LLM像人一样思考。如果全部封装，LLM就变成了一个单纯的“传话筒”，它无法根据编译报错的具体内容（如是找不到符号，还是类型转换错误）来动态决定下一步是去查依赖树（Skill 6），还是去改代码（Skill 3）。
3. **上下文丢失**：
   大Skill在内部调用小Skill时，中间的报错信息往往会被截断或总结，LLM拿不到最原始的报错堆栈，导致修复成功率大幅下降。

### 二、 正确的设计：通过“SOP（标准作业程序）”和“状态机”调度

这9个Skills应该是平级的、暴露给LLM的工具。我们需要在**System Prompt（系统提示词）**中为LLM定义一套“调度规范”，让它按需调用。

具体的调度逻辑应该是一个**带有反馈闭环的状态机**：

#### 1. 接收任务阶段
* **LLM思考**：这是一个新功能开发，我需要先了解现有代码。
* **调度动作**：调用 `Skill 1（检索）` 和 `Skill 9（规范检索）`。

#### 2. 编码阶段
* **LLM思考**：我已经拿到了上下文和规范，现在开始写代码。
* **调度动作**：调用 `Skill 2（读取）` 确认行号 -> 调用 `Skill 3（编写与修改）` 写入代码。

#### 3. 验证阶段（核心调度逻辑）
* **LLM思考**：代码写完了，必须先保证能跑通。
* **调度动作**：调用 `Skill 4（编译）`。
  * **分支 A（成功）**：LLM思考 -> 编译过了，该检查规范了。调度 `Skill 8（Lint守卫）`。
  * **分支 B（失败）**：LLM拿到报错信息 -> 思考错误原因。
    * 如果是“缺少依赖包” -> 调度 `Skill 6（依赖管理）` -> 加完包后重新调度 `Skill 4`。
    * 如果是“语法错误/类型不匹配” -> 调度 `Skill 3（修改代码）` -> 改完后重新调度 `Skill 4`。

#### 4. 测试与排错阶段
* **LLM思考**：编译和Lint都过了，开始跑单测。
* **调度动作**：调用 `Skill 5（测试执行）`。
  * **分支 A（成功）**：任务完成，输出总结。
  * **分支 B（失败）**：LLM拿到断言失败信息 -> 调度 `Skill 3（修改代码）` 修复业务逻辑 -> 重新调度 `Skill 5`。
  * **分支 C（运行时崩溃）**：如果是空指针或Bean注入失败 -> 调度 `Skill 7（运行时日志分析）` -> 拿到根因后 -> 调度 `Skill 3` 修复。

### 三、 如何在工程上实现这种调度？

你不需要写代码去硬编码这个调度逻辑，而是通过设计良好的 **System Prompt** 来约束大模型。例如，在Prompt中写入如下“宪法级”指令：

> **【Java开发Agent工作守则】**
> 1. **禁止盲写**：在调用 `write_code` 之前，必须先调用 `search_codebase` 或 `read_file` 获取确切上下文。
> 2. **强制闭环**：任何代码修改后，必须立即调用 `compile_code`。禁止在编译通过前调用 `run_tests`。
> 3. **错误自愈**：当你收到 `compile_code` 或 `run_tests` 的错误输出时，你必须分析错误并调用 `write_code` 进行修复，直到成功。如果连续修复3次仍失败，请停止并向上级汇报。
> 4. **规范前置**：当任务涉及中间件（Redis/MQ等），必须先调用 `fetch_standards` 获取规范，再调用 `write_code`。

### 总结

这9个Skills是Agent的**“手脚”**，它们必须是独立、原子化的。
统一的调度不应该在代码层面用一个Skill包裹，而应该交由**LLM的大脑（基于Prompt的ReAct模式：思考-行动-观察）**来进行动态的、有状态的智能调度。这才是AI Agent区别于传统自动化脚本的核心价值。

## Skills

为了解决前面提到的“不包含中间件规范”以及“操作颗粒度”的问题，我对这8个Skills进行了**深度升级与扩展**。

升级的核心思路是：**从“纯文本操作”升级为“AST（抽象语法树）感知操作”，从“被动执行”升级为“主动引入规范上下文”，并增加“架构守卫”能力。**

以下是升级优化后的完整 Skills 设计（扩展为 9 个核心 Skills）：

---

### 1. 智能代码库检索与模式分析 Skill (原 Skill 1 升级)
* **作用**：理解现有代码结构，寻找依赖关系，**并主动发现项目中已有的中间件使用模式和规范代码**。
* **输入**：关键词、类名、方法签名、或意图描述（如“Redis缓存实现”）。
* **底层实现**：
  * 结合 `ripgrep` 进行文本检索。
  * 结合 JavaParser AST 解析类继承树和方法调用链。
  * **增强点**：支持“找相似”查询，例如检索带有 `@Cacheable` 或特定 `RedisUtil` 调用的类，提取其实现模式。
* **输出**：匹配的文件路径、方法签名、依赖树，**以及推荐复用的现有工具类或规范模板路径**。
* **设计理由**：强制Agent在写代码前“抄作业”，避免它凭空捏造不符合项目规范的中间件调用方式。

### 2. AST感知精准代码读取 Skill (原 Skill 2 升级)
* **作用**：精确读取指定文件的内容，支持按行号或**按语法结构**读取。
* **输入**：文件绝对路径、读取范围（行号区间，或“类名”、“方法名”、“import区”）。
* **底层实现**：基于 JavaParser，不仅返回纯文本，还能识别出当前光标/目标所在的方法体、变量作用域。
* **输出**：带有行号前缀的文件内容，**附带当前代码块的AST上下文摘要**（如：当前位于 `UserService#getUser` 方法内，类成员变量包含 `redisUtil`）。
* **设计理由**：Java是强作用域语言，提供AST上下文摘要，能让LLM知道当前可用哪些变量和Bean，避免生成未定义的引用。

### 3. 高阶代码编写与自动格式化 Skill (原 Skill 3 升级)
* **作用**：执行具体的写文件操作，并**强制保证代码风格统一**。
* **输入**：文件路径、操作类型（新建、覆盖、精准替换）。
* **底层实现**：
  * 支持基于字符串的精准替换。
  * **增强点**：底层集成 Google Java Format 或 Spotless 插件。每次写入或替换后，底层自动触发格式化。
* **输出**：修改成功状态、实际修改的行号范围。
* **设计理由**：LLM生成的缩进往往混乱，对于Python是灾难，对于Java也会导致CheckStyle报错。底层自动格式化彻底解决了这个问题，同时保护了文件的其他部分不被破坏。

### 4. 增量编译与结构化报错 Skill (原 Skill 4 升级)
* **作用**：验证代码语法和依赖，**加速编译过程并提供易于LLM理解的错误格式**。
* **输入**：模块路径、是否跳过测试（`-DskipTests`）。
* **底层实现**：
  * 执行 `mvn compile -q -o`（开启增量编译，离线模式优先，提速）。
  * **增强点**：解析编译输出日志，过滤掉Maven自身的日志，提取 `ERROR` 行，生成结构化错误。
* **输出**：`BUILD SUCCESS` 或 结构化错误列表（包含：文件绝对路径、错误行号、错误类型如 `cannot find symbol`、具体的错误描述）。
* **设计理由**：Maven原生日志极其冗长，会浪费大量Token且干扰LLM判断。结构化输出能让Agent一针见血地定位到“第45行少了个分号”或“未导入某个包”。

### 5. 测试执行与覆盖率分析 Skill (原 Skill 5 升级)
* **作用**：运行单元测试，验证业务逻辑，**并评估测试质量**。
* **输入**：测试类路径或具体的方法名。
* **底层实现**：执行 `mvn test -Dtest=YourTestClass#yourMethod`，并结合 JaCoCo 获取覆盖率。
* **输出**：测试通过率、失败断言信息、异常堆栈跟踪、**当前修改代码的行覆盖率百分比**。
* **设计理由**：不仅告诉Agent测试是否通过，还告诉它“你写的测试有没有真正覆盖到修改的代码行”，防止Agent写无意义的空测试（如只写一个 `assertTrue(true)`）来蒙混过关。

### 6. 依赖管理与冲突排查 Skill (原 Skill 6 升级)
* **作用**：添加、升级、移除第三方库依赖，**并能解决Jar包冲突**。
* **输入**：依赖 `groupId:artifactId:version`、操作类型。
* **底层实现**：
  * 解析并修改 `pom.xml` / `build.gradle`。
  * **增强点**：执行 `mvn dependency:tree`，底层内置冲突检测逻辑。
* **输出**：更新状态。如果检测到冲突（如 `NoSuchMethodError` 常见原因），输出冲突的依赖链路及**建议排除的依赖路径**。
* **设计理由**：Java的“依赖地狱”非常出名。Agent引入新包时极易引发版本冲突（如同时引入不同版本的FastJSON）。此Skill让Agent具备自动排查和解决 `ClassNotFoundException` 的能力。

### 7. 运行时控制与根因分析 Skill (原 Skill 7 升级)
* **作用**：不仅获取日志，还能**控制本地应用的启停**，并智能解析异常。
* **输入**：操作指令（启动/停止/获取日志）、过滤条件（如只看 `com.demo` 包下的错误）。
* **底层实现**：
  * 支持以非阻塞方式启动 `spring-boot:run`。
  * 监控控制台输出或 `spring.log`。
  * **增强点**：针对冗长的Java StackTrace（尤其是嵌套的 `Caused by:`），底层进行截断和根因提取。
* **输出**：应用运行状态（RUNNING/STOPPED）、**Root Cause 摘要**（如：`BeanCreationException: UserService依赖的OrderMapper未找到，发生在UserService.java:35`）。
* **设计理由**：Spring Boot启动时的报错堆栈往往长达上百行。LLM读长堆栈容易丢失焦点。底层提取最终的 `Caused by` 交给LLM，大幅提升纠错效率。

### 8. 架构守卫与规范Lint Skill (原 Skill 8 升级)
* **作用**：执行静态检查，**强制卡控业务规范和中间件使用规范**。
* **输入**：待检查的文件路径或模块。
* **底层实现**：
  * 底层调用 CheckStyle / SonarQube。
  * **增强点**：集成 ArchUnit 或自定义AST规则，检查架构规范（如：Controller层禁止直接调用DAO；中间件必须使用公司统一的 `RedisUtil` 而不能裸用 `RedisTemplate`）。
* **输出**：规范违规列表（文件、行号、违规规则编号、修复建议）。
* **设计理由**：这是Agent的“质检员”。如果Agent写了不符合规范的中间件代码（如硬编码Redis Key），此Skill会报错，强制Agent回到 Skill 3 进行修改，形成闭环。

---
*(新增)*
### 9. 规范检索与上下文注入 Skill (针对中间件规范痛点新增)
* **作用**：在编写复杂业务或中间件代码前，**主动拉取团队沉淀的规范文档、最佳实践代码片段**。
* **输入**：任务意图（如“Kafka生产者配置”、“分页查询规范”）。
* **底层实现**：
  * 集成 RAG（检索增强生成）机制，连接企业的Wiki或Markdown规范库。
  * 也可从代码库中抽取被标记为 `@StandardTemplate` 的 exemplary 代码。
* **输出**：与任务相关的规范条款、推荐的API用法、甚至是可直接复用的代码模板。
* **设计理由**：前8个Skill是“手”，这个Skill是“脑”。它能从根本上解决LLM“写出的代码能跑但不规范”的问题。通过在执行 Skill 3 之前调用此Skill，确保Agent在正确的上下文指导下编写代码。

---

### 总结：升级后的优势

1. **防幻觉能力更强**：通过 Skill 1 找现有模式 + Skill 9 注入规范，大幅减少LLM胡编乱造中间件API的情况。
2. **容错率更高**：Skill 3 的自动格式化和 Skill 4 的结构化报错，让Agent在“修改->编译->报错->再修改”的循环中更加丝滑。
3. **符合企业级标准**：Skill 8 从普通的Lint升级为“架构守卫”，能够拦截越权调用、硬编码等工程恶习，产出的代码真正达到可直接Merge的标准。

# PA-RPC-Center 支付模块编码规范

## 概述

本 Skill 定义了 PA-RPC-Center 项目中支付模块（pay/csf）的编码规范，用于指导代码审查、代码生成和代码补全。

## 项目架构

### 多仓库/多模块结构

本项目采用 **多仓库 + 多模块** 架构，共包含 4 个核心模块：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              多模块架构                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐              │
│  │   ctrl       │─────▶│   center     │◀─────│   domain     │              │
│  │  (接口层)     │      │  (业务中心)   │      │  (实体类)     │              │
│  │              │      │              │      │              │              │
│  │ • Controller │      │ • CSF接口     │      │ • Entity     │              │
│  │ • 参数校验    │      │ • Business   │      │ • DTO/VO     │              │
│  │ • 接口文档    │      │ • Dao        │      │ • Request    │              │
│  │              │      │              │      │ • Response   │              │
│  └──────────────┘      └──────────────┘      └──────────────┘              │
│         │                     │                     ▲                      │
│         │                     │                     │                      │
│         └─────────────────────┴─────────────────────┘                      │
│                               │                                            │
│                               ▼                                            │
│                        ┌──────────────┐                                    │
│                        │   common     │                                    │
│                        │  (公共模块)   │                                    │
│                        │              │                                    │
│                        │ • 常量定义    │                                    │
│                        │ • 错误码      │                                    │
│                        │ • 工具类      │                                    │
│                        │ • 国际化消息  │                                    │
│                        └──────────────┘                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 模块职责与路径

| 模块       | 职责                                         | 代码路径                                                  | Git 仓库 |
| ---------- | -------------------------------------------- | --------------------------------------------------------- | -------- |
| **domain** | 实体类定义（Entity/DTO/VO/Request/Response） | `pa-rpc-domain/src/main/java/com/cmi/rpc/domain/pay/app/` | 独立     |
| **common** | 公共常量、错误码、工具类、国际化消息         | `pa-rpc-common/src/main/java/com/cmi/rpc/common/`         | 独立     |
| **ctrl**   | 对外接口层（Controller），对接外部系统       | `pa-rpc-ctrl/src/main/java/...`                           | 独立     |
| **center** | 核心业务逻辑（CSF接口实现）                  | `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/`     | 当前项目 |

### 模块间调用关系

```
ctrl ────────▶ center ────────▶ domain
  │              │                ▲
  │              │                │
  └──────────────┴────────────────┘
                 │
                 ▼
               common (被所有模块依赖)
```

### 各模块详细结构

#### 1. domain 模块（实体类）

```
pa-rpc-domain/
└── src/main/java/com/cmi/rpc/domain/pay/
    ├── app/entity/          # 实体类（数据库映射）
    │   ├── CostPaymentRequest.java
    │   ├── PaymentHeader.java
    │   └── ...
    ├── app/payment/         # 支付相关实体
    ├── app/payMaster/       # 付款主数据
    └── app/resale/          # 转售相关实体
```

#### 2. common 模块（公共类）

```
pa-rpc-common/
└── src/main/java/com/cmi/rpc/common/
    ├── constants/           # 常量定义
    │   ├── CostPaymentConstants.java
    │   ├── PaymentHeaderConstants.java
    │   └── ResalePaymentConstants.java
    ├── enums/               # 枚举类
    │   └── MessageAccessor.java    # 国际化消息访问器
    ├── exception/           # 异常类
    ├── util/                # 工具类
    └── sequence/            # 序列号生成
```

#### 3. ctrl 模块（接口层）

```
sa-rpc-ctrl/
└── src/main/java/com/ai/mvno/business/pay/controller/
    ├── payment/             # 付款相关接口
    │   ├── PaymentHeaderController.java
    │   └── PaymentLineController.java
    └── ...
```

#### 4. center 模块（业务中心）

```
pa-rpc-center/
└── src/main/java/com/cmi/rpc/domain/pay/
    ├── csf/                 # CSF接口层
    │   ├── interfaces/      # 接口定义
    │   └── impl/            # 接口实现
    ├── business/            # 业务逻辑层
    │   ├── interfaces/      # 业务接口
    │   └── impl/            # 业务实现
    └── dao/                 # 数据访问层
        ├── interfaces/      # Dao接口
        └── impl/            # Dao实现
```

## 代码注释规范

### 类注释

所有类必须包含 Javadoc 注释，标注 `@author` 和 `@date`：

```java
/**
 * @author 作者名
 * @Description: 类功能描述
 * @date 2024年01月01日 10:00
 */
public class PaymentHeaderCSVImpl implements IPaymentHeaderCSV {
```

### 方法注释

方法必须有中文注释说明功能逻辑：

```java
/**
 * 查询付款单列表
 * @param request 查询条件
 * @return 付款单列表
 * @throws Exception 异常
 */
public Response<PageResult<PaymentInvoiceQueryDTO>> queryList(PaymentInvoiceQueryDTO request) throws Exception {
```

### 行内注释

复杂业务逻辑需要添加行内注释：

```java
// 计算付款金额：取申请金额和发票余额的最小值
BigDecimal paymentAmount = least(requestAmount, invoiceBalance);
```

## Git 分支使用规范

### 分支命名规范

**格式**: `日期@工号@功能描述`

**示例**:

```
20260408@yangdi5@补充参与方税号
20260323@yangdi5@统一支付批量提交
```

### 分支创建流程

1. **从 master 拉取最新代码**

   ```bash
   git checkout master
   git pull origin master
   ```

2. **创建功能分支**

   ```bash
   git checkout -b 20260408@yangdi5@功能描述
   ```

3. **开发完成后提交**

   ```bash
   git add .
   git commit -m "@工号@问题描述@详细说明"
   git push origin 分支名
   ```

### 分支合并流程

1. **开发完成** → 合并到 `uat` 分支测试
2. **测试通过** → 合并到 `master` 分支上线

```bash
# 合并到 uat
git checkout uat
git merge 20260408@yangdi5@功能描述
git push origin uat

# 合并到 master
git checkout master
git merge 20260408@yangdi5@功能描述
git push origin master
```

## Git 提交信息规范

### 规范一：标准格式（推荐）

**格式**: `<type>: <subject>`

| Type       | 说明                       |
| ---------- | -------------------------- |
| `feat`     | 新功能                     |
| `fix`      | 修复 bug                   |
| `docs`     | 文档更新                   |
| `style`    | 代码格式调整（不影响功能） |
| `refactor` | 重构                       |
| `perf`     | 性能优化                   |
| `test`     | 测试相关                   |
| `chore`    | 构建/工具相关              |

**示例**:

```
feat: 添加付款单批量导出功能

fix: 修复发票金额计算精度问题

refactor: 优化查询 SQL 性能
```

### 规范二：生产问题修复格式（必填）

**格式**: `@<工号>@<问题描述>@<详细说明>`

**规则**:

- 必须以 `@` 符号开头
- 第一个 `@` 后填写**工号**（如：yangdi5）
- 第二个 `@` 后填写**问题描述**，长度**不得少于15个字**
- 第三个 `@` 后填写**详细说明**，长度**不得少于15个字**

**示例**:

```
@yangdi5@生产问题问题修复生产问题修复@生产问题问题修复生产问题修复修复
```

**说明**:

- 问题描述：简要概括修复的问题类型或模块
- 详细说明：具体描述修复内容和影响范围
- 该格式用于生产环境问题修复的强制规范

## CSF 接口层规范

### 接口定义规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/csf/interfaces/`

**命名规范**: `I` + 业务名 + `CSV.java`

**示例**:

```java
public interface IPaymentHeaderCSV {
    // 取消打包
    Response<PageResult<PaymentHeader>> unpack(List<PaymentHeaderDTO> request) throws Exception;
    
    // 查询列表
    Response<PageResult<PaymentInvoiceQueryDTO>> queryList(PaymentInvoiceQueryDTO request) throws Exception;
    
    // 保存数据
    Response<?> paymentSave(PaymentHeader request) throws Exception;
}
```

### 接口实现规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/csf/impl/`

**命名规范**: 业务名 + `CSVImpl.java`

**示例**:

```java
/**
 * @author houlinhao
 * @Description: 付款单CSF接口实现
 * @date 2024年08月14日 17:06
 */
public class PaymentHeaderCSVImpl implements IPaymentHeaderCSV {
    
    private static final Logger log = Logger.getLogger(PaymentHeaderCSVImpl.class);
    
    // 注入其他服务
    IPaymentHeaderDaoSV iPaymentHeaderDaoSV = (IPaymentHeaderDaoSV) ServiceFactory.getService(IPaymentHeaderDaoSV.class);
    
    @Override
    public Response<PageResult<PaymentInvoiceQueryDTO>> queryList(PaymentInvoiceQueryDTO request) throws Exception {
        // 1. 参数校验
        // 2. 调用 Business 层或 Dao 层
        // 3. 结果处理
        // 4. 返回响应
    }
}
```

### CSF 接口职责

1. **参数校验** - 基础参数合法性检查
2. **权限检查** - 用户操作权限验证
3. **业务编排** - 调用 Business 层完成业务逻辑
4. **结果封装** - 统一包装返回 `Response<T>`
5. **异常处理** - 捕获并转换异常为统一错误码

### Response 返回规范

CSF 接口实现中返回 `Response` 对象时，**必须使用构造函数创建**，禁止使用 `Response.success()` 静态方法。

**正确写法**:

```java
// 成功响应 - 使用构造函数
return new Response<>(true, "200", "查询成功", null, result);

// 失败响应
return new Response<>(false, "500", "申请单信息不能为空", null, null);
```

**错误写法**:

```java
// 禁止 - 使用静态方法
return Response.success(resultList);
```

**构造函数参数说明**:

| 参数      | 类型    | 说明                                   |
| --------- | ------- | -------------------------------------- |
| success   | boolean | 是否成功（true/false）                 |
| code      | String  | 状态码（"200"表示成功，"500"表示失败） |
| message   | String  | 返回消息（中文描述）                   |
| errorCode | String  | 错误码（可选，失败时使用）             |
| data      | T       | 返回数据对象                           |

**参考示例**: 参见 `ErpPaymentRequestCSVImpl.java` 中的返回方式

## Business 层规范

### 接口定义规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/business/interfaces/`

**命名规范**: `I` + 业务名 + `BizSV.java`

**示例**:

```java
public interface IPaymentHeaderBizSV {
    // 业务方法
    PaymentHeader processPayment(PaymentHeaderDTO request) throws Exception;
}
```

### 实现规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/business/impl/`

**命名规范**: 业务名 + `BizSVImpl.java`

**示例**:

```java
/**
 * @author 作者名
 * @Description: 付款单业务逻辑实现
 * @date 2024年01月01日 10:00
 */
public class PaymentHeaderBizSVImpl implements IPaymentHeaderBizSV {
    
    @Override
    public PaymentHeader processPayment(PaymentHeaderDTO request) throws Exception {
        // 1. 业务规则校验
        // 2. 调用 Dao 层操作数据
        // 3. 执行业务逻辑
        // 4. 返回结果
    }
}
```

### Business 层职责

1. **业务规则校验** - 复杂的业务逻辑校验
2. **数据转换** - DTO 与 Entity 之间的转换
3. **业务编排** - 多个 Dao 操作的组合
4. **事务控制** - 手动控制事务（特殊场景）

## Dao 层规范

### 接口定义规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/dao/interfaces/`

**命名规范**: `I` + 业务名 + `DaoSV.java`

**示例**:

```java
public interface IPaymentHeaderDaoSV {
    // 查询列表（分页）
    PageResult<PaymentInvoiceQueryDTO> queryList(PaymentInvoiceQueryDTO request) throws Exception;
    
    // 根据ID查询
    PaymentHeader queryById(Long headerId) throws Exception;
    
    // 插入数据
    int insert(PaymentHeader record) throws Exception;
    
    // 更新数据
    int updateByPrimaryKey(PaymentHeader record) throws Exception;
}
```

### 实现规范

**位置**: `pa-rpc-center/src/main/java/com/cmi/rpc/domain/pay/dao/impl/`

**命名规范**: 业务名 + `DaoSVImpl.java`

**示例**:

```java
/**
 * @author 作者名
 * @Description: 付款单数据访问实现
 * @date 2024年01月01日 10:00
 */
public class PaymentHeaderDaoSVImpl implements IPaymentHeaderDaoSV {
    
    private static final Logger log = LoggerFactory.getLogger(PaymentHeaderDaoSVImpl.class);
    
    @Override
    public PageResult<PaymentInvoiceQueryDTO> queryList(PaymentInvoiceQueryDTO request) throws Exception {
        PageResult pageResult = new PageResult();
        Map<String,Object> params = SqlHandleUtil.objectToMap(request);
        
        // 1. 查询总数
        if (request.getPage() != null) {
            StringBuilder countSql = new StringBuilder("SELECT COUNT(1) COUNT_NO FROM (" + buildQuerySql(request, params) + ")");
            List<Map<String, Object>> countList = DBCaller.execQuery(RPCDBType.RPC.getConnName(), countSql.toString(), params, true);
            pageResult.setCount(Maps.getInt(countList.get(0), "COUNT_NO"));
        }
        
        // 2. 查询分页数据
        StringBuilder sqlBuilder = new StringBuilder();
        sqlBuilder.append("SELECT * FROM (");
        sqlBuilder.append("SELECT T.*, ROWNUM RN FROM (");
        sqlBuilder.append(buildQuerySql(request, params));
        sqlBuilder.append(") T WHERE ROWNUM <= ").append(request.getPage().getEndRowIndex());
        sqlBuilder.append(") WHERE RN >= ").append(request.getPage().getStartRowIndex());
        
        List<Map<String, Object>> resultList = DBCaller.execQuery(RPCDBType.RPC.getConnName(), sqlBuilder.toString(), params, true);
        List<PaymentInvoiceQueryDTO> list = SqlHandleUtil.convertList(resultList, PaymentInvoiceQueryDTO.class);
        pageResult.setResult(list);
        
        return pageResult;
    }
}
```

### Dao 层职责

1. **SQL 构建** - 动态 SQL 拼接
2. **参数处理** - 使用参数化查询（禁止 SQL 拼接）
3. **结果映射** - 将查询结果转换为对象
4. **分页处理** - 实现三层嵌套分页

## Controller 层规范

### 位置

`sa-rpc-ctrl/src/main/java/com/ai/mvno/business/pay/controller/`

### 命名规范

业务名 + `Controller.java`

### 示例

```java
/**
 * @author 作者名
 * @Description: 付款单Controller
 * @date 2024年01月01日 10:00
 */
@RestController("paymentHeaderController.v1")
@RequestMapping("/api/v1/payment-headers")
public class PaymentHeaderController {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentHeaderController.class);
    
    @GetMapping("/queryList")
    public Response<PageResult<PaymentInvoiceQueryDTO>> queryList(PaymentInvoiceQueryDTO request, Page page) {
        request.setPage(page);
        Response<PageResult<PaymentInvoiceQueryDTO>> response = ServiceHelper.call("rpc_IPaymentHeaderCSV_queryList_request", request);
        return response;
    }
    
    @PostMapping("/paymentSave")
    public Response<?> paymentSave(@RequestBody PaymentHeader request) {
        Response<?> response = ServiceHelper.call("rpc_IPaymentHeaderCSV_paymentSave_request", request);
        return response;
    }
}
```

### Controller 层职责

1. **接收请求** - 接收 HTTP 请求，参数解析
2. **参数校验** - 基础参数合法性校验
3. **调用业务** - 通过 `ServiceHelper.call()` 调用 center 模块的 CSF 接口
4. **返回响应** - 统一包装返回 `Response<T>`
5. **Swagger 文档** - 提供 API 接口文档

## 分页查询规范

### 标准分页 SQL 结构

```sql
-- 1. 查询总数
SELECT COUNT(1) COUNT_NO FROM (
    -- 主查询 SQL
)

-- 2. 查询分页数据
SELECT * FROM (
    SELECT T.*, ROWNUM RN FROM (
        -- 主查询 SQL
        ORDER BY xxx
    ) T WHERE ROWNUM <= #{endNum}
) WHERE RN >= #{startNum}
```

### 三层嵌套结构

1. **最内层**：主查询（包含所有 JOIN 和 WHERE 条件）
2. **中间层**：添加 `ROWNUM <= endNum` 限制最大行数
3. **最外层**：添加 `RN >= startNum` 限制起始行号

### Java 实现示例

```java
public PageResult<PaymentInvoiceQueryDTO> queryList(PaymentInvoiceQueryDTO request) throws Exception {
    PageResult pageResult = new PageResult();
    Map<String,Object> params = SqlHandleUtil.objectToMap(request);
    
    if (request.getPage() != null) {
        pageResult.setPageNum(request.getPage().getPageNum());
        pageResult.setPageSize(request.getPage().getPageSize());
        
        // 查询总数
        StringBuilder countSql = new StringBuilder("SELECT COUNT(1) COUNT_NO FROM (" + buildQuerySql(request, params) + ")");
        List<Map<String, Object>> countList = DBCaller.execQuery(RPCDBType.RPC.getConnName(), countSql.toString(), params, true);
        if (countList != null && countList.size() > 0) {
            pageResult.setCount(Maps.getInt(countList.get(0), "COUNT_NO"));
        } else {
            pageResult.setCount(0);
        }
    }
    
    // 查询分页数据
    StringBuilder sqlBuilder = new StringBuilder();
    if (request.getPage() != null) {
        sqlBuilder.append("SELECT * FROM (");
        sqlBuilder.append("SELECT T.*, ROWNUM RN FROM (");
    }
    sqlBuilder.append(buildQuerySql(request, params));
    if (request.getPage() != null) {
        int startNum = request.getPage().getStartRowIndex();
        int endNum = request.getPage().getEndRowIndex();
        sqlBuilder.append(") T WHERE ROWNUM <= ").append(endNum);
        sqlBuilder.append(") WHERE RN >= ").append(startNum);
    }
    
    List<Map<String, Object>> resultList = DBCaller.execQuery(RPCDBType.RPC.getConnName(), sqlBuilder.toString(), params, true);
    List<PaymentInvoiceQueryDTO> list = SqlHandleUtil.convertList(resultList, PaymentInvoiceQueryDTO.class);
    pageResult.setResult(list);
    
    return pageResult;
}
```

## 事务管理规范

### 框架自动事务

默认情况下，使用框架自动事务管理。

### 手动控制事务

特殊场景需要手动控制事务：

```java
if (!ServiceManager.getSession().isStartTransaction()) {
    logger.info("手动开启事务");
    ServiceManager.getSession().startTransaction();
}
try {
    // 业务逻辑
    ServiceManager.getSession().commitTransaction();
    logger.info("手动提交事务");
} catch (Exception e) {
    logger.error("事务回滚", e);
    ServiceManager.getSession().rollbackTransaction();
    throw e;
}
```

## SQL 编写规范

### 参数化查询

必须使用参数化查询，禁止 SQL 拼接：

```java
// 正确
queryBuilder.append("AND sai.INVOICE_NO = :INVOICE_NO ");
params.put("INVOICE_NO", invoiceNo);

// 错误（禁止）
queryBuilder.append("AND sai.INVOICE_NO = '" + invoiceNo + "' ");
```

### 表别名规范

| 表名                  | 别名     |
| --------------------- | -------- |
| RPC_AP_INVOICE_INS    | sai      |
| RPC_PAYMENT_REQUEST   | rpr/scpr |
| RPC_PAYMENT_INVOICE   | rpi/scpi |
| RPC_PAYMENT_HEADER    | sph/spa  |
| RPC_PAYMENT_LINE      | spl      |
| RPC_PAYMENT_PRE_CHECK | sppc     |

### SQL 关键字规范

- 关键字使用大写（SELECT、FROM、WHERE、AND、OR 等）
- 表名、列名使用大写
- 使用 `||` 进行字符串拼接（Oracle 语法）
- 日期转换使用 `TO_DATE()` 和 `TO_CHAR()`

## 实体类设计规范

### 继承规范

所有 DTO/VO/Entity 等实体类**必须继承 `Base`**，禁止实现 `Serializable`：

**正确写法**:

```java
public class ResaleInvoiceQueryDTO extends Base {
    // ...
}
```

**错误写法**:

```java
// 禁止 - 实现 Serializable
public class ResaleInvoiceQueryDTO implements Serializable {
    // ...
}
```

### 序列化规范

**禁止**在类中显式定义 `serialVersionUID` 字段：

**正确写法**:

```java
public class ResaleInvoiceQueryDTO extends Base {
    private String circuitIdQuery;
    // ...
}
```

**错误写法**:

```java
// 禁止 - 添加 serialVersionUID
public class ResaleInvoiceQueryDTO extends Base {
    private static final long serialVersionUID = 1L;
    private String circuitIdQuery;
    // ...
}
```

## 编码规范检查清单

### 代码审查维度

1. **代码注释规范**
   - [ ] 类是否包含 Javadoc 注释（@author、@date）
   - [ ] 方法是否有中文注释说明功能
   - [ ] 复杂逻辑是否有行内注释
   - [ ] 注释是否使用中文编写

2. **项目架构规范**
   - [ ] 是否遵循分层架构（CSF -> Business -> Dao）
   - [ ] 模块间调用是否正确
   - [ ] 是否直接操作数据库（必须通过 Dao 层）
   - [ ] 常量是否定义在 common 模块

3. **实体类设计规范**
   - [ ] DTO/VO/Entity 是否继承 `Base`
   - [ ] 是否禁止实现 `Serializable`
   - [ ] 是否禁止定义 `serialVersionUID`

4. **代码质量**
   - [ ] 命名是否清晰（驼峰命名法）
   - [ ] 是否存在重复代码
   - [ ] 方法是否过长（建议不超过 50 行）
   - [ ] 类是否过大（建议不超过 500 行）

5. **异常处理**
   - [ ] 是否有完善的异常处理
   - [ ] 是否捕获了具体异常而非 Exception
   - [ ] 异常信息是否清晰有用

6. **性能与安全**
   - [ ] 循环中是否有 SQL 查询（N+1 问题）
   - [ ] 是否有硬编码的敏感信息
   - [ ] 输入参数是否进行了校验
   - [ ] 是否存在 SQL 注入风险

7. **Git 提交规范**
   - [ ] 提交信息是否符合格式要求
   - [ ] 生产问题修复是否使用 `@工号@问题描述@详细说明` 格式
   - [ ] 问题描述和详细说明是否 ≥ 15 个字

## 错误码国际化规范

### 错误码定义

**位置**: `pa-rpc-common/src/main/java/com/cmi/rpc/common/constants/`

错误码常量定义在各个 `Constants` 类的内部接口中：

```java
public class CostPaymentConstants {
    public interface ErrorCode {
        String COST_PAYMENT_REQUEST_VENDOR_NOT_NULL = "cost_payment_request_vendor_not_null";
        String COST_PAYMENT_REQUEST_INVOICE_NOT_EMPTY = "cost_payment_request_invoice_not_empty";
        // ...
    }
}
```

### 国际化消息注册

**位置**: `pa-rpc-common/src/main/java/com/cmi/rpc/common/enums/MessageAccessor.java`

在 `MESSAGE_ACCESSORS` 列表中注册错误码对应的中英文消息：

```java
private static final List<MessageAccessor> MESSAGE_ACCESSORS = Arrays.asList(
    new MessageAccessor(COST_PAYMENT_REQUEST_VENDOR_NOT_NULL, 
        "供应商不能为空", 
        "Vendor cannot be empty"),
    new MessageAccessor(COST_PAYMENT_REQUEST_INVOICE_NOT_EMPTY, 
        "不存在发票头信息", 
        "Invoice header information does not exist"),
    // ...
);
```

### 使用方式

#### 1. 添加错误信息（推荐）

```java
private void addErrorMsg(List<String> errorList, String code, Locale locale, Object... parameters) {
    String desc;
    if (locale != null && locale.equals(Locale.US)) {
        desc = MessageAccessor.getMessageEngDesc(code, parameters);
    } else {
        desc = MessageAccessor.getMessageDesc(code, parameters);
    }
    if (!errorList.contains(desc)) {
        errorList.add(desc);
    }
}
```

#### 2. 带参数的错误信息

```java
// 错误码定义
String COST_PAYMENT_REQUEST_INVOICE_LINE_NOT_EMPTY = "cost_payment_request_invoice_line_not_empty";

// MessageAccessor 注册
new MessageAccessor(COST_PAYMENT_REQUEST_INVOICE_LINE_NOT_EMPTY, 
    "发票编号【{0}】不存在发票行", 
    "Invoice number [{0}] does not have invoice lines.");

// 使用
addErrorMsg(errorList, COST_PAYMENT_REQUEST_INVOICE_LINE_NOT_EMPTY, locale, invoiceNum);
```

#### 3. 直接获取错误描述

```java
// 获取中文描述
String cnDesc = MessageAccessor.getMessageDesc(code, parameters);

// 获取英文描述
String enDesc = MessageAccessor.getMessageEngDesc(code, parameters);

// 获取 MessageAccessor 对象
MessageAccessor msg = MessageAccessor.getMessage(code);
String desc = msg.desc();           // 中文
String engDesc = msg.getEngDesc();  // 英文
```

### 新增错误码步骤

1. **在 Constants 类中定义错误码常量**

   ```java
   String ERP_PAYMENT_REQUEST_INVOICE_EMPTY = "erp_payment_request_invoice_empty";
   ```

2. **在 MessageAccessor.java 中注册消息**

   ```java
   new MessageAccessor(ERP_PAYMENT_REQUEST_INVOICE_EMPTY, 
       "发票列表不能为空", 
       "Invoice list cannot be empty"),
   ```

3. **在代码中使用**

   ```java
   addErrorMsg(errorList, ERP_PAYMENT_REQUEST_INVOICE_EMPTY, locale);
   ```

### 常用可复用错误码

| 错误码                                                     | 中文描述                                      | 使用场景         |
| ---------------------------------------------------------- | --------------------------------------------- | ---------------- |
| `COST_PAYMENT_REQUEST_VENDOR_NOT_NULL`                     | 供应商不能为空                                | 供应商校验       |
| `COST_PAYMENT_REQUEST_CURRENCY_CODE_SCALE_NOT_NULL`        | 申请单币种财务精度不存在                      | 币种精度校验     |
| `BANK_INFO_IS_NULL`                                        | 银行信息为空                                  | 银行信息校验     |
| `COST_BANK_VENDOR_NUM_DIFFERENT`                           | 银行信息不存在于该供应商【{0}】下             | 银行供应商一致性 |
| `COST_PAYMENT_REQUEST_INVOICE_NOT_EMPTY`                   | 不存在发票头信息                              | 发票头存在性     |
| `COST_PAYMENT_REQUEST_INVOICE_LINE_NOT_EMPTY`              | 发票编号【{0}】不存在发票行                   | 发票行存在性     |
| `COST_PAYMENT_REQUEST_INVOICE_AMOUNT_NOT_NULL`             | 【{0}】发票头金额为空                         | 发票金额非空     |
| `COST_PAYMENT_REQUEST_INVOICE_AMOUNT_NOT_GREATER_ZERO`     | 发票头金额不大于0                             | 发票金额大于0    |
| `COST_PAYMENT_REQUEST_SCALE_ERROR`                         | {0} 币种精度应不超过 {1} 位                   | 金额精度校验     |
| `COST_PAYMENT_REQUEST_ENTITY_SUPPLIER_CURRENCY_CODE_ERROR` | 付款申请头和发票行 主体 供应商 币种不完全一致 | 一致性校验       |

## 注意事项

1. **master 分支保护**: master 分支为生产环境代码，禁止直接推送
2. **uat 分支**: 用于测试环境，开发完成后先合并到 uat 测试
3. **及时同步**: 开发周期较长时，定期从 master 同步最新代码
4. **冲突解决**: 合并时如有冲突，在本地解决后再推送
5. **分支清理**: 已合并到 master 的分支及时删除

这份 Skill（编码规范）整体结构清晰，涵盖了模块划分、分层架构、Git 工作流、SQL规范和国际化等核心内容，已经是一份相当不错的指导文档。

## 不足之处

但针对一个**支付模块**以及**代码生成/审查场景**，这份规范还存在一些明显的不足和可以补充的地方。以下是从**支付业务特性、代码健壮性、AI 代码生成友好度、工程实践**四个维度的改进建议：

### 一、 缺少支付业务核心特性的规范（最致命的缺失）

作为支付模块，规范中完全没有提及金融/支付系统最核心的业务约束，这会导致 AI 生成代码时忽略关键防线：

1. **金额计算与处理规范（极度重要）**：
   - 未规定必须使用 `BigDecimal`，禁止使用 `double`/`float`。
   - 未规定 `BigDecimal` 的比较规范（必须使用 `compareTo()` 而非 `equals()`，因为 `1.0` 和 `1.00` 不相等）。
   - 未规定精度和舍入模式（如必须指定 `RoundingMode.HALF_UP`）。
   - 未规定金额单位转换规范（如数据库存“分”，业务层用“元”时的转换约定）。
2. **幂等性设计规范**：
   - 支付核心操作（如扣款、退款、状态流转）必须支持幂等。
   - 缺乏对幂等键（如 `paymentId + requestId`）的使用规范，以及如何处理并发重复请求的指导。
3. **并发与锁控制规范**：
   - 支付单状态变更时，未提及是否需要使用乐观锁（如 `version` 字段更新）或分布式锁。
   - 未提供防重入、防超卖（如发票余额被多次扣减）的代码规范。
4. **敏感数据安全规范**：
   - 银行卡号、账户号、密码等敏感信息在日志中如何脱敏（如 `log.info("银行卡号: {}", maskBankCard(cardNo))`）？
   - 接口出参是否需要对敏感字段进行脱敏处理？

### 二、 缺少日志规范（工程实践缺失）

文档中多次出现了 `Logger`，但缺乏统一的日志规范：

1. **日志级别使用规范**：什么场景用 `error`，什么场景用 `warn`，什么场景用 `info`/`debug`。
2. **日志内容规范**：
   - 入口接口必须打印入参和出参（但要注意敏感信息脱敏）。
   - 异常捕获时，`log.error` 必须带异常堆栈（如 `log.error("msg", e)`），不能只打印消息。
3. **日志框架混用**：文档示例中 `PaymentHeaderCSVImpl` 用了 `Logger.getLogger`（Log4j/JUL），而 `PaymentHeaderDaoSVImpl` 用了 `LoggerFactory.getLogger`（SLF4J）。**这是严重的规范冲突**，必须明确统一使用 SLF4J + Logback 等门面框架。

### 三、 异常处理规范不够严谨

1. **异常抛出方式**：所有的接口都 `throws Exception`，这在规范中是不推荐的。应该定义业务模块的自定义异常（如 `PaymentBusinessException`），并规定优先抛出自定义异常。
2. **Controller 层异常兜底**：未规定 Controller 层是否需要全局异常处理器（`@ControllerAdvice` / `@RestControllerAdvice`），如果 CSF 层抛出异常，Controller 如何捕获并转换为友好的 `Response`。

### 四、 代码生成（AI友好度）与复用性不足

1. **分页代码过度冗余**：
   - `Dao` 层示例中的三层嵌套分页 SQL 拼接代码非常长且重复度高。应当提供一个基础的 `BaseDao` 或分页工具类（如 `PageHelper`），让 AI 生成代码时只需关注核心业务 SQL，而不是每次都重写 ROWNUM 拼接逻辑。
2. **DTO/VO 转换规范**：
   - 未规定 Entity 与 DTO 之间的转换规范。是手动 `set`？还是用 `BeanUtils.copyProperties`？或者推荐使用 MapStruct？AI 需要明确的指令来生成转换代码。
3. **空指针防御规范**：
   - 未提供集合判空、字符串判空的工具类使用规范（如推荐使用 `CollectionUtils.isEmpty()` 而非 `list != null && list.size() > 0`）。
4. **时间日期规范**：
   - 未明确 Java 8 的 `LocalDateTime`/`LocalDate` 与传统 `Date` 的选择，以及与 Oracle 数据库 `DATE`/`TIMESTAMP` 交互时的转换规则。

### 五、 其他细节完善建议

1. **枚举类的规范**：
   - 支付状态、单据类型等强依赖状态的字段，应强制要求使用枚举，而不是在代码中硬编码魔法值（如 `"PAID"`，`"UNPAID"`）。规范中应指明枚举类放在 `common` 模块的 `enums` 包下。
2. **Git 提交规范冲突**：
   - 文档中同时存在“标准格式（feat: xxx）”和“生产问题修复格式（@工号@问题@说明）”。这对 AI 来说会产生困惑。建议明确：常规开发用标准格式，生产工单修复用工单格式，并给出 AI 判断的依据。
3. **Controller 的 Swagger 注解规范**：
   - 既然提到提供 Swagger 文档，就应该给出 `@Api`、`@ApiOperation`、`@ApiModelProperty` 等注解的示例，要求 AI 在生成 Controller 和 DTO 时自动带上这些注解。
4. **常量类的使用边界**：
   - 规定了魔法数字和字符串应提取为常量，但未说明业务状态常量、SQL 字段常量应该放在哪个具体的 Constants 类中。

### 总结修改建议（针对 Skill）

如果希望 AI 基于这份 Skill 更好地生成支付代码，建议补充以下章节：
- `## 金额与数值计算规范`（BigDecimal 强制使用、比较与舍入规则）
- `## 日志与安全脱敏规范`（统一 SLF4J、日志级别、敏感数据打码规则）
- `## 异常与空处理规范`（自定义业务异常、全局异常捕获、集合/对象判空工具使用）
- `## 支付业务核心要求`（幂等性设计要求、并发控制/乐观锁要求）
- 修正示例代码中 `Logger` 导包不一致的问题。
- 补充 DTO 转换工具或 MapStruct 的使用推荐。