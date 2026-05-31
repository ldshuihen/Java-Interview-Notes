# Trae

## 规则（Rules）

Trae 的规则（Rules）是用来**约束 AI 行为、统一输出风格、固化个人/项目规范**的配置，避免每次对话重复说明要求，分为**全局（个人）规则**和**项目规则**两类。

---

### 一、规则有什么用？

1. **全局规则（user_rules.md）**
   - 作用：**所有项目生效**，定义你的个人偏好。
   - 常用场景：
     - 语言：始终用中文回复、专业术语用英文。
     - 风格：代码简洁、注释详细、回答严谨。
     - 系统：默认按 Windows/macOS 给出命令。
     - 交互：直接给结论，不要多余铺垫。

2. **项目规则（project_rules.md）**
   - 作用：**仅当前项目生效**，定义团队/项目强制规范。
   - 常用场景：
     - 代码风格：缩进2空格、变量 camelCase、常量全大写。
     - 技术栈：React+TS、Python3.10、禁止用 jQuery。
     - 架构：状态管理用 Zustand、接口统一封装。
     - 安全：禁止使用指定 API、敏感数据加密。

3. **生效方式（项目规则）**
   - 始终生效：项目内所有对话都遵守。
   - 指定文件：仅匹配 `globs`（如 `src/**/*.ts`）时生效。
   - 智能生效：AI 根据对话内容自动判断是否启用。

---

### 二、怎么用（快速上手）
#### 1. 打开规则面板
- 右上角 ⚙️（设置）→ 规则（Rules & Skills）。


#### 2. 创建全局规则（个人）
- 个人规则 → + 创建 user_rules.md。
- 编写规则（Markdown），保存即生效。
```markdown
# 个人全局规则
- 所有回复使用中文，专业术语保留英文。
- 代码必须包含注释，关键步骤加说明。
- 优先给出可直接运行的完整代码，不要片段。
- 解释问题时先结论、后原因、再示例。
```

#### 3. 创建项目规则
- 项目规则 → + 创建 project_rules.md。
- 选择生效方式（始终/指定文件/智能）。
- 编写项目规范，文件保存在 `.trae/rules/`。
```markdown
# 项目规则：React+TS 电商前端
## 技术栈
- 框架：React 18 + TypeScript
- 构建：Vite，包管理：pnpm
## 代码规范
- 变量/函数：camelCase；组件：PascalCase；常量：UPPER_CASE
- 缩进：2空格；禁止用 any
- 状态管理：Zustand；路由：React Router v6
## 接口规范
- 所有请求封装到 src/api/，统一错误处理
- 禁止直接在组件内写 fetch/axios
```

#### 4. 验证规则是否生效
- 新建对话，让 AI 生成代码或回答问题，检查是否符合规则。
- 规则会**自动注入上下文**，无需每次手动引用。

---

### 三、最佳实践
1. **全局规则**：写通用偏好，控制在 **5–10 条**，简洁明确。
2. **项目规则**：聚焦项目专属规范，**不超过 1000 字符**，按模块分类。
3. **语言建议**：核心规则用**英文**（AI 理解更准），说明用中文。
4. **团队协作**：把 `.trae/rules/` 提交到 Git，**全员共享同一规范**。

---

### 四、常见问题
- 规则不生效：检查文件路径是否为 `.trae/rules/`、是否保存、是否选择正确生效方式。
- AI 偶尔违反：规则**不要太冗长**，关键要求放在最前面，避免歧义。

##  Skills

有，Trae 里**真的有 Skills（技能）**，而且是和 Rules 并列、但用途完全不同的核心功能。下面用最清楚的方式讲：什么是 Skills、在哪、怎么用、和 Rules 区别。

---

### 一、Trae 里的 Skills 是什么？

- **Rules = 约束/规矩**（强制 AI “必须怎么输出”）
- **Skills = 能力包/操作手册**（给 AI “一套可复用的专业流程”）

Skill 本质是一个文件夹 + `SKILL.md`，里面写：
- 适用场景
- 执行步骤
- 约束与模板
- 甚至可附带脚本/示例

AI 会**自动判断何时触发**，不需要你每次重复prompt。

---

### 二、在哪看/创建 Skills？

打开 Trae → 右上角 ⚙️（设置）→ **规则和技能（Rules & Skills）**
- 下面分两块：
  - **Rules（规则）**：`user_rules.md` / `project_rules.md`
  - **Skills（技能）**：全局技能 + 项目技能

#### 目录结构（很重要）

- 全局技能（所有项目生效）
  ```
  ~/.trae/skills/技能名/SKILL.md
  ```
- 项目技能（只当前项目生效）
  ```
  .trae/skills/技能名/SKILL.md
  ```

---

### 三、怎么用（极简流程）

#### 1）创建 Skill（两种方式）

##### 方式A：在设置里手动建

- 进入 Rules & Skills → 技能 → +创建
- 填：名称、描述、详细指令（Markdown）
- 保存即可

##### 方式B：对话里让 AI 帮你建（最爽）

直接说：
> 帮我创建一个前端组件开发的 Skill，要求用 React+TS、函数组件、自动加 JSDoc、用 Tailwind、禁止内联样式。

AI 会自动生成文件夹和 `SKILL.md`，并放到正确目录。

#### 2）触发 Skill

不用手动开启，AI **自动匹配**。
你只要说：

> 帮我写一个登录表单组件

它会自动调用「前端组件开发」Skill，按你定的规范输出。

#### 3）启用/禁用

在设置里，每个 Skill 有开关；关闭后 AI 不再调用。

---

### 四、Skills 和 Rules 到底哪里不一样？（一句话）

- **Rules = 长期全局约束**（风格、语言、代码规范，**一直生效**）
- **Skills = 场景化能力流程**（做某类事的完整步骤，**按需触发**）

可以同时用：
- Rules 管「代码必须加注释、用中文」
- Skills 管「写组件的完整流程、目录结构、命名规范」

---

### 五、常见误区澄清

1. ❌ 不是“高级版 Rules”：**机制完全不同**，Rules 是全局 prompt 注入，Skills 是动态加载的能力包。
2. ❌ 不需要写代码：纯 Markdown 即可，**零编程**。
3. ❌ 不是插件：是内置功能，**免费可用**。

---

如果你愿意，我可以直接给你一个**可复制的 React+TS 组件开发 Skill 模板（SKILL.md）**，你粘贴就能用。要不要？

### Trae Skills 最实用示例（直接复制可用）

我给你**3 个最常用、开箱即用的 Skill 示例**，全部是 Trae 支持的 `SKILL.md` 格式，**直接复制粘贴就能用**。

#### 先记住：

Skill = **做某件事的固定流程/专业能力**
文件位置：
- 全局：`~/.trae/skills/技能名/SKILL.md`
- 项目：`项目/.trae/skills/技能名/SKILL.md`

---

#### 示例 1：React + TS 组件开发 Skill（最常用）

文件名/技能名：`react-ts-component/SKILL.md`
```markdown
# Skill: React + TS 组件开发
## 适用场景
开发 React 18 + TypeScript 项目的业务组件、UI 组件

## 强制规范
1. 使用函数组件 + Hooks，禁止 class 组件
2. 必须定义 interface Props
3. 必须使用 JSDoc 注释组件和 Props
4. 禁止使用 any
5. 组件名使用 PascalCase
6. 导出方式：export default 组件

## 输出格式
- 先给出完整代码
- 再给出使用示例
- 最后说明注意点

## 示例输出风格
​```tsx
/**
 * 登录表单组件
 */
interface LoginFormProps {
  onSuccess: () => void;
}

export default function LoginForm({ onSuccess }: LoginFormProps) {
  return <div>表单内容</div>;
}
```
---

#### 示例 2：API 接口封装 Skill（前端项目必备）

技能名：`api-service/SKILL.md`
```markdown
# Skill: API 接口封装
## 适用场景
统一封装项目请求接口，基于 axios/fetch

## 规范
1. 所有接口放在 src/api/ 目录下
2. 按模块拆分文件（user.ts, order.ts）
3. 统一返回类型：Promise<ApiResponse<T>>
4. 统一错误处理
5. 接口必须加 JSDoc

## 固定结构
​```typescript
// 统一响应类型
interface ApiResponse<T> {
  code: number;
  data: T;
  message: string;
}

// 接口示例
/**
 * 获取用户信息
 */
export async function getUserInfo(id: number) {
  return request.get<ApiResponse<UserInfo>>(`/user/${id}`);
}
```

---

#### 示例 3：代码审查 Skill（自动帮你查错）

技能名：code-review/SKILL.md
```markdown
# Skill: 代码审查
## 适用场景
审查代码质量、规范、潜在 bug

## 审查步骤
1. 检查语法错误
2. 检查类型安全（TS）
3. 检查命名规范
4. 检查是否有冗余代码
5. 检查性能问题
6. 给出优化建议 + 优化后代码

## 输出格式
- 问题列表（每条带行号）
- 风险等级
- 修复方案
- 最终优化代码
```

---

#### 怎么在 Trae 里使用这些 Skill？

1. 打开 Trae 设置 → **规则和技能**
2. 进入 **技能** → **创建技能**
3. 把上面任意一段复制进去
4. 保存，**立刻生效**

之后你只要说：
- `帮我写一个用户列表组件`
- `帮我封装一个获取订单的接口`
- `帮我审查这段代码`

AI 会**自动匹配 Skill**，按你的规范输出。

---

#### 最简单区分（再帮你巩固）

- **Rules**：我必须怎么做（规矩）
- **Skills**：我会怎么专业完成某件事（流程/能力）
- **MCP**：我能调用哪些外部工具（接口）

---

需要我再给你 **Vue 版、小程序版、Java 版、Python 版** 的 Skill 吗？你要哪个我直接给你可复制模板。

## MCP

先给一句话结论：**MCP 是 Trae 用来安全、标准化地连接外部工具/服务的协议层**，和你之前问的 Rules、Skills 完全不是一类东西。

---

### 一、MCP 到底是什么

**MCP = Model Context Protocol（模型上下文协议）**，是一套开放标准（类似“工具 USB 接口”）。

- 作用：让 Trae 的 AI 智能体（Agent）能**统一、安全地调用外部服务/工具/数据库/API**
- 形态：**MCP Server**（外部服务端） + **Trae MCP Client**（IDE 里的调用方）
- 传输：支持 **stdio（本地进程）**、**HTTP/SSE（远程服务）**

简单理解：
- Rules = 约束（怎么写、怎么说）
- Skills = 场景流程（做某件事的步骤）
- **MCP = 外部工具连接器（让 AI 能“摸”到外面的世界）**

---

### 二、MCP 能做什么（核心用途）

在 Trae 里最常见场景：

1. **连接云服务/第三方 API**
   - GitHub：自动创建 PR、查 Issue、看代码
   - Figma：从设计图直接生成前端代码
   - 数据库（PostgreSQL/MySQL）：AI 直接查数据、写 SQL

2. **本地工具深度集成**
   - Puppeteer：AI 帮你打开网页、操作、截图
   - 本地脚本/命令：安全调用你电脑上的程序

3. **企业内部系统对接**
   - 自研 MCP Server：把内部平台、数据、能力暴露给 AI

一句话：**Skills 是“内置能力模板”，MCP 是“外接工具总线”**。

---

### 三、Trae 里怎么用 MCP（极简步骤）

#### 1. 入口

右上角 ⚙️（设置）→ **MCP**

#### 2. 添加 MCP Server（两种方式）

##### 方式 A：从「MCP 市场」一键添加（推荐）

- 进入 MCP → 添加 → 选市场（如 GitHub、Figma、Puppeteer）
- 填密钥/Tokens（如 GitHub Token）→ 确认

##### 方式 B：手动配置（自定义/私有服务）

- MCP → 添加 → 手动配置
- 写 JSON（示例：本地 stdio 模式）
```json
{
  "name": "my-local-tool",
  "transport": "stdio",
  "command": "python3",
  "args": ["./my_tool.py"],
  "env": {}
}
```

#### 3. 在对话里调用

- 进入智能体模式：输入 **@Builder with MCP**
- 直接说任务：
  - “帮我用 GitHub MCP 查一下这个仓库的 Issues”
  - “用 Figma MCP 把这个设计转成 React 组件”


---

### 四、MCP vs Skills vs Rules（彻底分清）

| 概念       | 本质                | 作用                      | 生效方式            |
| ---------- | ------------------- | ------------------------- | ------------------- |
| **Rules**  | 全局约束/风格       | 规定输出格式、语言、规范  | 全程自动生效        |
| **Skills** | 场景化流程模板      | 定义“做一类事”的步骤      | AI 按需触发         |
| **MCP**    | 外部工具协议/连接器 | 连接外部服务、API、数据库 | 在 Agent 中显式调用 |

记忆：
- Rules = 规矩
- Skills = 套路
- **MCP = 外挂接口**

---

### 五、常见误区

1. ❌ MCP 不是 Skills：Skills 是纯文本流程，MCP 是**真实外部工具调用**
2. ❌ MCP 不是插件系统：是**标准化协议**，所有符合 MCP 的服务都能接入
3. ❌ 不需要写复杂代码：市场里的 MCP 基本**一键配置**即可用





# qoder



# claude code



# codex



# antgravity

