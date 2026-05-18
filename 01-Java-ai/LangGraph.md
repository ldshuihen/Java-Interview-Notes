# **LangGraph**

LangGraph 是 LangChain 生态的**状态化多智能体编排框架**，核心是用**有向循环图（DCG）**建模工作流，支持**状态持久化、循环迭代、条件分支与多Agent协作**，解决传统Chain线性、无状态、难回溯的痛点。下面从**核心组件、图类型、工作流模式、高级特性**四大维度系统分类介绍。

---

## 一、核心组件分类（四大基石）

### 1. State（状态）：全局共享内存

- **定义**：所有节点读写的**唯一数据源**，强类型约束（TypedDict/Pydantic）。
- **核心特性**：
  - **不可变更新**：每次生成新状态，支持**回溯（Time Travel）**。
  - **Reducer（归约器）**：控制状态合并逻辑（覆盖/追加/消息专用）。
  - **预构建State**：`MessagesState`（内置`messages`字段与消息Reducer）。
- **常用字段**：`messages`（对话历史）、`tool_calls`（工具记录）、`status`（执行状态）。

---

#### 一、State 到底是什么？

一句话：
**State = 整个图的全局共享记事本（唯一数据源）**

- 所有节点（LLM、工具、路由、人类输入）**都只能读写这个 State**
- 节点之间**不直接通信**，全部通过 State 传递数据
- 每执行一步，State 就更新一次
- 整个流程的**历史、输入、输出、记忆、工具结果**全都存在这里

它就是：
**多轮对话的记忆 + 工作流的上下文 + 节点间的通信桥梁**

---

#### 二、State 三大核心特性（必考必懂）

##### 1. 不可变更新（Immutable）

- 每次节点执行完，**不是修改旧状态**
- 而是**创建一个新状态**替换旧的
- 旧状态依然保留 → 支持**时间回溯（Time Travel）**

好处：
- 可回到任意一步重新跑
- 调试超级方便
- 不会因为中途出错污染状态

##### 2. Reducer 归约器（最关键！）

**控制状态怎么更新：覆盖？追加？合并？**

LangGraph 默认提供 2 种最常用 reducer：

###### ① `append` 追加（对话消息必备）

```
旧消息：[A,B,C]
新消息：[D]
新状态：[A,B,C,D]
```
用于：对话历史、工具调用记录

###### ② `overwrite` 覆盖（默认）

```
旧值：18
新值：20
新状态：20
```
用于：单值变量（当前阶段、分数、状态）

##### 3. 强类型（TypedDict / Pydantic）

必须提前定义：
- 状态里有哪些字段
- 每个字段是什么类型
- 用什么归约器

**结构清晰、不会乱、不会错**

---

#### 三、最常用的 State：MessagesState（官方内置）

##### 作用：**标准对话状态**

自带一个字段：
**`messages: list[BaseMessage]`**

并且自带：
**`reducer=messages_state_reducer` → 自动追加对话历史**

所以你不用自己写，直接用：
```python
from langgraph.graph import MessagesState
```

它就是：
**多轮聊天 + 工具调用 + 记忆** 的标准状态。

---

#### 四、自定义 State 完整写法（必须掌握）

##### 方式1：TypedDict（最常用）

```python
from typing import TypedDict, Annotated
from langgraph.graph import add_messages

class MyState(TypedDict):
    # 消息列表：自动追加
    messages: Annotated[list, add_messages]

    # 普通字段：覆盖模式
    query: str
    result: str
    is_finish: bool
```

###### 关键语法解释

```
Annotated[list, add_messages]
```
= 列表类型 + 使用追加归约器
= **新消息不会覆盖旧消息，只会往后加**

---

#### 五、State 如何在节点中使用？

##### 所有节点都接收 state，返回新 state

```python
def llm_node(state: MyState):
    # 从 state 读取数据
    history = state["messages"]

    # 调用模型
    response = model.invoke(history)

    # 返回新状态（自动追加）
    return {"messages": [response]}
```

##### 规则：

- 你返回什么字段，就更新什么字段
- 没返回的保持不变
- `messages` 因为是追加模式，所以直接返回新消息列表即可

---

#### 六、State 核心工作流程（超级重要）

```
用户输入 → 写入 State
LLM节点 → 读取 State → 生成回答 → 写入 State
工具节点 → 读取 State → 调用工具 → 写入结果
路由节点 → 读取 State → 判断下一步
...
直到结束
```

**整个图的灵魂就是 State 在流动！**

---

#### 七、State 常用字段总结

```python
state = {
    "messages": [],           # 对话历史（必备）
    "query": "",              # 用户问题
    "context": [],            # RAG检索内容
    "tool_results": [],        # 工具调用结果
    "is_need_tool": False,    # 路由判断
    "status": "success",      # 执行状态
}
```

---

#### 八、一句话终极总结（背下来就懂了）

##### **State = 全局共享内存 + 对话记忆 + 节点通信 + 工作流上下文**

##### **Reducer = 控制状态是追加还是覆盖**

##### **MessagesState = 开箱即用的对话状态（90%场景直接用）**

---

#### 九、面试/考试必考点

1. **State 作用？**
全局共享状态，所有节点通过它通信、存储记忆、传递上下文。

2. **什么是 Reducer？**
控制状态更新规则：追加（对话）或覆盖（普通值）。

3. **MessagesState 是什么？**
内置对话状态，自动维护消息列表，适合聊天、Agent、工具调用。

4. **为什么状态不可变？**
保留历史，支持回溯、调试、断点续跑。

5. **节点之间怎么通信？**
不直接通信，全部通过读写 State。

---

#### 🔥 你已经掌握 LangGraph 最核心灵魂！

接下来我可以带你：
- **快速搭建一个带记忆的对话图**
- **搭建一个带工具调用的 Agent**
- **搭建一个带 RAG 的检索问答图**
- **搭建多智能体协作图**

你想继续学哪一个？我直接给你**可运行代码**！



### 2. Node（节点）：执行单元（函数）

- **定义**：接收State→执行逻辑→返回新State的同步/异步函数。
- **按功能分类**：
  - **LLM节点**：调用大模型（思考/回答/决策）。
  - **Tool节点**：调用外部工具（搜索/API/计算器）。
  - **Router节点**：条件判断，决定下一步流向。
  - **Human节点**：暂停流程，等待人工输入/审核。
  - **Subgraph节点**：嵌套子图，复杂流程模块化。

**Node 就是图里干活的“员工/任务单元”**
整个图的逻辑，全是靠一个个 Node 执行出来的。

我用**最简单、最清晰、最实战**的方式给你讲透👇

---

#### 一、Node 到底是什么？

**一句话：Node = 接收 State → 做事 → 返回新 State 的函数**

它是 LangGraph 里**真正执行逻辑**的地方：
- 调用 LLM
- 调用工具
- 条件判断
- 数据处理
- 等待人工输入
- 执行子流程

所有功能都靠 Node 实现。

---

#### 二、Node 的统一规则（必须记住）

1. **必须接收 state 作为参数**
2. **必须返回字典形式的新状态 `{key: value}`**
3. 不能直接修改 state，只能返回更新
4. 执行完自动把结果合并到全局 State 里

**标准格式：**
```python
def 节点名(state: 你的State):
    # 1. 从 state 拿数据
    数据 = state["xxx"]
    
    # 2. 执行逻辑（调用模型/工具/计算）
    
    # 3. 返回要更新的字段
    return {"xxx": 新数据}
```

---

#### 三、5 类最常用 Node（实战全覆盖）

##### 1. LLM 节点（最常用）

**调用大模型，生成回答/思考/决策**
```python
def llm_node(state):
    messages = state["messages"]
    response = llm.invoke(messages)  # 调用模型
    return {"messages": [response]}  # 追加新消息
```

---

##### 2. Tool 节点（工具执行）

**执行函数/搜索/计算器/API/数据库**
由模型决定是否调用。
```python
def tool_node(state):
    last_msg = state["messages"][-1]
    tool_results = tool.execute(last_msg)  # 执行工具
    return {"messages": tool_results}
```

---

##### 3. Router 节点（路由/条件分支）

**不做业务逻辑，只做一件事：下一步去哪？**
返回值是**下一个节点的名字**。
```python
def router_node(state):
    last_msg = state["messages"][-1]
    if last_msg.tool_calls:
        return "tool"  # 去工具节点
    else:
        return "end"   # 结束
```

---

##### 4. Human-in-the-Loop 节点（人工介入）

**暂停流程，等待人审核/输入/确认**
企业级工作流必备。
```python
def human_approval_node(state):
    print("等待人工审核...")
    input("按回车继续")
    return {"status": "approved"}
```

---

##### 5. Subgraph 节点（子图节点）

**把复杂流程打包成一个节点，实现模块化**
大图套小图，代码更干净。
```python
subgraph_node = subgraph.compile()  # 子图变成一个节点
```

---

#### 四、Node 如何注册到图里？

```python
from langgraph.graph import StateGraph

builder = StateGraph(MyState)

# 添加节点
builder.add_node("llm", llm_node)
builder.add_node("tool", tool_node)

# 连接边
builder.add_edge("llm", "tool")
```

---

#### 五、Node 执行特点

1. **每次只跑一个节点**
2. 跑完根据边（Edge）跳到下一个
3. 支持**循环**：A → B → A → B…
4. 支持**条件跳转**（Router）
5. 支持**中断继续**（Human Node）

---

#### 六、一句话终极总结（背会就懂）

##### **Node = 图里干活的最小单元**

##### **所有功能 = 一堆 Node 按顺序执行**

##### **LLM/工具/路由/人工/子图 都是 Node**

---

#### 七、你已经掌握 LangGraph 第二核心！

现在你可以理解：
- 图怎么跑
- 谁在干活
- 数据怎么流动

---

#### 我可以马上带你写 **完整可运行的图**

你想做哪个？
1. **对话机器人图**
2. **带工具调用的 Agent**
3. **带 RAG 检索的问答图**
4. **带路由分支的流程图**
5. **带人工审核的企业级图**

你说一声，我直接给你**能跑的完整代码**！



### 3. Edge（边）：流程控制器

- **定义**：连接节点的**有向跳转规则**，分两类：
  - **普通边（Normal Edge）**：无条件流转（A→B）。
  - **条件边（Conditional Edge）**：基于State动态分支（如“是否需要工具？”）。

#### 核心定义

**Edge 是连接各个 Node 的通路，用来规定节点之间的执行流向**，决定走完当前节点后，下一步跳转到哪个节点，是整个工作流的**流程调度核心**。
所有数据流转、分支判断、循环执行，全靠边来控制。

#### 两大分类

##### 一、普通边 Normal Edge

1. **特点**
    - 无条件固定流转，路径写死
    - 执行完A节点，**必然直接走到B节点**，没有任何判断
    - 结构简单，适合线性顺序流程
2. **适用场景**
    固定步骤流水线、串行执行任务
3. **代码用法**
```python
# 语法：add_edge(起点节点名, 终点节点名)
builder.add_edge("llm思考节点", "结果整理节点")
```
4. **流向示例**
`开始 → LLM节点 → 工具节点 → 结束`

---

##### 二、条件边 Conditional Edge

1. **特点**
    - 基于**全局State状态**做逻辑判断，动态选择下一步路径
    - 同一个节点执行完成后，可以走向多个不同分支
    - 是实现**分支、路由、循环**的核心
2. **执行逻辑**
    读取State里的字段/消息内容 → 自定义判断逻辑 → 返回目标节点名称 → 自动跳转
3. **适用场景**
    意图分发、是否调用工具、循环重试、流程分流
4. **代码用法**
```python
# 1. 定义路由判断函数（返回下一个节点名称）
def route_decision(state):
    # 读取状态做判断
    has_tool_call = bool(state["messages"][-1].tool_calls)
    if has_tool_call:
        return "go_tool"  # 走工具分支
    else:
        return "go_end"   # 直接结束对话

# 2. 绑定条件边
builder.add_conditional_edges(
    source="llm节点",       # 判断起点
    path=route_decision,    # 判断逻辑函数
    path_map={              # 映射：函数返回值 → 真实节点名
        "go_tool": "tool执行节点",
        "go_end": "结束节点"
    }
)
```
5. **经典流向示例**
LLM决策节点 → 判断是否需要调用工具 → 是→走工具节点 / 否→直接结束

---

#### 三、Edge 核心作用

1. **串联流程**：把零散独立的Node拼接成完整业务流程
2. **分支分流**：实现多路径业务逻辑，适配不同用户请求
3. **构建循环**：搭配条件边实现`思考-执行-再思考`Agent循环
4. **终止流程**：指向内置结束标记`END`，终止整个图运行

#### 四、常用固定结束节点

LangGraph内置终止常量，所有边指向它即可结束流程
```python
from langgraph.graph import END
builder.add_edge("最终回复节点", END)
```

#### 五、普通边 vs 条件边 对比

| 类型   | 流转规则          | 灵活性 | 用途                       |
| ------ | ----------------- | ------ | -------------------------- |
| 普通边 | 固定单向流转      | 低     | 串行固定流程               |
| 条件边 | 依据State动态跳转 | 极高   | 分支、路由、循环、智能决策 |

#### 六、面试核心考点

1. Edge的作用是什么？
连接节点，控制工作流执行顺序与跳转逻辑。
2. 普通边和条件边区别？
普通边无条件固定跳转；条件边根据全局State状态动态选择执行分支。
3. 如何实现Agent循环调用？
利用**条件边**判断工具调用结果，满足条件重新回到LLM节点，形成闭环循环。
4. 怎么终止LangGraph流程？
将边指向内置常量`END`即可终止运行。

### 4. Graph（图）：编排容器

- **基础图**：`StateGraph`（核心类，绑定State，管理节点/边）。
- **编译图**：`graph.compile()`生成可执行Runnable，支持流式输出、回调、持久化。

这是**把 State、Node、Edge 组装成一个可运行 AI 工作流的总容器**！

我用**最清晰、最实战、最容易记住**的方式给你讲透👇

---

#### 一、Graph 到底是什么？

一句话：
**Graph = 总编排器 + 流程图本体**

它负责：
1. 绑定你定义的 **State（状态）**
2. 注册所有 **Node（节点）**
3. 连接所有 **Edge（边）**
4. 设置 **START 起点** 和 **END 终点**
5. 最后 **compile() 编译成可运行程序**

它就是你整个 **AI 智能体/工作流的本体**。

---

#### 二、Graph 的两个阶段（必须掌握）

##### 1. 构建图：StateGraph（蓝图）

```python
from langgraph.graph import StateGraph

# 1. 创建构建器（绑定 State）
builder = StateGraph(MyState)
```

作用：
- 只是**画蓝图**
- 还不能运行
- 用来添加节点、添加边、设置起点

##### 2. 编译图：graph = builder.compile()（可执行）

```python
# 2. 编译 → 生成真正可运行的图
graph = builder.compile()
```

作用：
- 变成可调用对象（Runnable）
- 支持 `invoke` / `stream` / `batch`
- 支持调试、日志、持久化、断点
- **真正能跑的版本**

---

#### 三、Graph 完整构建四步公式（背下来）

所有 LangGraph 程序，**永远都是这 4 步**：

##### ① 创建构建器

```python
builder = StateGraph(MyState)
```

##### ② 添加节点

```python
builder.add_node("llm", llm_node)
builder.add_node("tool", tool_node)
```

##### ③ 连接边（流程）

```python
builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", route_func)
builder.add_edge("tool", "llm")
```

##### ④ 编译

```python
graph = builder.compile()
```

**完成！这就是一个完整 Agent！**

---

#### 四、Graph 核心能力（超级重要）

##### 1. 绑定 State（全局状态）

- 所有 Node 共享同一个 State
- 消息、工具结果、上下文、记忆 全部存在这里

##### 2. 管理 Node（执行单元）

- 统一注册
- 按流程调度
- 按顺序/按条件执行

##### 3. 调度 Edge（流向控制）

- 线性执行
- 条件分支
- **循环执行（Agent 核心）**
- 终止流程

##### 4. 提供运行接口（编译后）

编译后的图可以直接运行：
```python
# 运行一次
graph.invoke({"messages": [HumanMessage(content="你好")]})

# 流式输出（打字机效果）
for chunk in graph.stream(...):
    print(chunk)
```

---

#### 五、最精简可运行模板（必收藏）

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState

# 1. 定义 State
class State(MessagesState):
    pass

# 2. 定义 Node
def llm_node(state):
    return {"messages": ["我是AI助手"]}

# 3. 构建 Graph
builder = StateGraph(State)
builder.add_node("llm", llm_node)
builder.add_edge(START, "llm")
builder.add_edge("llm", END)

# 4. 编译
graph = builder.compile()

# 运行
print(graph.invoke({"messages": []}))
```

---

#### 六、一句话终极总结

##### **Graph = 编排总引擎**

##### **StateGraph = 画蓝图**

##### **compile() = 生成可运行AI工作流**

##### **所有Agent、流程图、多智能体 → 都是一张Graph**

---

#### 🔥 你已经学完 LangGraph 四大核心！

✅ **State**：全局内存
✅ **Node**：执行单元
✅ **Edge**：流程控制
✅ **Graph**：总编排器

---

#### 我现在可以直接给你：

##### **一套完整可运行的 LangGraph Agent（带工具调用+记忆+循环）**

##### 复制就能跑，界面用 Streamlit 展示！

你要我现在直接给你吗？



---

## 二、图的类型分类（按复杂度）

### 1. 基础图（Basic Graph）

- **结构**：线性DAG（有向无环图），节点顺序执行。
- **适用**：简单流水线（如RAG：加载→分割→检索→生成）。
- **示例**：`A → B → C → END`

#### 核心定义

由**有向无环图 DAG**构成，节点按固定先后顺序串行执行，**无分支、无循环**，走完既定流程直接结束。

#### 结构特点

1. 流向唯一，从上到下单向流转
2. 流程预先写死，运行中不会改变执行路径
3. 逻辑简单、易调试、执行效率高
4. 不支持回头重试、条件跳转

#### 适用场景

- 固定步骤流水线任务
- 标准RAG问答：文档加载 → 文本分割 → 向量入库 → 语义检索 → 大模型生成答案
- 内容批量处理：翻译、摘要、格式统一转换
- 表单信息提取、固定模板文案生成

#### 流转示例

```
START → A节点 → B节点 → C节点 → END
```

#### 极简代码实现

```python
from langgraph.graph import StateGraph,START,END
from langgraph.graph import MessagesState

# 定义状态
class GraphState(MessagesState):
    pass

# 定义串行节点
def node_a(state):
    return {"messages":["执行步骤A"]}

def node_b(state):
    return {"messages":["执行步骤B"]}

def node_c(state):
    return {"messages":["执行步骤C"]}

# 构建基础线性图
builder = StateGraph(GraphState)
builder.add_node("A",node_a)
builder.add_node("B",node_b)
builder.add_node("C",node_c)

# 固定线性连线
builder.add_edge(START,"A")
builder.add_edge("A","B")
builder.add_edge("B","C")
builder.add_edge("C",END)

graph = builder.compile()

# 执行
res = graph.invoke({"messages":[]})
print(res)
```

#### 优缺点

- 优点：结构清晰、开发快、维护简单、无死循环风险
- 缺点：灵活性差，无法应对动态需求、不能自主决策

#### 实战定位

LangGraph 入门首选结构，**所有复杂图都是在基础线性图之上扩展分支与循环**。

### 2. 循环图（Cyclic Graph）

- **结构**：支持**闭环**（核心优势），节点可重复执行直到满足退出条件。
- **适用**：需要迭代优化的任务（如Agent“思考→工具→观察→再思考”循环）。
- **示例**：`START → LLM → Tool → LLM → ... → END`

这是 **LangGraph 最强大、最核心、Agent 必用** 的图结构！
也是它比传统 LangChain 强一万倍的原因！

我用**最通俗、最实战、最容易理解**的方式给你讲透👇

---

#### 一、循环图到底是什么？

一句话：
**能走“回头路”，形成闭环，反复执行，直到条件满足才结束的图**

基础图是：
`A→B→C→END`（一条路走到黑）

循环图是：
`A→B→A→B→A→B……直到满足条件才结束`

**这就是 Agent（智能体）的灵魂！**

---

#### 二、核心特点

1. **支持闭环（回头路）**
   节点可以跳回之前的节点，形成循环
2. **动态退出**
   不是固定次数，而是**看 State 状态决定是否继续循环**
3. **Agent 标准结构**
   `思考 → 行动 → 观察 → 再思考 → 再行动……`
4. **无限迭代（可控）**
   可设置最大轮次，防止死循环

---

#### 三、标准循环图结构（Agent 专用）

```
START → LLM（思考/决策）→ 路由判断
               ↙        ↖
          工具执行       结束
               ↘        ↙
                    END
```

也就是：
**LLM → 工具 → LLM → 工具 → … → 结束**

---

#### 四、为什么循环图是 Agent 核心？

因为真实世界任务**不是一次就能完成**的：
- 要先联网搜索 → 再计算 → 再查资料 → 最后总结
- 要写代码 → 跑代码 → 报错 → 修改 → 再跑
- 要多工具调用、多步骤推理

**必须循环迭代！**

传统 Chain 做不到，**只有 LangGraph 循环图能做到**。

---

#### 五、最经典实战代码（可直接运行）

##### 实现：LLM → 工具 → LLM 循环

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

# 1. 定义状态（自带消息追加）
class State(MessagesState):
    pass

# 2. 定义工具
@tool
def calculator(a: int, b: int) -> int:
    """计算器 a + b"""
    return a + b

llm = ChatOpenAI(model="gpt-3.5-turbo").bind_tools([calculator])

# 3. 定义节点
def llm_node(state: State):
    return {"messages": [llm.invoke(state["messages"])]}

def tool_node(state: State):
    # 执行工具并返回结果
    last = state["messages"][-1]
    result = calculator.invoke(last.tool_calls[0]["function"]["arguments"])
    return {"messages": [f"工具结果：{result}"]}

# 4. 路由函数（循环核心！）
def should_continue(state):
    last_msg = state["messages"][-1]
    # 如果有工具调用 → 回到工具节点（循环）
    if hasattr(last_msg, "tool_calls") and last_msg.tool_calls:
        return "tool"
    # 否则结束
    return END

# ========================
# 构建循环图（重点看这里）
# ========================
builder = StateGraph(State)

builder.add_node("llm", llm_node)
builder.add_node("tool", tool_node)

# 起点 → LLM
builder.add_edge(START, "llm")

# 条件边：实现循环！
builder.add_conditional_edges(
    "llm",
    should_continue,
    path_map={
        "tool": "tool",  # 走工具 → 然后会回到 LLM（形成循环）
        END: END         # 结束
    }
)

# 工具执行完 → 回到 LLM（闭环！循环关键）
builder.add_edge("tool", "llm")

# 编译
graph = builder.compile()
```

##### 执行流程（循环形成）

1. START → llm节点
2. llm 判断需要工具 → 路由到 tool
3. tool 执行完 → **回到 llm（形成循环）**
4. llm 再判断 → 还要工具？继续循环
5. 不需要了 → 结束

这就是**标准 ReAct 智能体**！

---

#### 六、一句话终极总结

##### **循环图 = 带回头路的流程图 = Agent 智能体的底层架构**

##### **条件边 + 回头边 = 循环**

##### **LLM → 工具 → LLM → 工具…… 就是最强 AI 工作模式**

---

#### 七、你已经掌握 LangGraph 最强能力！

现在你彻底理解：
- 基础图 = 线性流水线
- **循环图 = AI 自主思考 + 工具调用（Agent）**

---

#### 我可以直接送你：

##### **一套完整可运行的 LangGraph + Streamlit 智能体项目**

包含：
- 循环图
- 工具调用
- 多轮记忆
- 流式输出
- 前端界面

你要我现在直接发给你吗？



### 3. 子图（Subgraph）

- **结构**：大图嵌套小图，**模块化拆分复杂逻辑**，可复用。
- **适用**：多模块协作（如内容创作：研究子图→写作子图→审核子图）。

#### 核心定义

将**独立完整的小型Graph**封装成一个普通节点，嵌入到大图中运行，实现**大图套小图**的分层编排，逻辑解耦、代码复用。

#### 结构特点

1. 层级嵌套：主图统一调度，内部业务逻辑由子图独立完成
2. 隔离性强：子图拥有自己独立的节点、边、局部流转，不污染主图逻辑
3. 可复用：同一张子图能被多个主图直接调用，减少重复开发
4. 易维护：复杂业务按功能拆分成多个子图，各司其职

#### 适用场景

- 大型复杂工作流拆分：内容创作（调研子图 → 撰写子图 → 审核子图）
- 业务模块解耦：RAG检索子图、数据清洗子图、结果格式化子图
- 功能复用：通用审批流程、通用数据解析流程多处复用
- 团队协作开发：不同成员独立开发不同子图，最后统一拼装

#### 流转示例

```
START → 研究子图 → 写作子图 → 审核子图 → END
每个子图内部可包含线性流程、分支、小型循环
```

#### 核心使用流程

1. 独立编写并编译**子图**
2. 将编译后的子图，通过 `add_node` 注册为主图的一个普通节点
3. 主图正常配置边，调度子图执行
4. 子图执行完毕自动回流主图继续往下走

#### 极简代码示例

```python
from langgraph.graph import StateGraph, START, END, MessagesState

# 全局统一状态
class AllState(MessagesState):
    pass

# ========== 1. 构建第一个子图：调研子图 ==========
def research_node(state):
    return {"messages":["完成资料调研"]}

sub_builder1 = StateGraph(AllState)
sub_builder1.add_node("research", research_node)
sub_builder1.add_edge(START, "research")
sub_builder1.add_edge("research", END)
research_subgraph = sub_builder1.compile()

# ========== 2. 构建第二个子图：写作文本子图 ==========
def write_node(state):
    return {"messages":["完成文案撰写"]}

sub_builder2 = StateGraph(AllState)
sub_builder2.add_node("write", write_node)
sub_builder2.add_edge(START, "write")
sub_builder2.add_edge("write", END)
write_subgraph = sub_builder2.compile()

# ========== 3. 主图拼装所有子图 ==========
main_builder = StateGraph(AllState)
# 把子图当作普通节点添加
main_builder.add_node("research_task", research_subgraph)
main_builder.add_node("write_task", write_subgraph)

# 主图线性调度
main_builder.add_edge(START, "research_task")
main_builder.add_edge("research_task", "write_task")
main_builder.add_edge("write_task", END)

main_graph = main_builder.compile()

# 运行
res = main_graph.invoke({"messages":[]})
print(res)
```

#### 优缺点

- 优点：逻辑分层清晰、代码高度复用、业务解耦、便于迭代维护
- 缺点：层级过多会增加调用链路复杂度，调试链路变长

#### 实战定位

中小型业务用基础图、循环图；**企业级大型AI工作流、多模块组合业务，优先用子图拆分**，是项目工程化必备写法。

### 4. 多智能体图（Multi-Agent Graph）

- **结构**：多个独立Agent（子图）通过共享State协作，支持**并行/串行/层级调度**。
- **适用**：复杂团队任务（如“产品→研发→测试→部署”多角色协作）。

#### 核心定义

把**多个独立智能体（各自为完整子图）** 编排在一起，共用一份全局`State`实现数据互通、分工协作，模拟多人团队协同办公，是 LangGraph 最高阶工作流架构。

#### 结构特点

1. **角色独立**：每个Agent拥有专属Prompt、专属能力、专属执行逻辑，各司其职
2. **状态共享**：所有智能体读写同一全局State，任务成果互相传递
3. **调度灵活**：支持串行依次执行、并行同时执行、层级上下级调度
4. **自主交接**：可通过路由自动判定由哪个Agent接手任务，实现任务流转

#### 适用场景

- 团队式复杂任务：产品策划→代码开发→代码测试→文档整理
- 领域分工协作：检索Agent+推理Agent+总结Agent+格式化Agent
- 分工式问答：意图识别Agent+专业解答Agent+兜底闲聊Agent
- 长流程复杂业务流水线、企业级复杂AI系统

#### 流转逻辑

全局State统一存放任务需求、中间结果、最终产出
AgentA完成工作写入状态 → 路由分发 → AgentB读取状态继续处理 → 依次流转直至全部完成

#### 核心协作模式

1. **串行协作**：按固定顺序依次交接执行
2. **并行协作**：多个Agent同时开工，完成后合并结果
3. **竞选协作**：根据用户需求自动匹配最合适的Agent执行
4. **上下级协作**：总指挥Agent分配任务，执行Agent落地完成

#### 极简实战示例

```python
from langgraph.graph import StateGraph,START,END,MessagesState

# 全局共享状态
class TeamState(MessagesState):
    task_content: str
    work_result: str

# 角色1：产品Agent
def product_agent(state:TeamState):
    return {"work_result":"产品已梳理需求与方案"}

# 角色2：研发Agent
def dev_agent(state:TeamState):
    return {"work_result":"根据方案完成功能开发"}

# 角色3：测试Agent
def test_agent(state:TeamState):
    return {"work_result":"完成全流程测试，验收通过"}

# 搭建多智能体主图
builder = StateGraph(TeamState)
builder.add_node("product",product_agent)
builder.add_node("develop",dev_agent)
builder.add_node("test",test_agent)

# 串行团队协作流程
builder.add_edge(START,"product")
builder.add_edge("product","develop")
builder.add_edge("develop","test")
builder.add_edge("test",END)

team_graph = builder.compile()

# 执行团队协同任务
res = team_graph.invoke({"task_content":"开发后端接口系统"})
print(res)
```

#### 与普通单Agent区别

| 类型          | 执行主体               | 能力范围               | 适用场景                           |
| ------------- | ---------------------- | ---------------------- | ---------------------------------- |
| 单Agent循环图 | 一个智能体自主调用工具 | 全能通用，自主决策     | 日常问答、工具调用、简单任务       |
| 多智能体图    | 多个专属角色Agent协同  | 分工明确，专精单一领域 | 大型复杂流程、团队化作业、分层业务 |

#### 优缺点

- 优点：任务拆分清晰、专业能力最大化、易拓展新增角色、贴合真实业务团队流程
- 缺点：架构复杂度高、状态流转调试成本更高、需要设计合理任务交接逻辑

#### 实战定位

单智能体解决**单点问题**，多智能体图解决**系统性复杂工程问题**，是搭建企业级大型AI应用、智能工作流平台的核心架构。

---

## 三、工作流模式分类（四大经典模式）

### 1. 链式模式（Chain）

- **特点**：线性顺序，无分支无循环，最简单。
- **场景**：固定流程（如文档摘要：加载→分割→总结）。

#### 核心特点

1. 全程**线性串行**执行，节点按预设顺序依次运行
2. 无条件分支、无闭环循环，流程写死不可动态变更
3. 结构极简、逻辑直观、上手成本低
4. 执行路径唯一，执行完毕直接结束流程

#### 运行流程

`起始节点 → 节点1 → 节点2 → 节点3 → 结束`
全程单向流转，不会跳转回溯、不会分流走不同路径

#### 适用业务场景

- 文档标准化处理：加载文档 → 文本切块 → 内容摘要
- 批量数据流水线：数据读取 → 清洗过滤 → 格式转换 → 入库存储
- 固定话术生成、统一模板填充
- 简易RAG标准流程：加载文档→切片→向量化→检索→答案生成

#### 代码极简示例

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState

# 定义状态
class WorkState(MessagesState):
    pass

# 流程节点
def load_doc(state):
    return {"messages":["完成文档加载"]}

def split_text(state):
    return {"messages":["完成文本分割"]}

def summary_text(state):
    return {"messages":["完成内容摘要"]}

# 构建链式流程图
builder = StateGraph(WorkState)
builder.add_node("加载文档", load_doc)
builder.add_node("文本分割", split_text)
builder.add_node("内容总结", summary_text)

# 线性串联
builder.add_edge(START, "加载文档")
builder.add_edge("加载文档", "文本分割")
builder.add_edge("文本分割", "内容总结")
builder.add_edge("内容总结", END)

chain_graph = builder.compile()

# 执行
result = chain_graph.invoke({"messages":[]})
print(result)
```

#### 优缺点

- 优点：开发快、易调试、稳定性高、无死循环风险
- 缺点：灵活性极差，无法适配动态需求、不能自主决策

#### 实战定位

LangGraph最基础的工作流模式，作为复杂流程的底层基础，**固定不变的标准化流程优先使用链式模式**。



### 2. 路由模式（Router）

- **特点**：**条件分支**，根据State选择不同路径。
- **场景**：多任务分发（如用户意图识别：闲聊→LLM；查天气→工具）。

#### 核心特点

1. 依靠**全局State状态**做逻辑判断，实现**条件分支分流**
2. 同一入口，不同请求走不同执行链路，流程不固定
3. 核心依靠**条件边**实现路径选择
4. 可灵活分发任务、区分业务走向

#### 执行逻辑

用户输入/状态数据 → 路由判定函数识别意图/条件 → 匹配对应分支节点执行 → 各自走完流程统一收尾

#### 适用场景

- 用户意图分发：闲聊对话走普通LLM、查数据走工具调用、知识库问答走RAG
- 业务分类处理：简单问题直答、复杂问题拆分处理、异常请求兜底回复
- 权限分流：普通用户简易流程、管理员完整流程
- 内容分类：文案改写、信息提取、情感分析自动分流

#### 流程示意图

```
START → 意图识别节点
          ├→ 闲聊分支 → 普通对话回复 → END
          ├→ 工具分支 → 调用外部工具 → END
          └→ 知识库分支 → RAG检索问答 → END
```

#### 完整代码示例

```python
from langgraph.graph import StateGraph,START,END,MessagesState
from langchain_core.messages import HumanMessage

# 自定义状态
class RouterState(MessagesState):
    intent: str

# 分支节点
def chat_chat(state):
    return {"messages":["日常闲聊回复"]}

def use_tool(state):
    return {"messages":["调用工具查询结果"]}

def rag_answer(state):
    return {"messages":["知识库检索回答"]}

# 路由判断函数
def route_intent(state):
    intent = state.get("intent","chat")
    if intent == "tool":
        return "use_tool_node"
    elif intent == "rag":
        return "rag_node"
    return "chat_node"

# 搭建路由图
builder = StateGraph(RouterState)
builder.add_node("chat_node",chat_chat)
builder.add_node("use_tool_node",use_tool)
builder.add_node("rag_node",rag_answer)

# 条件路由
builder.add_conditional_edges(
    START,
    route_intent,
    {
        "chat_node":"chat_node",
        "use_tool_node":"use_tool_node",
        "rag_node":"rag_node"
    }
)

# 所有分支统一收尾
builder.add_edge("chat_node",END)
builder.add_edge("use_tool_node",END)
builder.add_edge("rag_node",END)

router_graph = builder.compile()

# 测试不同分支
print(router_graph.invoke({"intent":"chat","messages":[HumanMessage(content="你好")]}))
print(router_graph.invoke({"intent":"tool","messages":[HumanMessage(content="查天气")]}))
```

#### 优缺点

- 优点：业务解耦、分类清晰、适配多类型请求、扩展性强
- 缺点：分支过多时路由逻辑臃肿，需统一规范判定规则

#### 实战定位

智能问答系统、统一AI入口平台**必用模式**，实现一个入口承接多种不同业务需求。

### 3. 循环模式（Loop）

- **特点**：**迭代执行**，直到退出条件（如“回答正确”或“最大轮次”）。
- **场景**：Agent自主决策、代码调试、多轮对话优化。

这是 **Agent 智能体的灵魂！**
也是 LangGraph 最核心、最强大、最能体现“智能”的工作流模式。

我用**最通俗、最实战、最容易记住**的方式给你讲透👇

---

#### 一、循环模式到底是什么？

一句话：
**节点可以反复、迭代执行，直到满足退出条件才停止**

不是跑一次就结束，而是：
**跑 → 判断 → 再跑 → 再判断 → 直到合格才结束**

这就是 AI 能**自主思考、反复试错、多轮优化**的核心原理！

---

#### 二、核心特点

1. **支持闭环迭代**
   可以跑回之前的节点，重复执行
2. **动态退出条件**
   不是固定次数，而是看 **State 状态** 决定是否继续
3. **自动重试/优化**
   代码写错 → 重写；答案不对 → 重答；工具失败 → 重试
4. **可设置最大轮次**
   防止死循环

---

#### 三、标准循环流程（Agent 标准结构）

```
START
  ↓
LLM 思考/决策 → 路由判断
  ↗            ↘
工具执行 ←←←←←   结束 END
```

也就是：
**思考 → 行动 → 观察 → 再思考 → 再行动……直到完成**

这就是 **ReAct 智能体** 的标准工作模式！

---

#### 四、为什么必须用循环模式？

因为现实任务**不可能一次就完成**：
- 数学题：需要计算 → 验证 → 再计算
- 写代码：写 → 跑 → 报错 → 修改 → 再跑
- 复杂查询：搜索 → 查资料 → 整合 → 再搜索
- 多工具调用：查天气 + 查时间 + 查地点

**必须循环！**

---

#### 五、最经典实战代码（可直接运行）

##### 实现：LLM ↔ 工具 循环调用

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState

# 1. 状态（自带消息追加）
class State(MessagesState):
    pass

# 2. 节点：LLM 思考
def llm_node(state):
    # 模拟：判断是否需要调用工具
    return {"messages": ["需要调用计算器"]}

# 3. 节点：工具执行
def tool_node(state):
    return {"messages": ["工具计算完成：1+1=2"]}

# 4. 路由：循环的核心！
def should_continue(state):
    last = state["messages"][-1]
    if "需要调用" in last:
        return "tool"      # 继续循环 → 去工具
    return END             # 退出循环

# ========================
# 构建循环图（重点）
# ========================
builder = StateGraph(State)

builder.add_node("llm", llm_node)
builder.add_node("tool", tool_node)

builder.add_edge(START, "llm")

# 条件边：决定是否循环
builder.add_conditional_edges(
    "llm",
    should_continue,
    {"tool": "tool", END: END}
)

# 工具执行完 → 回到 LLM（形成闭环！）
builder.add_edge("tool", "llm")

# 编译运行
graph = builder.compile()
```

##### 执行流程（循环形成）

1. START → LLM
2. LLM 判断需要工具 → 路由到 tool
3. tool 执行完 → **回到 LLM**
4. LLM 再次判断 → 不需要工具了
5. 结束流程

---

#### 六、一句话终极总结（背下来！）

##### **循环模式 = 迭代执行 + 条件退出 = Agent 智能体的核心**

##### **条件边 + 回头边 = 真正的 AI 自主思考**

##### **不用循环，就不是真正的智能体！**

---

#### 七、你已经掌握 LangGraph 三大最强模式！

✅ 链式模式：固定流程
✅ 路由模式：条件分支
✅ **循环模式：自主迭代（Agent 灵魂）**

---

#### 我可以直接送你一套：

##### **🔥 完整可运行的 LangGraph 智能体（循环+工具+记忆+Streamlit 界面）**

复制就能跑，带前端聊天界面！

你要我现在直接发给你吗？

### 4. 协作模式（Collaborator）

- **特点**：**多Agent分工**，共享State，并行/串行完成任务。
- **场景**：复杂项目（如研究Agent+写作Agent+编辑Agent联合创作）。

#### 核心特点

1. **多智能体分工协作**，每个Agent独立负责一类专项任务，角色职责清晰
2. 全员共享**全局State**，任务数据、中间结果互通流转
3. 调度灵活，支持**串行依次执行、并行同步执行、主次调度**
4. 可自主流转交接任务，模拟团队协同办公

#### 执行逻辑

全局状态统一承载需求、过程数据、最终成果
专属Agent1完成本职工作写入状态 → 调度分发 → Agent2读取状态接续处理 → 多角色依次配合，直至整体任务闭环完成

#### 适用场景

- 内容创作流水线：调研Agent → 撰稿Agent → 润色编辑Agent
- 复杂问题拆解：拆解Agent → 推理Agent → 整合总结Agent
- 项目全流程：需求Agent → 设计Agent → 执行Agent → 审核Agent
- 专业领域分工：检索知识库Agent + 逻辑推理Agent + 格式整理Agent

#### 流程示意

```
START
  ├→ 研究Agent → 收集整理资料
  ├→ 写作Agent → 依托资料完成正文
  └→ 编辑Agent → 校对优化定稿
          ↓
         END
```

#### 极简代码示例

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState

# 全局共享状态
class CollaborateState(MessagesState):
    source_info: str    # 调研资料
    draft_content: str  # 初稿内容
    final_article: str  # 定稿文案

# 角色1：资料调研智能体
def research_agent(state: CollaborateState):
    return {"source_info": "已收集完整行业相关参考资料"}

# 角色2：文案撰写智能体
def write_agent(state: CollaborateState):
    return {"draft_content": "依据资料完成文章初稿"}

# 角色3：审核编辑智能体
def edit_agent(state: CollaborateState):
    return {"final_article": "完成语句优化、纠错排版，输出终稿"}

# 搭建多Agent协作流程图
builder = StateGraph(CollaborateState)
builder.add_node("research_agent", research_agent)
builder.add_node("write_agent", write_agent)
builder.add_node("edit_agent", edit_agent)

# 串行团队协作流转
builder.add_edge(START, "research_agent")
builder.add_edge("research_agent", "write_agent")
builder.add_edge("write_agent", "edit_agent")
builder.add_edge("edit_agent", END)

collab_graph = builder.compile()

# 执行协同任务
res = collab_graph.invoke({})
print(res)
```

#### 三种协作调度方式

1. **串行协作**：按固定先后顺序依次完成，流程规整易管控
2. **并行协作**：多个Agent同时执行独立任务，完成后合并结果，提升效率
3. **自主委派协作**：由总控Agent分配任务，自动指派对应专项Agent执行

#### 优缺点

- 优点：任务拆分精细、专业能力各司其职、易扩展新增角色、高度贴合真实业务团队流程
- 缺点：架构复杂度偏高，状态流转与任务交接需要合理设计，调试链路更长

#### 实战定位

单Agent解决单点任务，**协作模式面向大型系统化复杂业务**，是搭建企业级AI工作流、智能协同平台的主流落地模式。

---

## 四、高级特性分类（企业级能力）

### 1. 状态持久化（Persistence）

- **机制**：内置**Checkpoint**（检查点），自动保存每一步状态，支持：
  - **断点续跑**：中断后从最近检查点恢复。
  - **时间旅行**：回溯到任意历史状态，调试优化。
- **存储后端**：内存（默认）、SQLite、Redis、PostgreSQL。

这是 **LangGraph 最强大、企业级必须用** 的高级功能！
让 AI 工作流**断电不死、中断可续、历史可查、随时回溯**。

我用**最通俗、最实战、最容易记住**的方式给你讲透👇

---

#### 一、状态持久化是什么？

一句话：
**把每一步执行的 State（状态）自动保存到数据库，让流程可以中断后恢复、回到过去重新跑。**

就像游戏的**存档点**：
- 打一关存一个档
- 死机了 → 从最近存档继续
- 想回去打前面关卡 → 直接读档

---

#### 二、两大超级能力

##### 1. 断点续跑（Resume）

- 流程跑一半断了（网络挂了、服务重启、用户暂停）
- **不用从头再来**
- 从**最近检查点**直接继续执行

##### 2. 时间旅行（Time Travel）

- 回到流程**任意历史步骤**
- 重新执行、修改逻辑、调试优化
- 完全不影响当前状态

这是传统 LangChain、普通代码**完全做不到**的！

---

#### 三、核心机制：Checkpoint（检查点）

- 每执行完一个 Node，自动存一个 **Checkpoint**
- 保存内容：
  - 完整 State
  - 当前执行到哪个节点
  - 历史消息记录
- 存起来后，随时可恢复、可回溯

---

#### 四、支持的存储后端

1. **内存（默认）**：重启丢失
2. **SQLite**：本地文件，适合开发、单机部署
3. **Redis**：分布式、高性能、正式环境首选
4. **PostgreSQL**：企业级持久化、大规模部署

---

#### 五、最精简实战代码（开箱即用）

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.sqlite import SqliteCheckpointSaver
from langgraph.graph import MessagesState

# 1. 定义状态
class State(MessagesState):
    pass

# 2. 定义节点
def simple_node(state):
    return {"messages": ["执行一步操作"]}

# 3. 创建持久化存储器（SQLite 文件保存）
checkpointer = SqliteCheckpointSaver.from_conn_string("checkpoints.db")

# 4. 构建流程图
builder = StateGraph(State)
builder.add_node("node", simple_node)
builder.add_edge(START, "node")
builder.add_edge("node", END)

# 5. 编译时传入 checkpointer！！！
graph = builder.compile(checkpointer=checkpointer)
```

##### 运行时传入 thread_id（用户ID/会话ID）

```python
# 第一次运行
graph.invoke(
    {"messages": ["开始"]},
    config={"configurable": {"thread_id": "user_123"}}
)

# 关闭程序、重启电脑
# 再次运行，会自动从上次状态继续！
graph.invoke(
    {"messages": ["继续"]},
    config={"configurable": {"thread_id": "user_123"}}
)
```

---

#### 六、为什么这是企业级必备功能？

- **多轮对话不丢失**：重启服务也能接着聊
- **人工审核流程**：暂停 → 审核 → 继续
- **复杂长流程**：跑几天的任务，中断可恢复
- **可审计、可回溯**：每一步都有记录，合规必备
- **调试神器**：回到错误现场，直接复现问题

---

#### 七、一句话终极总结（背下来！）

##### **持久化 = 自动存档 + 断点续跑 + 时间旅行**

##### **Checkpoint = 每一步的存档点**

##### **thread_id = 每个用户/每个任务的独立存档槽**

##### **企业级 AI 工作流必须开持久化！**

---

#### 🔥 你已经掌握 LangGraph 最高级能力！

现在你真正理解：
- 基础图：线性流程
- 循环图：智能体
- 子图：模块化
- 多智能体：团队协作
- **持久化：企业级健壮性**

---

#### 我可以直接送你一套：

##### **🔥 完整可运行：LangGraph + 持久化 + 多用户记忆 + 前端界面**

一套就是**正式上线级AI智能体系统**！

你要我现在直接发给你吗？



### 2. 人工介入（Human-in-the-Loop）

- **模式**：
  - **审批模式**：Agent生成结果→人工审核→通过/驳回重跑。
  - **输入模式**：流程暂停→等待用户输入→继续执行。

#### 核心定义

AI流程执行中途**主动暂停**，把控制权交给人，完成审核、确认、补充信息后，再恢复自动执行，实现**人机协同**，规避模型幻觉、决策失误、不合规输出等问题。

#### 两大核心模式

##### 一、审批模式

1. **执行流程**
Agent自动生成方案/答案/操作内容 → 流程暂停 → 人工审核判断 → 审核通过直接放行 / 驳回退回重写重生成
2. **适用场景**
- 文案、合同、话术合规审核
- 业务操作、数据变更人工确认
- 高危指令拦截复核
- 模型输出内容质检

3. **业务价值**
过滤错误输出、严控内容风险、满足企业审批流程、降低线上故障概率

---

##### 二、输入模式

1. **执行流程**
流程走到指定节点自动暂停 → 等待用户补充输入信息 → 拿到人工输入后写入State → 自动继续走完剩余流程
2. **适用场景**
- 信息缺失时主动索要关键字段
- 多轮分步填表式任务
- 复杂需求分步引导用户完善诉求
- 个性化定制类任务收集偏好

#### 实现原理

1. 借助 LangGraph **中断机制**暂停图执行
2. 暂停后保留完整现场State与执行位置
3. 外部传入人工确认结果/新增内容
4. 调用`invoke`传入同一份`thread_id`，自动从中断点继续运行
5. 搭配**状态持久化**，重启服务也能保留待审批状态

#### 极简代码示例

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph import MessagesState

class State(MessagesState):
    content: str
    pass_audit: bool

# AI生成内容节点
def gen_content(state):
    return {"content":"AI自动生成正式文案"}

# 人工介入中断节点
def human_interrupt(state):
    # 流程在此暂停，等待外部传入审核结果
    return {}

# 最终发布节点
def publish_node(state):
    if state["pass_audit"]:
        return {"messages":["审核通过，正式发布"]}
    return {"messages":["审核驳回，重新生成"]}

# 构建流程
builder = StateGraph(State)
builder.add_node("gen", gen_content)
builder.add_node("human_check", human_interrupt)
builder.add_node("publish", publish_node)

builder.add_edge(START, "gen")
builder.add_edge("gen", "human_check")
builder.add_edge("human_check", "publish")
builder.add_edge("publish", END)

graph = builder.compile()

# 1. 执行到人工节点自动暂停
thread_cfg = {"configurable":{"thread_id":"task_001"}}
graph.invoke({}, config=thread_cfg)

# 2. 外部传入人工审核结果，继续执行
graph.invoke({"pass_audit":True}, config=thread_cfg)
```

#### 搭配能力

1. **结合持久化**：待审批任务持久入库，随时处理、延时审核
2. **结合多智能体**：分工流程关键节点统一人工把关
3. **结合回调监控**：记录审核人、审核时间、审核结果，留痕审计

#### 优缺点

- 优点：大幅提升流程可靠性、弥补大模型能力短板、适配企业规范流程、人机协同更贴合实际业务
- 缺点：增加人工操作成本，不适合超高并发全自动纯无人场景

#### 实战定位

企业级AI办公流程、智能审批系统、内容生产平台、自动化运维流程**标配能力**，是从纯自动脚本走向可落地商用系统的关键特性。

### 3. 可观测性（Observability）

- **原生集成LangSmith**：全链路追踪，可视化每一步节点/边/状态变化、耗时、Token消耗。

#### 核心定义

对 LangGraph 工作流全流程做**日志采集、链路追踪、指标统计、问题排查**，全程透明化，告别黑盒调用，线上生产环境必备。

#### 核心依托

原生无缝集成 **LangSmith**，无需额外改造代码，一键开启全链路监控。

#### 可观测核心监控维度

1. **流程链路可视化**
    直观看到整张图执行路径：走过哪些节点、走了哪条条件分支、循环执行次数、流转走向。
2. **状态全量变化**
    每一步 Node 执行前后 State 字段变更、消息追加、数据修改全程留痕。
3. **性能指标统计**
    每个节点执行耗时、整体流程总耗时、调用延迟分析。
4. **资源消耗统计**
    输入/输出 Token 数量、模型调用次数、整体计费开销统计。
5. **内容全览**
    完整 Prompt、模型返回结果、工具入参、工具返回数据全部可查。
6. **异常捕获**
    节点报错、路由异常、调用超时、参数错误自动标记定位。

#### 快速开启使用

##### 1. 配置环境变量

```python
import os
# 开启追踪
os.environ["LANGCHAIN_TRACING_V2"] = "true"
# 填入自己 LangSmith API Key
os.environ["LANGCHAIN_API_KEY"] = "你的key"
# 项目名分类管理
os.environ["LANGCHAIN_PROJECT"] = "LangGraph智能体项目"
```

##### 2. 正常运行Graph即可自动上报

无需修改节点、边、持久化代码，`invoke` / `stream` 执行自动推送链路数据到后台。

###### 实战价值

1. **调试排错**
线上流程出错直接回溯完整执行步骤，快速定位是提示词、检索、路由还是工具问题。
2. **流程优化**
依据耗时数据优化慢节点、精简冗余分支、减少无效循环调用。
3. **成本管控**
统计日常 Token 消耗，合理选型模型，控制调用成本。
4. **业务复盘**
统计用户请求流向、常用分支、高频任务，反向优化业务流程。
5. **运维审计**
企业级系统满足调用留痕、操作溯源、合规审查需求。

#### 搭配组合用法

- 结合**状态持久化**：既存存档数据，又存执行追踪日志
- 结合**人工介入**：记录每一次人工审核节点的暂停时长、操作行为
- 结合**回调Callbacks**：自定义业务埋点 + LangSmith 官方链路双层监控

#### 适用场景

所有上线生产的 LangGraph 应用、企业级多智能体系统、RAG 知识库平台、自动化办公AI流程，**必开可观测追踪**。

#### 面试考点

1. LangGraph 如何实现全流程观测？
原生集成 LangSmith，开启环境变量即可实现节点、状态、耗时、Token 全链路追踪。
2. 可观测性主要解决什么问题？
解决AI工作流执行黑盒问题，实现调试、性能优化、成本统计、线上故障快速定位。
3. 能监控哪些核心数据？
执行链路、状态变更、节点耗时、Token消耗、模型输入输出、异常报错。



### 4. 异步与并行（Async & Parallel）

- 支持**异步节点**（提高吞吐量）、**并行分支**（同时执行多个节点，结果合并）。

#### 核心定义

LangGraph 原生支持**异步执行**与**并行并发**能力，打破串行排队执行限制，大幅提升工作流吞吐效率与整体运行速度。

#### 一、异步节点 Async

##### 特点

1. 定义`async`异步函数作为节点，不阻塞主线程
2. 适合IO密集型任务：模型调用、接口请求、数据库查询、文件读写
3. 同一线程内可调度多个异步任务，提升并发承载量

##### 适用场景

- 大批量批量问答、批量文档处理
- 多接口异步拉取数据
- 高并发对话服务，提升接口吞吐量

##### 基础写法

```python
# 异步节点定义
async def async_llm_node(state):
    # 异步调用模型/请求接口
    resp = await llm.ainvoke(state["messages"])
    return {"messages":[resp]}

# 注册使用和普通节点完全一致
builder.add_node("async_think", async_llm_node)
```

#### 二、并行分支 Parallel

##### 核心特点

1. 同一触发点**同时开启多条执行分支**，多个节点同步运行
2. 所有并行分支全部执行完毕后，再统一合并结果、汇总状态，继续往下流转
3. 无需等待A执行完再跑B，时间开销由“累加”变为“取最大值”

##### 适用场景

1. 多维度信息同时搜集：并行查百科+联网搜索+知识库检索
2. 多角度并行分析：情感分析+关键词提取+内容摘要同步执行
3. 多方案并行生成：一次性产出多个回答版本、多个设计思路
4. 异构数据源并行拉取数据后统一整合

##### 执行流程

```
START
    ├─分支1→节点A
    ├─分支2→节点B
    └─分支3→节点C
全部执行完成 → 合并State数据 → 统一进入下一节点 → END
```

#### 核心优势

1. **提速显著**：串行总耗时 = 各节点耗时之和；并行总耗时 = 最慢分支耗时
2. **资源利用率高**：充分利用网络、IO、模型调用空闲时间
3. **架构灵活**：可局部并行、全局串行混用，兼顾效率与流程顺序
4. **兼容原有能力**：并行流程同样支持状态持久化、人工介入、LangSmith链路追踪

#### 实战价值

- 优化RAG问答：并行多路检索，再统一整合答案，缩短响应时长
- 多智能体协同：同级Agent并行作业，提升团队协作效率
- 线上服务压优：高并发场景用异步+并行提升服务承载能力

#### 面试重点

1. 异步节点作用
非阻塞执行IO/模型调用，提升服务并发吞吐量。
2. 并行分支原理
同一节点触发多条链路同时执行，全部完成后合并状态再继续执行。
3. 串行和并行选型
有先后依赖必须串行；无依赖、可同时执行的任务优先并行提速。
4. 并行执行后如何汇总数据
依靠全局共享`State`自动合并各分支写入的字段数据。



---

## 五、与LangChain Chain的核心区别

| 特性          | LangChain Chain            | LangGraph                       |
| :------------ | :------------------------- | :------------------------------ |
| **结构**      | 线性DAG，无循环            | 有向循环图，支持闭环            |
| **状态**      | 无状态（靠Memory临时存储） | 原生状态管理，持久化+回溯       |
| **控制流**    | 固定顺序，难分支           | 动态条件分支，灵活路由          |
| **Agent能力** | 简单工具调用，无迭代       | 原生循环，支持自主决策+多轮优化 |
| **复杂度**    | 低（适合简单任务）         | 高（适合复杂/企业级任务）       |

---

## 六、一句话总结

**LangGraph = 状态化循环编排引擎**，用**节点/边/状态**三大基石，构建**线性、分支、循环、多Agent协作**等复杂工作流，解决传统Chain的**无状态、难回溯、流程僵化**痛点，是**企业级Agent与复杂AI应用的核心框架**。



# Command API & StateGraph

## **LangGraph Command API**

---

### 一、Command 是什么？

**Command** 是 LangGraph 的**统一控制原语**，用来在**节点 / 工具 / 外部恢复点**同时做两件事：

1. **更新 State**（像普通节点返回 dict）
2. **控制流向**（像条件边）

一句话：**Command = 状态更新 + 动态路由**，把分散的“节点返回 + 条件边”合并成更内聚、更易读的写法。

适用三个场景：
- ✅ **从节点返回**：替代条件边
- ✅ **从工具返回**：在工具内改状态 + 跳转
- ✅ **外部 invoke/stream 传入**：恢复中断（human‑in‑the‑loop）

---

### 二、Command 类签名（Python）

```python
from langgraph.types import Command

Command(
    update: dict | None = None,      # 状态更新
    goto: str | list | Send | None = None,  # 下一步节点
    graph: str | None = None,        # 目标图（主子图）
    resume: Any = None                # 中断恢复值
)
```
#### 核心参数详解

##### 1）update：状态更新（必用）

- 类型：`dict`，与节点返回格式完全一致
- 作用：**合并到当前 State**（遵循 reducer 规则）
- 示例：
```python
update={"messages": [("user", "hello")], "step": 1}
```

##### 2）goto：动态路由（核心）

支持多种形式：
- **str**：单个节点名 / `END`
- **list[str]**：并行执行多个节点
- **Send**：带参数调用指定节点（子图常用）
- **None**：按默认边执行

示例：
```python
goto="agent"
goto=["tool1", "tool2"]
goto=Send("subgraph", {"input": "xxx"})
```

##### 3）graph：主子图跳转

- `None`（默认）：当前图
- `"parent"` / `Command.PARENT`：最近父图（子图返回时用）
- 具体图名：指定目标子图

##### 4）resume：中断恢复（HITL）

- 用于 `interrupt_before` / `interrupt_after` 后的恢复
- 外部调用 `graph.invoke(Command(resume=...))` 传入

---

### 三、三种典型用法（附完整可运行示例）

#### 场景 1：节点返回 Command（替代条件边）

传统写法（节点返回 + 条件边）：
```python
def router(state):
    if state["step"] < 3:
        return "loop"
    else:
        return "end"

builder.add_conditional_edges("node", router, {"loop": "node", "end": END})
```

**Command 写法（更简洁）**：
```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command
from typing import TypedDict

class State(TypedDict):
    step: int

def node(state) -> Command[State]:
    new_step = state["step"] + 1
    return Command(
        update={"step": new_step},
        goto="node" if new_step < 3 else END
    )

builder = StateGraph(State)
builder.add_node("node", node)
builder.add_edge(START, "node")
graph = builder.compile()

print(graph.invoke({"step": 0}))  # {'step': 3}
```
**优势**：路由逻辑内聚在节点，无需单独写 router 函数和条件边映射。

---

#### 场景 2：工具返回 Command（工具内改状态 + 跳转）

```python
from langchain_core.tools import tool
from langgraph.types import Command, InjectedToolCallId
from langchain_core.runnables import RunnableConfig

@tool
def add_message(
    text: str,
    tool_call_id: InjectedToolCallId,
    config: RunnableConfig
) -> Command:
    """添加消息并直接跳转到 agent"""
    return Command(
        update={"messages": [("user", text)]},
        goto="agent"
    )
```
**关键点**：工具返回 Command，**自动更新状态 + 跳转**，无需 ToolNode 额外处理。

---

#### 场景 3：外部恢复中断（Human‑in‑the‑Loop）

```python
from langgraph.checkpoint.memory import MemorySaver

builder = StateGraph(State)
builder.add_node("agent", agent_node)
builder.add_node("dangerous_tool", dangerous_tool)
builder.add_edge(START, "agent")
builder.add_edge("agent", "dangerous_tool")

# 编译时指定在 dangerous_tool 前中断
checkpointer = MemorySaver()
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["dangerous_tool"]
)

# 1. 执行到中断点
thread = {"configurable": {"thread_id": "1"}}
graph.invoke({"step": 0}, thread)

# 2. 人类审批后，用 Command(resume=...) 恢复
graph.invoke(Command(resume="approved"), thread)
```
**resume 作用**：把外部输入传给中断节点，继续执行。

---

### 四、goto 高级用法：Send 与并行

#### 1）Send：带参数调用节点（子图）

```python
from langgraph.types import Send

def parent_node(state) -> Command:
    return Command(
        goto=Send("subgraph", {"input": state["query"]})
    )
```

#### 2）并行执行（list）

```python
def parallel_node(state) -> Command:
    return Command(goto=["tool1", "tool2"])
```

---

### 五、Command vs 条件边（Conditional Edges）

| 维度     | 条件边                | Command               |
| -------- | --------------------- | --------------------- |
| 逻辑位置 | 节点外，独立 router   | 节点内，内聚          |
| 状态更新 | 节点返回 dict         | Command(update=...)   |
| 路由跳转 | add_conditional_edges | Command(goto=...)     |
| 子图跳转 | 复杂                  | 原生支持 graph/PARENT |
| 恢复中断 | 需额外处理            | 原生 resume           |
| 可读性   | 分散                  | 高（逻辑集中）        |

**结论**：**优先用 Command**，除非路由逻辑特别复杂、需要复用多个节点。

---

### 六、常见坑与最佳实践

1. **update 必须是 dict**：不要返回非 dict 类型
2. **goto 节点必须存在**：否则抛 `NodeNotFound`
3. **子图跳转用 graph=Command.PARENT**：避免硬编码图名
4. **resume 仅用于中断恢复**：不要在普通节点返回
5. **并行 goto 用 list**：不要嵌套 list

---

### 七、完整速查（可直接复制）

```python
# 节点返回 Command
def node(state) -> Command:
    return Command(update={"k": "v"}, goto="next")

# 工具返回 Command
@tool
def my_tool(...) -> Command:
    return Command(update={...}, goto="agent")

# 恢复中断
graph.invoke(Command(resume=value), config)

# 子图返回父图
return Command(..., graph=Command.PARENT)
```

---

要不要我把这些示例整理成一个**可直接运行的完整脚本**（含状态定义、节点、工具、中断恢复），你直接复制就能跑？

## LangGraph StateGraph

### 一、核心概念

`StateGraph` 是 LangGraph **最核心的图构建器**，基于**状态驱动**构建多智能体/流程编排图。

- 核心思想：**整个流程靠统一 State 流转，节点只读+修改状态，边决定流向**
- 定位：替代手动写循环、分支、状态机，快速搭建 Agent 工作流
- 依赖：`TypedDict` / `dataclass` 定义全局状态

#### 导入

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Optional, List
```

---

### 二、基础架构组成

1. **State 状态**：全局唯一数据流，所有节点共享读写
2. **Node 节点**：执行逻辑，接收 State，返回**状态更新**
3. **Edge 普通边**：固定流向 `A -> B`
4. **Conditional Edge 条件边**：根据状态动态跳转
5. **START**：图入口起点
6. **END**：图终止终点
7. **Checkpointer 持久化**：断点续跑、记忆存储
8. **Compile 编译**：把构图转为可执行 runnable

---

### 三、1. 定义 State 状态（重中之重）

#### 方式1：TypedDict（最常用、轻量）

```python
class AgentState(TypedDict):
    messages: List[str]    # 对话消息
    query: str             # 用户问题
    result: Optional[str]  # 最终结果
    step: int              # 流程步数
```

#### 方式2：dataclass（支持默认值、类型约束更强）

```python
from dataclasses import dataclass, field

@dataclass
class AgentState:
    messages: List[str] = field(default_factory=list)
    query: str = ""
    result: str = ""
    step: int = 0
```

**状态合并规则**
- 节点返回 `{字段:新值}` 只会**增量更新**，不会覆盖全量
- 列表默认**追加合并**，可自定义 `reducer` 改写合并逻辑

---

### 四、2. 定义 Node 节点 & 添加节点

#### 节点标准格式

```python
def 节点名(state: 状态类) -> dict:
    # 1. 读取状态数据
    user_query = state["query"]
    # 2. 执行业务逻辑
    res = f"处理：{user_query}"
    # 3. 返回状态更新字典
    return {"result": res, "step": state["step"] + 1}
```

#### 节点返回两种形式

1. **普通 dict**：只更新状态
2. **Command**：更新状态 + 动态跳转（搭配上篇 Command API）
```python
from langgraph.types import Command
def router_node(state) -> Command:
    return Command(update={"step":1}, goto="next_node")
```

#### `builder.add_node`

**`builder.add_node(节点名, 执行函数)`
= 在流程图里添加一个「步骤/任务/节点」**

```python
builder.add_node("节点名字", 节点要执行的函数)
```

1. **"节点名字"**
   - 给这个步骤起个代号（随便起）
   - 后面用 `add_edge` 跳转时会用到
   - 例：`"parse_query"`、`"supervisor"`、`"search"`

2. **节点要执行的函数**
   - 流程走到这一步时**真正运行的代码**
   - 必须接收 `state`
   - 必须返回 **要更新的状态字典**
   - 例：`parse_query_node`、`supervisor_node`

---

##### 最经典例子

```python
builder.add_node("parse_query", parse_query_node)
```

意思：
> 添加一个步骤，名字叫 **parse_query**
> 当流程走到这里时，执行 **parse_query_node 函数**

---

##### 节点函数长什么样？（固定模板）

```python
def parse_query_node(state: AgentState):
    # 1. 从 state 里拿数据
    user_query = state["query"]

    # 2. 做业务逻辑（解析、处理、判断）
    parsed = {"intent": "search", "keyword": "AI"}

    # 3. 返回要更新的状态
    return {"parsed_query": parsed}
```

---

##### 在流程图里的意义

```
START → parse_query → next_node → ...
```

**每一个 add_node，就是流程图里的一个方框。**

**add_node = 添加一个步骤
名字是代号，函数是干活的**

---

### 五、3. 构建 StateGraph 全流程

#### 完整基础模板

```python
# 1. 初始化图，绑定状态
builder = StateGraph(AgentState)

# 2. 添加节点
builder.add_node("parse_query", parse_query_node) # 给流程图注册一个节点：名字叫 parse_query，执行函数是 parse_query_node
builder.add_node("think", think_node)
builder.add_node("answer", answer_node)

# 3. 设置入口
builder.add_edge(START, "parse_query")

# 4. 普通固定边
builder.add_edge("parse_query", "think")
builder.add_edge("think", "answer")

# 5. 出口结束
builder.add_edge("answer", END)

# 6. 编译图
graph = builder.compile()

# 7. 调用执行
res = graph.invoke({"query":"今天天气"})
print(res)
```

---

### 六、4. 核心边用法

#### 1）普通边 add_edge

固定单向流转
```python
builder.add_edge("A", "B")  # 建立一条固定的 “单向执行边”，表示**当 A 执行完成后 → 自动跳转到 B 执行**
```

- `A`：起点节点（可以是 Agent / 工具 / 函数）
- `B`：终点节点
- 这条边是**单向、固定、无条件**的
- 没有判断、没有选择，就是**执行完 A 必须走 B**

#### 2）条件边 add_conditional_edges（核心分支）

**add_conditional_edges**：**条件判断跳转**根据结果选择走不同节点

**语法**
```python
builder.add_conditional_edges(
    source: 源节点名,
    path: 判断函数(接收state，返回跳转key),
    path_map: {key: 目标节点名},
    then: 可选，走完分支后统一走这个节点
)
```

**实战示例：循环/终止**
```python
def loop_judge(state: AgentState) -> str:
    if state["step"] < 3:
        return "continue"
    else:
        return "finish"

builder.add_conditional_edges(
    source="think",
    path=loop_judge,
    path_map={
        "continue": "think",
        "finish": "answer"
    }
)
```

---



### 七、5. 高级核心能力

#### ① 自定义状态合并 Reducer

修改列表/字段合并逻辑
```python
from langgraph.graph import add_messages

class State(TypedDict):
    messages: List[str]  # 使用内置消息合并器

# 初始化指定reducer
builder = StateGraph(State)
```
内置常用：
- `add_messages`：对话消息追加
- `operator.add`：数值累加
- 自定义函数实现覆盖/清空

#### ② 子图 Subgraph 嵌套

实现复杂业务模块化拆分
```python
# 1. 构建子图
sub_builder = StateGraph(SubState)
# ...子图节点+边...
sub_graph = sub_builder.compile()

# 2. 主图添加子图作为节点
builder.add_node("sub_workflow", sub_graph)
```

#### ③ 断点暂停 + 人工介入 HITL

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()
# 编译时指定暂停节点
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["risk_node"]  # 执行前暂停
    # interrupt_after=["node"] 执行后暂停
)
```
搭配 `Command(resume=xxx)` 恢复执行。

#### ④ 并行节点执行

同一个状态同时跑多个节点
```python
builder.add_edge("start", ["node1", "node2"])
```

#### ⑤ 图可视化

```python
# 生成Mermaid流程图
graph.get_graph().draw_mermaid_png(output_file="graph.png")
```

---

### 八、6. 调用执行三种方式

#### 1. invoke 同步一次性执行

```python
graph.invoke({"query":"你好"})
```

#### 2. stream 流式逐节点输出

```python
for chunk in graph.stream({"query":"流式问答"}):
    print(chunk)
```

#### 3. ainvoke 异步调用

```python
import asyncio
async def run():
    await graph.ainvoke({"query":"异步"})
```

---

### 九、7. StateGraph 常用API速查表

| API                       | 作用           |
| ------------------------- | -------------- |
| `StateGraph(StateClass)`  | 初始化状态图   |
| `add_node(name, func)`    | 注册节点       |
| `add_edge(s,t)`           | 普通流向边     |
| `add_conditional_edges()` | 条件分支边     |
| `compile()`               | 编译成可执行图 |
| `invoke()`                | 同步执行       |
| `stream()`                | 流式执行       |
| `get_graph()`             | 获取图结构     |
| `interrupt_before/after`  | 流程断点       |
| `Command(goto=...)`       | 节点内动态路由 |

---

### 十、8. 最佳实践 & 避坑

1. **状态尽量扁平化**，不要嵌套过深，维护困难
2. 节点只做**纯逻辑**，路由判断尽量内聚或统一抽函数
3. 循环流程优先用**条件边**，不要手写死循环
4. 多Agent场景拆分**子图**解耦
5. 长对话必加 `Checkpointer` 做记忆持久化
6. 优先 `TypedDict` 定义状态，开发提示更友好
7. 节点禁止直接修改入参 state，**只返回更新字典**

---

### 十一、最简可运行完整示例

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict

# 1. 定义状态
class MyState(TypedDict):
    num: int
    total: int

# 2. 定义节点
def add_node(state: MyState):
    return {"total": state["num"] + 10}

# 3. 构图
builder = StateGraph(MyState)
builder.add_node("add", add_node)
builder.add_edge(START, "add")
builder.add_edge("add", END)

# 4. 运行
graph = builder.compile()
print(graph.invoke({"num":5}))
# 输出: {'num': 5, 'total': 15}
```

---

## LangGraph

### LangGraph 的结构梳理

LangGraph 作为 LangChain 生态系统的重要扩展，代表了智能体框架设计的一个全新方向。与前面介绍的基于“对话”的框架（如 AutoGen 和 CAMEL）不同，LangGraph 将智能体的执行流程建模为一种**状态机（State Machine）**，并将其表示为**有向图（Directed Graph）**。在这种范式中，图的**节点（Nodes）**代表一个具体的计算步骤（如调用 LLM、执行工具），而**边（Edges）**则定义了从一个节点到另一个节点的跳转逻辑。这种设计的革命性之处在于它天然支持循环，使得构建能够进行迭代、反思和自我修正的复杂智能体工作流变得前所未有的直观和简单。

要理解 LangGraph，我们需要先掌握它的三个基本构成要素。

**首先，是全局状态（State）**。整个图的执行过程都围绕一个共享的状态对象进行。这个状态通常被定义为一个 Python 的 `TypedDict`，它可以包含任何你需要追踪的信息，如对话历史、中间结果、迭代次数等。所有的节点都能读取和更新这个中心状态。

```python
from typing import TypedDict, List

# 定义全局状态的数据结构
class AgentState(TypedDict):
    messages: List[str]      # 对话历史
    current_task: str        # 当前任务
    final_answer: str        # 最终答案
    # ... 任何其他需要追踪的状态Copy to clipboardErrorCopied
```

**其次，是节点（Nodes）**。每个节点都是一个接收当前状态作为输入、并返回一个更新后的状态作为输出的 Python 函数。节点是执行具体工作的单元。

```python
# 定义一个“规划者”节点函数
def planner_node(state: AgentState) -> AgentState:
    """根据当前任务制定计划，并更新状态。"""
    current_task = state["current_task"]
    # ... 调用LLM生成计划 ...
    plan = f"为任务 '{current_task}' 生成的计划..."
    
    # 将新消息追加到状态中
    state["messages"].append(plan)
    return state

# 定义一个“执行者”节点函数
def executor_node(state: AgentState) -> AgentState:
    """执行最新计划，并更新状态。"""
    latest_plan = state["messages"][-1]
    # ... 执行计划并获得结果 ...
    result = f"执行计划 '{latest_plan}' 的结果..."
    
    state["messages"].append(result)
    return stateCopy to clipboardErrorCopied
```

**最后，是边（Edges）**。边负责连接节点，定义工作流的方向。最简单的边是常规边，它指定了一个节点的输出总是流向另一个固定的节点。而 LangGraph 最强大的功能在于**条件边（Conditional Edges）**。它通过一个函数来判断当前的状态，然后动态地决定下一步应该跳转到哪个节点。这正是实现循环和复杂逻辑分支的关键。

```python
def should_continue(state: AgentState) -> str:
    """条件函数：根据状态决定下一步路由。"""
    # 假设如果消息少于3条，则需要继续规划
    if len(state["messages"]) < 3:
        # 返回的字符串需要与添加条件边时定义的键匹配
        return "continue_to_planner"
    else:
        state["final_answer"] = state["messages"][-1]
        return "end_workflow"Copy to clipboardErrorCopied
```

在定义了状态、节点和边之后，我们可以像搭积木一样将它们组装成一个可执行的工作流。

```python
from langgraph.graph import StateGraph, END

# 初始化一个状态图，并绑定我们定义的状态结构
workflow = StateGraph(AgentState)

# 将节点函数添加到图中
workflow.add_node("planner", planner_node)
workflow.add_node("executor", executor_node)

# 设置图的入口点
workflow.set_entry_point("planner")

# 添加常规边，连接 planner 和 executor
workflow.add_edge("planner", "executor")

# 添加条件边，实现动态路由
workflow.add_conditional_edges(
    # 起始节点
    "executor",
    # 判断函数
    should_continue,
    # 路由映射：将判断函数的返回值映射到目标节点
    {
        "continue_to_planner": "planner", # 如果返回"continue_to_planner"，则跳回planner节点
        "end_workflow": END               # 如果返回"end_workflow"，则结束流程
    }
)

# 编译图，生成可执行的应用
app = workflow.compile()

# 运行图
inputs = {"current_task": "分析最近的AI行业新闻", "messages": []}
for event in app.stream(inputs):
    print(event)Copy to clipboardErrorCopied
```

### 2 三步问答助手

在理解了 LangGraph 的核心概念之后，我们将通过一个实战案例来巩固所学。我们将构建一个简化的问答对话助手，它会遵循一个清晰、固定的三步流程来回答用户的问题：

1. **理解 (Understand)**：首先，分析用户的查询意图。
2. **搜索 (Search)**：然后，模拟搜索与意图相关的信息。
3. **回答 (Answer)**：最后，基于意图和搜索到的信息，生成最终答案。

这个案例将清晰地展示如何定义状态、创建节点以及将它们线性地连接成一个完整的工作流。我们将代码分解为四个核心步骤：定义状态、创建节点、构建图、以及运行应用。

#### （1）定义全局状态

首先，我们需要定义一个贯穿整个工作流的全局状态。**这是一个共享的数据结构，它在图的每个节点之间传递，作为工作流的持久化上下文。** 每个节点都可以读取该结构中的数据，并对其进行更新。

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class SearchState(TypedDict):
    messages: Annotated[list, add_messages]
    user_query: str      # 经过LLM理解后的用户需求总结
    search_query: str    # 优化后用于Tavily API的搜索查询
    search_results: str  # Tavily搜索返回的结果
    final_answer: str    # 最终生成的答案
    step: str            # 标记当前步骤Copy to clipboardErrorCopied
```

我们创建了 `SearchState` 这个 `TypedDict`，为状态对象定义了一个清晰的数据模式（Schema）。一个关键的设计是同时包含了 `user_query` 和 `search_query` 字段。这允许智能体先将用户的自然语言提问，优化成更适合搜索引擎的精炼关键词，从而显著提升搜索结果的质量。

#### （2）定义工作流节点

定义好状态结构后，下一步是创建构成我们工作流的各个节点。在 LangGraph 中，每个节点都是一个执行具体任务的 Python 函数。这些函数接收当前的状态对象作为输入，并返回一个包含更新后字段的字典。

在开始定义节点之前，我们先完成项目的初始化设置，包括加载环境变量和实例化大语言模型。

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from tavily import TavilyClient

# 加载 .env 文件中的环境变量
load_dotenv()

# 初始化模型
# 我们将使用这个 llm 实例来驱动所有节点的智能
llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL_ID", "gpt-4o-mini"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1"),
    temperature=0.7
)
# 初始化Tavily客户端
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))Copy to clipboardErrorCopied
```

现在，我们来逐一创建三个核心节点。

##### （1） 理解与查询节点

此节点是工作流的第一步，此节点的职责是理解用户意图，并为其生成一个最优化的搜索查询。

```python
def understand_query_node(state: SearchState) -> dict:
    """步骤1：理解用户查询并生成搜索关键词"""
    user_message = state["messages"][-1].content
    
    understand_prompt = f"""分析用户的查询："{user_message}"
请完成两个任务：
1. 简洁总结用户想要了解什么
2. 生成最适合搜索引擎的关键词（中英文均可，要精准）

格式：
理解：[用户需求总结]
搜索词：[最佳搜索关键词]"""

    response = llm.invoke([SystemMessage(content=understand_prompt)])
    response_text = response.content
    
    # 解析LLM的输出，提取搜索关键词
    search_query = user_message # 默认使用原始查询
    if "搜索词：" in response_text:
        search_query = response_text.split("搜索词：")[1].strip()
    
    return {
        "user_query": response_text,
        "search_query": search_query,
        "step": "understood",
        "messages": [AIMessage(content=f"我将为您搜索：{search_query}")]
    }Copy to clipboardErrorCopied
```

该节点通过一个结构化的提示，要求 LLM 同时完成“意图理解”和“关键词生成”两个任务，并将解析出的专用搜索关键词更新到状态的 `search_query` 字段中，为下一步的精确搜索做好准备。

##### （2）搜索节点

该节点负责执行智能体的“工具使用”能力，它将调用 Tavily API 进行真实的互联网搜索，并具备基础的错误处理功能。

```python
def tavily_search_node(state: SearchState) -> dict:
    """步骤2：使用Tavily API进行真实搜索"""
    search_query = state["search_query"]
    try:
        print(f"🔍 正在搜索: {search_query}")
        response = tavily_client.search(
            query=search_query, search_depth="basic", max_results=5, include_answer=True
        )
        # ... (处理和格式化搜索结果) ...
        search_results = ... # 格式化后的结果字符串
        
        return {
            "search_results": search_results,
            "step": "searched",
            "messages": [AIMessage(content="✅ 搜索完成！正在整理答案...")]
        }
    except Exception as e:
        # ... (处理错误) ...
        return {
            "search_results": f"搜索失败：{e}",
            "step": "search_failed",
            "messages": [AIMessage(content="❌ 搜索遇到问题...")]
        }Copy to clipboardErrorCopied
```

此节点通过 `tavily_client.search` 发起真实的 API 调用。它被包裹在 `try...except` 块中，用于捕获可能的异常。如果搜索失败，它会更新 `step` 状态为 `"search_failed"`，这个状态将被下一个节点用来触发备用方案。

##### （3）回答节点

最后的回答节点能够根据上一步的搜索是否成功，来选择不同的回答策略，具备了一定的弹性。

```python
def generate_answer_node(state: SearchState) -> dict:
    """步骤3：基于搜索结果生成最终答案"""
    if state["step"] == "search_failed":
        # 如果搜索失败，执行回退策略，基于LLM自身知识回答
        fallback_prompt = f"搜索API暂时不可用，请基于您的知识回答用户的问题：\n用户问题：{state['user_query']}"
        response = llm.invoke([SystemMessage(content=fallback_prompt)])
    else:
        # 搜索成功，基于搜索结果生成答案
        answer_prompt = f"""基于以下搜索结果为用户提供完整、准确的答案：
用户问题：{state['user_query']}
搜索结果：\n{state['search_results']}
请综合搜索结果，提供准确、有用的回答..."""
        response = llm.invoke([SystemMessage(content=answer_prompt)])
    
    return {
        "final_answer": response.content,
        "step": "completed",
        "messages": [AIMessage(content=response.content)]
    }Copy to clipboardErrorCopied
```

该节点通过检查 `state["step"]` 的值来执行条件逻辑。如果搜索失败，它会利用 LLM 的内部知识回答并告知用户情况。如果搜索成功，它则会使用包含实时搜索结果的提示，来生成一个有时效性且有据可依的回答。

#### （3）构建图

我们将所有节点连接起来。

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver

def create_search_assistant():
    workflow = StateGraph(SearchState)
    
    # 添加节点
    workflow.add_node("understand", understand_query_node)
    workflow.add_node("search", tavily_search_node)
    workflow.add_node("answer", generate_answer_node)
    
    # 设置线性流程
    workflow.add_edge(START, "understand")
    workflow.add_edge("understand", "search")
    workflow.add_edge("search", "answer")
    workflow.add_edge("answer", END)
    
    # 编译图
    memory = InMemorySaver()
    app = workflow.compile(checkpointer=memory)
    return appCopy to clipboardErrorCopied
```

#### （4）运行案例展示

运行此脚本后，您可以提出一些需要实时信息的问题，例如我们第一章中的案例：`明天我要去北京，天气怎么样？有合适的景点吗`

您会看到终端清晰地展示出智能体的“思考”过程：

```
🔍 智能搜索助手启动！
我会使用Tavily API为您搜索最新、最准确的信息
支持各种问题：新闻、技术、知识问答等
(输入 'quit' 退出)

🤔 您想了解什么: 明天我要去北京，天气怎么样？有合适的景点吗

============================================================
🧠 理解阶段: 我理解您的需求：理解：用户想了解明天北京的天气情况以及合适的景点推荐。  
搜索词：北京 明天 天气 景点推荐 Beijing weather tomorrow attractions
🔍 正在搜索: 北京 明天 天气 景点推荐 Beijing weather tomorrow attractions
🔍 搜索阶段: ✅ 搜索完成！找到了相关信息，正在为您整理答案...

💡 最终回答:
明天（2025年9月17日）北京的天气预报显示，预计将是多云，气温范围在17°C（62°F）到25°C（77°F）之间。这种温和的天气非常适合户外活动。

### 合适的景点推荐：
1. **长城**：作为中国最著名的历史遗址之一，长城是必游之地。你可以选择八达岭或慕田峪这些较为受欢迎的段落进行游览。

2. **故宫**：故宫是明清两代的皇宫，拥有丰富的历史和文化，适合对中国历史感兴趣的游客。

3. **天安门广场**：这是中国的象征之一，广场上有许多重要的建筑和纪念碑，适合拍照留念。

4. **颐和园**：一个非常美丽的皇家园林，适合漫步和欣赏自然风光，尤其是湖泊和古建筑。

5. **798艺术区**：如果你对现代艺术感兴趣，798艺术区是一个集艺术、文化和创意于一体的地方，适合探索和拍摄。

### 小贴士：
- 由于明天天气良好，建议提前规划出行路线，并准备一些水和小吃，以便在游览时保持充足的体力。
- 由于天气变化可能会影响游览体验，建议查看实时天气更新。

希望这些信息能帮助你安排一个愉快的北京之旅！如果你需要更多关于景点的信息或者旅行建议，欢迎随时询问。

============================================================

🤔 您想了解什么:Copy to clipboardErrorCopied
```

并且他是一个可以持续交互的助手，你也可以继续向他发问。

# LangGraph 全体系核心组件

## 一、流程控制 & 路由类 

### 总览

流程控制+路由是 LangGraph 核心流转能力，用来实现**顺序、分支、循环、并行、暂停、动态跳转、子图调用**，脱离硬编码逻辑，纯声明式编排。

**所属包**
```python
from langgraph.graph import START, END
from langgraph.types import Command, Send, interrupt
```

---

### 1. START / END 全局固定节点

#### 作用

- `START`：流程图**唯一入口**，所有流程必须从这里出发
- `END`：流程图**终止出口**，走到这里流程直接结束

#### 使用规则

1. 不能给 START 写业务逻辑，只做起点
2. 所有结束分支统一指向 END
3. 不允许从 END 再往外连边

#### 示例

```python
builder.add_edge(START, "agent")  # 入口进智能体
builder.add_edge("finish", END)  # 完成结束
```

---

### 2. 静态路由：普通边 add_edge

#### 语法

```python
add_edge(source: str | list[str], target: str | list[str])
```
#### 两种用法

##### 1）串行单流向（顺序执行）

```python
builder.add_edge("A", "B")
builder.add_edge("B", "C")
```
固定 A → B → C 顺序

##### 2）一对多并行路由

```python
builder.add_edge("dispatch", ["search", "calc", "summary"])
```
执行完 dispatch **同时并发执行**三个节点，全部跑完再走后续流程

---

##### 3. 条件路由：add_conditional_edges（最常用分支）

###### 完整参数

```python
builder.add_conditional_edges(
    source: str,                # 来源节点
    path: Callable[[State], str], # 判断函数：入状态，出分支标识
    path_map: dict[str, str],  # 标识映射真实节点名
    then: Optional[str] = None # 所有分支走完统一走这个节点
)
```

###### 核心流程

1. 节点执行完毕
2. 执行 path 函数拿到**分支key**
3. 通过 path_map 匹配目标节点
4. 自动跳转

#### 实战：循环 + 终止

```python
def judge_loop(state):
    if state["round"] < 3:
        return "continue"
    return "stop"

builder.add_conditional_edges(
    source="think",
    path=judge_loop,
    path_map={
        "continue": "think",
        "stop": "output"
    }
)
```

#### 简写省略 path_map

key 和节点名一致可直接返回节点字符串
```python
def route(state):
    return "tool_call" if state["need_tool"] else "direct_answer"

builder.add_conditional_edges("agent", route)
```

---

### 4. 动态最强路由：Command 内置跳转

**优先级高于所有外部条件边**，在**节点内部直接决定下一跳**，逻辑内聚。

#### 核心路由字段

```python
Command(
    update={},       # 更新状态
    goto= 路由目标,  # 核心路由
    graph=None       # 主子图跳转
)
```

#### goto 支持 5 种路由形式

1. **字符串节点名**
```python
return Command(goto="search")
```
2. **直接结束流程**
```python
return Command(goto=END)
```
3. **多节点并行**
```python
return Command(goto=["node1", "node2"])
```
4. **Send 带参调用节点/子图**
```python
from langgraph.types import Send
return Command(goto=Send("sub_node", {"query": state["q"]}))
```
5. **回到父图**
```python
return Command(goto="parent_node", graph=Command.PARENT)
```

#### 优势

- 路由逻辑写在业务节点内，不用单独抽路由函数
- 复杂多分支比 `add_conditional_edges` 更整洁
- 天然支持子图、并行、结束终止

---

### 5. Send 定向传参路由

#### 作用

专门用于：**主动调用指定节点 + 单独传局部参数**，不依赖全局 State，常用于批量分发、子任务派发。

#### 语法

```python
Send(node_name: str, arg_dict: dict)
```

#### 实战：批量拆分任务

```python
def split_task(state):
    tasks = ["任务1", "任务2", "任务3"]
    sends = [Send("exec_task", {"task": t}) for t in tasks]
    return Command(goto=sends)
```
一次性并发调用 `exec_task` 并传入各自参数。

#### 特点

- 被调用节点**优先读取 Send 传入参数**，再合并全局 State
- 适合分发式、批量式、多子任务架构

---

### 6. 流程暂停路由：interrupt 内置中断

#### 作用

在**节点代码内部手动暂停流程**，实现人工审核、等待用户输入、外部确认。

#### 两种中断方式

##### 方式1：代码内主动中断（interrupt()）

```python
from langgraph.types import interrupt

def audit_node(state):
    # 暂停流程，弹出提示，等待外部传入恢复值
    user_input = interrupt("请确认是否执行高危操作？")
    if user_input == "yes":
        return {}
    else:
        return Command(goto=END)
```

##### 方式2：编译时全局断点（更常用）

```python
# 执行该节点之前暂停
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["risk_tool"]
)
# 执行完该节点之后暂停
# interrupt_after=["node"]
```

##### 恢复路由

外部使用 `Command(resume=值)` 唤醒流程
```python
graph.invoke(Command(resume="yes"), config)
```

---

### 7. 主子图跨图路由

#### 1）子图跳回父图

```python
return Command(
    update={},
    goto="parent_target_node",
    graph=Command.PARENT
)
```
#### 2）主图调用子图

直接把子图编译后当成普通节点添加即可，路由自动连通。

---

### 8. 三大路由方案选型对比

| 路由方式              | 适用场景                   | 优点               | 缺点                 |
| --------------------- | -------------------------- | ------------------ | -------------------- |
| add_edge 普通边       | 固定顺序、简单串行         | 最简单、直观       | 无分支能力           |
| add_conditional_edges | 分支多、路由逻辑可复用     | 解耦路由与业务     | 逻辑分散，多分支臃肿 |
| Command.goto          | 复杂判断、循环、子图、并行 | 逻辑内聚，功能最全 | 集中写在节点内       |

**开发通用选型原则**
1. 简单顺序流程 → `add_edge`
2. 通用公共分支 → `add_conditional_edges`
3. 业务强关联分支/循环/人机交互/子图 → **Command**

---

### 9. 路由执行顺序规则（必记）

1. 节点执行完成
2. 优先判断**节点返回 Command.goto**
3. 无 Command 再走外部 `add_conditional_edges`
4. 无条件边走默认普通边
5. 命中 END 直接终止整个图
6. 并行路由：所有目标节点**全部执行完毕**才进入下一阶段

---

### 10. 路由常见错误避坑

1. goto 写不存在节点名 → `NodeNotFoundError`
2. 循环无终止条件 → 死循环卡死
3. 同时写 Command + 外部条件边 → Command 优先级更高，外部失效
4. interrupt 不配置 checkpointer → 无法暂停恢复
5. Send 传参和 State 字段重名 → Send 参数优先覆盖

---

### 11. 流程控制全套导入一键复制

```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import (
    Command,
    Send,
    interrupt
)
```

需要我给你写一份**「顺序+分支+循环+并行+中断」五合一完整路由实战代码**吗？直接可运行。

## 二、状态（State）& 合并器（Reducer）

这是 **LangGraph 最核心、最容易踩坑**的知识点，没有之一。
我会用**最通俗、最完整、可直接落地**的方式讲透：

- 状态是什么
- 为什么需要合并器
- 内置合并器怎么用
- 自定义合并器怎么写
- 字段级 Annotated 精确控制
- 实战避坑

---

### 一、核心一句话

**LangGraph 所有节点都不会“覆盖”状态，只会“合并”状态。
合并规则 = 由 Reducer（合并器）决定。**

- 节点返回 `dict` → 不是覆盖，是**合并更新**
- 列表默认不是替换，是**追加**
- 消息默认不是清空，是**添加**
- 你可以完全自定义每个字段怎么合并

---

### 二、状态（State）两种定义方式

#### 1. TypedDict（最常用）

```python
from typing import TypedDict, Annotated

class MyState(TypedDict):
    messages: list          # 普通字段
    count: int              # 普通字段
```

#### 2. dataclass

```python
from dataclasses import dataclass, field

@dataclass
class MyState:
    messages: list = field(default_factory=list)
    count: int = 0
```

**状态本质：全局共享的只读数据流，节点只能返回更新，不能直接改 state。**

---

### 三、什么是 Reducer（合并器）？

#### 定义

**Reducer = 两个值怎么合并成一个**
格式：
```python
def reducer(left, right) -> 新值:
```
- left：当前状态值
- right：节点返回的新值
- 返回：合并后的值

#### 默认规则（非常重要）

1. **int / str / bool**：**直接覆盖**
2. **list**：**追加（extend）**
3. **dict**：**逐层合并**

---

### 四、内置合并器（开箱即用）

#### 1. add_messages（最常用！对话专用）

自动合并消息，去重、保持顺序、兼容 LangChain 消息格式
```python
from langgraph.graph import add_messages
```

使用方式：
```python
class State(TypedDict):
    messages: Annotated[list, add_messages]
```

效果：
- 节点返回 `{"messages": [new_msg]}`
- 不会覆盖，只会**追加**到列表末尾

#### 2. operator.add（数值累加）

```python
import operator

class State(TypedDict):
    count: Annotated[int, operator.add]
```

效果：
- 当前 count=5
- 节点返回 `{"count":3}`
- 最终 count = 5+3 = **8**

#### 3. operator.or_（布尔 OR 合并）

```python
class State(TypedDict):
    success: Annotated[bool, operator.or_]
```

只要有一个节点返回 True，最终就是 True。

---

### 五、Annotated 字段级合并器（核心技能）

LangGraph 最强大的功能：
**每个字段可以用不同的合并规则！**

语法：
```python
字段名: Annotated[类型, 合并器]
```

示例：
```python
class State(TypedDict):
    # 消息：追加
    messages: Annotated[list, add_messages]
    
    # 计数：累加
    count: Annotated[int, operator.add]
    
    # 结果：覆盖
    result: str
    
    # 日志：自定义合并
    logs: Annotated[list, custom_log_reducer]
```

---

### 六、自定义 Reducer（万能）

你可以完全控制任何字段的合并逻辑。

#### 示例 1：列表去重合并

```python
def unique_reducer(current: list, new: list) -> list:
    combined = current + new
    return list(dict.fromkeys(combined))  # 去重保留顺序
```

使用：
```python
class State(TypedDict):
    unique_list: Annotated[list, unique_reducer]
```

#### 示例 2：总是覆盖（不追加）

```python
def override_reducer(current, new):
    return new  # 直接用新值替换旧值
```

#### 示例 3：只在为空时设置

```python
def set_if_empty(current, new):
    return current if current else new
```

---

### 七、完整可运行实战（必看）

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END, add_messages
import operator

# ----------------------
# 1. 定义带合并器的状态
# ----------------------
class State(TypedDict):
    messages: Annotated[list, add_messages]   # 消息追加
    count: Annotated[int, operator.add]      # 计数累加
    result: str                              # 普通覆盖
    logs: Annotated[list, lambda a,b: a+b]    # 列表追加

# ----------------------
# 2. 节点
# ----------------------
def node1(state: State):
    return {
        "messages": ["节点1执行"],
        "count": 1,
        "result": "节点1结果",
        "logs": ["log1"]
    }

def node2(state: State):
    return {
        "messages": ["节点2执行"],
        "count": 1,
        "result": "节点2结果",  # 覆盖！
        "logs": ["log2"]
    }

# ----------------------
# 3. 构建图
# ----------------------
builder = StateGraph(State)
builder.add_node("n1", node1)
builder.add_node("n2", node2)

builder.add_edge(START, "n1")
builder.add_edge("n1", "n2")
builder.add_edge("n2", END)

graph = builder.compile()

# ----------------------
# 4. 运行
# ----------------------
res = graph.invoke({
    "messages": [],
    "count": 0,
    "result": "",
    "logs": []
})

print(res)
```

#### 输出结果（你能完全看懂就精通了状态合并）

```
{
    'messages': ['节点1执行', '节点2执行'],  # 追加
    'count': 2,                              # 累加 0+1+1
    'result': '节点2结果',                   # 覆盖
    'logs': ['log1', 'log2']                 # 追加
}
```

---

### 八、执行机制（必懂）

1. 节点**只读**状态
2. 节点返回**更新字典**
3. 引擎**按每个字段的 Reducer 合并**到全局状态
4. 进入下一个节点

---

### 九、最常见的 6 个坑

#### 1. 以为返回 dict 会覆盖列表 → 错！默认是追加

要用覆盖必须写 `override_reducer`

#### 2. 忘记 Annotated → 合并器不生效

```python
# 错误
messages: list[add_messages]

# 正确
messages: Annotated[list, add_messages]
```

#### 3. 直接修改 state 对象 → 不生效！

```python
# 错误 ❌
def bad(state):
    state["messages"].append("hi")

# 正确 ✅
def good(state):
    return {"messages": ["hi"]}
```

#### 4. 多个节点同时更新同一个非列表字段 → 覆盖

#### 5. 子图与主图状态必须兼容

#### 6. 不设置 Reducer 就想实现消息累加 → 必错

---

### 十、速查模板（直接复制用）

```python
from typing import TypedDict, Annotated
from langgraph.graph import add_messages
import operator

# 覆盖
def override(left, right): return right

# 累加
def add(left, right): return left + right

# 去重
def unique(left, right):
    return list(dict.fromkeys(left + right))

# 最终状态模板
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    score: Annotated[int, operator.add]
    result: str
    history: Annotated[list, unique]
```

---

### 总结（最核心 3 句）

1. **State 是全局只读数据**
2. **节点只返回更新，不直接修改状态**
3. **每个字段的合并规则由 Reducer 决定**
   - `Annotated[类型, 合并器]`

---

需要我给你一份 **「10 种常用 Reducer 工具箱」** 或者 **「多智能体状态最佳实践」** 吗？

## 三、节点专用组件

### 一、整体概述

节点是 LangGraph 执行逻辑的最小单元，**节点组件 = 标准化节点 + 注入参数 + 工具专用封装 + 配置上下文**
专门用来统一：LLM推理、工具调用、参数注入、上下文透传、权限配置等场景

**核心导入**
```python
from langgraph.prebuilt import ToolNode, create_react_agent
from langgraph.types import InjectedToolCallId
from langchain_core.runnables import RunnableConfig
from langchain_core.tools import tool
```

---

### 二、1. 标准内置节点

#### 1. ToolNode 工具执行专用节点

##### 作用

**统一解析模型输出的工具调用、自动执行工具、封装结果返回**
不用手动解析 `tool_calls`、不用手写工具分发，开箱即用。

##### 核心特点

- 自动识别 `messages[-1].tool_calls`
- 批量串行/并行执行工具
- 自动拼装工具返回消息格式
- 完美适配 React 智能体流程

##### 基础用法

```python
from langgraph.prebuilt import ToolNode

# 定义工具
@tool
def search(query: str) -> str:
    """搜索信息"""
    return f"搜索结果：{query}"

tools = [search]
# 初始化工具节点
tool_node = ToolNode(tools=tools)

# 直接加入图
builder.add_node("tools", tool_node)
```

##### 常用参数

```python
ToolNode(
    tools: list,                # 工具列表
    name: str = "tools",        # 节点名
    handle_tool_errors: bool = True, # 是否捕获工具异常
    parallel_tool_calls: bool = True # 多工具是否并行执行
)
```

##### 标准智能体流程搭配

```
START → LLM推理节点 → 条件判断是否调用工具
    是 → ToolNode 执行工具 → 回到LLM
    否 → END
```

---

#### 2. create_react_agent 一键生成React智能体节点

##### 作用

**一行代码生成完整 ReAct 模式智能体**
内置：思考→工具调用→结果汇总全流程，企业级快速搭建Agent

```python
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    model=llm,
    tools=tools,
    checkpointer=None,
    state_schema=None
)
# 直接作为节点加入图
builder.add_node("react_agent", agent)
```

---

### 三、2. 节点内置注入参数（高频必用）

#### 1. RunnableConfig 全局上下文配置

##### 作用

在**任意节点、任意工具**中获取全局透传参数：
会话ID、用户ID、租户ID、权限、自定义业务参数

##### 结构

```python
RunnableConfig(
    configurable: dict  # 自定义配置字典
)
```

##### 节点中使用

```python
def business_node(state, config: RunnableConfig):
    # 取出全局参数
    thread_id = config["configurable"]["thread_id"]
    user_id = config["configurable"]["user_id"]
    return {}
```

##### 调用时传入

```python
graph.invoke(
    {"messages": ["你好"]},
    config={"configurable": {"thread_id":"1001", "user_id":"u001"}}
)
```

##### 工具内同样可注入

```python
@tool
def get_user_info(config: RunnableConfig):
    uid = config["configurable"]["user_id"]
    return f"当前用户：{uid}"
```

---

#### 2. InjectedToolCallId 工具调用ID注入

##### 作用

自动注入**当前工具调用唯一ID**
常用于：
- 工具内部状态标记
- 工具执行后定向路由
- Command 跳转绑定本次调用上下文

##### 用法

```python
from langgraph.types import InjectedToolCallId
from langgraph.types import Command

@tool
def safe_check(
    content: str,
    tool_call_id: InjectedToolCallId  # 自动注入
) -> Command:
    """敏感内容审核工具"""
    if len(content) > 100:
        # 携带工具ID更新状态并跳转
        return Command(
            update={"audit_id": tool_call_id},
            goto="human_audit"
        )
    return {"result":"通过"}
```

**规则**
- 无需手动传参，LangGraph自动注入
- 仅在 `ToolNode` 调度的工具内生效

---

### 四、3. 自定义普通节点 标准范式

#### 无配置基础节点

```python
def normal_node(state: AgentState) -> dict:
    # 只读状态
    msg = state["messages"]
    # 业务逻辑
    # 返回状态更新
    return {"messages": [("ai", "回复内容")]}
```

#### 带配置高阶节点

```python
def advance_node(state: AgentState, config: RunnableConfig) -> dict:
    # 读取上下文
    app_key = config["configurable"].get("app_key")
    # 逻辑处理
    return {}
```

#### 返回Command路由节点（最强节点）

```python
def route_node(state) -> Command:
    if state["need_search"]:
        return Command(goto="tools", update={})
    return Command(goto="answer", update={})
```

---

### 五、4. 消息专用节点组件

#### 1. MessagesState 内置对话状态节点

无需手动定义 TypedDict，**开箱即用对话状态**
自带 `messages` + `add_messages` 合并器
```python
from langgraph.graph import MessagesState

# 直接使用
class ChatState(MessagesState):
    # 额外扩展字段
    query: str
    source: str
```

#### 2. 消息格式化工具

```python
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    ToolMessage,
    SystemMessage
)
```
节点内统一构造标准消息，兼容所有LLM与ToolNode

---

### 六、5. 节点执行生命周期回调组件

#### CallbackHandler 节点全生命周期监听

监听节点执行全流程：
- 节点开始执行
- 节点执行结束
- LLM调用开始/结束
- 工具调用开始/结束
- 异常报错

```python
from langchain_core.callbacks import BaseCallbackHandler

class NodeMonitor(BaseCallbackHandler):
    def on_chain_start(self, serialized, inputs, **kwargs):
        print("节点开始执行")
```
初始化图时传入 `callbacks=[monitor]` 全局监听所有节点

---

### 七、6. 子图节点封装

#### 作用

把一整套流程**封装成单个节点**，实现业务解耦
```python
# 1. 构建子图
sub_builder = StateGraph(SubState)
# ...子图逻辑...
sub_graph = sub_builder.compile()

# 2. 主图直接当作节点使用
builder.add_node("sub_work_flow", sub_graph)
```
子图 = 一个**可复用大型节点**

---

### 八、7. 节点异常处理组件

#### 1. ToolNode 内置异常捕获

```python
tool_node = ToolNode(tools=tools, handle_tool_errors=True)
```
工具报错自动封装错误消息进messages，不中断流程

#### 2. 节点手动异常捕获

```python
def safe_node(state):
    try:
        # 业务逻辑
        return {}
    except Exception as e:
        return {"error": str(e)}
```

---

### 九、节点组件分类速查表

| 组件                | 包路径                   | 核心用途            |
| ------------------- | ------------------------ | ------------------- |
| ToolNode            | langgraph.prebuilt       | 统一执行工具调用    |
| create_react_agent  | langgraph.prebuilt       | 快速创建ReAct智能体 |
| RunnableConfig      | langchain_core.runnables | 全局上下文参数透传  |
| InjectedToolCallId  | langgraph.types          | 注入工具调用ID      |
| MessagesState       | langgraph.graph          | 内置对话状态        |
| 各类Message         | langchain_core.messages  | 构造标准对话消息    |
| BaseCallbackHandler | langchain_core.callbacks | 节点生命周期监听    |

---

### 十、节点开发三大规范（必遵守）

1. **节点只读State，只返回更新字典，禁止原地修改state**
```python
# 禁止
state["messages"].append(...)
# 正确
return {"messages": [...]}
```

2. **通用业务节点统一接收 `config: RunnableConfig`**
方便后期扩展用户、权限、租户、会话信息

3. **工具逻辑统一交给 ToolNode，不要手写工具分发**
减少冗余代码，统一异常、统一消息格式

---

### 十一、最简组合实战（可直接运行）

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END, add_messages
from langgraph.prebuilt import ToolNode
from langchain_core.tools import tool
from langchain_core.runnables import RunnableConfig

# 状态
class State(TypedDict):
    messages: Annotated[list, add_messages]

# 工具
@tool
def weather(city: str, config: RunnableConfig) -> str:
    user = config["configurable"].get("user","未知用户")
    return f"{user}查询 {city} 天气晴朗"

tools = [weather]
tool_node = ToolNode(tools)

# LLM模拟节点
def llm_node(state):
    return {"messages": [{"role":"tool_call","name":"weather","args":{"city":"北京"}}]}

# 构图
builder = StateGraph(State)
builder.add_node("llm", llm_node)
builder.add_node("tools", tool_node)
builder.add_edge(START, "llm")
builder.add_edge("llm", "tools")
builder.add_edge("tools", END)

graph = builder.compile()
print(graph.invoke({"messages":[]}, config={"configurable":{"user":"张三"}}))
```

---

需要我把**节点组件 + 状态合并器 + 流程路由**三者整合，给你写一套**企业级标准 LangGraph Agent 完整架构模板**吗？

## 四、记忆 & 持久化（Checkpointer 全家桶）

这是 **LangGraph 最核心、生产环境必须用**的模块，没有它就做不到**多轮对话、断点续跑、人工审核、历史回溯**。

我会用**最清晰、最落地、最不绕弯**的方式讲透：

---

### 一、一句话核心定位

**Checkpointer = 图的记忆体 + 持久化引擎**
- 自动保存**每一步状态**
- 支持**中断、恢复、重试、回溯**
- 支持**多用户会话隔离（thread_id）**
- 支持**内存、本地文件、数据库**多种存储

> 没有 Checkpointer → 流程跑完就忘，不能中断，不能多轮对话。

---

### 二、核心概念（必须懂）

#### 1. Checkpoint（快照）

图执行**每一步**都会保存一个快照：
- 完整 State
- 当前执行到哪个节点
- 边、路由、配置信息

#### 2. Thread（会话/线程）

**每个用户一个独立 thread_id**，数据完全隔离。
- 你对话 = 一个 thread
- 他对话 = 另一个 thread
- 永不混淆

#### 3. Resume（恢复）

中断后用 `Command(resume=...)` 唤醒流程。

#### 4. Persistence（持久化）

- 内存：重启丢失
- SQLite / Postgres：永久保存

---

### 三、Checkpointer 全家桶（全部 5 个）

```python
# 内存（测试用）
from langgraph.checkpoint.memory import MemorySaver

# 本地文件（小型项目）
from langgraph.checkpoint.sqlite import SqliteSaver

# 生产分布式（大型项目）
from langgraph.checkpoint.postgres import PostgresSaver

# 基类（自定义存储）
from langgraph.checkpoint.base import BaseCheckpointSaver

# 同步/异步适配
from langgraph.checkpoint import AnyCheckpointSaver
```

#### 按项目规模直接选：

- **本地调试** → MemorySaver
- **个人/小型项目** → SqliteSaver
- **生产/多服务/高可用** → PostgresSaver
- **自研存储（Redis/Mongo）** → 继承 BaseCheckpointSaver

---

### 四、1. MemorySaver（内存版，最简单）

#### 作用

- 把快照存在**内存**中
- 程序重启 → 记忆清空
- 适合**调试、demo、测试**

#### 代码

```python
from langgraph.checkpoint.memory import MemorySaver

# 1. 创建记忆
checkpointer = MemorySaver()

# 2. 编译图时传入
graph = builder.compile(checkpointer=checkpointer)
```

#### 调用必须带 thread_id

```python
config = {"configurable": {"thread_id": "user_123"}}

graph.invoke({"messages": ["你好"]}, config=config)
```

---

### 五、2. SqliteSaver（本地文件持久化，推荐个人用）

#### 作用

- 存在 **checkpoints.db**
- 重启不丢失
- 单机完美

#### 使用

```python
from langgraph.checkpoint.sqlite import SqliteSaver
import sqlite3

# 连接数据库
conn = sqlite3.connect("checkpoints.db")
checkpointer = SqliteSaver(conn)

graph = builder.compile(checkpointer=checkpointer)
```

---

### 六、3. PostgresSaver（生产级，企业必备）

#### 作用

- 多服务共享
- 分布式部署
- 高并发、高可靠

#### 使用

```python
from langgraph.checkpoint.postgres import PostgresSaver
from psycopg_pool import ConnectionPool

DSL = "postgresql://user:pass@host:5432/db"
pool = ConnectionPool(DSL)

checkpointer = PostgresSaver(pool)
checkpointer.setup()  # 建表

graph = builder.compile(checkpointer=checkpointer)
```

---

### 七、4. BaseCheckpointSaver（自定义存储）

如果你想存在 **Redis、Mongo、MySQL** 等，继承这个基类实现：
- save
- get
- list

适合大厂自研存储。

---

### 八、记忆核心功能（全部讲透）

#### 1. 多轮对话（最常用）

```python
# 第一次
config = {"configurable": {"thread_id": "user_1"}}
graph.invoke({"messages": ["我是张三"]}, config=config)

# 第二次（自动记住）
graph.invoke({"messages": ["我叫什么？"]}, config=config)
```

#### 2. 中断 + 人工审核（HITL）

```python
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["danger_node"]  # 执行前暂停
)
```

恢复：
```python
graph.invoke(Command(resume="yes"), config=config)
```

#### 3. 查看历史记录

```python
# 获取所有历史快照
history = checkpointer.list(config)
```

#### 4. 回溯到之前步骤

```python
checkpointer.put(config, old_snapshot)
```

#### 5. 清空记忆

```python
checkpointer.clear(config)
```

#### 6. 多节点并发安全（Postgres 支持）

分布式多实例同时调用不冲突。

---

### 九、config 结构（必须掌握）

```python
config = {
    "configurable": {
        "thread_id": "user_001",      # 会话ID（必须）
        "checkpoint_id": "xxx",       # 可选：指定恢复到某一步
        "user_id": "u_123",           # 自定义
        "tenant_id": "t_456"          # 自定义
    }
}
```

---

### 十、完整可运行代码（记忆 + 中断 + 多轮对话）

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated
from langgraph.graph import add_messages
from langgraph.types import Command

# 状态
class State(TypedDict):
    messages: Annotated[list, add_messages]

# 节点
def hello_node(state):
    return {"messages": ["你好！"]}

def wait_node(state):
    return {"messages": ["等待人工审核..."]}

# 构建
builder = StateGraph(State)
builder.add_node("hello", hello_node)
builder.add_node("wait", wait_node)

builder.add_edge(START, "hello")
builder.add_edge("hello", "wait")
builder.add_edge("wait", END)

# 记忆
checkpointer = MemorySaver()

# 编译 + 中断
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_after=["wait"]
)

# 执行
config = {"configurable": {"thread_id": "test1"}}
graph.invoke({"messages": []}, config=config)

# 恢复
graph.invoke(Command(resume="ok"), config=config)
```

---

### 十一、最常见的 6 个坑（90% 的人都踩过）

#### 1. 没传 checkpointer → 不能中断、没有记忆

#### 2. 没传 thread_id → 记忆不生效

#### 3. 每次用不同 thread_id → 记不住对话

#### 4. Sqlite 多线程报错 → 必须用单连接

#### 5. 中断后不 resume 直接重新 invoke → 重新跑流程

#### 6. 生产用 MemorySaver → 重启丢失所有用户对话

---

### 十二、终极速查表（直接复制）

```python
# 内存记忆
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()

# SQLite
import sqlite3
from langgraph.checkpoint.sqlite import SqliteSaver
conn = sqlite3.connect("db.db")
checkpointer = SqliteSaver(conn)

# Postgres
from langgraph.checkpoint.postgres import PostgresSaver
checkpointer = PostgresSaver(pool)

# 编译
graph = builder.compile(checkpointer=checkpointer)

# 调用
config = {"configurable": {"thread_id": "user_1"}}
graph.invoke(..., config=config)
```

---

### 十三、总结（最核心 3 句）

1. **Checkpointer = 记忆 = 多轮对话 = 中断恢复**
2. **必须用 thread_id 区分用户**
3. **开发用 Memory，个人用 Sqlite，生产用 Postgres**

---

如果你愿意，我可以马上给你写一套：
**「LangGraph 企业级标准架构：状态+路由+节点+记忆+人工审核+多轮对话」完整版可直接上线使用**

## 五、人机交互 HITL 核心

**HITL = Human-in-the-Loop 人机介入**
流程运行中**主动暂停**，等待人工输入/审批/确认，再继续执行，是智能体落地风控、审核、决策的必备能力。

### 一、核心原理

1. 图执行到指定节点自动**冻结状态**
2. 保存当前快照到 `Checkpointer`
3. 让出执行权，等待外部传入**恢复值**
4. 拿到恢复值后，从断点处继续向后执行
5. 全程**状态不丢失、上下文完整连贯**

### 二、三大核心实现方式

#### 1. 编译级断点（最常用、企业首选）

在 `compile` 时全局配置暂停规则，**不用改业务节点代码**
##### 两个关键字

- `interrupt_before`：**进入节点前暂停**
- `interrupt_after`：**执行完节点后暂停**

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

graph = builder.compile(
    checkpointer=checkpointer,
    # 即将执行 risk_tool 节点时暂停
    interrupt_before=["risk_tool"],
    # 执行完 audit_result 节点后暂停
    interrupt_after=["audit_result"]
)
```

##### 适用场景

- 高危工具调用前人工审批
- 敏感操作二次确认
- 固定流程卡点等待确认

---

#### 2. 代码内手动中断 `interrupt()`

在**节点函数内部**主动暂停，可携带提示文案，灵活性最高
```python
from langgraph.types import interrupt

def confirm_node(state):
    # 暂停流程，弹出提示，阻塞执行
    user_choice = interrupt("是否同意执行当前操作？输入 yes / no")
    
    if user_choice == "yes":
        return {}
    else:
        return Command(goto=END)
```
- 可自定义提示语
- 可动态判断是否需要暂停
- 适合**动态条件式人机等待**

---

#### 3. 外部恢复统一入口 `Command(resume)`

所有暂停流程，**统一用这个 API 唤醒**
```python
from langgraph.types import Command

# 传入恢复值，继续往下跑
graph.invoke(Command(resume="yes"), thread_config)
```
- `resume` 传入的值，会被 `interrupt()` 函数接收
- 支持字符串、数字、字典、复杂对象任意格式

#### 三、标准完整流程（固定五步）

1. 初始化 `Checkpointer`（必须有，否则无法存断点）
2. 构图 + 配置 `interrupt_before / after`
3. 首次执行走到断点，**自动暂停**
4. 前端/控制台获取暂停状态，展示给用户
5. 用户输入确认后，调用 `Command(resume=xxx)` 恢复执行

### 四、获取当前是否处于暂停状态

```python
# 获取最新状态快照
snapshot = graph.get_state(thread_config)

# 判断是否暂停
if snapshot.next:
    print("流程已暂停，等待人工输入")
    print("下一步将要执行节点：", snapshot.next)
```
- `snapshot.next` 不为空 = 处于中断等待态
- 可用于前端渲染：展示等待弹窗、审核面板

### 五、恢复值传递机制

1. 节点内 `val = interrupt("提示")`
2. 外部 `Command(resume=实际值)`
3. `val` 直接接收 `resume` 内容
**双向直通，无额外参数转发**

示例：
```python
# 节点内
reason = interrupt("请输入驳回原因")

# 外部恢复
graph.invoke(Command(resume="内容违规驳回"), config)
# 此时 reason = "内容违规驳回"
```

### 六、HITL 搭配路由组合用法

#### 1. 审批通过走工具，拒绝直接结束

```python
def audit_pass_node(state):
    decision = interrupt("审批是否通过")
    if decision == "pass":
        return Command(goto="call_tool")
    else:
        return Command(goto=END)
```

#### 2. 暂停后动态修改状态再继续

拿到快照 → 修改 state → 写入快照 → 再恢复执行
```python
snapshot = graph.get_state(config)
# 修改状态
snapshot.values["query"] = "修改后的问题"
# 覆写状态
graph.update_state(config, snapshot.values)
# 继续执行
graph.invoke(None, config)
```

### 七、HITL 常用实战场景

1. **联网搜索前人工确认是否搜索**
2. **支付/下单/高危操作二次审核**
3. **AI 生成内容人工校对后再输出**
4. **多轮对话中途修正用户意图**
5. **流程异常暂停，人工介入排错**
6. **表单补全：缺少字段暂停等待用户填写**

### 八、HITL 核心限制 & 注意事项

1. **必须依赖 Checkpointer**
   无持久化存储无法保存断点，中断直接失效
2. **必须携带唯一 thread_id**
   不同会话断点互相隔离，不会串流程
3. **中断期间不能重新从头 invoke**
   会重置流程，丢失暂停状态
4. **流式 stream 同样支持中断**
   stream 读到暂停节点自动停止，恢复后继续流式输出
5. **子图内也支持 HITL**
   子图内部中断，主图同样感知暂停状态

### 九、最简可运行 HITL 完整代码

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated
from langgraph.graph import add_messages
from langgraph.types import interrupt, Command

# 状态
class State(TypedDict):
    messages: Annotated[list, add_messages]

# 普通节点
def start_node(state):
    return {"messages": ["任务准备完成"]}

# 人机审核节点
def human_audit(state):
    # 主动暂停
    res = interrupt("确认执行任务？y/n")
    if res == "n":
        return Command(goto=END)
    return {}

# 执行节点
def run_task(state):
    return {"messages": ["任务已执行完毕"]}

# 构图
builder = StateGraph(State)
builder.add_node("start", start_node)
builder.add_node("audit", human_audit)
builder.add_node("run", run_task)

builder.add_edge(START, "start")
builder.add_edge("start", "audit")
builder.add_edge("audit", "run")
builder.add_edge("run", END)

# 记忆持久化
check = MemorySaver()
graph = builder.compile(checkpointer=check)

# 会话配置
thread_cfg = {"configurable": {"thread_id": "hitl_001"}}

# 1. 执行到审核处暂停
graph.invoke({"messages": []}, thread_cfg)

# 2. 人工确认恢复执行
graph.invoke(Command(resume="y"), thread_cfg)
print(graph.get_state(thread_cfg).values)
```

### 十、HITL 核心API速记

```python
# 1. 编译级断点
interrupt_before = ["节点名"]
interrupt_after = ["节点名"]

# 2. 代码内暂停
from langgraph.types import interrupt
ret = interrupt("提示文案")

# 3. 恢复流程
from langgraph.types import Command
Command(resume=传入值)

# 4. 查询暂停状态
graph.get_state(config)

# 5. 手动修改断点状态
graph.update_state(config, new_state_dict)
```

### 十一、选型建议

- **固定流程卡点** → 用 `interrupt_before / after`
- **动态灵活等待、带交互文案** → 用内置 `interrupt()`
- **前后端对接、前端弹窗审核** → 统一 `get_state` 判断状态 + `Command` 恢复

需要我给你写**前后端分离版 HITL 接口封装**，直接对接 Web 页面做人工审核吗？

## 六、子图 & 模块化

### 一、核心概念

**子图 Subgraph**：把一组关联节点+边封装成独立可复用流程图，**当作普通节点嵌入主图**
- 作用：解耦复杂业务、拆分大流程、逻辑分层、统一复用
- 本质：`StateGraph.compile()` 编译后的可运行图 = 标准可加入主图的节点
- 通信：**主图与子图共用一套 State**，天然共享状态，无需额外传参

### 二、核心优势

1. **业务拆分**：大流程拆成登录、查询、审批、总结等子流程
2. **代码解耦**：不同模块单独编写、单独测试
3. **复用能力**：同一子图可在主图多处调用
4. **层级清晰**：主子图结构直观，易维护易排错
5. **统一状态**：全局一份 State，数据互通无隔阂

### 三、主子图状态规则

1. **默认共用同一个 State 类型**，字段完全互通
2. 子图内修改状态，**自动同步到主图**
3. 可定义**子图专属扩展字段**，主图也能读取
4. 合并器规则全局统一，主子图行为一致

### 四、两种创建使用方式

#### 方式1：同状态主子图（最常用）

主子图使用**同一个 State**，直接嵌入
```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END, add_messages
from langgraph.types import Command

# 全局统一状态
class GlobalState(TypedDict):
    messages: Annotated[list, add_messages]
    task_type: str
    result: str
```

##### 1）编写子图

```python
# 构建子图：数据处理子流程
def sub_process_node(state):
    return {"result": f"处理完成：{state['task_type']}"}

# 初始化子图构造器
sub_builder = StateGraph(GlobalState)
sub_builder.add_node("sub_process", sub_process_node)
sub_builder.add_edge(START, "sub_process")
sub_builder.add_edge("sub_process", END)

# 编译得到子图（可直接当节点用）
sub_graph = sub_builder.compile()
```

##### 2）主图引入子图

```python
# 主图节点
def main_entry(state):
    return {"task_type": "数据分析"}

# 主图构图
main_builder = StateGraph(GlobalState)
main_builder.add_node("main_entry", main_entry)
# 把子图直接添加为主图节点
main_builder.add_node("data_sub_flow", sub_graph)

# 主图流程
main_builder.add_edge(START, "main_entry")
main_builder.add_edge("main_entry", "data_sub_flow")
main_builder.add_edge("data_sub_flow", END)

main_graph = main_builder.compile()
```

##### 3）调用执行

```python
res = main_graph.invoke({"messages": []})
print(res)
```

---

#### 方式2：独立状态子图（字段映射）

子图自有独立 State，通过**节点转发**做字段映射转换，适合完全独立业务模块
1. 定义子图私有 State
2. 子图内部独立流转
3. 主图入口节点映射字段传入
4. 子图出口节点映射结果回写主状态

适用：第三方独立流程、跨业务隔离模块

### 五、子图内部路由全部支持

子图内部可以任意使用：
- 普通边 `add_edge`
- 条件分支 `add_conditional_edges`
- 动态路由 `Command(goto=...)`
- 并行节点
- HITL人机中断 `interrupt`
- Checkpointer 记忆持久化
- ToolNode 工具调用

**子图内部中断，主图同样感知暂停状态**

### 六、主子图互相跳转（Command 跨图路由）

#### 1. 子图跳回父主图

```python
def sub_back_to_parent(state) -> Command:
    return Command(
        update={},
        goto="main_target_node",  # 主图节点名
        graph=Command.PARENT      # 固定标识：回到父图
    )
```

#### 2. 主图主动跳入指定子图

直接边连接子图节点名即可，无需特殊语法

### 七、Send 跨图传参调用子图

不依赖全局状态，**单独传局部参数**调用子图内部节点
```python
def main_dispatch(state) -> Command:
    return Command(
        goto=Send("sub_inner_node", {"custom_param": "自定义参数"})
    )
```
子图节点优先接收 Send 传入参数，再合并全局 State

### 八、模块化工程目录最佳实践

```
langgraph_agent/
├── main_graph.py        # 主流程入口图
├── states.py            # 全局状态统一定义
├── subgraphs/
│   ├── __init__.py
│   ├── search_sub.py    # 搜索子图
│   ├── audit_sub.py     # 审核子图
│   └── summary_sub.py   # 总结子图
├── nodes/               # 通用公共节点
├── tools/               # 工具集合
└── config.py            # 记忆、配置、线程管理
```
- 每个子图单独文件编写，导出编译好的 `sub_graph`
- 主图统一导入组装，实现大型项目工程化

### 九、子图可视化

主子图嵌套结构同样支持绘图，自动展示层级关系
```python
main_graph.get_graph().draw_mermaid_png()
```

### 十、常见使用场景

1. **智能体多职能拆分**：思考子图、工具调用子图、回复整理子图
2. **业务流程分段**：表单填写→校验子图→提交子图→结果通知子图
3. **权限分级流程**：普通流程子图 + 管理员审批子图
4. **多轮会话分层**：意图识别子图 + 业务执行子图
5. **定时/批量任务**：批量拆分任务子图 + 单任务执行子图

### 十一、避坑要点

1. **子图必须 compile 之后才能加入主图**，不能直接加 builder
2. 共用状态时，字段命名尽量统一，避免冲突
3. 子图内部 END 只代表**子图结束**，自动退回主图继续执行
4. 全局 checkpointer 配置一次，主子图共用记忆快照
5. 不要在子图内直接修改原始 state 对象，依旧遵循**只返回更新字典**原则
6. 多层嵌套子图跳转，统一使用 `graph=Command.PARENT` 向上回溯

### 十二、极简完整可运行示例

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END, add_messages
from langgraph.types import Command

# 全局状态
class State(TypedDict):
    messages: Annotated[list, add_messages]
    content: str
    final_out: str

# ========== 子图定义 ==========
def sub_handle(state):
    return {"content": f"子图处理：{state['content']}"}

sub_builder = StateGraph(State)
sub_builder.add_node("handle", sub_handle)
sub_builder.add_edge(START, "handle")
sub_builder.add_edge("handle", END)
sub_graph = sub_builder.compile()

# ========== 主图定义 ==========
def main_in(state):
    return {"content": "原始业务数据"}

def main_out(state):
    return {"final_out": state["content"] + " 主图最终封装"}

main_builder = StateGraph(State)
main_builder.add_node("input", main_in)
main_builder.add_node("sub_module", sub_graph)
main_builder.add_node("output", main_out)

main_builder.add_edge(START, "input")
main_builder.add_edge("input", "sub_module")
main_builder.add_edge("sub_module", "output")
main_builder.add_edge("output", END)

graph = main_builder.compile()
print(graph.invoke({"messages": []}))
```

### 十三、核心API速记

```python
# 1. 创建子图
sub_builder = StateGraph(State)
# 2. 子图构图完成编译
sub_graph = sub_builder.compile()
# 3. 主图添加子图为节点
main_builder.add_node("sub_name", sub_graph)
# 4. 子图回跳父图
Command(goto="父图节点", graph=Command.PARENT)
# 5. 带参调用子图内部节点
Send("子图节点名", 参数字典)
```

需要我给你搭建**企业级多子图分层架构模板**，直接拆分意图识别、工具调用、结果整理三大子图吗？

## 七、流式输出 & 事件回调

这是 **生产级必备**：前端实时打字机效果、监控节点执行、日志上报、异常捕获、进度展示全靠它。

我会一次性讲透：
- 流式输出（stream）3 种模式
- 事件回调（Callbacks）全生命周期
- 实战可直接复制的代码
- 前端对接最佳实践

---

### 一、核心一句话

- **流式输出**：让 AI 回答像打字机一样**逐字/逐节点返回**，不用等全部跑完
- **事件回调**：监听图执行**每一个事件**（节点开始/结束、工具调用、LLM 调用、报错）

---

### 二、流式输出 Stream（最常用）

#### 1. 三种 stream_mode（必须掌握）

```python
graph.stream(input, config, stream_mode="values")
```

##### ① `values`（全局状态流）

- 每一步**完整状态**推送到前端
- 适合：状态监控、实时展示全部数据
```python
for chunk in graph.stream(inputs, stream_mode="values"):
    print(chunk)
```

##### ② `updates`（仅更新部分，推荐！）

- 只返回**节点更新的字典**，不返回全量状态
- 流量小、速度快、前端最常用
```python
for chunk in graph.stream(inputs, stream_mode="updates"):
    print(chunk)
```

##### ③ `debug`（调试流）

- 输出**元数据**：节点名、耗时、路由信息
```python
for chunk in graph.stream(inputs, stream_mode="debug"):
    print(chunk)
```

---

### 三、流式输出实战（打字机效果）

#### 完整可运行代码

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Annotated
from langgraph.graph import add_messages
import time

class State(TypedDict):
    messages: Annotated[list, add_messages]

# 模拟 LLM 流式输出
def llm_node(state):
    full_ans = "我是AI助手，我正在一步一步回复你..."
    for word in full_ans.split(" "):
        time.sleep(0.1)
        yield {"messages": [("assistant", word)]}  # 逐词返回

# 构建
builder = StateGraph(State)
builder.add_node("llm", llm_node)
builder.add_edge(START, "llm")
builder.add_edge("llm", END)
graph = builder.compile()

# 流式调用（updates 模式）
for chunk in graph.stream({"messages": []}, stream_mode="updates"):
    print(chunk, end=" ", flush=True)
```

---

### 四、异步流式 astream（高并发必备）

```python
async for chunk in graph.astream(inputs, stream_mode="updates"):
    print(chunk)
```

---

### 五、事件回调 Callback（全生命周期监控）

#### 作用

监听**所有内部事件**，用于：
- 日志系统
- 监控告警
- 前端推送执行进度
- 审计记录
- 性能统计

#### 核心事件

- `on_chain_start`：节点/图开始
- `on_chain_end`：节点/图结束
- `on_chain_error`：执行报错
- `on_llm_start`：大模型调用开始
- `on_tool_start`：工具调用开始

#### 自定义回调处理器（直接复制用）

```python
from langchain_core.callbacks import BaseCallbackHandler

class MyMonitor(BaseCallbackHandler):
    def on_chain_start(self, serialized, inputs, **kwargs):
        print(f"【开始】节点: {kwargs.get('name')}")

    def on_chain_end(self, outputs, **kwargs):
        print(f"【结束】节点: {kwargs.get('name')} → 输出: {outputs}")

    def on_chain_error(self, error, **kwargs):
        print(f"【异常】错误: {str(error)}")

    def on_tool_start(self, serialized, input_str, **kwargs):
        print(f"【工具调用】{serialized['name']}")
```

#### 使用回调

```python
monitor = MyMonitor()

graph.invoke(
    inputs,
    config={"callbacks": [monitor]}  # 传入回调
)
```

---

### 六、全局回调（所有节点都监听）

```python
graph = builder.compile(
    callbacks=[monitor]  # 全局生效
)
```

---

### 七、流式 + 回调 组合（企业级标准）

```python
for chunk in graph.stream(
    inputs,
    stream_mode="updates",
    config={"callbacks": [monitor]}
):
    # 前端实时推送
    send_to_frontend(chunk)
```

---

### 八、最常用 4 个场景模板

#### 1. 前端打字机效果

```python
for chunk in graph.stream(..., stream_mode="updates"):
    if "messages" in chunk:
        yield chunk["messages"][0]  # SSE 推前端
```

#### 2. 节点执行进度

```python
def on_chain_start(…):
    websocket.send(f"执行中：{node_name}")
```

#### 3. 工具调用日志

```python
def on_tool_start(…):
    logger.info(f"调用工具：{name}")
```

#### 4. 异常自动告警

```python
def on_chain_error(…):
    send_alert(error)
```

---

### 九、核心 API 速查表

| API                      | 用途             |
| ------------------------ | ---------------- |
| `stream(mode="values")`  | 全量状态流       |
| `stream(mode="updates")` | 增量更新（推荐） |
| `stream(mode="debug")`   | 调试信息         |
| `astream`                | 异步流式         |
| `callbacks=[handler]`    | 注册事件监听     |
| `on_chain_start`         | 节点开始         |
| `on_chain_end`           | 节点结束         |
| `on_chain_error`         | 异常捕获         |
| `on_llm_start`           | LLM 调用         |
| `on_tool_start`          | 工具调用         |

---

### 十、最常见 4 个坑

1. **stream 必须用 for 循环接收**
2. **前端必须用 SSE/WebSocket** 推送流式结果
3. **回调不能阻塞执行**（不要写耗时逻辑）
4. **yield 节点必须用流式调用**

---

### 十一、总结（最核心 3 句）

1. **stream_mode="updates" 前端永远首选**
2. **Callback 监听所有事件**
3. **异步 + 流式 + 回调 = 企业级标准架构**

---

用！

## 八、工具生态配套

### 一、整体概述

LangGraph 完全兼容 **LangChain 工具体系**，同时新增**图流程专属工具规范、工具节点、工具路由、工具权限、工具批量调度**能力，是智能体实现**联网查询、函数调用、外部能力对接**的核心生态。

**核心导入**
```python
# 基础工具装饰器
from langchain_core.tools import tool, StructuredTool
# 工具参数校验
from pydantic import BaseModel, Field
# LangGraph 专用工具节点
from langgraph.prebuilt import ToolNode
# 工具注入参数
from langgraph.types import InjectedToolCallId
# 工具上下文配置
from langchain_core.runnables import RunnableConfig
```

---

### 二、1. 标准工具定义体系

#### 1. 最简装饰器工具 @tool（最常用）

##### 基础无参工具

```python
@tool
def get_time() -> str:
    """获取当前系统时间"""
    import datetime
    return str(datetime.datetime.now())
```

##### 带入参结构化工具

```python
@tool
def calculate(a: float, b: float, op: str) -> str:
    """
    数学计算器
    :param a: 数字1
    :param b: 数字2
    :param op: 运算符号 + - * /
    """
    if op == "+":
        return str(a + b)
    elif op == "-":
        return str(a - b)
    return "不支持运算"
```

#### 2. Pydantic 强类型参数工具（企业级首选）

适合参数多、需要严格校验、对接大模型自动填参场景
```python
class SearchInput(BaseModel):
    query: str = Field(description="用户搜索关键词")
    limit: int = Field(default=5, description="返回结果条数")

@tool(args_schema=SearchInput)
def web_search(query: str, limit: int) -> str:
    """联网搜索外部信息"""
    return f"搜索【{query}】获取{limit}条结果"
```

#### 3. 自定义 StructuredTool 类工具

适合复杂初始化、携带实例、持久化连接的工具
```python
class DbQueryTool(StructuredTool):
    name: str = "db_query"
    description: str = "数据库查询工具"
    args_schema: type[BaseModel] = SearchInput

    def _run(self, query: str, limit: int, **kwargs) -> str:
        return f"数据库查询结果：{query}"
```

---

### 三、2. LangGraph 专用工具增强能力

#### 1. 工具内注入全局上下文 RunnableConfig

工具直接读取**会话ID、用户ID、租户、权限**，无需状态传参
```python
@tool
def user_info_tool(config: RunnableConfig) -> str:
    """获取当前登录用户信息"""
    cfg = config["configurable"]
    uid = cfg.get("user_id", "匿名用户")
    return f"当前操作用户：{uid}"
```

#### 2. 注入工具调用ID InjectedToolCallId

绑定本次调用唯一标识，用于日志追踪、状态标记、定向跳转
```python
@tool
def audit_tool(content: str, tool_call_id: InjectedToolCallId) -> str:
    """内容审核工具"""
    print(f"本次调用ID：{tool_call_id}")
    return "审核通过"
```

#### 3. 工具直接返回 Command 实现流程跳转

**工具执行完直接控制图路由**，不用回到LLM节点再判断
```python
from langgraph.types import Command

@tool
def risky_operate(content: str) -> Command:
    """高危操作工具，执行后直接跳转人工审核"""
    if len(content) > 50:
        return Command(goto="human_audit", update={"risk_content": content})
    return Command(goto="llm_reply")
```

#### 4. 工具异常统一捕获

##### ToolNode 全局异常处理

```python
# 自动捕获工具报错，封装成ToolMessage，不中断流程
tool_node = ToolNode(
    tools=[calculate, web_search],
    handle_tool_errors=True,   # 开启异常捕获
    parallel_tool_calls=True   # 支持并行调用多个工具
)
```

##### 自定义工具异常返回

```python
@tool
def safe_div(a: float, b: float) -> str:
    try:
        return str(a / b)
    except ZeroDivisionError:
        return "错误：除数不能为0"
```

---

### 四、3. ToolNode 工具执行核心节点

#### 核心作用

1. 自动解析 LLM 输出的 `tool_calls` 格式
2. 自动匹配对应工具并执行
3. 自动封装 **ToolMessage** 消息回填状态
4. 支持串行/并行批量调用工具
5. 统一异常收敛

#### 标准智能体工具调用流程

```
START → LLM思考节点 → 判断是否需要调用工具
    是 → ToolNode 批量执行工具 → 结果回到LLM
    否 → 直接结束输出
```

#### 条件路由判断是否调用工具

```python
def tool_route(state):
    last_msg = state["messages"][-1]
    # 判断是否存在工具调用
    if hasattr(last_msg, "tool_calls") and last_msg.tool_calls:
        return "tools"
    return "end_reply"

# 绑定条件边
builder.add_conditional_edges("llm_node", tool_route)
```

---

### 五、4. 工具权限 & 隔离生态

#### 1. 动态挂载工具（运行时切换工具集）

不同用户、不同场景加载不同工具，实现权限管控
```python
# 普通用户工具
user_tools = [get_time, calculate]
# 管理员额外工具
admin_tools = [*user_tools, db_query_tool]
```
根据 `config` 内角色动态初始化 `ToolNode`

#### 2. 工具黑名单/白名单过滤

在LLM输出tool_calls后拦截禁止调用的高危工具

#### 3. 工具调用次数限流

结合 `state` 内调用计数，限制单轮最大调用次数，防止死循环

---

### 六、5. 批量 & 并行工具调度

#### 1. 单次多工具并行执行

开启 `parallel_tool_calls=True`，LLM一次输出多个工具调用，**同时并发执行**

#### 2. Send 批量派发工具任务

拆分任务批量调用同一工具，实现批量处理
```python
def batch_task_dispatch(state):
    tasks = ["任务1", "任务2", "任务3"]
    sends = [Send("tool_exe", {"task": t}) for t in tasks]
    return Command(goto=sends)
```

---

### 七、6. 开箱即用内置常用工具库

可直接导入使用，无需自己实现
```python
# 网页抓取
from langchain_community.tools import WebBrowserTool
# 维基百科查询
from langchain_community.tools import WikipediaQueryRun
# 天气查询
from langchain_community.tools.openweathermap import OpenWeatherMapQueryRun
# 本地文件读写
from langchain_community.tools.file_management import ReadFileTool, WriteFileTool
# 终端命令（谨慎使用）
from langchain_community.tools import ShellTool
```

---

### 八、7. 工具与 HITL 人机交互结合

#### 高危工具调用前强制人工审核

```python
# 编译时在工具节点前暂停
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["tools"]
)
```
用户确认通过后再执行工具，完美风控。

---

### 九、8. 工具记忆与历史调用记录

1. 工具执行结果自动存入全局 `messages` 状态
2. 依托 Checkpointer 自动保存**每一次工具调用记录**
3. 多轮对话可回溯历史工具执行过程与结果

---

### 十、9. 工具生态最佳实践规范

1. **工具职责单一**：一个工具只做一件事
2. **描述写清楚**：`docstring` 决定大模型是否会主动调用
3. **入参结构化**：复杂场景统一使用 Pydantic 声明参数
4. **统一使用 ToolNode**：禁止手写工具分发逻辑
5. **高危工具加拦截**：支付、删数据、联网抓取必须加HITL审核
6. **工具内优先读取 RunnableConfig** 做身份与权限判断
7. **返回结果简洁**，交由LLM统一整理输出

---

### 十一、完整可运行工具调用最简示例

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END, add_messages
from langgraph.prebuilt import ToolNode
from langchain_core.tools import tool

# 1. 定义工具
@tool
def add_num(a:int, b:int) -> str:
    """两数相加"""
    return f"结果：{a+b}"

tools = [add_num]
tool_node = ToolNode(tools)

# 2. 状态
class State(TypedDict):
    messages: Annotated[list, add_messages]

# 3. 模拟LLM生成工具调用
def llm_mock(state):
    return {
        "messages": [
            {"role":"assistant","tool_calls":[{"name":"add_num","args":{"a":10,"b":20}}]}
        ]
    }

# 4. 路由判断
def route_func(state):
    msg = state["messages"][-1]
    if hasattr(msg, "tool_calls") or "tool_calls" in msg:
        return "tools"
    return END

# 5. 构图
builder = StateGraph(State)
builder.add_node("llm", llm_mock)
builder.add_node("tools", tool_node)
builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", route_func)
builder.add_edge("tools", END)

graph = builder.compile()
print(graph.invoke({"messages":[]}))
```

---

### 十二、工具生态速查清单

| 组件                | 用途                 |
| ------------------- | -------------------- |
| @tool               | 快速定义普通工具     |
| StructuredTool      | 自定义类式复杂工具   |
| args_schema         | Pydantic参数强校验   |
| ToolNode            | 统一工具执行调度节点 |
| InjectedToolCallId  | 注入工具调用唯一ID   |
| RunnableConfig      | 注入用户/会话上下文  |
| handle_tool_errors  | 工具全局异常捕获     |
| parallel_tool_calls | 开启多工具并行调用   |
| Send                | 批量派发工具任务     |

---

需要我给你整理一份**LangGraph 常用业务工具合集（天气、搜索、数据库、文件、接口调用）**直接复制即用吗？

## 九、调试 & 可视化

### 一、核心作用

1. **可视化**：一键生成流程图，直观看清节点、流向、分支、嵌套子图
2. **调试**：查看状态变更、执行轨迹、断点、路由走向、报错定位
3. **排错**：快速定位死循环、路由错误、状态合并异常、流程卡死

---

### 二、可视化全套用法

#### 1. 基础获取图结构

```python
graph = builder.compile()
# 获取图对象
graph_struct = graph.get_graph()
```

#### 2. 生成 Mermaid 流程图文本

```python
mermaid_code = graph_struct.draw_mermaid()
print(mermaid_code)
```
可直接粘贴到 [Mermaid 在线编辑器](https://mermaid.live/) 渲染

#### 3. 导出 PNG 图片（最常用）

##### 依赖安装

```bash
pip install pillow
# 需本地安装 graphviz 软件 + 配置环境变量
```

##### 代码导出

```python
# 保存为图片
graph.get_graph().draw_mermaid_png(output_file="langgraph_flow.png")
```

#### 4. 层级可视化（主子图自动分层）

子图嵌套结构会**自动折叠/分层展示**，无需额外配置，编译后直接绘图即可看出层级关系。

#### 5. 可视化配置参数

```python
graph.get_graph(
    xray=True  # 穿透子图，展示内部所有节点
).draw_mermaid_png()
```
- `xray=True`：穿透子图，把嵌套内部节点全部展开
- 默认：只显示主图节点，子图合并为一个块

---

### 三、五大调试手段

#### 1. debug 流式调试模式

**stream_mode="debug"**，输出每一步完整执行元信息
```python
for info in graph.stream(
    inputs,
    stream_mode="debug"
):
    print("="*50)
    print(info)
```
输出内容：
- 当前执行节点
- 入状态、出更新
- 路由跳转目标
- 执行耗时、线程信息
- 中断标记

#### 2. 实时查看状态快照

##### 获取当前最新状态

```python
config = {"configurable":{"thread_id":"1"}}
snapshot = graph.get_state(config)

# 当前完整状态值
print(snapshot.values)
# 下一步将要执行的节点
print("待执行节点：", snapshot.next)
# 是否中断
if snapshot.next:
    print("流程已暂停")
```

##### 查看历史所有快照

```python
# 列出该会话所有历史步骤
history_list = list(checkpointer.list(config))
for idx, step in enumerate(history_list):
    print(f"第{idx}步：{step.values}")
```

#### 3. 手动修改状态调试

断点处直接篡改状态，测试不同分支走向
```python
# 覆写状态
graph.update_state(
    config,
    {"query":"修改后的问题","step":5}
)
# 继续执行
graph.invoke(None, config=config)
```

#### 4. 单步执行调试

借助 Checkpointer 快照，**一步一停**手动推进流程
1. 执行到节点自动暂停
2. 查看状态是否符合预期
3. 确认无误再继续下一步

#### 5. 日志+回调埋点调试

利用之前讲的 `BaseCallbackHandler` 精准打印执行日志
```python
class DebugHandler(BaseCallbackHandler):
    def on_chain_start(self, serialized, inputs, **kwargs):
        print(f"【进入节点】{kwargs.get('name')} 入参：{inputs}")
    def on_chain_end(self, outputs,** kwargs):
        print(f"【退出节点】{kwargs.get('name')} 输出：{outputs}")
```
调用时传入 `callbacks=[DebugHandler()]`

---

### 四、内置调试工具函数

#### 1. 打印图所有节点与边

```python
graph_info = graph.get_graph()
# 所有节点
print(graph_info.nodes)
# 所有流向边
print(graph_info.edges)
```

#### 2. 校验图合法性

```python
# 编译时自动校验：节点是否存在、闭环是否合法
# 非法路由、不存在节点直接抛错
graph = builder.compile()
```

#### 3. 追踪 Command 动态路由

debug 流中会明确打印：
- 节点返回的 `goto` 目标
- 跨图跳转 `graph=PARENT` 标记
- 并行节点列表

---

### 五、常见问题可视化+调试定位方案

| 问题现象       | 调试定位方法                                     |
| -------------- | ------------------------------------------------ |
| 流程死循环     | debug流看跳转节点，检查循环终止条件              |
| 状态字段不更新 | 查看reducer合并规则，打印snapshot.values         |
| 分支走不对     | 打印条件路由返回key，核对path_map映射            |
| HITL无法恢复   | 检查有无checkpointer、thread_id是否一致          |
| 工具不执行     | 查看LLM是否生成tool_calls，路由是否走到tools节点 |
| 子图执行无反应 | xray可视化展开，检查子图入口出口                 |

---

### 六、开发调试最佳流程

1. 先**绘图**确认整体流程逻辑无问题
2. 开启 `stream_mode="debug"` 跑一遍完整链路
3. 关键节点埋回调日志
4. 多轮对话用 `get_state` 核对历史状态
5. 异常场景手动 `update_state` 模拟边界值
6. 上线前关闭debug日志，改用正式日志组件

---

### 七、极简调试可视化代码模板

```python
from langgraph.graph import StateGraph,START,END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

class State(TypedDict):
    num:int

def add_node(state):
    return {"num":state["num"]+1}

builder = StateGraph(State)
builder.add_node("add",add_node)
builder.add_edge(START,"add")
builder.add_edge("add",END)

check = MemorySaver()
graph = builder.compile(checkpointer=check)

# 1. 导出流程图
graph.get_graph().draw_mermaid_png("flow.png")

# 2. debug调试执行
cfg = {"configurable":{"thread_id":"debug_001"}}
for d in graph.stream({"num":0},cfg,stream_mode="debug"):
    print(d)

# 3. 查看状态
print(graph.get_state(cfg).values)
```

---

### 八、速查命令

```python
# 导出图片
graph.get_graph().draw_mermaid_png("flow.png")

# 文本流程图
graph.get_graph().draw_mermaid()

# 调试流式
stream(..., stream_mode="debug")

# 获取状态
graph.get_state(config)

# 修改状态
graph.update_state(config, new_dict)

# 查看历史
checkpointer.list(config)
```

---
需要我给你写一套**开发专用一键调试工具类**，封装绘图、状态打印、单步执行、异常捕获，直接项目导入即用吗？

## 十、异步 & 并发

这是 **生产环境、高并发、多任务、高性能** 必须掌握的核心模块。
我会用**最清晰、最落地、可直接复制**的方式讲透：

- 全套异步 API（invoke/stream/StateGraph 全异步）
- 并发执行（并行节点）
- 批量任务（Batch）
- 异步 + 并发组合实战
- 性能最佳实践

---

### 一、一句话核心

- **异步**：非阻塞 I/O，适合高并发、多用户、API 服务场景
- **并发**：同时执行多个节点/任务，大幅提升速度
- **LangGraph 全链路异步**：从节点、LLM、工具到图执行，100% 异步兼容

> 同步 = 排队执行
> 异步 = 非阻塞等待
> 并发 = 多任务同时跑

---

### 二、全套异步 API（必须背下来）

所有同步方法都有**异步版本**，直接加 `a` 即可：

| 同步           | 异步            | 用途         |
| -------------- | --------------- | ------------ |
| `invoke`       | `ainvoke`       | 异步执行一次 |
| `stream`       | `astream`       | 异步流式输出 |
| `batch`        | `abatch`        | 异步批量执行 |
| `get_state`    | `aget_state`    | 异步获取状态 |
| `update_state` | `aupdate_state` | 异步更新状态 |

---

### 三、1. 基础异步执行 ainvoke

#### 最简单可运行代码

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
import asyncio

class State(TypedDict):
    msg: str

# 异步节点 ✅
async def async_node(state: State):
    return {"msg": "异步执行成功！"}

# 构建图
builder = StateGraph(State)
builder.add_node("async_node", async_node)
builder.add_edge(START, "async_node")
builder.add_edge("async_node", END)
graph = builder.compile()

# 异步执行
async def main():
    result = await graph.ainvoke({"msg": ""})
    print(result)

asyncio.run(main())
```

#### 关键点

- **节点可以直接定义 async 函数**
- 执行用 `await graph.ainvoke()`
- 完全非阻塞，适合 Web 服务（FastAPI）

---

### 四、2. 异步流式输出 astream

前端打字机效果 + 高并发必备
```python
async def async_stream_demo():
    async for chunk in graph.astream({"msg": ""}, stream_mode="updates"):
        print(chunk)
```

---

### 五、3. 并发执行（真正多任务并行）

LangGraph 有 **3 种并发模式**

#### ① 静态并行边（最简单）

```python
# 同时执行 node1、node2、node3
builder.add_edge(START, ["node1", "node2", "node3"])

# 全部完成后进入下一个节点
builder.add_edge(["node1", "node2", "node3"], "final_node")
```
- 自动并发
- 全部完成才会继续
- 无需写代码

#### ② 动态并发：Command + 列表

```python
from langgraph.types import Command

def dispatch_node(state):
    return Command(
        update={"task": "并发开始"},
        goto=["worker_1", "worker_2", "worker_3"]  # 并发执行
    )
```

#### ③ 高级并发：Send（带参并发）

最强大，可**批量并发 + 每个任务传不同参数**
```python
from langgraph.types import Send

def batch_send(state):
    # 并发 3 个任务，各自参数不同
    return Command(
        goto=[
            Send("worker", {"idx": 1}),
            Send("worker", {"idx": 2}),
            Send("worker", {"idx": 3}),
        ]
    )
```

---

### 六、4. 批量执行 abatch

一次跑 **N 个独立任务**，高并发吞吐提升巨大
```python
# 批量输入
inputs = [{"msg": "task1"}, {"msg": "task2"}, {"msg": "task3"}]

# 异步批量执行
results = await graph.abatch(inputs)
```

---

### 七、5. 全异步 + 并发 + 批量 完整示例

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, Send
import asyncio

class State(TypedDict):
    result: list

async def worker(state: State):
    idx = state.get("idx", 0)
    return {"result": [f"worker {idx} 完成"]}

async def main_node(state: State):
    return Command(
        goto=[Send("worker", {"idx": 1}), Send("worker", {"idx": 2})]
    )

builder = StateGraph(State)
builder.add_node("main_node", main_node)
builder.add_node("worker", worker)
builder.add_edge(START, "main_node")
builder.add_edge("worker", END)

graph = builder.compile()

async def run():
    res = await graph.ainvoke({"result": []})
    print(res)

asyncio.run(run())
```

---

### 八、七、异步 & 并发 核心规则（必看）

1. **异步节点必须用 async def**
2. **执行必须用 ainvoke / astream / abatch**
3. **并发节点会自动同步等待**（全部完成才进入下一步）
4. **Send 是最高效并发方式**
5. **工具、LLM 都支持异步**（async 工具）
6. **Checkpointer 也支持异步**（AsyncMemorySaver）

---

### 九、生产级性能最佳实践

1. **API 服务必须用异步**（FastAPI + ainvoke）
2. **多工具/多节点一定用并发**
3. **批量任务用 abatch** 提升吞吐量
4. **I/O 密集型（LLM/网络/数据库）异步收益巨大**
5. **CPU 密集型不适合异步**（用多进程）

---

### 十、速查模板（直接复制）

```python
# 异步执行
await graph.ainvoke(input)

# 异步流式
async for chunk in graph.astream(input):
    ...

# 异步批量
await graph.abatch([...])

# 并发执行
builder.add_edge(A, [B, C, D])

# 动态并发
return Command(goto=[B, C])

# 带参并发
return Command(goto=[Send(B, {"x":1}), ...])
```

---

### 十一、总结（最核心 3 句）

1. **异步 = 非阻塞 = 高性能 API**
2. **并发 = 多节点同时执行 = 速度翻倍**
3. **异步 + 并发 + Send = LangGraph 性能天花板**

---

需要我给你写一套 **「FastAPI + LangGraph 异步 + 并发 + SSE 流式输出」生产完整模板** 吗？
直接上线可用！

## 十一、高级架构组件

### 一、整体定位

面向**企业级分布式部署、托管运维、会话管理、状态快照、底层引擎、智能体高阶封装**，是从「写流程」进阶到「搭架构」必备组件，适合中台服务、多租户系统、云端托管智能体场景。

**核心导入总览**
```python
# 云端托管
from langgraph.cloud import RemoteGraph, CloudClient
# 状态快照与历史
from langgraph.graph.state import StateSnapshot
# 底层引擎
from langgraph.pregel import Pregel, PregelConfig
# 会话/线程管理
from langgraph.thread import Thread, ThreadManager
# 开箱即用助手封装
from langgraph.assistant import Assistant, AssistantGraph
# 分布式锁/事务
from langgraph.distributed import GraphLock
```

---

### 二、1. Pregel 底层调度引擎

#### 核心作用

LangGraph **所有图的底层核心执行引擎**，一切节点调度、状态流转、并行计算、断点续跑都基于 Pregel 图计算模型实现。
#### 核心特性

1. 天然支持**有向图依赖调度**，自动判定节点执行先后顺序
2. 内置**并行执行调度器**，自动拆分并行任务
3. 统一状态聚合、Reducer 合并调度
4. 原生兼容 Checkpointer 快照持久化
5. 支持执行层级、迭代深度、最大轮次限制

#### 常用配置 `PregelConfig`

```python
from langgraph.pregel import PregelConfig

config = PregelConfig(
    max_iterations=50,       # 限制最大执行轮次，防死循环
    interrupt_after_steps=10,# 每N步自动暂停
    debug_mode=False,        # 全局调试开关
    stream_subgraphs=True    # 子图流式透传
)
# 编译图时注入底层引擎配置
graph = builder.compile(config=config)
```

#### 实战价值

- 限制智能体无限循环调用工具
- 统一全局执行规则
- 底层性能调优、执行流程管控

---

### 三、2. StateSnapshot 状态快照组件

#### 作用

对任意时刻**全局状态做完整快照存档**，支持回滚、比对、复刻流程。
#### 核心能力

1. 一键保存当前全量状态+执行位点
2. 快照之间差异比对
3. 历史快照回溯重跑
4. 会话克隆：复制别人的流程状态重新执行

#### 基础用法

```python
# 获取当前会话最新快照
snapshot: StateSnapshot = graph.get_state(config)

# 快照字段
snapshot.values      # 完整状态数据
snapshot.next        # 下一步待执行节点
snapshot.metadata    # 执行元数据、时间、版本

# 用历史快照覆写状态，实现回滚
graph.update_state(config, snapshot.values)
```

#### 业务场景

- 流程出错一键回退到上一步
- 客服会话复盘、问题复现
- A/B 流程状态对比测试

---

### 四、3. Thread 线程/会话管理体系

#### 核心概念

`Thread` = 独立会话隔离单元，**多租户、多用户、多对话**隔离核心架构。
#### 核心组件

1. **Thread**：单会话实体，绑定唯一 `thread_id`
2. **ThreadManager**：批量会话管理、创建、销毁、过期清理
3. **ThreadScope**：会话级变量隔离，不同线程互不干扰

#### 批量会话管理

```python
from langgraph.thread import ThreadManager

manager = ThreadManager(checkpointer=checkpointer)
# 创建新会话
new_thread = manager.create_thread()
# 销毁闲置会话
manager.delete_thread(thread_id="xxx")
# 批量查询在线会话
active_threads = manager.list_active_threads()
```

#### 企业级规范

- 前端用户ID → 映射为固定 thread_id
- 后台定时清理超时闲置会话
- 会话隔离实现**数据隔离、记忆隔离、流程隔离**

---

### 五、4. Assistant API 开箱即用智能体封装

#### 作用

屏蔽底层 StateGraph 细节，**快速标准化构建业务助手**，内置对话记忆、工具调用、多轮会话、格式输出能力。
#### 快速创建业务助手

```python
from langgraph.assistant import Assistant
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")
assistant = Assistant(
    name="业务数据分析助手",
    model=llm,
    tools=[搜索工具、计算工具],
    enable_memory=True,    # 自动开启会话记忆
    enable_human_loop=True # 自动支持人机介入
)

# 直接调用，无需手动构图
res = assistant.invoke("帮我分析月度数据")
```

#### 适用场景

- 内部办公助手、客服机器人、轻量业务Agent
- 快速上线，无需自定义状态与路由

---

### 六、5. LangGraph Cloud 云端托管架构

#### 核心定位

**生产级云端托管方案**，脱离本地部署，实现图服务云端化运维。
#### 核心能力

1. 图代码云端一键部署，生成远程 API 接口
2. 云端统一会话存储、日志、监控
3. 弹性扩容、流量负载均衡
4. 版本灰度发布、图流程版本管理
5. 远程调用本地编写的 Graph

#### 两大核心用法

##### 1）本地调用云端已部署图

```python
from langgraph.cloud import RemoteGraph

# 连接云端托管图
remote_graph = RemoteGraph(
    deployment_url="https://xxx.langgraph.app/graph",
    api_key="你的云端密钥"
)
# 远程执行，本地无任何环境依赖
res = remote_graph.invoke({"query":"云端执行任务"})
```

##### 2）本地图一键上传云端

```python
from langgraph.cloud import CloudClient

client = CloudClient(api_key="xxx")
# 上传编译好的图
client.deploy_graph(graph, name="business_agent_v1")
```

#### 适用场景

- 多团队协作开发智能体
- 无服务器轻量化上线
- 统一中台智能体服务管理

---

### 七、6. 分布式协同组件

#### 1）GraphLock 分布式图锁

解决**多实例并发操作同一会话**引发的状态错乱
```python
from langgraph.distributed import GraphLock

with GraphLock(thread_id="user_1001"):
    # 同一时间只允许一个实例执行该会话流程
    graph.invoke(inputs, config)
```

#### 2）跨服务状态同步

结合 PostgresSaver 全局共享存储，多服务实例读写同一份会话状态，实现分布式会话打通。

#### 3）子图远程调用

主子图不在同一服务，通过 `RemoteGraph` 实现跨服务流程调用，完成微服务式智能体架构拆分。

---

### 八、7. 版本与灰度架构组件

1. **Graph Version**：流程图版本管理，支持 v1/v2 流程平滑切换
2. **Canary Deploy**：流量灰度，部分用户走新流程，其余走旧流程
3. **Schema Migrate**：状态字段结构变更自动兼容迁移，升级不丢历史会话数据

---

### 九、8. 权限与链路鉴权架构

#### 1）Scope 权限域划分

- GlobalScope：全局公共配置
- UserScope：用户私有状态
- TenantScope：租户级隔离数据

#### 2）调用链路鉴权

在 `RunnableConfig` 统一注入租户ID、角色、权限标识，在节点/工具内做权限拦截，实现**RBAC 权限体系**融入流程。

---

### 十、9. 事件总线 & 流程消息推送

高阶架构支持**图执行事件对外推送**
- 节点开始/结束事件推送MQ
- 工具调用日志统一投递消息队列
- 流程完成、异常终止事件回调通知业务系统
实现智能体流程与业务系统解耦联动。

---

### 十一、高级架构分层总图（企业标准分层）

```
【应用层】Assistant / 业务自定义Agent
    ↓
【编排层】StateGraph + 子图模块化 + 路由体系
    ↓
【调度层】Pregel底层引擎 + 异步并发调度
    ↓
【存储层】Checkpointer全家桶(内存/SQLite/PG) + 状态快照
    ↓
【分布式层】Thread会话管理 + 分布式锁 + 跨服务调用
    ↓
【运维托管层】LangGraph Cloud 云端部署+监控+版本管理
```

---

### 十二、选型使用建议

1. **个人/项目Demo**：只用 StateGraph + 基础组件即可
2. **中小型业务系统**：加入 Thread会话管理 + 异步并发 + HITL
3. **企业中台/多租户**：全套上 Checkpointer(PG) + ThreadManager + 分布式锁
4. **轻量化快速上线**：直接用 Assistant + LangGraph Cloud 托管
5. **高性能复杂流程**：深入调优 PregelConfig 底层执行规则

---

### 十三、高级架构组件速查表

| 组件                    | 核心用途               | 适用场景               |
| ----------------------- | ---------------------- | ---------------------- |
| Pregel+PregelConfig     | 底层执行调度、轮次限制 | 防死循环、全局执行管控 |
| StateSnapshot           | 状态快照、回滚复现     | 问题排查、流程复盘     |
| Thread/ThreadManager    | 会话隔离、多用户管理   | 多轮对话、C端用户系统  |
| Assistant               | 快速标准化助手         | 轻量业务Agent快速开发  |
| RemoteGraph/CloudClient | 云端部署远程调用       | 无服务上线、团队协作   |
| GraphLock               | 分布式并发锁           | 多实例集群部署         |
| 分布式事件总线          | 流程事件推送           | 打通业务上下游系统     |

---

需要我给你画一份**LangGraph 企业级完整架构部署图**，并配套写一套**多租户+分布式会话+云端部署**的项目工程目录结构吗？

## 十二、类型注解专用（Type Hints）

这是**写规范、优雅、可维护、无报错**的 LangGraph 代码必须掌握的**全套类型注解**。
你会学到：
- 所有核心类型从哪里导入
- State、节点、Command、Send、Config 怎么写类型
- 企业级规范写法（直接复制）
- 彻底消除编辑器红色波浪线

---

### 一、一句话作用

**类型注解 = 给 Python 加上“类型检查”**
- 编辑器自动提示
- 避免传参错误
- 代码可读性飙升
- 大型项目必备

---

### 二、最全类型注解导入清单（直接复制到项目顶部）

```python
# 基础类型
from typing import TypedDict, Annotated, Optional, Any, Callable

# 状态与图
from langgraph.graph import StateGraph, START, END, add_messages
from langgraph.graph.state import StateSnapshot

# 流程控制 & 路由
from langgraph.types import (
    Command,
    Send,
    interrupt,
    InjectedToolCallId,
    StreamMode,
)

# 配置
from langchain_core.runnables import RunnableConfig

# 消息
from langchain_core.messages import (
    BaseMessage,
    HumanMessage,
    AIMessage,
    ToolMessage,
    SystemMessage,
)

# 工具
from langchain_core.tools import BaseTool
from langgraph.prebuilt import ToolNode
```

---

### 三、1. State 类型注解（最常用）

#### ① 标准 TypedDict 状态

```python
class AgentState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
    query: str
    result: Optional[str]
    step: int
```

#### ② 继承 MessagesState（官方推荐）

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    query: str
    result: str
```

---

### 四、2. 节点函数 标准类型注解

#### ① 普通节点（返回 Dict）

```python
def my_node(state: AgentState, config: RunnableConfig) -> dict:
    return {"messages": [AIMessage(content="回答")]}
```

#### ② 异步节点

```python
async def async_node(
    state: AgentState,
    config: RunnableConfig
) -> dict:
    return {"result": "ok"}
```

#### ③ 返回 Command 的节点

```python
def router_node(
    state: AgentState
) -> Command:
    return Command(goto=END, update={})
```

#### ④ 返回 Send / 并发列表

```python
def send_node(state: AgentState) -> Command:
    return Command(
        goto=[
            Send("worker", {"task": 1}),
            Send("worker", {"task": 2}),
        ]
    )
```

---

### 五、3. 工具函数 类型注解

#### ① 标准工具

```python
@tool
def search(
    query: str,
    config: RunnableConfig
) -> str:
    return "结果"
```

#### ② 带 ToolCallId 注入

```python
@tool
def audit(
    content: str,
    tool_call_id: InjectedToolCallId
) -> Command:
    return Command(goto="human_audit")
```

---

### 六、4. 路由函数 类型注解

```python
def my_router(
    state: AgentState
) -> Literal["tools", "end", "continue"]:
    if "tool_calls" in state:
        return "tools"
    return "end"
```

---

### 七、5. 配置 Config 注解

```python
config: RunnableConfig = {
    "configurable": {
        "thread_id": "user_123",
        "user_id": "u_123"
    }
}
```

---

### 八、6. 状态快照 StateSnapshot

```python
snapshot: StateSnapshot = graph.get_state(config)
print(snapshot.values)
print(snapshot.next)
```

---

### 九、7. Stream 流式模式 StreamMode

```python
mode: StreamMode = "updates"  # values / updates / debug
```

---

### 十、8. 完整可运行 企业级规范示例（带全类型）

```python
from typing import TypedDict, Annotated, Literal
from langgraph.graph import StateGraph, START, END, add_messages, MessagesState
from langgraph.types import Command, Send, InjectedToolCallId
from langchain_core.runnables import RunnableConfig
from langchain_core.messages import AIMessage
from langchain_core.tools import tool

# --------------------------
# 状态（全类型）
# --------------------------
class State(MessagesState):
    query: str
    result: str

# --------------------------
# 工具（全类型）
# --------------------------
@tool
def add(a: int, b: int) -> str:
    return str(a + b)

# --------------------------
# 节点（全类型）
# --------------------------
def llm_node(
    state: State,
    config: RunnableConfig
) -> dict:
    return {"messages": [AIMessage(content="你好")]}

def router(
    state: State
) -> Literal["tools", "end"]:
    return "end"

# --------------------------
# 构图（全类型）
# --------------------------
builder: StateGraph = StateGraph(State)
builder.add_node("llm", llm_node)
builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", router)
graph = builder.compile()
```

---

### 十一、最常用类型速查表（背会这张就够）

| 类型     | 注解写法                 | 用途                   |
| -------- | ------------------------ | ---------------------- |
| 状态     | `state: AgentState`      | 节点入参               |
| 配置     | `config: RunnableConfig` | 全局上下文             |
| 节点返回 | `-> dict`                | 普通节点               |
| 路由返回 | `-> Literal["a", "b"]`   | 条件路由               |
| 命令返回 | `-> Command`             | 动态路由节点           |
| 消息     | `BaseMessage`            | 对话消息               |
| 快照     | `StateSnapshot`          | 获取状态               |
| 工具ID   | `InjectedToolCallId`     | 工具内注入             |
| 流式模式 | `StreamMode`             | `updates/values/debug` |

---

### 十二、为什么一定要用类型注解？

1. **编辑器自动补全** → 写代码飞快
2. **提前发现错误** → 运行前就报错
3. **代码可读性 100%** → 别人一眼看懂
4. **大型项目必备** → 无类型 = 灾难

---

### 十三、你已经学完 **LangGraph 全体系**

我帮你回顾一下你已经掌握的全套知识：
1. **StateGraph + Command**
2. **状态 & 合并器**
3. **节点专用组件**
4. **记忆持久化 Checkpointer**
5. **人机交互 HITL**
6. **子图 & 模块化**
7. **流式 & 事件回调**
8. **工具生态**
9. **调试 & 可视化**
10. **异步 & 并发**
11. **高级架构**
12. **类型注解**

---

### 如果你需要，我可以马上给你：

#### ✅ **一套完整可上线的企业级 LangGraph 项目模板**

包含：
- 多轮对话
- 工具调用
- 人工审核
- 异步流式输出
- 多子图模块化
- 状态管理
- 类型注解齐全
- FastAPI 接口

要我直接给你生成吗？