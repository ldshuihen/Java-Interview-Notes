# Fast API

我给你按**最常用、最实战、面试必问**的方式，把 FastAPI 完整分类讲透，**你一看就懂、一用就会**。

---

## 一、FastAPI 是什么？

**FastAPI = 现代、高性能、快速开发的 Python Web 框架**
专门用来做：

- **后端 API 接口**
- **微服务**
- **AI 服务部署（LangChain/LangGraph 必备）**
- **高并发服务**

---

## 二、FastAPI 核心分类（4 大类）

### 1. 路由与请求分类（最基础）

控制接口路径、请求方式、参数传递

#### ① 路径路由

```python
@app.get("/")       # 首页
@app.get("/user")   # 用户接口
@app.post("/chat")  # 对话接口
```

#### ② 请求方法分类

- `GET`：查询数据（查）
- `POST`：提交数据（增/改）
- `PUT`：更新
- `DELETE`：删除

**LangChain 项目 90% 用 POST**

#### ③ 参数传递分类

1. **查询参数** `?key=xx`
2. **路径参数** `/user/123`
3. **请求体（Body）** → **最常用（JSON）**
4. **表单数据**
5. **文件上传**

---

### 2. 请求体与数据验证（FastAPI 最强功能）

基于 **Pydantic**，自动验证 JSON 格式

#### ① 基础模型

```python
from pydantic import BaseModel

class Request(BaseModel):
    question: str
    user_id: str
```

#### ② 自动校验

- 类型不对 → 直接报错
- 字段缺失 → 直接报错
- 自动生成接口文档

---

### 3. 响应分类（返回数据格式）

#### ① JSON 响应（最常用）

```python
return {"code": 200, "msg": "成功", "data": "xxx"}
```

#### ② 流式响应（Streaming）

**LangChain/LangGraph 必备 → 打字机效果**

```python
from fastapi.responses import StreamingResponse
return StreamingResponse(stream_generator())
```

#### ③ 文件响应

#### ④ HTML / 模板响应

---

### 4. 高级功能分类（企业级必备）

#### ① 异步支持 async/await

**FastAPI + LangChain 必用！**

```python
@app.post("/chat")
async def chat(req: Request):
    return await chain.ainvoke(...)
```

#### ② 依赖注入 Depends

数据库、认证、日志统一管理

#### ③ 中间件 Middleware

跨域、请求日志、权限控制

#### ④ 异常处理

全局捕获错误

#### ⑤ 自动 API 文档

- http://127.0.0.1:8000/docs
- http://127.0.0.1:8000/redoc

**自动生成、可直接调试！**

---

## 三、FastAPI 核心架构分类

### 1. 基础层

- 路由
- 请求/响应
- 参数解析

这是 **FastAPI 所有功能的地基**，也是和 **LangChain + LangGraph** 对接时**最常用**的部分。

我用**最简单、最实战、最不绕弯**的方式给你讲透👇

---

#### 一、基础层包含 3 个核心

1. **路由（Router）**：定义接口地址 + 请求方式
2. **请求（Request）**：接收前端传过来的数据
3. **响应（Response）**：返回数据给前端

这 3 个学会，你就能写 **完整后端 API** 对接 AI。

---

#### 二、1. 路由（Router）

##### 作用

**定义接口路径 + 请求方法**

##### 最常用 2 种请求方式

```python
from fastapi import FastAPI

app = FastAPI()

# 1. GET：查询、获取数据
@app.get("/hello")
def hello():
    return {"message": "你好"}

# 2. POST：提交数据（LangChain 必用！）
@app.post("/chat")
def chat():
    return {"answer": "AI回答"}
```

##### 为什么 AI 项目用 POST？

- 可以传大量文本
- 更安全
- 符合接口规范

---

#### 三、2. 参数解析（接收前端数据）

**3 种最常用传参方式**

##### ① 查询参数（?key=xxx）

```python
@app.get("/question")
def question(question: str):
    return {"q": question}
```

调用：

```
http://localhost:8000/question?question=你好
```

##### ② 路径参数（/user/123）

```python
@app.get("/user/{user_id}")
def user(user_id: int):
    return {"user_id": user_id}
```

##### ③ 请求体（JSON，最强！AI 必用！）

用 **Pydantic** 接收 JSON

```python
from pydantic import BaseModel

class ChatRequest(BaseModel):
    question: str
    user_id: str

@app.post("/chat")
def chat(req: ChatRequest):
    return {"answer": "回答：" + req.question}
```

---

#### 四、3. 请求与响应

##### 请求（Request）

前端 → 后端的数据  
可以是：

- 文本
- JSON
- 文件
- 表单

##### 响应（Response）

后端 → 前端的数据  
**永远返回 JSON**（最常用）

```python
return {"code": 200, "msg": "成功", "data": "xxx"}
```

---

#### 五、FastAPI 基础层 完整模板（可直接运行）

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="AI接口基础模板")

# 请求体模型
class ChatRequest(BaseModel):
    question: str
    user_id: str

# 路由：POST接口（对接LangChain必用）
@app.post("/chat")
def chat(req: ChatRequest):
    # 这里未来可以接 LangChain / LangGraph
    answer = "AI回答：" + req.question
    return {
        "code": 200,
        "msg": "成功",
        "answer": answer
    }
```

运行：

```bash
uvicorn main:app --reload
```

打开文档：
**http://127.0.0.1:8000/docs**

---

#### 六、一句话终极总结

##### **路由 = 接口地址**

##### **参数解析 = 接收前端数据**

##### **请求/响应 = 前后端数据通道**

##### **这就是 FastAPI 地基！**

---

#### 你已经掌握 FastAPI 最核心基础！

接下来我可以带你写：

##### **FastAPI + LangChain 对话接口**

##### **FastAPI + LangGraph 智能体接口**

##### **FastAPI 流式输出（打字机效果）**

你想学哪一个？我直接给你**可运行代码**！

### 2. 验证层

- Pydantic 模型
- 自动数据校验

**验证层 = Pydantic + 自动数据校验**
这是 FastAPI 最牛的功能，**没有之一**。
作用：**前端传过来的数据，自动检查对错，不用写一行判断代码！**

我用**最清晰、最实战、一看就会**的方式讲👇

---

#### 一、验证层到底是什么？

一句话：
**定义数据格式 → 自动校验 → 自动报错 → 自动生成文档**

你告诉系统：

- 这个字段必须是字符串
- 那个字段必须是数字
- 这个字段不能为空
- 那个字段长度必须大于 2

FastAPI 会**自动检查**，不符合直接返回错误，不用你写 `if` 判断！

---

#### 二、两大核心

##### 1. Pydantic 模型（数据定义）

```python
from pydantic import BaseModel

class ChatRequest(BaseModel):
    user_id: str       # 字符串
    question: str      # 字符串
    age: int | None    # 整数 或 空
```

这就是**数据格式合同**，前后端必须遵守。

##### 2. 自动数据校验（核心能力）

FastAPI 自动帮你做：

- 字段是否缺失？
- 类型是否正确？（字符串 vs 数字）
- 格式是否合法？
- 长度是否满足？

**错了直接返回清晰错误，不用写代码！**

---

#### 三、最常用实战示例（对接 LangChain 必用）

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义验证模型（验证层核心）
class ChatRequest(BaseModel):
    user_id: str       # 必须传，字符串
    question: str      # 必须传，问题内容
    temperature: float = 0.7  # 可选，默认0.7

# 接口：自动校验数据
@app.post("/chat")
def chat(req: ChatRequest):
    # 这里一定是合法数据，直接用！
    return {
        "code": 200,
        "answer": f"回答：{req.question}"
    }
```

##### 如果前端传错数据（例如 temperature=文本）

**FastAPI 自动返回错误：**

```json
{
  "detail": [
    {
      "loc": ["body", "temperature"],
      "msg": "Input should be a valid number",
      "type": "float_type"
    }
  ]
}
```

你**完全不用写校验代码**！全自动！

---

#### 四、高级校验（企业级常用）

##### 1. 字段必填 / 可选

```python
class Request(BaseModel):
    must_field: str       # 必传
    optional_field: str | None = None  # 可选
```

##### 2. 数字范围限制

```python
from pydantic import Field

class Request(BaseModel):
    age: int = Field(ge=0, le=120)  # 0~120
```

##### 3. 字符串长度限制

```python
content: str = Field(min_length=1, max_length=1000)
```

##### 4. 邮箱、正则等复杂校验

```python
from pydantic import EmailStr

email: EmailStr
```

---

#### 五、验证层 3 大超级好处（背下来）

1. **不用写 if 判断，代码少 50%**
2. **前端传错直接报错，安全稳定**
3. **自动生成接口文档，一键调试**

这就是 FastAPI 比 Django、Flask 强的地方！

---

#### 六、和 LangChain / LangGraph 关系

**验证层 = AI 接口的大门**
所有传给 AI 的：

- user_id
- question
- 会话ID
- 模型参数

都必须通过 **Pydantic 验证**，保证 AI 拿到干净合法的数据。

---

#### 七、一句话终极总结

##### **Pydantic 模型 = 数据格式**

##### **自动校验 = 不用写判断代码**

##### **验证层 = FastAPI 的核心灵魂**

---

#### 你已经掌握 FastAPI 第二大核心！

接下来我可以带你写：

- **FastAPI + LangChain 对话（带验证）**
- **FastAPI + LangGraph 智能体（带验证）**
- **完整可运行项目**

你想学哪个？我直接给你**代码**！

### 3. 高级服务层

- 异步
- 依赖注入
- 中间件
- CORS 跨域
- 认证授权



这是 **FastAPI 从“能用”到“生产上线”** 的核心功能，也是对接 **LangChain、LangGraph、高并发AI服务** 必须掌握的部分。

我给你**最清晰、最实战、最不绕弯**的完整讲解👇

---

#### 一、高级服务层 5 大核心

1. **异步（async/await）**
2. **依赖注入（Depends）**
3. **中间件（Middleware）**
4. **CORS 跨域（前端必用）**
5. **认证授权（安全）**

这 5 个学会，你就能写 **商用级 AI 后端服务**。

---

#### 1. 异步 async / await（AI 项目必用！）

###### 作用

- **不阻塞、高并发、性能提升巨大**
- LangChain / LangGraph 全部支持异步
- FastAPI 天生为异步而生

###### 实战代码（对接 LangChain）

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Request(BaseModel):
    question: str

# 异步接口（必用）
@app.post("/chat")
async def chat(req: Request):  # async 声明
    # 异步调用 AI
    # response = await llm.ainvoke(...)
    return {"answer": "异步AI回复：" + req.question}
```

###### 为什么 AI 服务必须用异步？

- 模型调用是 IO 等待
- 异步可以同时处理 100+ 请求
- 不卡顿、不排队

---

#### 2. 依赖注入 Depends

###### 作用

- 公共逻辑抽离
- 数据库连接、日志、权限、配置统一管理
- 代码更干净

##### 示例：全局日志依赖

```python
from fastapi import Depends

# 公共依赖
def get_logger():
    print("记录请求日志")
    return "logger"

# 接口使用依赖
@app.post("/chat")
async def chat(req: Request, logger=Depends(get_logger)):
    return {"answer": "ok"}
```

---

#### 3. 中间件 Middleware

##### 作用

- 全局请求拦截
- 统一处理请求/响应
- 计时、日志、请求头处理

```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    process_time = time.time() - start
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

---

#### 4. CORS 跨域（前端连接必用！）

##### 作用

让 **前端网页（Vue/React/小程序）** 可以调用你的后端接口。

**这是上线必备！**

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 允许所有域名
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

#### 5. 认证授权（安全）

##### 作用

- 接口保护
- 只有登录用户能访问 AI 服务
- 企业级必备

```python
from fastapi import Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    if token != "valid_token":
        raise HTTPException(status_code=401)
    return "user"

@app.post("/chat")
async def chat(req: Request, user=Depends(get_current_user)):
    return {"answer": "安全回复"}
```

---

#### 二、高级服务层 超级总结（背会就懂）

1. **异步** → 高并发、AI服务必用
2. **依赖注入** → 公共代码复用
3. **中间件** → 全局请求处理
4. **CORS** → 允许前端调用（最重要）
5. **认证** → 接口安全

---

#### 三、一句话终极总结

##### **高级服务层 = FastAPI 企业级战斗力**

##### **异步 + CORS 是 LangChain/LangGraph 上线必用功能！**

---

##### 你已经掌握 FastAPI 全部核心！

现在你可以直接搭建：
✅ FastAPI + LangChain 对话
✅ FastAPI + LangGraph 智能体
✅ FastAPI + 流式输出
✅ 高并发AI后端服务

---

#### 我可以直接给你一套

#### **🔥 FastAPI + LangGraph + 异步 + 跨域 + 上线完整版代码**

直接复制运行，带前端对接！

你要吗？

### 4. 部署层

- uvicorn 运行
- 多进程
- 高并发



这是把 **FastAPI + LangChain / LangGraph** 从本地代码变成**线上可访问服务**的最后一步，也是生产环境必须掌握的核心。

我用**最实战、最简洁、直接能用**的方式讲👇

---

#### 1. Uvicorn 运行（核心运行器）

FastAPI 本身**不能直接运行**，必须依靠 **Uvicorn**（ASGI 服务器）启动。

##### 基础运行命令（本地开发）

```bash
uvicorn main:app --reload
```

- `main`：你的文件名 `main.py`
- `app`：FastAPI 实例名 `app = FastAPI()`
- `--reload`：代码修改自动重启（**仅限开发**）

##### 生产环境运行（正式上线）

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

- `--host 0.0.0.0`：让外部设备能访问
- `--port 8000`：端口号

---

#### 2. 多进程（多核利用，高并发关键）

**单进程跑 AI 服务太慢！必须开多进程！**

Uvicorn 开启多进程：

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

- `--workers 4`：开启 4 个进程（一般 = CPU 核心数）

多核 CPU 服务器必须用这个，**并发能力翻倍**。

---

#### 3. 高并发优化（AI 服务必配）

LangChain / LLM 调用是 IO 密集型，高并发必须优化：

##### 最优高并发启动命令

```bash
uvicorn main:app \
  --host 0.0.0.0 \
  --port 8000 \
  --workers 4 \
  --loop uvloop \
  --http httptools
```

- `--loop uvloop`：Python 最快异步事件循环
- `--http httptools`：更快的 HTTP 解析
- 性能提升 **30%~200%**

---

#### 4. 完整部署配置（企业级标准）

##### 方式1：直接命令启动（简单）

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

##### 方式2：后台常驻运行（生产推荐）

```bash
nohup uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4 &
```

- 关闭终端也不会停
- 日志输出到 nohup.out

---

#### 5. 部署层一句话总结

##### **Uvicorn = 运行引擎**

##### **多进程 = 多核加速**

##### **高并发配置 = AI 服务稳定上线**

---

#### 🔥 终极上线启动命令（直接复制用）

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4 --loop uvloop --http httptools
```

---

#### 你已经学完 FastAPI 全部四层！

✅ 基础层：路由、请求、响应  
✅ 验证层：Pydantic 自动校验  
✅ 高级服务层：异步、跨域、中间件  
✅ **部署层：uvicorn + 多进程 + 高并发**

---

#### 我可以直接给你一套

##### **🔥 FastAPI + LangGraph + 异步 + 跨域 + 部署 完整上线模板**

复制就能跑，直接对接前端！

要吗？

---

## 四、FastAPI 在 LangChain 中的分类用法

### 1. 普通对话接口

```python
@app.post("/chat")
async def chat(req: ChatRequest):
    res = await llm.ainvoke(...)
    return res
```

### 2. RAG 知识库接口

```python
@app.post("/rag")
async def rag(question: str):
    return await rag_chain.ainvoke(...)
```

### 3. LangGraph 智能体接口

```python
@app.post("/agent")
async def agent(question: str):
    return await graph.ainvoke(...)
```

### 4. 流式输出（打字机）

```python
@app.post("/stream")
async def stream(question: str):
    return StreamingResponse(chain.astream(...))
```

---

## 五、一句话总结（超级好记）

### **FastAPI = 做 API 接口的神器**

### **路由 + 请求体 + 异步 + 自动文档 = 企业级后端标配**

### **FastAPI + LangChain/LangGraph = AI 服务标准架构**

---

## **完整代码**

1. 基础 FastAPI 模板
2. FastAPI + LangChain 对话
3. FastAPI + LangGraph 智能体
4. FastAPI 流式输出（打字机）
5. FastAPI + 文件上传 + RAG

我给你整理了**一套可直接运行、完整无坑、带注释**的 FastAPI + AI 全场景模板，包含你要的 5 大类：

- 基础模板
- LangChain 对话
- LangGraph 智能体
- 流式输出（打字机效果）
- 文件上传 + RAG（本地知识库）

所有代码**Python 3.10+** 直接跑，依赖统一安装。

---

### 先统一安装依赖

```bash
pip install fastapi uvicorn langchain langgraph langchain-openai pypdf faiss-cpu python-multipart
```

---

### 1. 基础 FastAPI 模板

最简单、最标准的 FastAPI 启动模板

```python
from fastapi import FastAPI

# 创建 FastAPI 应用
app = FastAPI(title="基础 FastAPI 模板", version="1.0")

# 根路由（健康检查）
@app.get("/", tags=["基础"])
def root():
    return {"message": "FastAPI 服务运行正常！"}

# 带参数的 GET 接口
@app.get("/hello/{name}", tags=["基础"])
def hello(name: str):
    return {"message": f"你好，{name}！"}

# 启动命令：uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

**启动命令**

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

访问文档：

- Swagger UI：http://127.0.0.1:8000/docs
- ReDoc：http://127.0.0.1:8000/redoc

---

### 2. FastAPI + LangChain 对话接口

直接对接大模型（OpenAI 格式，可换成通义千问、文心一言、DeepSeek 等）

```python
from fastapi import FastAPI
from pydantic import BaseModel
from langchain_openai import ChatOpenAI

# 1. 初始化应用
app = FastAPI(title="FastAPI + LangChain 对话")

# 2. 配置 LLM（换成你自己的 key 和地址）
llm = ChatOpenAI(
    model="deepseek-chat",
    api_key="你的API_KEY",
    base_url="https://api.deepseek.com/v1",
    temperature=0.7
)

# 3. 请求体模型
class ChatRequest(BaseModel):
    query: str

# 4. 对话接口
@app.post("/chat", tags=["LangChain 对话"])
async def chat(request: ChatRequest):
    response = llm.invoke(request.query)
    return {"answer": response.content}
```

---

### 3. FastAPI + LangGraph 智能体（带工具调用）

最实用的**AI 智能体**，支持联网、计算、工具调用

```python
from fastapi import FastAPI
from pydantic import BaseModel
from langgraph.graph import StateGraph, MessagesState
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

# 1. 初始化
app = FastAPI(title="FastAPI + LangGraph 智能体")

# 2. 定义工具（智能体能调用的函数）
@tool
def calculator(expression: str) -> str:
    """计算数学表达式，例如 3+5*2"""
    try:
        return str(eval(expression))
    except:
        return "计算错误"

# 3. LLM + 绑定工具
llm = ChatOpenAI(
    model="deepseek-chat",
    api_key="你的API_KEY",
    base_url="https://api.deepseek.com/v1"
).bind_tools([calculator])

# 4. 构建 LangGraph 智能体工作流
def agent_node(state: MessagesState):
    return {"messages": [llm.invoke(state["messages"])]}

# 构建图
workflow = StateGraph(MessagesState)
workflow.add_node("agent", agent_node)
workflow.set_entry_point("agent")
agent = workflow.compile()

# 5. 请求体
class AgentRequest(BaseModel):
    query: str

# 6. 智能体接口
@app.post("/agent", tags=["LangGraph 智能体"])
async def run_agent(request: AgentRequest):
    result = agent.invoke({"messages": [("user", request.query)]})
    return {"answer": result["messages"][-1].content}
```

---

### 4. FastAPI 流式输出（打字机效果）

前端能看到**逐字输出**，像 ChatGPT 一样的流畅体验

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from langchain_openai import ChatOpenAI

app = FastAPI(title="FastAPI 流式输出（打字机）")

llm = ChatOpenAI(
    model="deepseek-chat",
    api_key="你的API_KEY",
    base_url="https://api.deepseek.com/v1",
    streaming=True  # 开启流式
)

# 流式生成器
async def stream_generator(query: str):
    async for chunk in llm.astream(query):
        yield chunk.content

# 流式接口
@app.get("/stream", tags=["流式输出"])
async def stream_chat(query: str):
    return StreamingResponse(
        stream_generator(query),
        media_type="text/plain"
    )
```

访问测试：

```
http://127.0.0.1:8000/stream?query=写一篇50字的短文
```

---

### 5. FastAPI + 文件上传 + RAG（本地知识库）

支持 **PDF / TXT** 上传 → 向文档提问 → 检索回答

```python
from fastapi import FastAPI, UploadFile, File
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain.chains import RetrievalQA
import tempfile

app = FastAPI(title="FastAPI + 文件上传 + RAG")

# 全局向量库
vector_store = None

# 1. 上传文件并构建知识库
@app.post("/upload", tags=["RAG 文件上传"])
async def upload_file(file: UploadFile = File(...)):
    global vector_store

    # 保存临时文件
    with tempfile.NamedTemporaryFile(delete=False) as tmp:
        tmp.write(await file.read())
        tmp_path = tmp.name

    # 加载文档
    if file.filename.endswith(".pdf"):
        loader = PyPDFLoader(tmp_path)
    elif file.filename.endswith(".txt"):
        loader = TextLoader(tmp_path, encoding="utf-8")
    else:
        return {"error": "只支持 PDF / TXT"}

    # 切分 + 向量化 + 建库
    docs = loader.load()
    splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
    splits = splitter.split_documents(docs)

    embeddings = OpenAIEmbeddings(
        api_key="你的API_KEY",
        base_url="https://api.deepseek.com/v1"
    )
    vector_store = FAISS.from_documents(splits, embeddings)
    return {"message": f"文件 {file.filename} 上传并构建知识库成功"}

# 2. 向知识库提问
@app.get("/rag", tags=["RAG 检索问答"])
async def rag_query(query: str):
    if not vector_store:
        return {"error": "请先上传文件构建知识库"}

    llm = ChatOpenAI(
        model="deepseek-chat",
        api_key="你的API_KEY",
        base_url="https://api.deepseek.com/v1"
    )

    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        retriever=vector_store.as_retriever()
    )
    result = qa_chain.invoke({"query": query})
    return {"answer": result["result"]}
```

---

### 如何把它们合并成一个完整项目？

直接新建一个 `main.py`，把**所有代码按顺序粘贴**，共用一个 `app = FastAPI()` 即可。

最终你会得到：

- 一个服务端口
- 一套完整接口文档
- 基础API、对话、智能体、流式、文件RAG 全部可用

---

### 启动命令（统一）

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

---

### 我可以继续为你做的

你可以直接告诉我，我可以马上升级：

1. 换成**国产模型**（通义千问、文心、豆包、DeepSeek、本地 Ollama）
2. 加**身份认证 / API Key**
3. 加**历史对话记忆**
4. 加**前端页面**（HTML 聊天界面）
5. 打包成 **Docker** 一键部署

需要哪个我直接给你完整代码！

# LangChain

LangChain 是一个用于构建大语言模型（LLM）应用的模块化框架，其核心设计理念是**组件化**与**可组合性**。它主要可以从**包结构**、**核心组件**和**生态系统**三个维度进行分类介绍。

---

## 一、包结构分类（模块化架构）

LangChain V1.0 采用清晰的分层包设计，各包职责明确，依赖关系单向。

| 包名                      | 核心定位         | 主要内容                                                     | 依赖             |
| :------------------------ | :--------------- | :----------------------------------------------------------- | :--------------- |
| **`langchain-core`**      | 核心抽象层       | 定义基础接口（ChatModel, Prompt, Tool）、LCEL 表达式语言、Runnable 原语。 | 无               |
| **`langchain`**           | 主包（应用入口） | 高阶 API、预构建链/代理、中间件。                            | `langchain-core` |
| **`langchain-community`** | 第三方集成库     | 社区维护的海量集成（模型、向量库、工具、文档加载器）。       | `langchain-core` |
| **`langchain-openai`**    | 官方合作伙伴包   | OpenAI 系列模型的专用集成（更稳定、最新）。                  | `langchain-core` |
| **`langchain-groq`**      | 官方合作伙伴包   | Groq 高速推理模型集成。                                      | `langchain-core` |

---

## 二、核心组件分类（六大模块）

LangChain 的功能核心由六大组件构成，它们如同乐高积木，可灵活组合。

### 1. 模型输入输出 (Model I/O)

负责与大语言模型交互，是所有应用的基础。

- **模型 (Models)**:
  - **LLM**: 文本输入/文本输出（如 GPT-3.5-turbo-instruct）。
  - **ChatModel**: 消息列表输入/消息输出（如 GPT-4o），支持多轮对话。
  - **Embeddings**: 将文本转换为语义向量（如 text-embedding-ada-002）。
- **提示词模板 (Prompt Templates)**: 动态生成模型输入，封装工程化最佳实践。
- **输出解析器 (Output Parsers)**: 将模型输出（常为字符串）解析为结构化数据（如 JSON、List）。

#### 一、整体作用

**Model I/O** 是 LangChain 所有应用**最底层基础**，专门统一规范：
如何给大模型发请求、怎么拼装提示词、怎么接收模型返回结果、怎么把杂乱文本转成结构化数据。
屏蔽不同大模型（OpenAI、通义、文心、本地LLaMA）调用差异，一套写法通吃所有模型。

---

#### 二、三大模型分类（LLM / ChatModel / Embeddings）

##### 1. LLM 纯文本模型

###### 定义

**纯字符串输入 → 纯字符串输出**
老式补全式大模型，没有对话角色区分，只做文本续写。

###### 特点

1. 无`系统提示、用户、助手`角色概念
2. 直接拼接文本上下文
3. 只适合**单轮问答、文案生成、续写**
4. 不适合多轮连续对话

###### 常见模型

GPT-3.5-turbo-instruct、text-davinci-003、本地开源续写模型

###### 代码极简示例

```python
from langchain_openai import OpenAI

llm = OpenAI(model="gpt-3.5-turbo-instruct")
res = llm.invoke("写一句早安文案")
print(res)
```

---

##### 2. ChatModel 对话模型（**工作最常用**）

###### 定义

**消息对象列表输入 → 消息对象输出**
专门为**聊天对话**设计，区分角色，支持多轮上下文。

###### 三大固定角色

1. **SystemMessage 系统角色**：设定人设、规则、任务要求
2. **HumanMessage 用户角色**：用户提问
3. **AIMessage AI角色**：模型回答

###### 特点

1. 原生支持**多轮对话记忆**
2. 市面主流聊天模型全是此类：GPT4o、通义千问、讯飞星火、Qwen、DeepSeek
3. 支持**函数调用、工具调用**
4. RAG智能问答、AI聊天机器人首选

###### 代码示例

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage

chat = ChatOpenAI(model="gpt-4o")
msgs = [
    SystemMessage(content="你是专业Java面试官"),
    HumanMessage(content="讲一下JVM内存分区")
]
res = chat.invoke(msgs)
print(res.content)
```

Viewed llm_chain.py:1-63

在当前的 `langchain-demo` 项目中，**确实已经区分了这三种消息类型**。

特别是在 `demos/memory_chat.py` 中，你可以看到如下定义：

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."), # SystemMessage
    MessagesPlaceholder(variable_name="history"), # 历史消息（包含 Human 和 AI）
    ("human", "{input}"), # HumanMessage
])
```

以下是这三者的详细区别和作用：

###### 1. SystemMessage (系统消息)

*   **角色**：扮演“导演”或“规则制定者”。
*   **作用**：设置 AI 的身份、语气、行为准则或任务背景。
*   **示例**：`"你是一个资深的 Python 程序员"` 或 `"请用幽默的方式回答问题"`。
*   **特点**：通常放在对话的最开始，对 AI 的后续表现有全局性的引导作用。

###### 2. HumanMessage (人类消息)

*   **角色**：扮演“用户”。
*   **作用**：代表真实用户的提问、指令或输入。
*   **示例**：`"如何实现一个冒泡排序？"`。
*   **特点**：由用户发起，是对话的驱动力。

###### 3. AIMessage (AI 消息)

*   **角色**：扮演“AI 助手”。
*   **作用**：代表模型生成的回复。
*   **示例**：`"冒泡排序是一种简单的排序算法..."`。
*   **特点**：模型根据之前的 `SystemMessage` 和 `HumanMessage` 生成的内容。在对话记忆（Memory）中，AI 的回复会被保存为 `AIMessage`，以便下次提问时模型知道自己之前说过什么。

---

###### 在本项目中的体现：

如果你查看 `chat_histories/` 下的 JSON 文件（例如你打开过的 `user_02.json`），你会发现存储的结构就是按类型区分的：

```json
[
  {"type": "human", "data": {"content": "你知道小明几岁吗"}},
  {"type": "ai", "data": {"content": "我无法直接知道..."}}
]
```

**总结区别表：**

| 消息类型          | 对应角色    | 主要目的                 | 频率                |
| :---------------- | :---------- | :----------------------- | :------------------ |
| **SystemMessage** | 系统/开发者 | 设定 AI 角色、规则、约束 | 通常 1 条（在开头） |
| **HumanMessage**  | 用户        | 提出问题、给出指令       | 每轮对话 1 条       |
| **AIMessage**     | AI 模型     | 给出回答、执行任务       | 每轮对话 1 条       |

**为什么要区分？**
现代大模型（如 Gemini, GPT, Qwen 等）是专门针对这种“角色扮演”模式进行微调的。明确区分角色可以显著提高模型理解指令的准确度，并让对话流更加自然。

---

##### 3. Embeddings 嵌入向量模型

###### 定义

**文本 → 高密度数字向量**
把人类语言转成计算机能计算的**语义向量**。

###### 核心用途

1. **RAG知识库检索**（最核心）
2. 文本相似度对比
3. 聚类、内容匹配、问答召回文档

###### 原理

语义越相近的文本，向量距离越近。

###### 常用模型

text-embedding-3-small、bge-m3、m3e、百川向量模型

###### 代码示例

```python
from langchain_openai import OpenAIEmbeddings

emb = OpenAIEmbeddings()
vec = emb.embed_query("Java并发原理")
print(vec) # 输出浮点型向量数组
```



#### **ChatModel **

**ChatModel** 是 LangChain 中**所有大语言模型（LLM）的统一接口标准**，不管是 OpenAI、Anthropic、通义千问、文心一言、DeepSeek、Ollama 本地模型，全部都被它**标准化封装**。

它的核心作用：
**让你用同一套代码，无缝切换所有模型，不用改业务逻辑。**

---

##### 一、ChatModel 到底是什么？

一句话：
**ChatModel = LangChain 对「对话类大模型」的统一抽象层**

它不是某个具体模型，而是**接口规范**。

所有模型必须实现它规定的方法：

- `invoke()` → 同步调用
- `stream()` → 流式输出
- `batch()` → 批量调用
- `ainvoke()` → 异步调用

你只需要学这一个接口，就能用**全球所有模型**。

---

##### 二、核心能力（必须掌握）

###### 1. 统一消息格式

所有模型都必须接收 **Message 列表**：

- `SystemMessage`：系统提示
- `HumanMessage`：用户输入
- `AIMessage`：AI 回复

```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="你是一个助手"),
    HumanMessage(content="你好")
]

model.invoke(messages)
```

###### 2. 统一调用方式

不管什么模型，调用代码**完全一样**：

```python
# 同步
response = model.invoke(messages)

# 流式
for chunk in model.stream(messages):
    print(chunk.content, end="")

# 异步
await model.ainvoke(messages)

# 批量
model.batch([messages1, messages2])
```

###### 3. 统一输出格式

输出永远是：

```python
AIMessage(
    content="...",
    usage_metadata={"input_tokens": 10, "output_tokens": 5}
)
```

---

##### 三、创建 ChatModel 的 2 种方式（官方最新）

###### 方式1：通用创建（最推荐，一行切换所有模型）

```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    model="gpt-3.5-turbo",
    model_provider="openai",
    api_key="sk-xxx",
    base_url="https://xxx",
    temperature=0.7
)
```

**支持的 provider 几乎全覆盖：**

- openai
- anthropic
- qwen（通义千问）
- wenxin（文心一言）
- ollama（本地模型）
- deepseek
- zhipu
- mistral
- ……100+

###### 方式2：直接导入（适合固定模型）

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-3.5-turbo")
```

---

##### 四、最常用参数（所有模型通用）

```python
model = ChatOpenAI(
    temperature=0.7,      # 创造力 0~2
    max_tokens=1024,      # 最大输出长度
    api_key="sk-xxx",     # 密钥
    base_url="https://",  # 代理地址
    streaming=True,       # 开启流式
    verbose=True          # 打印调试信息
)
```

---

##### 五、ChatModel 工作流程

1. 输入：**消息列表**
2. 模型：**统一接口调用**
3. 输出：**AIMessage**
4. 可接：解析器、RAG、工具调用、记忆、链……

```
messages → ChatModel → AIMessage
```

---

##### 六、和 LCEL 无缝组合（核心）

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是专家"),
    ("user", "{input}")
])

chain = prompt | model | StrOutputParser()

chain.invoke({"input": "你好"})
```

**这就是 LangChain 最强大的地方：模型即组件。**

---

##### 七、和普通 LLM（Completion Model）的区别

| 类型          | 输入                        | 场景                   |
| ------------- | --------------------------- | ---------------------- |
| **ChatModel** | 多轮消息（system/human/ai） | 对话、助手、AI 应用    |
| LLM           | 纯文本提示                  | 补全、续写、旧接口模型 |

**现在 99% 的场景都用 ChatModel**

---

##### 八、支持的高级能力（所有模型通用）

###### 1. 结构化输出（JsonOutputParser）

```python
chain = prompt | model | JsonOutputParser(pydantic_object=Joke)
```

###### 2. 带记忆对话（RunnableWithMessageHistory）

```python
chain_with_history = RunnableWithMessageHistory(
    runnable=chain,
    get_session_history=get_session_history,
    input_messages_key="input"
)
```

###### 3. 工具调用（Function Calling）

```python
model.bind_tools([get_weather])
```

###### 4. 流式输出

```python
for chunk in model.stream(messages):
    print(chunk.content, end="")
```

---

##### 九、为什么必须学 ChatModel？

1. **统一代码**：一套代码跑所有模型
2. **生产稳定**：官方标准，长期支持
3. **接入极快**：切换模型只改 model 名
4. **LCEL 生态核心**：所有链、RAG、智能体都基于它

---

##### 十、一句话终极总结

**ChatModel = LangChain 的大模型统一标准接口**
**所有模型 → 封装成 ChatModel → 用同一套代码调用**

---

如果你愿意，我可以给你：
✅ **通用模型切换工具类（生产级）**
✅ **支持所有国产/海外/本地模型**
✅ **带流式、结构化输出、记忆、异常处理**

要我直接给你完整可复制代码吗？

---

#### 三、Prompt Templates 提示词模板

##### 作用

**动态拼接、统一管理提示词**，告别硬编码字符串，实现工程化开发。

##### 核心优势

1. 支持**占位符动态传参**
2. 提示词统一维护、方便修改
3. 适配不同模型格式
4. 快速切换业务场景（客服、面试、写作、总结）

##### 两种常用模板

###### 1. 普通字符串模板（适配LLM）

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate(
    template="请用{style}风格回答：{question}",
    input_variables=["style","question"]
)
text = prompt.format(style="幽默", question="如何学习Python")
```

###### 2. 聊天提示词模板（适配ChatModel）

```python
from langchain_core.prompts import ChatPromptTemplate

chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是{job}"),
    ("user", "回答：{query}")
])
msg = chat_prompt.format_messages(job="后端架构师", query="讲微服务优缺点")
```

#### **ChatPromptTemplate**

---

##### 一、它是什么？（一句话定义）

**ChatPromptTemplate 是 LangChain 中专门为「对话型大模型」设计的提示词模板**，用来**标准化、结构化、参数化**构造多角色对话（system / user / assistant），让你不用手动拼接字符串，代码更干净、更易维护。

简单理解：
**它 = 提示词的模具**
你往里填变量（参数），它自动输出标准格式的对话消息列表。

---

##### 二、为什么要用它？（核心价值）

1. **支持多角色消息**（system、user、assistant）
2. **支持动态传参**（{question}、{context}、{user_name}）
3. **代码更优雅**，不用拼接一堆字符串
4. **可复用、可维护**，适合生产环境
5. **和 LLM 接口完美对齐**（OpenAI、通义千问、文心一言等）

---

##### 三、核心结构（必背）

ChatPromptTemplate 由 **多条消息模板（MessagePromptTemplate）** 组成：

1. **System 系统消息**
   定义角色、规则、行为（如：你是客服助手、只回答技术问题）
2. **User 用户消息**
   用户真正的问题/输入
3. **Assistant 助手消息**
   可选，用于少样本学习（few-shot）

格式永远是：

```
[
    SystemMessage(content="..."),
    HumanMessage(content="..."),
    AIMessage(content="...")
]
```

---

##### 四、使用步骤（最常用写法）

###### 1. 从消息列表创建模板

```java
// Java 版（你是 Java 技术栈，我直接给 Java 版！）
List<ChatMessagePromptTemplate> messages = new ArrayList<>();

// 系统角色
messages.add(ChatMessagePromptTemplate.fromSystemMessage(
    "你是一个专业的Java技术助手，回答要简洁、准确、带示例"));

// 用户问题（带参数 {question}）
messages.add(ChatMessagePromptTemplate.fromUserMessage(
    "请解释：{question}"));

// 构建模板
ChatPromptTemplate promptTemplate = ChatPromptTemplate.of(messages);
```

###### 2. 填充变量，生成最终提示词

```java
Map<String, Object> variables = new HashMap<>();
variables.put("question", "ConcurrentHashMap put流程");

// 生成完整对话消息
PromptResult result = promptTemplate.format(variables);
List<ChatMessage> messages = result.getChatMessages();
```

###### 3. 直接丢给大模型

```java
chatModel.invoke(messages);
```

---

##### 五、最常用的 3 种创建方式（Java）

###### 1. 多角色模板（最常用）

```java
ChatPromptTemplate prompt = ChatPromptTemplate.builder()
    .system("你是一个AI助手，专业、简洁、有耐心")
    .user("用户问题：{query}")
    .build();
```

###### 2. 带上下文的 RAG 模板（企业级）

```java
ChatPromptTemplate prompt = ChatPromptTemplate.builder()
    .system("""
        使用下面的上下文回答用户问题。
        上下文：{context}
        """)
    .user("问题：{question}")
    .build();
```

###### 3. FewShot 少样本模板

```java
ChatPromptTemplate prompt = ChatPromptTemplate.builder()
    .system("你是分类助手")
    .user("订单问题")
    .assistant("订单类")
    .user("登录问题")
    .assistant("账号类")
    .user("{text}")  // 最终用户输入
    .build();
```

##### 历史对话

---

###### 一、它到底是干嘛的？（一句话）

**MessagesPlaceholder = 消息占位符**
作用：**把历史对话（history）动态插入到提示词模板里**，让大模型知道“之前聊过什么”。

也就是：
**让 AI 拥有记忆 → 实现真正的多轮对话。**

---

###### 二、代码完整含义

```python
ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),  # 系统角色
    MessagesPlaceholder(variable_name="history"),  # 历史对话占位符
    ("human", "{input}"),  # 用户最新问题
])
```

最终生成的结构是：

```
[
    系统提示词,
    历史对话1,
    历史对话2,
    历史对话3,
    ...
    用户最新问题
]
```

没有它会怎样？

- AI **没有记忆**
- 每次对话都是全新的
- 无法多轮聊天

有了它：

- AI **记得之前的对话**
- 能上下文理解
- 真正实现智能多轮交互

---

###### 三、MessagesPlaceholder 核心作用（必背）

1. **动态插入历史消息**
   把 `history` 里的多轮对话自动放进模板
2. **保持消息结构**
   不会破坏 system / user / assistant 格式
3. **支持任意长度历史**
   不管多少轮对话都能正确拼接
4. **LangChain 记忆（Memory）机制的标配**

---

###### 四、最精炼总结（填表/写成果直接复制）

**在 LangChain 对话模板中使用 MessagesPlaceholder 动态注入历史对话记录，实现多轮对话上下文记忆功能，让大模型能够基于历史交互内容进行连贯、上下文感知的智能回复，是构建具备长期记忆能力的 AI Agent 与对话系统的核心组件。**

---

##### 六、核心作用（写成果/面试都能用）

1. **结构化对话**
   严格按照 system / user / assistant 构造消息，符合大模型输入规范。
2. **动态参数注入**
   支持 {context}、{question}、{input} 等变量，适合 RAG、对话系统。
3. **提示词工程标准化**
   把提示词从代码中抽离，便于修改、管理、版本控制。
4. **提升模型效果稳定性**
   规范的角色设定 + 清晰的上下文 = 回答更稳定、可控。
5. **生产级可用**
   企业 Agent、智能问答、RAG 系统的**标配组件**。

---

##### 七、和普通 PromptTemplate 的区别（必背）

| 类型                   | 用途             | 格式               |
| ---------------------- | ---------------- | ------------------ |
| **PromptTemplate**     | 普通文本生成     | 单字符串           |
| **ChatPromptTemplate** | **对话模型专用** | **多角色消息列表** |

一句话区分：
**普通模板 = 一段话**
**Chat模板 = 一场对话（带角色）**

---

##### 八、最精炼总结（填表/写成果直接用）

**ChatPromptTemplate 是 LangChain 中面向对话大模型的标准化提示词模板，支持 system/user/assistant 多角色结构与动态参数注入，可实现提示词工程化、可复用、可维护，是构建 RAG 系统、智能对话 Agent、多轮交互应用的核心组件，大幅提升模型输出稳定性与开发效率。**

---

#### 四、Output Parsers 输出解析器

##### 作用

强制让大模型**按指定格式返回数据**，把自由文本 → 结构化数据。
解决模型回答杂乱、无法程序读取的痛点。

##### 主流3种解析器

###### 1. StrOutputParser 字符串解析

只提取纯文本，最基础，日常对话用

```python
from langchain_core.output_parsers import StrOutputParser
parser = StrOutputParser()
```

###### 2. JsonOutputParser JSON结构化解析

强制模型返回**标准JSON**，后端接口、数据提取必备
用途：抽取姓名、电话、地址、商品信息、表单数据

###### 3. PydanticOutputParser 实体类解析

绑定Python实体类，自动校验字段、类型、长度
企业级开发最规范，适合严肃业务数据提取

###### 完整流程串联（ModelI/O 标准链路）

**Prompt模板 → ChatModel大模型 → 输出解析器**

```
用户变量填充提示词 → 传给大模型 → 模型生成文本 → 解析成结构化数据
```

```python
# 完整标准链路
prompt = ChatPromptTemplate.from_messages([("user","总结这段话：{content}")])
model = ChatOpenAI()
parser = StrOutputParser()

# LCEL链式写法
chain = prompt | model | parser
result = chain.invoke({"content":"这里是长文本内容"})
```

#### **JsonOutputParser** 

##### 一、核心定位

**JsonOutputParser** 是 LangChain 内置的输出解析器，用于**将大模型的文本输出可靠地转为 JSON（Python dict/list）**，并支持**强制格式约束与流式增量解析**。

- 所属包：`langchain-core.output_parsers.json`
- 依赖：无强依赖（可选 Pydantic/Zod 做结构化校验）
- 适用场景：需要**结构化输出**（API 参数、配置、报表数据）、**机器直接处理**的场景

---

##### 二、工作原理

1. **提示词注入**：通过 `get_format_instructions()` 生成标准化格式要求，注入 Prompt，强制模型输出**纯净 JSON 字符串**（无解释、无 Markdown 包裹）。
2. **原始输出清洗**：自动移除首尾空格、换行、Markdown 代码块标记（```json），提取核心 JSON 文本。
3. **安全解析**：调用标准 JSON 解析器，转为 Python `dict`/`list`；失败则抛 `OutputParserException`。
4. **流式支持**：支持**增量解析**，边输出边生成部分 JSON 对象（区别于 PydanticOutputParser）。

---

##### 三、核心参数与方法

###### 1. 初始化参数（Python）

```python
class JsonOutputParser(BaseCumulativeTransformOutputParser[Any]):
    def __init__(
        self,
        *,
        schema: Optional[Union[BaseModel, Dict[str, Any]]] = None,
        diff: bool = False,
        name: Optional[str] = None
    )
```

- `schema`：可选，Pydantic 模型或 JSON Schema，用于**强校验输出结构**。
- `diff`：是否启用增量解析（默认 `False`）。
- `name`：解析器标识（调试用）。

###### 2. 关键方法

- `get_format_instructions() -> str`：生成**标准化格式提示**，注入 Prompt 指导模型输出合规 JSON。
- `parse(text: str) -> Any`：清洗并解析文本为 JSON 对象。
- `parse_stream(text: str) -> Iterator[Any]`：流式增量解析，逐块返回部分 JSON。

---

##### 四、基础用法（无 Schema）

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from langchain_openai import ChatOpenAI

# 1. 初始化组件
model = ChatOpenAI(model="gpt-3.5-turbo")
parser = JsonOutputParser()

# 2. 构建 Prompt，注入格式指令
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是数据提取专家，严格按格式输出。"),
    ("user", "提取以下文本的实体：{input}\n{format_instructions}")
]).partial(format_instructions=parser.get_format_instructions())

# 3. 构建链
chain = prompt | model | parser

# 4. 调用
result = chain.invoke({"input": "西安是陕西省省会，古称长安。"})
print(type(result))  # <class 'dict'>
print(result)
# 输出：{"city": "西安", "province": "陕西省", "ancient_name": "长安"}
```

**生成的 format_instructions 示例**：

```
输出必须是严格的 JSON 字符串，无任何额外解释、换行或代码块。
使用双引号，键名完整，值类型正确（字符串/数字/布尔/数组/对象）。
示例：{"key": "value"}
```

---

##### 五、高级用法（带 Pydantic Schema，强校验）

✔ **`JsonOutputParser` 同时支持两种写法：**

1. **`schema=PydanticModel`**
2. **`pydantic_object=PydanticModel`**

```python
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from langchain_openai import ChatOpenAI

# 1. 定义 Pydantic 模型（Schema）
class Entity(BaseModel):
    city: str = Field(description="城市名称")
    province: str = Field(description="省份名称")
    ancient_name: str = Field(description="古称")

# 2. 初始化带 Schema 的解析器
parser = JsonOutputParser(schema=Entity)

# 3. 构建 Prompt 与链
prompt = ChatPromptTemplate.from_messages([
    ("system", "严格按指定 JSON Schema 输出。"),
    ("user", "提取实体：{input}\n{format_instructions}")
]).partial(format_instructions=parser.get_format_instructions())

chain = prompt | ChatOpenAI() | parser

# 4. 调用（自动校验结构与类型）
result = chain.invoke({"input": "西安是陕西省省会，古称长安。"})
print(type(result))  # <class 'dict'>
print(result)  # {"city": "西安", "province": "陕西省", "ancient_name": "长安"}
```

**优势**：

- 自动生成**带字段描述的格式指令**，模型更易遵循。
- 解析时**强校验类型与必填字段**，不符合直接抛错，避免脏数据。

---

##### 六、流式解析（增量输出）

```python
# 沿用上面的 chain
for chunk in chain.stream({"input": "西安是陕西省省会，古称长安。"}):
    print(chunk)
```

**输出（增量过程）**：

```
{}
{'city': ''}
{'city': '西安'}
{'city': '西安', 'province': ''}
{'city': '西安', 'province': '陕西省'}
{'city': '西安', 'province': '陕西省', 'ancient_name': ''}
{'city': '西安', 'province': '陕西省', 'ancient_name': '长安'}
```

---

##### 七、常见问题与解决方案

1. **输出含 Markdown 代码块（```json ... ```）**
   - 原因：模型未遵循格式指令。
   - 解决：强化 Prompt，明确禁止代码块；或用 `SimpleJsonOutputParser`（自动剥离 Markdown）。

2. **解析失败（JSONDecodeError）**
   - 原因：输出有语法错误（单引号、 trailing comma、注释）。
   - 解决：
     - 用 **gpt-4/gpt-3.5-turbo** 等能力更强的模型。
     - 启用 `schema` 强校验，模型会更严谨。
     - 增加重试逻辑：`RetryOutputParser` 包裹。

3. **字段缺失/类型错误**
   - 解决：**必须用 Pydantic Schema**，定义必填字段与类型，解析时自动校验。

---

##### 八、与其他解析器对比

| 解析器                     | 核心能力          | 流式支持 | 结构化校验     | 适用场景         |
| -------------------------- | ----------------- | -------- | -------------- | ---------------- |
| **JsonOutputParser**       | 基础 JSON 解析    | ✅ 增量   | 可选（Schema） | 通用 JSON 输出   |
| **SimpleJsonOutputParser** | 自动清洗 Markdown | ✅ 增量   | ❌              | 模型易输出代码块 |
| **PydanticOutputParser**   | 强类型模型校验    | ❌ 全量   | ✅ 强制         | 复杂结构化数据   |

---

##### 九、总结

- **核心价值**：**标准化、可靠、可流式**地将 LLM 输出转为 JSON，打通“自然语言 → 结构化数据”的关键链路。
- **最佳实践**：
  - 简单场景：直接用 `JsonOutputParser()`。
  - 生产环境：**必加 Pydantic Schema**，强校验保障稳定性。
  - 流式需求：优先 `JsonOutputParser` 或 `SimpleJsonOutputParser`。

需要我把以上代码整理成一个可直接运行的完整示例（含依赖安装和异常重试逻辑）吗？

---

#### 五、Model I/O 整体工作流程总结

1. **PromptTemplate**：组装标准化、可动态修改的提问话术
2. **Model**：选择LLM/ChatModel/Embedding完成语义交互
3. **OutputParser**：规整模型输出，方便程序调用、入库、前端展示

#### 六、面试高频考点

1. LLM和ChatModel核心区别？
   LLM纯文本输入输出；ChatModel基于消息角色，支持多轮对话与工具调用。
2. Embedding模型作用？
   文本向量化，用于知识库相似度检索，是RAG核心。
3. PromptTemplate意义？
   解耦提示词与业务代码，动态传参，便于维护迭代。
4. OutputParser解决什么问题？
   统一模型输出格式，让大模型结果能被代码直接解析使用。

需要我给你整理**Model I/O全套可直接运行实战代码**吗？



### 2. 检索 (Retrieval)

解决 LLM 知识过时与私有数据问题，是**RAG（检索增强生成）**的核心。

- **文档加载器 (Document Loaders)**: 从各种来源（PDF、网页、数据库）加载数据为 `Document` 对象。
- **文本分割器 (Text Splitters)**: 将长文档切分为适合模型处理的小块（Chunk）。
- **向量存储 (Vector Stores)**: 存储和检索文本向量的数据库（如 Pinecone、Chroma、FAISS）。
- **检索器 (Retrievers)**: 根据用户查询，从向量库中召回最相关的文档片段。

#### 整体核心作用

解决**大模型知识库滞后、无法使用企业私有数据**两大痛点，是 **RAG 检索增强生成** 整套流程核心骨架。
完整流程：**加载文档 → 切片分块 → 向量化入库 → 语义检索 → 拼接上下文给大模型**

---

#### 1. 文档加载器 Document Loaders

##### 作用

从**不同数据源**读取内容，统一封装成 LangChain 标准 `Document` 文档对象。
`Document` 结构：

- `page_content`：正文文本内容
- `metadata`：元数据（文件名、路径、页码、来源、时间等）

##### 支持主流数据源

1. **本地文件**
   PyPDFLoader(PDF)、Docx2txtLoader(Word)、TextLoader(TXT)、CSVLoader、ExcelLoader
2. **网页数据**
   WebBaseLoader、SeleniumLoader、爬取博客/官网/教程
3. **知识库/平台**
   NotionLoader、MarkdownLoader
4. **数据库**
   SQLDatabaseLoader 读取 MySQL/PostgreSQL 数据

##### 简单示例

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("test.pdf")
docs = loader.load() # 加载全部文档
print(docs[0].page_content)
print(docs[0].metadata)
```

---

#### 2. 文本分割器 Text Splitters

##### 作用

大模型有**上下文长度限制**，长文档必须切成小块 Chunk，才能正常向量化+检索。
切分原则：**语义尽量完整，不割裂语句**

##### 常用分割器

1. **RecursiveCharacterTextSplitter 递归字符分割（最常用）**
   优先按段落、换行、句号、逗号逐级切割，语义保留最好
2. **CharacterTextSplitter 普通字符分割**
   单纯按固定符号切割，容易断句
3. **MarkdownTextSplitter**
   专门切分 Markdown 文档，保留标题层级
4. **TokenTextSplitter**
   按大模型 Token 数量切割，精准控制长度

##### 核心参数

- `chunk_size`：每块最大字符/Token 长度
- `chunk_overlap`：块与块重叠长度（**关键**，防止上下文断裂丢失信息）

##### 示例

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=800,
    chunk_overlap=150
)
split_docs = splitter.split_documents(docs)
```

---

#### 3. 向量数据库 Vector Stores

##### 作用

把**文本块 → 向量**，存入专门做**相似度搜索**的数据库，替代传统关键词搜索。
原理：语义相近文本，向量空间距离更近。

##### 主流向量库分类

1. **轻量本地免费（开发首选）**
   - **Chroma**：纯本地，零部署，本地RAG首选
   - **FAISS**：Facebook开源，检索速度极快，无持久化
2. **开源部署型**
   - Milvus、Qdrant、Weaviate
3. **云端商用**
   Pinecone、Weaviate Cloud、阿里云向量库

##### 存入向量库完整逻辑

1. 分割好的文本块
2. 调用 Embeddings 转为向量
3. 文本+向量+元数据 一起存入向量库

##### 极简入库示例

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = OpenAIEmbeddings()
# 存入向量库
vector_db = Chroma.from_documents(
    documents=split_docs,
    embedding=embeddings,
    persist_directory="./chroma_db" # 持久化保存
)
vector_db.persist()
```

---

#### 4. 检索器 Retrievers

##### 作用

用户输入问题 → 转为向量 → 在向量库**匹配相似度最高文档块**，召回作为参考上下文。
是连接**用户提问**和**私有知识库**的入口。

##### 检索常用模式

1. **相似度检索 Similarity Search**
   默认模式，返回语义最相似 Top-K 文档
2. **最大边际相关性 MMR**
   优先召回相似且内容不重复文档，避免检索结果同质化
3. **相似度阈值检索**
   只返回达到相似度分数的内容，过滤无关文档

##### 检索使用示例

```python
# 转为检索器
retriever = vector_db.as_retriever(search_kwargs={"k": 3})
# 根据问题召回相关文档
search_docs = retriever.invoke("Java并发锁原理")
```

---

##### 完整 RAG 检索整条链路

1. Document Loaders → 读取各类文件
2. Text Splitters → 长文本切块
3. Embeddings → 文本向量化
4. Vector Stores → 向量持久化存储
5. Retrievers → 用户问题语义召回文档
6. 召回文档 + 用户问题 拼接 Prompt → 送入 LLM/ChatModel 生成答案

---

##### 面试高频考点

1. 检索模块解决什么问题？
   解决大模型知识过时、无法使用本地私有业务数据，实现知识库问答。
2. 文本分割为什么要设置 overlap 重叠？
   防止重要上下文被切分断裂，保证语义连贯。
3. 向量库和普通数据库区别？
   普通库关键词匹配，向量库**语义相似度匹配**，理解句子含义。
4. Retriever 作用是什么？
   从海量知识库中快速筛选和用户问题最相关的片段，减少Prompt冗余Token。
5. 轻量级本地RAG优先用什么？
   Chroma + FAISS，无需搭建服务，开箱即用。

需要我给你写**一整套可直接运行的完整RAG检索代码**吗？

### 3. 记忆 (Memory)

为对话系统提供**上下文感知**能力，记录历史交互信息。

- **ConversationBufferMemory**: 简单缓存所有历史消息。
- **SummaryMemory**: 随对话进行，动态生成并维护历史摘要，节省 token。
- **VectorStoreRetrieverMemory**: 将历史对话存入向量库，按语义召回相关上下文。
- **RunnableWithMessageHistory**: 现代、可组合的记忆封装，兼容 LCEL。



#### 核心作用

给对话赋予**上下文记忆**，让大模型记住历史聊天内容，实现连续对话；解决单轮问答无上下文、多轮对话token爆炸、长对话遗忘关键信息等问题，是聊天机器人必备模块。

#### 整体原理

把用户历史提问、AI历史回答统一存储，每次新对话自动拼接历史上下文，再送入大模型。

---

#### 1. ConversationBufferMemory 缓存对话记忆

##### 核心特点

- **最简单基础**，完整存储**所有原始聊天记录**
- 不做任何压缩、精简，原样保存对话
- 优点：上下文最全、不会丢失信息
- 缺点：对话越长，token消耗越大，容易超出模型上下文限制

##### 适用场景

短轮次闲聊、简单客服对话、对话轮次少的场景

##### 代码示例

```python
from langchain_openai import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI()
memory = ConversationBufferMemory()
conv_chain = ConversationChain(llm=llm, memory=memory, verbose=True)

conv_chain.predict(input="我叫小明")
conv_chain.predict(input="我叫什么名字") # 能记住
```

---

#### 2. SummaryMemory 对话摘要记忆

##### 核心特点

- **不存完整对话**，自动把历史对话浓缩成**简短摘要**
- 边聊天边总结，持续精简历史内容
- 大幅减少token占用，支持**超长多轮对话**
- 缺点：过度精简会丢失细节信息

##### 细分两类

1. **ConversationSummaryMemory**：实时生成全局对话摘要
2. **ConversationSummaryBufferMemory**：先存原始对话，达到阈值再生成摘要，兼顾细节与节省token

##### 适用场景

长对话陪伴聊天、连续多轮业务咨询、智能助手

---

#### 3. VectorStoreRetrieverMemory 向量库记忆

##### 核心特点

- 把**历史对话存入向量数据库**
- 不再全量拼接历史，而是**根据当前问题语义召回相关历史对话**
- 精准调取有用记忆，彻底解决长对话冗余问题
- 属于**长期记忆**，可持久化保存聊天记录

##### 工作流程

1. 聊天记录自动向量化存入向量库
2. 用户发起新提问
3. 语义匹配召回最相关历史对话
4. 只拼接相关记忆，过滤无关内容

##### 适用场景

私人知识库对话、个性化长期AI助手、历史聊天复盘问答

---

#### 4. RunnableWithMessageHistory 现代标准记忆（主流推荐）

##### 核心特点

- LangChain 新版官方标准记忆方案
- 完美兼容 **LCEL 链式语法**，可自由组合提示词、模型、检索链
- 支持**多用户会话隔离**，不同用户独立记忆互不干扰
- 灵活搭配任意存储：内存、Redis、数据库、文件
- 企业级开发首选，替代老式 ConversationChain

##### 核心优势

1. 解耦记忆与业务链，结构清晰
2. 支持会话ID区分用户
3. 可接入分布式存储，支持线上部署
4. 适配RAG+记忆混合对话场景

##### 核心使用结构

```
对话链 + 会话历史管理 + 独立用户记忆
```

`RunnableWithMessageHistory` 是 LangChain 为 **LCEL 链式语法**设计的「会话记忆包装器」，核心作用是**给任意链自动加上多轮对话记忆**，彻底替代旧的 `ConversationChain`，是现在官方推荐的标准用法。

下面从**定义、核心参数、工作原理、完整示例、存储后端、对比旧方案**六方面讲透。

---

##### 一、它是什么？（一句话）

**给任意 Runnable（链/模型）套一层“记忆壳”**：

- 自动根据 `session_id` 加载历史
- 自动把历史注入 Prompt
- 自动把新问答保存回历史
- 全程无手动拼接历史，开箱即用

---

##### 二、核心参数（必懂）

```python
RunnableWithMessageHistory(
    runnable: Runnable,                # 你原来的链（prompt | model | parser）
    get_session_history: Callable,     # 工厂函数：根据 session_id 返回历史对象
    input_messages_key: str,           # 用户输入在 dict 里的 key（如 "input"）
    history_messages_key: str,         # 历史消息在 prompt 里的 key（对应 MessagesPlaceholder）
    output_messages_key: Optional[str] = None  # AI 输出的 key（默认自动识别）
)
```

- **`get_session_history`**：最关键，是个**工厂函数**，不是对象！每次调用按 `session_id` 拿到独立历史，天然支持多用户/多会话。
- **`session_id`**：在 `config={"configurable": {"session_id": "xxx"}}` 里传，区分不同会话。

---

##### 三、工作原理（四步走）

每次调用 `invoke` 自动执行：

1. **加载历史**：从 config 取 `session_id` → 调用 `get_session_history` → 拿到该会话的历史消息列表。
2. **注入 Prompt**：把历史列表塞进 `history_messages_key`（对应 Prompt 里的 `MessagesPlaceholder`）。
3. **执行链**：用「历史+当前输入」调用你的 Runnable，拿到输出。
4. **保存新消息**：把「用户输入」和「AI 输出」追加到该会话的历史里。

> 无历史时自动创建；同 `session_id` 共享历史；不同 `session_id` 完全隔离。

---

##### 四、完整可运行示例（直接复制可用）

###### 1. 依赖导入

```python
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
```

###### 2. 1分钟实现多轮对话记忆

```python
# 1. 内存存储（生产换 Redis/DB）
store = {}
def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

# 2. 构建基础链（含历史占位符）
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是助手，记住对话历史，简洁回答。"),
    MessagesPlaceholder(variable_name="history"),  # 历史消息注入这里
    ("human", "{input}")
])
base_chain = prompt | ChatOpenAI() | StrOutputParser()

# 3. 包装记忆（核心一行）
chain_with_mem = RunnableWithMessageHistory(
    runnable=base_chain,
    get_session_history=get_session_history,
    input_messages_key="input",       # 用户输入 key
    history_messages_key="history"    # 历史对应 Prompt 里的 key
)

# 4. 调用（带 session_id，多轮记忆）
config = {"configurable": {"session_id": "user_001"}}  # 会话ID

# 第一轮
res1 = chain_with_mem.invoke({"input": "我叫小明"}, config=config)
print(res1)  # 你好小明！有什么可以帮你？

# 第二轮（自动记住名字）
res2 = chain_with_mem.invoke({"input": "我叫什么？"}, config=config)
print(res2)  # 你叫小明。
```

---

##### 五、支持的存储后端（开发→生产）

- **开发/测试**：`InMemoryChatMessageHistory`（内存，重启丢失）。
- **文件持久化**：`FileChatMessageHistory`（JSON 文件，简单持久化）。
- **生产环境（推荐）**：`RedisChatMessageHistory`（高性能、分布式、支持 TTL）。
- **数据库**：`PostgresChatMessageHistory`/`SQLiteChatMessageHistory`（持久化到 DB）。

切换存储只需改 `get_session_history` 的实现，上层代码完全不用动，**解耦极强**。

---

##### 六、对比旧方案（为什么用它？）

###### 旧：ConversationChain + Memory（淘汰）

- 耦合重：链和记忆绑定，无法复用基础链。
- 不灵活：难与 LCEL、RAG、工具调用组合。
- 扩展性差：多会话/多存储后端支持弱。

###### 新：RunnableWithMessageHistory（官方推荐）

- **解耦**：记忆是独立包装器，基础链可复用。
- **灵活**：无缝搭配 LCEL、RAG、工具、流式输出。
- **强扩展**：原生支持多会话、多存储后端、分布式。
- **类型安全**：参数清晰，IDE 友好提示。

---

##### 七、关键注意点

1. **Prompt 必须有 `MessagesPlaceholder`**：`history_messages_key` 要和 `variable_name` 一致，否则历史注入不进去。
2. **`session_id` 必须传**：在 `config` 里指定，否则报错；不同 ID 隔离历史。
3. **工厂函数无状态**：`get_session_history` 不能带缓存，每次按 `session_id` 返回最新历史。
4. **内存存储不适合生产**：重启丢失、多实例不同步，生产必换 Redis/DB。

---

要不要我把上面示例改造成**带Redis持久化+流式输出+多会话隔离**的生产可用版本？





---

#### 四类记忆对比总结

| 记忆类型                   | 存储形式         | 优点                       | 缺点                | 适用场景               |
| -------------------------- | ---------------- | -------------------------- | ------------------- | ---------------------- |
| ConversationBufferMemory   | 完整原始对话     | 信息最全、最简单           | 耗token、长对话超限 | 短轮次简单对话         |
| SummaryMemory              | 对话精简摘要     | 极度省token、支持长聊      | 丢失细节            | 超长连续闲聊           |
| VectorStoreRetrieverMemory | 向量语义记忆     | 精准调取、长期存储         | 依赖向量库          | 个性化长期记忆         |
| RunnableWithMessageHistory | 会话式结构化记忆 | 工程化强、多用户、兼容LCEL | 配置稍复杂          | 线上项目、企业级AI对话 |

---

#### 面试高频考点

1. Memory 模块作用是什么？
   存储对话历史，实现多轮上下文连续对话，让模型记住聊天内容。
2. BufferMemory 和 SummaryMemory 区别？
   前者存全部原文，信息全耗token；后者压缩成摘要，节省token丢失细节。
3. 长对话token溢出怎么解决？
   使用摘要记忆、向量检索记忆、分段压缩记忆。
4. 线上项目做用户聊天记忆用什么？
   优先 **RunnableWithMessageHistory**，支持会话隔离、持久化、分布式部署。
5. 向量记忆优势？
   不用拼接全部历史，语义匹配调取相关对话，大幅提升对话效率。

需要我给你写**RunnableWithMessageHistory 多用户记忆完整可运行代码**吗？

### 4. 链 (Chains)

将多个组件**按固定顺序**串联，形成可复用的工作流。

- **基础链**: `LLMChain`（提示词+模型）、`RetrievalChain`（检索+生成）。

- **LCEL (LangChain Expression Language)**: 新一代组合语法，用 `|` 运算符连接组件，支持流式输出、并行执行、类型检查。

  ```python
  # LCEL 示例：提示词 -> 模型 -> 解析器
  chain = prompt | chat_model | output_parser
  ```

这是**把所有组件串起来干活的核心**，没有 Chains，模型、提示词、检索、记忆都是散的，串不起来就无法形成完整AI应用。

---

#### 一、Chains 到底是什么？

一句话：
**把多个组件按固定顺序串成一条流水线 = 链（Chain）**

就像工厂流水线：
原料（用户问题）→ 提示词加工 → 模型处理 → 输出解析 → 成品（最终答案）

**Chains = 组件 + 执行顺序**

---

#### 二、两大类链：传统基础链 vs 新一代 LCEL 链

##### 1. 传统基础链（Legacy Chains）

早期 LangChain 用的封装链，简单但不灵活，**现在逐步被淘汰**。

###### 最常用的两种：

1. **LLMChain**
   提示词 + 模型 最基础链

   ```python
   from langchain.chains import LLMChain
   chain = LLMChain(prompt=prompt, llm=chat_model)
   ```

```
   
2. **RetrievalQAChain**
   检索 + 问答链，RAG 专用
   
   ```python
   from langchain.chains import RetrievalQA
   chain = RetrievalQA.from_llm(llm=chat_model, retriever=retriever)
```

###### 缺点

- 写法臃肿
- 不支持流式输出
- 不支持并行
- 调试困难
- 不兼容新组件

---

#### 三、LCEL（新一代标准，必须掌握）

##### 1. 什么是 LCEL？

**LangChain Expression Language**
LangChain 官方主推的**链式组合语法**。

**用 | 符号把组件串起来**，像管道一样。

```python
chain = 提示词 | 模型 | 输出解析器
```

##### 2. 为什么必须用 LCEL？

✅ **超级简洁**，一行串完
✅ **支持流式输出**（打字机效果）
✅ **支持异步、并行**
✅ **自动重试、日志、监控**
✅ **完美兼容记忆、检索、代理**
✅ **企业级、线上项目唯一标准写法**

##### 3. 标准 LCEL 结构

###### 最基础对话链

```python
prompt = ChatPromptTemplate.from_template("回答：{question}")
model = ChatOpenAI()
parser = StrOutputParser()

# LCEL 管道
chain = prompt | model | parser
```

###### RAG 检索链

```python
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | parser
)
```

###### 带记忆对话链

```python
chain = RunnableWithMessageHistory(
    基础链,
    get_session_history=get_history
)
```

##### 4. LCEL 超级能力

1. **流式输出**

   ```python
   for chunk in chain.stream({"question": "讲Java"}):
       print(chunk, end="")
   ```

2. **并行执行**
   同时调用多个模型/检索器，速度翻倍

3. **自动异常重试**
   模型超时自动重试，不用写代码

4. **完整可观测**
   每一步输入输出都能在 LangSmith 看到

---

#### 四、Chains 完整执行流程

用户输入 → 链接收 → 按顺序执行组件 → 输出最终结果

**标准流水线：**

1. 接收用户问题
2. 填充提示词模板
3. 发送给大模型
4. 接收模型返回
5. 解析输出格式
6. 返回给用户

---

#### 五、最常用的 4 条标准链

##### 1. 基础对话链

提示词 → 模型 → 解析
用途：聊天、问答

##### 2. RAG 检索链

检索 → 提示词 → 模型 → 解析
用途：知识库问答（最常用）

##### 3. 带记忆对话链

记忆历史 → 提示词 → 模型 → 解析
用途：多轮对话机器人

##### 4. 结构化输出链

提示词 → 模型 → JSON解析
用途：提取信息、表单、数据解析

---

#### 六、一句话总结（最重要）

- **Chains = 组件流水线**
- **传统链 = 老写法，不推荐**
- **LCEL = 现代标准，必须会**
- **| 符号 = 串联组件**
- **所有企业级AI项目 = 100% 用 LCEL**

---

#### 面试必问

1. Chains 作用？
   把多个组件串成流水线，实现完整业务逻辑。

2. LCEL 是什么？
   Langchain 表达式语言，用 | 串联组件，现代标准写法。

3. LCEL 优势？
   简洁、流式、并行、重试、可观测、企业级标准。

4. LLMChain 和 LCEL 区别？
   LLMChain 是旧封装；LCEL 灵活、现代、功能更强。

---

需要我给你**4 条最常用 LCEL 完整可运行代码（基础对话、RAG、记忆、结构化输出）**吗？

### 5. 代理 (Agents)—Tool

赋予 LLM**动态决策**能力，使其能根据当前状态选择下一步行动（调用工具）。

- **核心要素**:
  - **工具 (Tools)**: LLM 可调用的外部能力（如搜索引擎、计算器、API）。
  - **工具调用 (Tool Calling)**: 模型输出结构化的工具调用请求。
- **常见类型**:
  - **ReAct Agent**: 基于“思考-行动-观察”循环。
  - **OpenAI Tool Calling Agent**: 利用模型原生工具调用能力，更稳定。

#### 核心作用

普通**链(Chain)**是**固定流程**，写死先做什么再做什么；
**代理(Agent)**是**自主决策**，让大模型自己判断：**要不要调用工具、调用哪个工具、调用几次**，自动完成复杂任务。

#### 核心逻辑

用户复杂问题 → LLM 自主思考 → 判断是否需要工具 → 调用外部工具 → 获取结果 → 整合答案输出
**思考 → 行动 → 观察 → 再思考** 循环执行

#### 一、核心概念

##### 1. 工具 Tools

**定义**：给大模型**外接执行能力**，弥补模型短板

- 局限：大模型**知识截止、不会计算、不能联网、无法操作业务接口**
- 作用：让模型能**查数据、算公式、搜实时信息、执行代码、调用业务接口**

**内置常用工具一览**

| 工具                     | 作用                                 |
| ------------------------ | ------------------------------------ |
| `Calculator`             | 高精度数学四则运算、复杂算式计算     |
| `SerpAPI`/`TavilySearch` | 联网实时搜索（新闻、天气、最新资讯） |
| `WikipediaQueryRun`      | 维基百科知识检索                     |
| `PythonREPL`             | 动态运行 Python 代码、数据分析       |
| `SQLDatabaseTool`        | 数据库查询工具                       |
| `FileTool`               | 本地文件读写                         |

##### 2. 工具调用 Tool Calling

**定义**：大模型不再只输出自然语言，而是输出**标准化结构化调用参数**
流程：

1. LLM 判断**需要调用工具**
2. 输出固定格式：**工具名 + 入参**
3. LangChain 自动解析、执行工具
4. 把工具执行结果丢回大模型
5. 模型整合结果给出最终答案

**核心优势**：全自动决策调用，无需人工判断

---

#### 二、三种自定义工具写法（必掌握）

##### 写法1：@tool 装饰器（最简最常用）

```python
from langchain.tools import tool

@tool
def get_city_weather(city: str) -> str:
    """
    查询指定城市实时天气
    Args:
        city: 城市中文名
    """
    # 这里可对接真实天气API
    return f"{city} 今日晴，气温26℃，微风"

@tool
def calculate_math(expr: str) -> str:
    """
    数学算式计算
    Args:
        expr: 数学表达式，如 12*5+99
    """
    return str(eval(expr))
```

- 函数**文档字符串必须写**：模型靠文档识别工具用途、入参
- 自动生成**工具名称、参数描述**，适配所有支持FunctionCall模型

##### 写法2：继承 BaseTool 类（复杂业务工具）

适合需要**初始化参数、状态、复杂逻辑**场景

```python
from langchain.tools.base import BaseTool
from langchain_core.callbacks import CallbackManagerForToolRun
from typing import Optional

class OrderQueryTool(BaseTool):
    name = "order_query"
    description = "根据订单号查询订单物流信息"

    def _run(self, order_no: str, run_manager: Optional[CallbackManagerForToolRun] = None) -> str:
        # 对接后端业务接口
        return f"订单{order_no}已发货，正在运输中"
```

##### 写法3：StructuredTool 结构化多参数工具

支持**多个入参、类型严格约束**

```python
from langchain.tools import StructuredTool

def user_info_query(name: str, age: int) -> str:
    """查询用户基础信息
    :param name: 用户名
    :param age: 用户年龄
    """
    return f"用户：{name}，年龄：{age}，信息正常"

user_tool = StructuredTool.from_function(user_info_query)
```

---

#### 三、完整工具调用实战（Agent + ToolCalling）

##### 1. 完整可运行代码

```python
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain.tools import tool

# 1. 定义工具
@tool
def weather_query(city:str) -> str:
    """查询城市天气，传入城市名称"""
    return f"{city}今天多云，气温24℃"

@tool
def calc(expr:str) -> str:
    """数学计算工具，传入数学表达式"""
    return str(eval(expr))

tools = [weather_query, calc]

# 2. 初始化支持工具调用的 ChatModel
model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# 3. 构建Agent提示词
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是智能助手，优先使用工具完成用户问题"),
    ("user", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# 4. 创建工具调用Agent
agent = create_openai_tools_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 5. 执行对话（自动调用工具）
res = agent_executor.invoke({"input": "帮我算 18*36 是多少，再查北京天气"})
print(res["output"])
```

##### 2. 执行流程拆解

1. 用户提问包含**计算+查天气**
2. 模型自动识别**两个工具**
3. 依次生成工具调用参数
4. 框架自动执行 `calc`、`weather_query`
5. 拿到结果后整理成自然语言回答

---

#### 四、ToolCalling 底层交互格式

模型输出**非文本**，而是 JSON 结构调用指令

```json
{
  "name": "calc",
  "parameters": {
    "expr": "18*36"
  }
}
```

LangChain 内置解析器自动识别、分发执行。

---

#### 五、核心使用场景

1. **实时信息问答**：联网搜索新闻、股市、赛事
2. **数理计算**：复杂公式、统计运算
3. **业务系统打通**：查订单、查会员、查库存、审批流程
4. **数据处理**：运行Python代码做统计、绘图、格式转换
5. **知识库增强**：结合百科、私有文档精准回答

---

#### 六、高频易错点

1. **工具无文档字符串** → 模型不知道什么时候调用
2. **参数类型不匹配** → 调用失败
3. 模型不支持 FunctionCall
   - 海外：gpt-3.5-turbo/gpt-4 全支持
   - 国产：通义、文心、DeepSeek 均适配 LangChain 工具调用
4. `verbose=True` 开启可查看**完整工具调用日志**，调试必备

---

#### 七、工具 + 记忆 组合（业务常用）

搭配 `RunnableWithMessageHistory` 实现**多轮对话+记忆+工具调用**

- 记住历史对话上下文
- 遇到需要外部能力自动调用工具
- 企业级智能客服、办公助手标准架构

需要我给你写**Streamlit 可视化工具调用Demo**，或者**联网搜索实时天气真实接口工具**吗？

### 5. 代理 (Agents)—Agent

#### 一、Agent 核心定义

**智能代理 = 大模型 + 思考决策 + 工具调用 + 自主执行**
不再是人指挥模型，**模型自己思考：要不要调用工具、调用哪个工具、传什么参数**，自动完成复杂任务。

**核心三要素**

1. **大脑**：ChatModel（负责思考、决策、判断）
2. **手脚**：Tools 工具（执行外部动作）
3. **调度**：Agent 调度逻辑（规划→调用→观察→总结）

#### 二、Agent 解决什么问题

- 单轮问答做不了**多步骤复杂任务**
- 不知道何时联网、何时计算、何时查数据
- 需要**自主规划、分步执行、纠错重试**

**典型场景**
旅行规划、数据分析、代码编写、智能办公助手、自动化流程、知识库问答+联网补充

#### 三、Agent 运行核心流程（思维链）

1. **思考 Thought**：分析用户需求，判断是否需要工具
2. **行动 Action**：选择工具 + 生成调用参数
3. **观察 Observation**：拿到工具返回结果
4. **循环迭代**：结果不足继续调用工具
5. **输出答案**：信息足够，整理最终回答

#### 四、LangChain 主流 Agent 类型

##### 1. OpenAI Tools Agent（最常用、生产首选）

- 基于**Function Calling**原生工具调用
- 稳定性最高、解析最准、支持多工具并行调用
- 适配：GPT3.5/4、通义、DeepSeek 等所有支持函数调用模型
- 创建函数：`create_openai_tools_agent()`

###### 原理

依托 GPT 系列**原生内置工具调用能力**，底层原生支持函数调用

###### 特点

- 调用格式标准、成功率极高、极少出错
- 执行效率高，无需复杂提示词约束
- 仅适配支持原生函数调用的模型

###### 适用

线上商业项目、生产环境、高精度工具调用场景

##### 2. ReAct Agent（经典思维框架）

- 思想：**思考+行动** 交替执行
- 不靠函数调用，靠模型文本推理解析工具
- 兼容所有大模型，无接口依赖
- 创建：`create_react_agent()`

###### 原理

遵循 **Thought(思考) → Action(行动) → Observation(观察)** 三段式思维链
靠提示词引导模型自己推理决策，**不依赖模型原生函数调用**，通用性最强。

###### 特点

- 开源模型、各类大模型全都适配
- 逻辑透明，能看到每一步思考过程
- 稳定性一般，复杂问题容易格式错乱

###### 适用

本地开源大模型、通用简单智能任务

##### 3. Structured Chat Agent

支持**多参数复杂工具**，适合企业复杂业务接口调用

##### 4. Plan-and-Execute 规划执行Agent

**先整体规划步骤，再分步执行**
适合长流程、多步骤复杂任务（写方案、做报表、流程自动化）

#### 五、核心组成代码结构

```python
# 1.导入
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_core.prompts import ChatPromptTemplate

# 2.定义工具
@tool
def search_info(keyword:str) -> str:
    """联网搜索实时信息"""
    return f"{keyword}最新资讯内容"

@tool
def calculate(formula:str) -> str:
    """数学计算"""
    return str(eval(formula))

tools = [search_info, calculate]

# 3.初始化模型
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# 4.构建Agent提示词
prompt = ChatPromptTemplate.from_messages([
    ("system","你是智能助手，合理使用工具完成用户问题"),
    ("human","{input}"),
    ("placeholder","{agent_scratchpad}")
])

# 5.创建Agent + 执行器
agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 6.调用执行
result = agent_executor.invoke({"input":"查今天新闻，再算25*48"})
print(result["output"])
```

#### 六、AgentExecutor 执行器核心参数

- `agent`：创建好的智能代理
- `tools`：可用工具列表
- `verbose=True`：**打印完整思考过程**（调试必备）
- `max_iterations`：最大迭代次数（防止死循环）
- `handle_parsing_errors=True`：自动解析错误重试

#### 七、Agent 五大特点

1. **自主决策**：模型自己选工具，不用人工指定
2. **多轮调用**：一个问题连续调用多个工具
3. **上下文感知**：结合对话历史做决策
4. **错误自愈**：参数错误自动修正重调
5. **可记忆**：搭配 `RunnableWithMessageHistory` 实现**记忆+Agent**多轮智能对话

#### 八、Agent 与 Chain 区别（面试必问）

1. **Chain 链**
   固定流程、固定顺序、**人定义执行步骤**，适合确定流程
2. **Agent 智能代理**
   动态流程、**模型自主决定步骤**，适合不确定、开放式复杂任务

#### 九、企业级常用组合架构

1. **Agent + 联网搜索**：实时资讯、舆情分析
2. **Agent + 计算器+代码工具**：数据分析、报表生成
3. **Agent + 数据库工具**：智能BI、自然语言查库
4. **Agent + 记忆 + 结构化输出**：智能客服、私人助理
5. **Agent + RAG**：私有知识库 + 外部实时信息 混合问答

#### 十、常见坑

1. 工具**缺少注释文档** → 模型不会调用
2. 温度 temperature 太高 → 决策混乱乱调用工具
3. 未限制最大迭代数 → 无限循环调用工具
4. 提示词没加 `agent_scratchpad` → Agent 无法记录思考过程

#### 十一、一句话总结

**Chain 是死板固定流水线，Agent 是会思考、会动手、会办事的智能员工。**

需要我给你整理 **Agent 面试题+答案**，或者 **带记忆的多轮Agent完整代码**吗？

### 5.代理区别

---

#### Agent 和 Chain 最核心区别

1. **Chain 链**
   流程**固定写死**，顺序不能变，适合确定流程任务
   例：知识库问答、文案生成、固定格式总结

2. **Agent 代理**
   流程**动态可变**，模型自主选工具、定步骤
   例：查天气+算数据+联网查资料组合复杂任务

---

#### 简单使用示例

```python
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain.tools import Calculator

# 1. 定义工具
tools = [Calculator()]

# 2. 模型 + 提示词
llm = ChatOpenAI(model="gpt-3.5-turbo")
prompt = ChatPromptTemplate.from_messages([
    ("system","你可以使用工具完成数学计算"),
    ("user","{input}"),
    ("placeholder","{agent_scratchpad}")
])

# 3. 创建代理
agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 4. 执行自动调用计算器
agent_executor.invoke({"input":"1234乘以5678等于多少"})
```

---

#### 面试高频考点

1. Agent 解决什么问题？
   打破大模型静态知识局限，自主调用外部工具完成不确定、多步骤复杂任务。

2. ReAct 与 OpenAI 工具调用代理区别？
   ReAct靠提示词推理，通用性强；OpenAI原生调用格式标准、稳定性更强。

3. Agent 执行流程？
   思考需求 → 选择工具 → 执行工具 → 接收结果 → 整合回答。

4. 什么时候用Chain，什么时候用Agent？
   流程固定用Chain；任务不确定、需要联网/计算/多方调用用Agent。

5. 工具(Tool)作用？
   拓展LLM能力边界，实现实时数据获取、运算、业务接口调用。

### 6. 回调 (Callbacks)

**事件驱动**系统，用于在组件生命周期的关键节点（如开始、结束、错误）注入自定义逻辑。

- **用途**: 日志记录、监控、限流、数据追踪、可视化。
- **集成**: 原生支持 LangSmith、OpenTelemetry 等。

这是 LangChain 的**监控、日志、调试、追踪系统**，相当于给你的 AI 应用装了一个**黑匣子 + 监控器**。

---

#### 一、Callbacks 到底是什么？

一句话解释：
**在模型运行的每一个关键节点，自动触发你写的自定义代码。**

它是**事件驱动**的：

- 链开始运行 → 触发 `on_chain_start`
- 模型开始调用 → 触发 `on_llm_start`
- 模型返回结果 → 触发 `on_llm_end`
- 出现报错 → 触发 `on_error`

你可以在这些节点**插入自己的逻辑**：打印日志、保存数据、报警、统计耗时。

---

#### 二、核心用途（工作中最常用）

##### 1. 日志打印（Verbose）

自动打印每一步执行细节：

- 提示词是什么
- 模型返回了什么
- 检索到哪些文档
- 代理思考过程
  不用自己写 print

##### 2. 调试排错

看到完整执行链路，快速定位：

- 提示词是否正确
- 模型是否调用成功
- 检索是否召回正确内容
- 工具调用是否失败

##### 3. 监控与埋点

- 记录调用耗时
- 统计 Token 消耗
- 监控错误率
- 用户行为追踪

##### 4. 限流与安全

- 接口调用次数限制
- 敏感内容拦截
- 异常请求拦截

##### 5. 对接观测平台

- **LangSmith**（官方首选）
- OpenTelemetry
- 自定义日志平台

---

#### 三、三种使用方式（由简单到高级）

##### 1. 全局开启 verbose（最简单）

直接打印所有流程，**调试神器**

```python
chain.invoke({"query": "问题"}, verbose=True)
```

##### 2. 自定义回调处理器（中级）

自己写一个类，监听事件：

```python
from langchain_core.callbacks import BaseCallbackHandler

class MyLogHandler(BaseCallbackHandler):
    def on_llm_start(self, *args, **kwargs):
        print("【日志】大模型开始调用")
        
    def on_llm_end(self, *args, **kwargs):
        print("【日志】大模型调用结束")

# 使用
chain.invoke({}, callbacks=[MyLogHandler()])
```

##### 3. 接入 LangSmith（企业级标准）

一键打通官方监控平台，**所有流程可视化**
只需设置环境变量：

```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "xxx"
```

然后：

- 每一次调用都能看见
- 每一步耗时、Token、输入输出
- 可分享、可测试、可标注优化

---

#### 四、支持监听的所有核心事件

你可以监听这些生命周期：

- `on_chain_start` / `end`
- `on_llm_start` / `end`
- `on_retriever_start` / `end`
- `on_tool_start` / `end`
- `on_agent_action` / `end`
- `on_error`

---

#### 五、Callbacks 最核心价值

1. **透明化**：AI 调用不再是黑盒
2. **可调试**：快速定位错误
3. **可观测**：生产环境必备监控
4. **可优化**：基于追踪数据优化提示词、模型、检索

---

#### 六、一句话总结

**Callbacks = LangChain 的监控与日志系统**
**所有生产级 AI 项目必须用！**

---

#### 面试高频考点

1. Callbacks 作用？
   监听组件运行事件，实现日志、监控、调试、追踪。

2. 最常用的 Callback 工具是什么？
   **LangSmith**，官方全链路追踪平台。

3. 能监听哪些事件？
   链开始/结束、模型调用、检索、工具调用、错误。

4. verbose=True 有什么用？
   自动打印完整执行流程，快速调试。

---

## 三、生态系统分类（周边工具）

LangChain 生态提供了从开发到部署的完整工具链。

- **LangGraph**: 用于构建**有状态、多轮、复杂**代理工作流的库，支持循环、分支和持久化。
- **LangSmith**: **调试、测试、评估和监控** LLM 应用的平台，可追踪每一步的输入输出。
- **LangServe**: 将 LangChain 链或代理**部署为 REST API** 的工具，支持流式输出。
- **LangFlow**: 基于 Web 的**可视化拖拽**界面，用于构建 LangChain 工作流，无需代码。

---

## 总结

简单来说：

- **包结构**解决了“**代码如何组织**”的问题。
- **核心组件**解决了“**AI 应用由哪些部分构成**”的问题。
- **生态系统**解决了“**如何开发、调试、部署**”的问题。

要不要我用一个表格对比一下 LangChain、LlamaIndex 和 Haystack 的核心差异和适用场景？