# AutoGen 0.75 最新版 超详细全解（2026官方稳定版）
AutoGen 0.75 是**微软官方最新稳定版**，完全重构了 API、架构、配置方式，**和 0.4/0.2 老版本不兼容**，是现在唯一推荐使用的版本。

我会用**最新 0.75 标准**从头讲：核心概念、安装、配置、基础用法、多智能体群聊、工具调用、代码执行、最佳实践。

---

## 一、AutoGen 0.75 核心定位
- **新一代多智能体框架**：基于 `Conversation Programming`（对话式编程）
- **0.75 最大变化**：
  1. 全新统一 API
  2. 全新配置系统
  3. 全新 Agent 体系
  4. 全新群聊 GroupChat
  5. 全新工具调用、代码执行、本地运行模式
- **一句话总结**：**多个 AI 角色自动对话协作，完成复杂任务**（写代码、做数据分析、做项目、查资料、写报告）

---

## 二、安装（0.75 官方最新）
```bash
pip install -U "autogen>=0.7.5"
```

验证版本：
```python
import autogen
print(autogen.__version__)  # 确保 ≥0.7.5
```

---

## 三、核心基础概念

### 1. 核心 Agent（只有 2 个最常用）

1. **AssistantAgent**：AI 助手（思考、回答、写代码、调用工具）
2. **UserProxyAgent**：用户代理（执行代码、调用工具、传递消息、模拟人类）

#### 一、整体定位

在 `autogen>=0.7.5` 版本中，**所有业务智能体都派生自这两个基础类**
- `AssistantAgent`：**思考决策方**，纯LLM驱动，负责推理、构思、生成内容、规划流程、发起工具调用
- `UserProxyAgent`：**执行交互方**，对接外部环境，负责运行代码、执行函数、接管人机输入、收发执行结果

---

#### 二、AssistantAgent 完整详解

##### 1. 核心作用

1. 接收对话上下文，依靠大模型**自主思考、逻辑推理**
2. 输出文本回答、业务方案、结构化代码
3. 主动识别任务需求，**主动发起工具函数调用**
4. 设定角色人设、任务立场、行为规范
5. 多智能体群聊中担任**专业角色**（开发、产品、分析师、架构师等）

##### 2. 核心构造参数（0.75 标准）

```python
AssistantAgent(
    name: str,                      # 智能体唯一名称，对话中身份标识
    llm_config: dict,               # 大模型全局配置（必传）
    system_message: str = "",       # 人设+指令+行为约束（核心灵魂）
    description: str = None,        # 群聊中角色简介，用于自动选发言人
    is_termination_msg: callable=None, # 自定义判断是否结束对话
    max_consecutive_auto_reply: int=None, # 最大连续自动回复次数
)
```

##### 3. 核心能力

###### （1）LLM 原生能力

- 上下文记忆自动维护，继承对话历史
- 支持`temperature`、`top_p`等采样参数调节风格
- 兼容OpenAI、Azure、通义、DeepSeek、Ollama本地模型

###### （2）工具调用能力

- 通过装饰器 `@assistant.register_for_llm()` 声明可用工具
- 自动分析是否需要调用函数、自动拼装入参
- 工具执行结果返回后，自动继续推理整合答案

###### （3）对话控制能力

- 可自定义终止话术识别
- 限制连续发言次数，防止死循环闲聊
- 支持修改默认回复逻辑，重写`generate_reply`

##### 4. 典型使用场景

- 代码编写、算法实现、脚本开发
- 需求梳理、方案设计、文案创作
- 数据分析思路规划、推理研判
- RAG检索后的答案整合、总结润色
- 群聊内专业角色输出观点

##### 5. 专属常用装饰器

```python
# 注册给LLM：让Assistant知道有这个工具、怎么调用
@assistant.register_for_llm(description="工具功能描述")
```

---

#### 三、UserProxyAgent 完整详解

##### 1. 核心作用

1. **模拟人类用户**，发起任务、下达指令
2. **代码沙盒执行者**：识别代码块，自动运行Python代码
3. **工具实际执行者**：接收Assistant发起的函数调用，本地执行
4. **人机交互入口**：控制是否需要人工介入确认
5. 收集执行结果、报错日志，回传给AI助手进行修正迭代

##### 2. 核心构造参数（0.75 重点）

```python
UserProxyAgent(
    name: str,
    human_input_mode: str,          # 人机交互模式【重中之重】
    code_execution_config: dict|False, # 代码执行配置
    llm_config: dict = None,        # 可选：也可给User挂载LLM
    system_message: str = "",
    max_consecutive_auto_reply: int=None,
)
```

##### 3. 三大核心必配参数详解

###### ① human_input_mode 三种模式

| 枚举值      | 作用                           | 适用场景                       |
| ----------- | ------------------------------ | ------------------------------ |
| `NEVER`     | **全程全自动，无需任何人干预** | 自动化脚本、批量任务、无人值守 |
| `ALWAYS`    | 每一轮对话都询问用户输入       | 人机交互式聊天、边聊边决策     |
| `TERMINATE` | 仅任务即将结束时询问确认       | 半自动化，收尾人工审核         |

###### ② code_execution_config 代码执行开关

```python
# 开启代码执行（最常用）
code_execution_config = {
    "work_dir": "workspace",    # 代码文件存放目录
    "use_docker": False,        # 关闭容器，本地直接运行（Windows/Mac首选）
    "timeout": 30,              # 代码运行超时时间
}

# 关闭代码执行
code_execution_config = False
```
- 自动提取```python```代码块
- 自动保存文件 → 自动运行 → 捕获stdout/stderr报错
- 把运行结果/错误信息原路发回Assistant自动排错

###### ③ llm_config

默认**不需要给UserProxy配模型**，只做执行；
特殊场景可挂载LLM，让用户代理也具备思考能力。

##### 4. 核心能力

1. **全自动代码运行+自动排错闭环**
   AI写代码 → User运行 → 报错回传 → AI改代码 → 再次运行
2. **函数调用落地执行**
   仅负责执行，不负责思考决策
3. **对话发起入口**
   唯一调用 `initiate_chat(recipient, message)` 启动对话的主体
4. 消息转发、会话状态托管

##### 5. 专属常用装饰器

```python
# 注册实际执行逻辑，对接本地函数
@user_proxy.register_for_execution()
```

##### 6. 典型使用场景

- 所有自动化任务**启动入口**
- 代码运行、脚本调试、数据运算
- 调用本地接口、本地文件读写、本地系统操作
- 人工审核介入、权限管控
- 群聊中的总指挥、任务发起方

---

#### 四、两大Agent 协作底层流程（0.75标准流程）

1. **UserProxy** 调用 `initiate_chat` 下发任务指令
2. **Assistant** 接收指令，LLM思考制定方案
3. 需要写代码 → 输出代码块
4. **UserProxy** 识别代码并自动执行
5. 执行成功/失败结果回传给Assistant
6. Assistant 基于结果继续优化、补全、收尾
7. 满足终止条件后对话结束

**极简口诀**
**Assistant 动脑规划，UserProxy 动手干活**

---

#### 五、核心区别对照表

| 维度        | AssistantAgent               | UserProxyAgent               |
| ----------- | ---------------------------- | ---------------------------- |
| 核心定位    | 思考者、决策者、创作者       | 执行者、调度者、用户化身     |
| 是否依赖LLM | **必须依赖**，无模型无法工作 | 可无LLM纯执行                |
| 核心行为    | 生成文本、代码、调用意图     | 运行代码、执行函数、接收输入 |
| 对话角色    | 应答方、专业角色             | 发起方、管控方               |
| 工具权限    | 声明工具、决定调用           | 真实运行工具逻辑             |
| 人机控制    | 无权限干预人工流程           | 全权控制是否需要人工介入     |
| 常用搭配    | 多个组成团队角色             | 全局唯一任务发起者           |

---

#### 六、0.75 标准最简配对模板（可直接复制运行）

```python
from autogen import AssistantAgent, UserProxyAgent

llm_cfg = {
    "config_list": [
        {"model": "gpt-4o-mini", "api_key": "xxx", "base_url": "xxx"}
    ],
    "temperature": 0
}

# 思考端
assist = AssistantAgent(
    name="智能助手",
    llm_config=llm_cfg,
    system_message="你是资深开发，输出完整可运行代码"
)

# 执行端
user = UserProxyAgent(
    name="执行代理",
    human_input_mode="NEVER",
    code_execution_config={"work_dir":"run_code","use_docker":False}
)

# 启动对话
user.initiate_chat(assist, message="写一个冒泡排序并测试")
```

需要我再补充**自定义继承重写这两个Agent**、修改内置对话逻辑的进阶写法吗？



### 2. 核心工作模式

- UserProxyAgent ↔ AssistantAgent **自动对话**
- 多 Agent **群聊 GroupChat**（0.75 完全重写）
- 支持**代码自动执行**
- 支持**工具调用**（联网、搜索、自定义函数）
- 支持**本地模型 + 云端模型**

#### 一、一对一自动对话模式（最基础）

##### 1. 流程逻辑

**UserProxyAgent（发起方） ↔ AssistantAgent（应答方）**
1. UserProxy 下发任务指令
2. Assistant 依靠 LLM 思考、生成内容/方案
3. 双向自动轮询对话，无需人工干预
4. 满足终止条件自动结束会话

##### 2. 核心特点

- 最简双智能体组合，开发最快
- 天然自带上下文记忆，连贯对话
- 支持自动多轮修正、迭代优化
- 适合：问答、文案、简单脚本、日常推理

##### 3. 0.75 标准代码

```python
from autogen import AssistantAgent, UserProxyAgent

llm_config = {"config_list": [{"model":"gpt-4o-mini","api_key":"xxx"}]}

# 思考助手
assist = AssistantAgent(name="AI助手", llm_config=llm_config)
# 用户执行代理
user = UserProxyAgent(name="用户端", human_input_mode="NEVER", code_execution_config=False)

# 启动一对一对话
user.initiate_chat(assist, message="讲解Java线程池核心原理")
```

---

#### 二、多Agent群聊 GroupChat 模式（0.75全新重构）

##### 1. 架构组成

- 多个**AssistantAgent** 担任不同专业角色
- 一个**UserProxyAgent** 作为任务发起入口
- **GroupChat**：管理会话成员、轮次、发言规则
- **GroupChatManager**：群聊调度中枢，决定谁发言

##### 2. 两大发言策略

1. **auto 自动选人**（默认）
   LLM 自主判断当前该由哪个角色发言，最贴合团队协作
2. **round_robin 轮询发言**
   按固定顺序轮流说话，流程固定可控

##### 3. 运行流程

1. 用户下发总需求
2. 产品→设计→开发→测试依次/自主分工讨论
3. 角色之间互相提问、纠错、评审
4. 达成统一方案后自动终止

##### 4. 适用场景

项目开发、方案评审、头脑风暴、流程化复杂任务

##### 5. 0.75 群聊标准写法

```python
from autogen import AssistantAgent, UserProxyAgent, GroupChat, GroupChatManager

llm_config = {"config_list": [{"model":"gpt-4o-mini","api_key":"xxx"}]}

# 多角色
pm = AssistantAgent(name="产品",llm_config=llm_config)
dev = AssistantAgent(name="开发",llm_config=llm_config)
test = AssistantAgent(name="测试",llm_config=llm_config)
user = UserProxyAgent(name="发起人",human_input_mode="NEVER")

# 群聊配置
groupchat = GroupChat(agents=[user,pm,dev,test],max_round=12)
manager = GroupChatManager(groupchat=groupchat,llm_config=llm_config)

# 启动群聊
user.initiate_chat(manager,message="设计一个简易学生管理系统")
```

---

#### 三、代码自动执行模式（最强生产力）

##### 1. 运行机制

1. Assistant 输出 ````python```` 格式代码块
2. UserProxy 自动识别代码块
3. 本地沙箱自动保存、运行代码
4. 捕获**控制台输出 + 异常报错**
5. 结果原路返回 AI，自动改错重跑

##### 2. 核心配置

```python
code_execution_config={
    "work_dir":"auto_code",   # 代码存放目录
    "use_docker":False,       # 本地直接运行（通用首选）
    "timeout":60              # 运行超时限制
}
```

##### 3. 能力范围

- 数据分析、可视化绘图
- 爬虫、接口脚本、自动化脚本
- 算法调试、批量处理任务
- 完整项目代码一键运行调试

##### 4. 闭环流程

写代码 → 自动运行 → 捕获错误 → AI 修复 → 再次运行 → 任务完成

---

#### 四、工具调用 Function Call 模式

##### 1. 分层职责

- **AssistantAgent**：声明工具、判断调用时机、生成入参
- **UserProxyAgent**：绑定本地函数、真实执行工具逻辑

##### 2. 双装饰器固定写法（0.75 官方标准）

```python
# 1. 定义本地工具函数
def search_internet(keyword:str)->str:
    return f"联网查询结果：{keyword}最新信息"

# 2. 注册给大模型（让AI知道有这个工具）
@assist.register_for_llm(description="联网搜索实时资讯")
# 3. 注册给执行端（本地真实运行）
@user.register_for_execution()
def search_tool(keyword:str)->str:
    return search_internet(keyword)
```

##### 3. 支持工具类型

- 联网搜索、天气查询、接口请求
- 本地文件读写、数据库操作
- 第三方API调用、企业内部接口
- RAG知识库检索、自定义业务函数

##### 4. 调用逻辑

AI 判断缺外部数据 → 自动发起函数调用 → 执行结果返回 → 整合输出答案

---

#### 五、混合模型部署模式（本地+云端双兼容）

##### 1. 支持模型体系

1. **云端大模型**
GPT系列、通义千问、DeepSeek、讯飞星火等
2. **本地开源模型**
Ollama、Qwen、Llama3、GLM、Phi 等本地私有化部署

##### 2. 0.75 统一配置格式（通用所有模型）

```python
# 云端模型
cloud_llm = {
    "config_list":[{"model":"gpt-4o-mini","api_key":"xxx","base_url":"xxx"}]
}

# 本地Ollama模型
local_llm = {
    "config_list":[{"model":"qwen2:7b","base_url":"http://127.0.0.1:11434/v1","api_key":"none"}]
}
```

##### 3. 两种使用方案

1. **全局统一模型**：所有Agent共用同一个大模型
2. **异构多模型**：思考用云端强模型，执行/简单角色用本地轻量化模型，节省成本

##### 4. 优势

- 无API密钥也可本地免费跑全流程
- 私有化部署，数据不外泄
- 线上线下环境无缝切换

---

#### 五大模式总结对照表

| 工作模式          | 核心组合                  | 核心用途                         |
| ----------------- | ------------------------- | -------------------------------- |
| 一对一自动对话    | UserProxy + Assistant     | 日常问答、简单任务、文案创作     |
| 多Agent群聊       | 多Assistant + GroupChat   | 团队协作、项目开发、方案研讨     |
| 代码自动执行      | 开启code_execution_config | 自动化编程、数据分析、脚本调试   |
| 工具函数调用      | 双装饰器注册函数          | 联网查询、本地能力拓展、业务对接 |
| 本地+云端混合模型 | 统一llm_config适配        | 私有化部署、降本、离线使用       |

需要我给你整理**0.75版本全套可直接运行合集**（一对一、群聊、代码执行、工具调用、本地Ollama五合一）吗？

---

## 四、必须掌握：LLM 配置

0.75 配置方式**完全变了**，这是最容易踩坑的地方！

### 标准配置（OpenAI / 深度求索 / 阿里 / 本地 Ollama 都通用）
```python
llm_config = {
    "config_list": [
        {
            "model": "gpt-4o-mini",  # 可换成 qwen、llama、deepseek-chat
            "api_key": "你的API_KEY",
            "base_url": "https://api.openai.com/v1",  # 反向代理可改
        }
    ],
    "temperature": 0.7,
}
```

---

## 五、AutoGen 0.75 最基础示例（可直接运行）

### 示例 1：一对一自动对话（AI 写代码 + 自动执行）
```python
from autogen import AssistantAgent, UserProxyAgent

# ===================== 1. 配置模型 =====================
llm_config = {
    "config_list": [
        {
            "model": "gpt-4o-mini",
            "api_key": "sk-xxxx",
            "base_url": "https://api.openai.com/v1"
        }
    ],
    "temperature": 0,
}

# ===================== 2. 创建两个 Agent =====================
# AI 助手
assistant = AssistantAgent(
    name="AI助手",
    llm_config=llm_config,
    system_message="你是专业Python工程师，直接输出可运行代码，不要废话。",
)

# 用户代理（自动执行代码，不需要人类输入）
user_proxy = UserProxyAgent(
    name="用户代理",
    human_input_mode="NEVER",  # 关闭人工输入
    code_execution_config={
        "work_dir": "coding",  # 代码保存目录
        "use_docker": False,   # 不使用 Docker（Windows/Mac 直接运行）
    },
)

# ===================== 3. 启动对话 =====================
user_proxy.initiate_chat(
    recipient=assistant,
    message="用Python写一个爬取天气信息的简单代码，使用requests库。",
)
```

**运行效果**：
- AI 自动写代码
- UserProxy 自动执行代码
- 自动报错修正
- 直到任务完成

---

## 六、最强功能：多智能体群聊 GroupChat

0.75 群聊**完全重写**，语法更简洁。

### 示例：4 个智能体自动协作
- 产品经理
- 程序员
- 测试工程师
- 用户代理

```python
from autogen import AssistantAgent, UserProxyAgent, GroupChat, GroupChatManager

# 1. 模型配置
llm_config = {
    "config_list": [{"model": "gpt-4o-mini", "api_key": "sk-xxx"}],
    "temperature": 0.1,
}

# 2. 创建多个角色 Agent
pm = AssistantAgent(name="产品经理", llm_config=llm_config, system_message="负责需求分析、方案设计。")
dev = AssistantAgent(name="程序员", llm_config=llm_config, system_message="负责写代码、实现功能。")
tester = AssistantAgent(name="测试", llm_config=llm_config, system_message="负责测试代码、找Bug。")

user_proxy = UserProxyAgent(
    name="用户代理",
    human_input_mode="NEVER",
    code_execution_config={"work_dir": "project", "use_docker": False},
)

# 3. 创建群聊
group_chat = GroupChat(
    agents=[user_proxy, pm, dev, tester],
    max_round=15,  # 最大对话轮数
)

# 4. 创建群聊管理器
manager = GroupChatManager(
    groupchat=group_chat,
    llm_config=llm_config,
)

# 5. 启动群聊
user_proxy.initiate_chat(
    manager,
    message="我要做一个待办事项（TODO）工具，用Python实现。",
)
```

**运行后**：
产品经理 → 程序员 → 测试 → 用户代理
自动对话、自动写代码、自动测试、自动完成项目。

---

## 七、 工具调用（Function Call）

0.75 工具调用**更简单、更稳定**。

```python
from autogen import AssistantAgent, UserProxyAgent
from typing import Annotated

# 自定义工具
def get_weather(city: Annotated[str, "城市名"]) -> str:
    return f"{city} 当前天气：晴天，25℃"

# 配置
llm_config = {"config_list": [{"model": "gpt-4o-mini", "api_key": "sk-xxx"}]}

# AI 助手
assistant = AssistantAgent(name="助手", llm_config=llm_config)

# 用户代理（注册工具）
user_proxy = UserProxyAgent(
    name="用户代理",
    human_input_mode="NEVER",
    code_execution_config=False,
)

# 注册工具
@user_proxy.register_for_execution()
@assistant.register_for_llm(description="获取城市天气")
def call_weather(city: str) -> str:
    return get_weather(city)

# 启动对话
user_proxy.initiate_chat(
    assistant,
    message="北京天气怎么样？",
)
```

---

## 八、常用关键参数（必须懂）

### 1. UserProxyAgent 参数
```python
human_input_mode="NEVER"   # 完全自动
# "ALWAYS" 每次都问你
# "TERMINATE" 结束时问你

code_execution_config={
    "work_dir": "输出目录",
    "use_docker": False,   # 关闭 Docker（最常用）
}
```

### 2. GroupChat 参数
```python
max_round=10        # 最大对话轮数
speaker_selection="auto"  # 自动选下一个发言者
```

---

## 九、0.75 版本 vs 老版本（关键区别）
| 功能      | 0.2 / 0.4 | **0.75**               |
| --------- | --------- | ---------------------- |
| 配置方式  | 旧版      | **全新统一配置**       |
| GroupChat | 复杂      | **全新简洁 API**       |
| 工具调用  | 不稳定    | **稳定、标准化**       |
| 代码执行  | 易报错    | **更稳定**             |
| 兼容性    | 低        | **官方未来唯一维护版** |

---

## 十、最佳实践（0.75 必看）

1. **一定用 0.75+**，不要看老教程
2. 关闭 Docker：`use_docker=False`
3. 自动运行：`human_input_mode="NEVER"`
4. 群聊不要超过 5 个 Agent，避免混乱
5. 给每个 Agent 清晰的**system_message**

## AutoGen 的优势与局限性分析

任何技术框架都有其特定的适用场景和设计权衡。在本节中，我们将客观地分析 AutoGen 的核心优势及其在实际应用中可能面临的局限性与挑战。

### （1）优势

- 如案例所示，我们无需为智能体团队设计复杂的状态机或控制流逻辑，而是将一个完整的软件开发流程，自然地映射为产品经理、工程师和审查员之间的对话。这种方式更贴近人类团队的协作模式，显著降低了为复杂任务建模的门槛。开发者可以将更多精力聚焦于定义“谁（角色）”以及“做什么（职责）”，而非“如何做（流程控制）”。
- 框架允许通过系统消息（System Message）为每个智能体赋予高度专业化的角色。在案例中，`ProductManager` 专注于需求，而 `CodeReviewer` 则专注于质量。一个精心设计的智能体可以在不同项目中被复用，易于维护和扩展。
- 对于流程化任务，`RoundRobinGroupChat` 这样机制提供了清晰、可预测的协作流程。同时，`UserProxyAgent` 的设计为“人类在环”（Human-in-the-loop）提供了天然的接口。它既可以作为任务的发起者，也可以是流程的监督者和最终的验收者。这种设计确保了自动化系统始终处于人类的监督之下。

### （2）局限性

- 虽然 `RoundRobinGroupChat` 提供了顺序化的流程，但基于 LLM 的对话本质上具有不确定性。智能体可能会产生偏离预期的回复，导致对话走向意外的分支，甚至陷入循环。
- 当智能体团队的工作结果未达预期时，调试过程可能非常棘手。与传统程序不同，我们得到的不是清晰的错误堆栈，而是一长串的对话历史。这被称为“对话式调试”的难题。

### （3）非 OpenAI 模型的配置补充

如果你想使用非 OpenAI 系列的模型（如 DeepSeek、通义千问等），在 0.7.4 版本中需要在 `OpenAIChatCompletionClient` 的参数中传入模型信息字典。以 DeepSeek 为例：

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    model_info={
        "function_calling": True,
        "max_tokens": 4096,
        "context_length": 32768,
        "vision": False,
        "json_output": True,
        "family": "deepseek",
        "structured_output": True,
    }
)Copy to clipboardErrorCopied
```

这个 `model_info` 字典帮助 AutoGen 了解模型的能力边界，从而更好地适配不同的模型服务。