# Git

Git 是分布式版本控制系统，可从 **版本控制类型、底层对象、命令、分支/工作流** 四大维度系统分类。下面是完整结构化介绍：

---

## 一、版本控制系统分类（Git 所处定位）

版本控制分三类，Git 属于 **分布式**：

1. **本地版本控制（Local VCS）**
   - 单机、无协作、仅本地记录补丁/快照
   - 代表：RCS


2. **集中式版本控制（CVCS）**
   - 单中心服务器、所有版本存在服务器、断网不可操作历史
   - 代表：SVN、CVS


3. **分布式版本控制（DVCS）**
   - 每人本地是完整仓库镜像、断网可提交/分支/查看历史
   - 代表：**Git、Mercurial**

---

## 二、Git 底层对象分类（.git/objects 核心存储）
Git 仓库由 4 种不可变对象构成，用 SHA-1 哈希唯一标识：

### 1. Blob（文件数据对象）
- 只存 **文件内容**，不含文件名/路径/权限
- 内容相同则 SHA-1 相同（自动去重）
- 对应：`git add` 后生成

### 2. Tree（目录树对象）
- 表示 **目录结构**，保存：文件名、权限、指向 blob / 子 tree 的哈希
- 对应：`git write-tree` / 提交时自动生成


### 3. Commit（提交对象）
- 项目**时间点快照**，包含：
  - 根 tree、父 commit、作者/提交者、时间、提交信息
- 对应：`git commit`

### 4. Tag（标签对象）
- 给 commit 起别名（如 v1.0.0），分**轻量标签**与**附注标签**（带信息）

---

## 三、Git 命令分类（按功能场景）

### 1. 配置与初始化

- `git config`：用户/仓库全局配置
- `git init`：新建本地仓库
- `git clone`：克隆远程仓库

```bash
# 配置用户名/邮箱（必须）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

# 查看配置
git config --list

# 初始化本地仓库
git init

# 克隆远程仓库
git clone 仓库地址
```

### 2. 工作区/暂存区/本地仓库

- `git status`：查看状态
- `git add`：加入暂存区
- `git commit`：提交到本地仓库
- `git diff`：对比差异

```bash
# 查看文件状态，查看工作区 当前状态

git status

# 添加到暂存区
git add 文件名       # 单个文件
git add .           # 所有修改

# 提交到本地仓库
git commit -m "提交说明"

# 对比差异
git diff            # 工作区 vs 暂存区 ，git diff 看完 → 按 q 退出
git diff --staged   # 暂存区 vs 本地仓库
```

### 3. 分支与合并（核心）

- `git branch`：创建/列出/删除分支
- `git checkout / switch`：切换分支
- `git merge`：合并分支
- `git rebase`：变基（线性提交）

```bash
# 查看所有分支
git branch

# 创建分支
git branch 分支名

# 切换分支
git switch 分支名    # 新版推荐
git checkout 分支名  # 旧版通用

# 创建并切换分支
git switch -c 分支名
git checkout -b 分支名

# 合并分支（把指定分支合并到当前分支）
git merge 分支名

# 变基（让提交更整洁线性）
git rebase 分支名

# 删除分支
git branch -d 分支名
```

### 4. 远程仓库操作

- `git remote`：管理远程地址
- `git fetch`：拉取远程（不合并）
- `git pull` = fetch + merge
- `git push`：推送本地到远程

```bash
# 查看远程仓库
git remote -v

# 关联远程仓库
git remote add origin 远程地址

# 拉取远程代码（不合并）
git fetch

# 拉取并合并（= fetch + merge）
git pull

# 推送到远程
git push -u origin 分支名  # 第一次推送加 -u
git push                   # 后续直接推
```

### 5. 查看日志与历史

- `git log`：提交日志
- `git show`：查看某提交详情
- `git blame`：逐行追溯修改人

```bash
# 查看提交日志
git log

# 简洁版日志（最常用）
git log --oneline

# 图形化分支日志
git log --graph --oneline

# 查看某一次提交详情
git show 提交哈希

# 查看文件每一行谁修改的
git blame 文件名
```

### 6. 撤销/回退/修复

- `git restore` / `checkout --`：丢弃工作区修改
- `git reset`：回退提交（soft/mixed/hard）
- `git revert`：反向提交（安全回退）

```bash
# 丢弃工作区修改（恢复成仓库里的版本）
git restore 文件名
git checkout -- 文件名

# 撤销暂存区（把 add 撤回来）
git restore --staged 文件名
git reset HEAD 文件名

# 回退到历史版本
git reset --soft 哈希    # 保留代码，只撤销提交
git reset --mixed 哈希   # 默认，保留代码，撤销提交+暂存
git reset --hard 哈希    # 彻底丢弃（慎用！）

# 安全回退（生成新提交抵消旧提交）
git revert 提交哈希
```

### 7. 标签与高级

- `git tag`：打版本标签
- `git stash`：临时储藏修改
- `git submodule`：子模块
- `git bisect`：二分定位 bug 提交

```bash
# 打标签
git tag 标签名          # 轻量标签
git tag -a v1.0 -m "说明" # 带说明标签

# 推送标签到远程
git push origin --tags

# 临时储藏未提交的修改
git stash           # 储藏
git stash pop       # 恢复

# 二分查找定位 bug 提交
git bisect start
git bisect bad
git bisect good 版本号
```

---

如果你需要，我还能帮你生成：
- **Git 命令一页纸速查表**
- **Git 常用命令思维导图**
- **Git 面试高频命令题**

你想要哪个版本？

---

## 四、Git 分支策略 / 工作流分类（团队协作模式）
### 1. 集中式工作流（Centralized）

- 单分支（main/master）、类似 SVN、所有人直接 push 到主分支

- 适合：**小团队、从 SVN 迁移**

- 核心：仅维护单一主干分支 `main/master`，所有人基于主分支开发、直接合并推送

  特点：流程简单，操作近似 SVN，学习成本低

  缺点：多人并行开发易冲突，主分支稳定性难保障

  适用场景：**小型团队、项目初期、SVN 迁移过渡场景**

#### 一、核心概念

集中式工作流是**最简单、最传统**的 Git 团队协作模式，完全对标 SVN 集中式管理思想。
整个项目**只维护唯一主干分支**（`main` / `master`），所有人都基于这一个主分支拉取代码、开发、提交、合并推送，没有额外功能分支、开发分支，远程主干作为团队唯一代码中心。

#### 二、核心流程

1. 所有人统一以远程 `master/main` 为唯一开发基准；
2. 开发前先执行 `pull` 拉取最新代码，保证本地同步远程；
3. 在本地直接修改代码、提交到本地仓库；
4. 推送前再次拉取，解决代码冲突；
5. 冲突处理完成后，直接 `push` 推送到远程主分支；
6. 所有迭代、修复、小需求全部合并在主干上。

#### 三、基础操作命令

```bash
# 1. 拉取主干最新代码
git pull origin master

# 2. 本地修改代码后，提交
git add .
git commit -m "功能开发/问题修复"

# 3. 推送前先更新，避免冲突
git pull origin master

# 4. 推送至远程主分支
git push origin master
```

#### 四、主要特点

1. 架构极简，**无多分支管理成本**，上手门槛极低；
2. 操作习惯和 SVN 高度一致，适合 SVN 开发人员无缝迁移；
3. 流程统一、规则简单，无需复杂分支规范；
4. 全员共用一条主干，代码集中管理。

#### 五、缺点

1. **主分支稳定性差**：多人直接提交，错误代码容易直接污染主干；
2. 并行开发冲突多：多人同时修改同一文件，频繁出现合并冲突；
3. 无法隔离开发：新功能、测试bug、线上修复全部混在一个分支，版本混乱；
4. 不支持版本分批发布，无法区分开发环境、生产环境代码；
5. 代码风险高：一旦强行推送错误代码，直接影响线上主干。

#### 六、适用场景

1. 人数少的**小型团队、个人项目、小组内部项目**；
2. 传统项目从 **SVN 迁移 Git** 的过渡阶段；
3. 项目初期、需求简单、迭代节奏慢、变更量小的项目；
4. 内部工具类项目，无严格线上版本管控要求。

#### 七、使用注意事项

1. 禁止强制推送 `git push -f`，防止覆盖他人代码；
2. 每次提交推送前，必须先 `pull` 拉取最新代码；
3. 关键项目不建议使用，无法保障生产代码安全；
4. 适合简单业务，**不适合中大型项目、迭代频繁的互联网项目**。

---

需要我把剩下五种工作流（功能分支、GitFlow、GitHub Flow、GitLab Flow、Forking），全部按**完全相同格式**整理成套吗？

### 2. 功能分支工作流（Feature Branch）

- 每个功能开独立分支、完成后合并回 main

- 适合：**中小团队、并行开发**

- 核心：基于主分支拉取独立功能分支，单一功能 / 需求对应一个分支

  流程：拉取功能分支→开发自测→合并至主干→删除临时分支

  特点：主干代码长期稳定，多需求并行开发互不干扰

  适用场景：**中小型团队、常规业务迭代、项目通用协作**

#### 一、核心概念

功能分支工作流是企业**最通用、最基础**的团队协作模式。
以**稳定主干分支(main/master)** 为唯一基准，**每一个需求、每一个功能、每一个Bug修复**，都单独新建独立分支开发；
功能开发完成后，再合并到主干分支，开发完毕删除临时分支。

#### 二、完整标准流程

1. 开发前切换到主干分支，执行拉取，保证本地代码为最新；
2. 基于主干分支，新建独立**功能分支**进行开发；
3. 在功能分支内完成代码编写、自测、多次本地提交；
4. 开发完成后，推送本地功能分支到远程仓库；
5. 提交 PR/MR 申请，进行代码评审、团队审核；
6. 审核通过后，合并到主干分支；
7. 合并完成，删除本地与远程无用功能分支，保持仓库干净。

#### 三、实操命令示例

```bash
# 1. 切换主干、更新最新代码
git checkout main
git pull

# 2. 新建并切换到功能分支
git checkout -b feature/login

# 3. 本地开发、提交
git add .
git commit -m "完成登录功能开发"

# 4. 推送远程功能分支
git push origin feature/login

# 5. 合并完毕后，删除无用分支
git checkout main
git branch -d feature/login
```

#### 四、核心特点

1. 主干分支长期保持稳定，不会被测试代码、半成品污染；
2. 多需求、多功能**并行开发**，互相隔离、互不干扰；
3. 支持代码评审（PR/MR），保证代码质量；
4. 分支职责清晰，结构简单，学习成本低；
5. 相比集中式工作流，极大减少主干冲突与线上风险。

#### 五、缺点

1. 分支数量较多，需要基础的分支管理规范；
2. 多个功能同时开发，合并主干时仍会产生代码冲突；
3. 没有严格区分测试分支、预发布分支，大型版本管控较弱。

#### 六、适用场景

1. 绝大多数**中小型企业、常规业务项目**；
2. 团队多人协作、并行迭代开发；
3. 不需要复杂多环境、强版本管控的项目；
4. 作为 GitFlow 等复杂工作流的基础入门模式。

#### 七、使用规范

1. **禁止直接在 main 主干修改、提交代码**；
2. 所有功能、迭代、bug 修复，必须新建独立分支；
3. 合并前先拉取主干最新代码，提前解决冲突；
4. 合并后及时删除临时分支，避免分支堆积混乱。

---

需要我继续按**同一份格式**，给你整理 **GitFlow 工作流 详细介绍** 吗？整套格式统一，方便背诵复习。

### 3. GitFlow 工作流（最严谨）

- 双长期分支：
  - `main`：生产稳定版
  - `develop`：开发集成分支
- 短期支撑分支：
  - `feature/*`：功能开发
  - `release/*`：版本发布
  - `hotfix/*`：线上紧急修复

核心：区分**长期常驻分支** + **临时短期分支**

长期双主干：

- `main`：存放线上稳定代码，对应生产环境
- `develop`：开发集成分支，汇总所有新功能

临时业务分支：

- `feature/*`：新功能、新需求开发
- `release/*`：版本测试、预发布、bug 修复
- `hotfix/*`：线上紧急故障、漏洞快速修复

特点：版本管控严格，环境隔离清晰，规范度高

适用场景：**传统企业项目、版本固定发布、需严格管控迭代的项目**



#### 一、核心概念

GitFlow 是**规范最严格、版本管控最强**的企业级分支工作流。
采用**双长期主干分支 + 多种临时短期分支**的结构，严格区分开发、测试、发布、线上紧急修复，适合有固定版本迭代、多环境部署、需要稳定发版的项目。

#### 二、两大长期常驻分支（永久保留）

1. **`main` / `master`**
- 生产环境分支，只存放**已上线、稳定可运行**的正式代码；
- 禁止直接开发、禁止直接提交，仅通过合并纳入新版本。
2. **`develop`**
- 开发集成分支，所有新功能开发完成后统一合并到此；
- 作为日常开发的统一基准，衔接功能分支与发布分支。

#### 三、四大临时短期分支（用完即删）

1. **`feature/*` 功能分支**
- 从 `develop` 拉出；
- 用于新需求、新业务功能开发；
- 开发完成合并回 `develop`，随后删除。
2. **`release/*` 版本发布分支**
- 从 `develop` 拉出；
- 用于版本冻结、整体测试、BUG 修复、版本号修改；
- 测试无误后，同时合并到 `main` 和 `develop`，完成发版。
3. **`hotfix/*` 紧急修复分支**
- 从 `main` 生产分支拉出；
- 用于线上突发故障、高危漏洞、紧急BUG修复；
- 修复完成同时合并到 `main` 和 `develop`，保证两边版本一致。
4. **`bugfix/*` 测试修复分支**
- 多用于测试环境问题修复，合并至 develop 或 release。

#### 四、标准完整工作流程

1. 所有新功能，从 `develop` 拉取 `feature` 分支开发；
2. 功能完成，合并到 `develop` 集成分支；
3. 版本迭代完毕，从 `develop` 拉出 `release` 分支进行预发布测试；
4. 测试通过，`release` 分别合并到 `main`（上线）和 `develop`（同步）；
5. 线上出现紧急问题，从 `main` 拉出 `hotfix` 修复；
6. 修复完毕，双向合并，打版本标签，清理临时分支。

#### 五、常用操作命令示例

```bash
# 1. 创建功能分支
git checkout develop
git pull
git checkout -b feature/order

# 2. 功能完成合并到 develop
git checkout develop
git merge feature/order

# 3. 拉取发布分支
git checkout -b release/v1.2.0

# 4. 线上紧急修复
git checkout main
git checkout -b hotfix/pay-bug
```

#### 六、核心特点

1. 分支职责划分明确，开发、测试、发布、线上修复完全隔离；
2. `main` 分支高度稳定，保障生产环境代码安全；
3. 版本迭代有序，支持计划性发版、版本回滚、版本标签管理；
4. 流程标准化，适合多人协作、大型团队、规范开发。

#### 七、缺点

1. 分支多、流程繁琐，学习成本与维护成本高；
2. 迭代节奏慢，不适合快速迭代、敏捷交付、持续部署项目；
3. 合并流程复杂，日常操作相对笨重。

#### 八、适用场景

- 传统中大型企业项目、政务、金融、医疗等**高稳定要求**系统；
- 固定周期版本发布、需要严格区分生产/测试/开发环境；
- 版本需要长期维护、多补丁迭代、合规要求高的项目。

#### 九、总结背诵

双长分支：**main（生产）、develop（开发）**
短分支：**feature开发、release发版、hotfix线上急救**
优势：规范严谨、生产稳定；缺点：流程复杂、迭代慢。

---

需要我继续按**完全一致格式**，接着写 **GitHub Flow 详细介绍**吗？整套统一排版，方便你整理笔记。



### 4. GitHub Flow（极简持续部署）

- 只有 `main` 长期分支
- 所有修改走 PR → 审查 → 合并 → 立即部署
- 适合：**持续交付、敏捷、Web 服务**

核心：极简模式，仅保留唯一长期分支 `main`

流程：新建临时分支开发→提交 PR 代码审查→合并主干→自动部署

特点：无复杂多分支，轻量灵活，支持快速迭代

适用场景：**敏捷开发、互联网 Web 服务、持续交付 / CI/CD 项目**

#### 一、核心概念

GitHub Flow 是一套**轻量、极简、敏捷**的持续交付分支工作流，摒弃 GitFlow 复杂多分支结构。
全程只保留**唯一长期主干分支 main**，所有功能、修复、迭代全部基于临时分支开发，配合 PR 代码审查，合并后直接部署，适配快速迭代、持续集成。

#### 二、核心分支结构

1. **唯一长期分支**
- `main`：永久稳定分支，随时可部署、可上线，严禁直接手写提交。
2. **临时短分支**
- 任意需求新建独立分支，用完合并后立即删除，不长期保留。

#### 三、标准工作流程

1. 每次开发前，基于最新 `main` 拉取**临时功能分支**；
2. 在独立分支完成开发、自测、连续多次提交；
3. 推送远程分支，发起 **PR（合并请求）**；
4. 团队进行代码评审、漏洞检测、自动化CI检测；
5. 审核通过、检查无误后，合并进入 `main` 主干；
6. 合并完成自动触发持续部署，直接上线；
7. 清理无用临时分支，保持仓库简洁。

#### 四、基础操作命令

```bash
# 1. 拉取主干最新代码
git pull origin main

# 2. 创建并切换临时开发分支
git checkout -b feature/new-demo

# 3. 开发完成提交
git add .
git commit -m "新增功能"
git push origin feature/new-demo

# 4. PR合并完成后，删除本地临时分支
git checkout main
git branch -d feature/new-demo
```

#### 五、主要特点

1. 结构极简，只有一条主干，无复杂分支层级；
2. 流程轻量化，迭代速度快，适合敏捷开发；
3. 强制 PR 审核 + CI 自动化校验，保障主干质量；
4. 合并即部署，天然适配 **CI/CD 持续集成、持续部署**；
5. 操作简单，学习成本低，协作门槛低。

#### 六、缺点

1. 无独立开发、预发布环境分支，不适合慢节奏固定版本发布；
2. 不区分测试/预发/生产环境，多环境管控能力弱；
3. 追求快速合并，对大型版本整体管控较弱。

#### 七、适用场景

1. 互联网项目、Web服务、SaaS产品，需要**快速迭代、频繁更新**；
2. 敏捷开发团队、小步快跑、持续交付模式；
3. 开源项目、轻量化团队协作；
4. 前后端分离项目、自动化部署成熟的项目。

#### 八、简短总结（背诵）

- 单主干 `main` + 临时分支，流程极简；
- 开发走分支、合并走PR、合并即上线；
- 优势：灵活高效、适配CI/CD；
- 劣势：无多环境隔离，不适合传统固定版本发布。

需要我继续按同款格式，写 **GitLab Flow、Forking工作流** 完整详细版吗？全套统一排版，直接整合进笔记。



### 5. GitLab Flow（折中）

- `main` + 环境分支（pre-production/production）
- 上游优先：只能从低环境往高环境合并
- 适合：**多环境发布、企业级**

核心：在主干基础上，新增多环境分支（测试 / 预发 / 生产）

规则：上游优先原则，只允许低环境向高环境合并，杜绝逆向合并

环境分支：`test`、`pre-production`、`production`

特点：兼顾灵活性与规范性，适配多环境部署

适用场景：**中大型企业、多环境隔离、定制化发布流程项目**

#### 一、核心概念

GitLab Flow 是介于 **GitFlow 严谨规范** 与 **GitHub Flow 极简敏捷** 之间的**折中企业级工作流**。
在单一主干基础上，引入**多环境分支**，遵循**上游优先**原则，兼顾快速迭代与多环境发布管控，专为 GitLab 企业团队、多环境部署设计。

#### 二、分支结构

1. **长期主干分支**
- `main`：代码开发主干，随时稳定，作为所有迭代基础。
2. **环境长期分支**
按部署环境划分，永久存在：
- `test` 测试环境
- `pre` / `pre-production` 预发布环境
- `prod` / `production` 生产环境
3. **临时业务分支**
- `feature/*` 新功能
- `bugfix/*` 问题修复
用完即删。

#### 三、核心规则（上游优先）

只允许：
`main → test → pre → production`
**禁止逆向合并**，防止生产不稳定代码回流，保证环境代码逐级纯净、可控。

#### 四、标准工作流程

1. 从 `main` 拉出功能分支进行日常开发；
2. 开发完成，经代码评审后合并到 `main`；
3. 将 `main` 合并到 `test` 测试环境，进行测试验证；
4. 测试通过，合并至预发环境，做线上仿真校验；
5. 预发无误，最终合并到生产环境完成上线；
6. 紧急Bug直接在对应环境分支修复，单向向上合并同步。

#### 五、简单操作示例

```bash
# 1. 基于main新建功能分支
git checkout main
git pull
git checkout -b feature/pay

# 2. 开发完成合并到main
git checkout main
git merge feature/pay

# 3. 合并到测试环境
git checkout test
git merge main

# 4. 测试通过合并预发、生产（单向流转）
```

#### 六、主要特点

1. 引入**多环境分支隔离**，满足企业测试、预发、生产隔离需求；
2. 上游优先单向合并，避免环境代码混乱，降低上线风险；
3. 舍弃 GitFlow 复杂的 develop、release 分支，流程更轻量化；
4. 兼容 CI/CD 流水线，可绑定不同分支自动部署对应环境；
5. 兼顾敏捷迭代与规范管控，适配中大型团队。

#### 七、缺点

1. 相比 GitHub Flow，分支数量更多，管理成本更高；
2. 多环境依次合并，发布流程比极简模式繁琐；
3. 环境分支长期维护，需要团队严格遵守合并规范。

#### 八、适用场景

1. **中大型企业团队、私有化部署项目、传统互联网项目**；
2. 必须严格区分**测试、预发、生产**多环境隔离的业务；
3. 使用 GitLab 做代码管理、流水线发布的团队；
4. 既不想用笨重 GitFlow，又不能用极简 GitHub Flow 的折中场景。

#### 九、背诵总结

- 模式：**单主干 + 多环境分支**，GitFlow 与 GitHub Flow 折中方案；
- 规则：**上游优先、单向合并**；
- 优点：多环境隔离、风险低、适配企业CI/CD；
- 缺点：环境分支多，流程有一定复杂度。

需要我接着按同款格式，写最后一个 **Forking 工作流 详细介绍**，六套工作流全套统一笔记？



### 6. Forking 工作流（开源标准）

- 贡献者先 **Fork 主仓库** → 在自己仓库开发 → 提交 PR → 维护者合并
- 适合：**开源社区、外部贡献者**

核心：隔离核心开发人员与外部贡献者

流程：开发者 Fork 官方主仓库→个人仓库独立开发→提交 PR/MR→管理员审核合并

特点：无需开放主仓库写入权限，安全隔离，互不影响

适用场景：**开源项目、跨团队协作、外部人员临时贡献代码**

#### 一、核心概念

Forking 工作流，也称**分叉工作流**，是**开源项目、跨团队协作**专属的 Git 协作模式。
核心区别：**不是所有人都拥有同一个仓库权限**，开发者不直接操作官方主仓库，
每个人先**Fork 一份独立副本**，在自己仓库开发，最后通过 PR/MR 提交贡献。

#### 二、分支结构

1. **官方主仓库**
- 唯一权威仓库，只有管理员拥有合并、推送权限；
- 维护稳定主干 `main`，统一管理版本。
2. **个人 Fork 仓库**
- 每个开发者独立仓库，完全自由创建分支、提交、修改；
- 不受主仓库权限限制。
3. **临时功能分支**
- 个人仓库内新建 `feature/xxx` 分支开发，互不影响。

#### 三、完整工作流程

1. 开发者在平台（GitHub/GitLab）**Fork 官方主仓库**，生成个人仓库；
2. 克隆**自己 Fork 后的仓库**到本地；
3. 基于主干新建功能分支，完成代码开发、修改、提交；
4. 推送到**个人远程仓库**；
5. 向**原官方主仓库**发起 **PR / MR 合并请求**；
6. 项目维护者审核代码、评论、修改建议；
7. 审核通过后，管理员合并代码到官方主仓库；
8. 贡献完成，开发者可删除临时分支，定期同步官方最新代码。

#### 四、基础操作命令

```bash
# 1. 克隆自己 fork 后的仓库
git clone 个人仓库地址

# 2. 新建功能分支开发
git checkout -b feature/demo

# 3. 提交并推送到个人仓库
git add .
git commit -m "开源功能贡献"
git push origin feature/demo

# 4. 绑定原官方仓库，后续同步最新代码
git remote add upstream 官方原仓库地址
git fetch upstream
```

#### 五、核心特点

1. **权限隔离**
   普通开发者无主仓库写入权限，仅管理员可合并，保障核心代码安全。
2. **完全解耦**
   每个人仓库独立，开发自由，互不干扰，不会误操作污染官方代码。
3. **开放协作**
   支持大量外部人员、陌生人参与项目贡献，门槛低。
4. **代码严格审核**
   所有外部修改必须经过 PR 审查，质量可控。

#### 六、缺点

1. 流程复杂，需要维护**个人仓库 + 官方仓库**双仓库；
2. 长期需要手动同步上游官方代码，容易版本落后；
3. 企业内部小团队协作使用过于繁琐，效率低。

#### 七、适用场景

1. **开源项目**（最主流），如框架、开源工具、公共组件；
2. 大型组织、多团队跨部门协作；
3. 外部外包人员、临时开发人员参与项目；
4. 严格控制仓库写入权限、高安全要求的公共项目。

#### 八、背诵总结

- 核心：**Fork 复制仓库 → 个人独立开发 → PR 贡献 → 管理员审核合并**；
- 优点：权限隔离、安全可控、适合开源；
- 缺点：双仓库维护、流程复杂；
- 适用：开源社区、跨团队协作、外部人员贡献。

---

已全部写完：
集中式、功能分支、GitFlow、GitHub Flow、GitLab Flow、Forking
**六种工作流格式完全统一**，直接复制即可整理成完整笔记。





---

## 五、Git 仓库分类（按位置）

1. **本地仓库（Local Repository）**
   - `.git` 目录、完整历史、单机操作
2. **远程仓库（Remote Repository）**
   - GitHub/GitLab/Gitee 等、团队共享、中央同步点
3. **裸仓库（Bare Repository）**
   - 无工作区、仅 `.git`、用于服务器共享（`git init --bare`）

### 本地仓库

#### 1. 概念

本地仓库是执行 `git init` 后，项目根目录自动生成的 **.git 隐藏文件夹**，
存储**完整版本历史、所有分支、标签、配置**，是 Git 版本控制的核心。

- 脱离网络也可提交、建分支、回退版本
- 工作区、暂存区的所有操作，最终都持久化到**本地仓库**

#### 2. 三个工作区域（核心）

1. **工作区**：实际编辑的项目文件
2. **暂存区**：临时存放待提交的修改（`git add`）
3. **本地仓库**：.git 目录，永久保存历史快照（`git commit`）

#### 3. 常用操作命令

```bash
# 初始化本地仓库
git init

# 将修改加入暂存区
git add .

# 提交到本地仓库（生成版本快照）
git commit -m "描述"

# 查看本地仓库提交日志
git log

# 版本回退（操作本地仓库历史）
git reset --hard 提交ID
```

#### 4. 本地仓库与远程仓库关系

- 本地仓库：独立完整，先本地提交，再按需推送到远程
- 远程仓库：多人共享中心，用来备份、协作

#### 5. 特点

- 完整镜像，包含全部历史
- 提交速度快、离线可用
- 可自由分支、合并、回退，不受远程限制

---

我把你前面所有模块统一排版成一套完整笔记：
配置初始化、三区、分支、合并、标签、自定义、远程仓库、本地仓库，需要整合完整版吗？

### 远程仓库

#### 1. 远程仓库基础概念

- 远程仓库：托管在网络上的 Git 仓库（GitHub / Gitee / GitLab）
- 作用：代码共享、团队协作、备份代码
- 默认远程仓库别名：**origin**

#### 2. 远程仓库查看

```bash
# 查看已关联远程
git remote

# 查看详细地址（常用）
git remote -v
```

#### 3. 关联 / 解绑 远程仓库

```bash
# 关联远程仓库
git remote add origin 远程仓库地址

# 修改远程仓库地址
git remote set-url origin 新地址

# 解绑远程仓库
git remote remove origin
```

#### 4. 拉取远程代码

```bash
# 只拉取远程分支信息，不合并代码
git fetch

# 拉取远程代码 + 自动合并（= fetch + merge）
git pull

# 指定分支拉取
git pull origin 分支名
```

#### 5. 推送本地代码到远程

```bash
# 首次推送，绑定上下游分支
git push -u origin 分支名

# 后续直接推送
git push

# 强制推送（慎用，多人协作禁止）
git push -f origin 分支名
```

#### 6. 远程分支管理

```bash
# 查看所有远程分支
git branch -a

# 删除远程分支
git push origin --delete 远程分支名

# 同步远程已删除的分支（清理本地无效远程记录）
git fetch --prune
```

#### 7. 克隆远程仓库

```bash
# 完整克隆
git clone 仓库地址

# 只克隆指定分支
git clone -b 分支名 仓库地址
```

---

#### 极简背诵总结

1. 查看远程：`git remote -v`
2. 关联远程：`git remote add origin 地址`
3. 拉取代码：`git fetch（不合并） / git pull（合并）`
4. 推送代码：`git push -u origin 分支名`
5. 修改远程地址：`git remote set-url`

需要我把**工作区/暂存区/本地仓库/远程仓库**四区关系给你配上总结吗？

---

## 六、Git 引用分类（指针）
- **分支（Branch）**：指向 commit 的可变指针
- **HEAD**：当前所在分支/提交的指针
- **标签（Tag）**：指向 commit 的不可变别名
- **远程跟踪分支**：`origin/main`，记录远程状态

---

## Github

### GitHub 完整 Git 使用流程（新手一步到位）

我给你整理一套**从 0 到 1、最完整、最实用**的 Git + GitHub 工作流，**直接照着做就能用**，包含本地开发 → 提交 → 推送到 GitHub 的全流程。

#### 一、前期准备（必须做）

1. 安装 Git：[git-scm.com](https://git-scm.com/)
2. 注册 GitHub 账号：[github.com](https://github.com/)
3. 配置本地 Git 身份（第一次使用必配）
```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```
查看配置是否成功：
```bash
git config --list
```

---

#### 二、两种最常用场景（完整流程）

##### 场景1：本地已有项目 → 上传到 GitHub

###### 1. 进入项目文件夹

```bash
cd 你的项目路径
```

###### 2. 初始化 Git 仓库

```bash
git init
```

###### 3. 添加文件到暂存区

```bash
git add .      # . 代表添加所有文件
```

###### 4. 提交到本地仓库

```bash
git commit -m "第一次提交：项目初始化"
```

###### 5. 在 GitHub 新建空仓库

**不要勾选** README、.gitignore、License（否则会冲突）

###### 6. 关联远程 GitHub 仓库

复制 GitHub 仓库的 HTTPS 链接，执行：
```bash
git remote add origin https://github.com/用户名/仓库名.git
```

###### 7. 推送到 GitHub

```bash
git push -u origin main
```
以后再次推送只需要：
```bash
git push
```

---

##### 场景2：GitHub 已有项目 → 克隆到本地修改

###### 1. 克隆项目到本地

```bash
git clone https://github.com/用户名/仓库名.git
```

###### 2. 进入项目文件夹

```bash
cd 仓库名
```

###### 3. 修改代码后提交

```bash
git add .
git commit -m "修改了XX功能"
```

###### 4. 推送到 GitHub

```bash
git push
```

#### 分支名称

- 你本地 Git 老版本默认分支叫：**master**
- GitHub 现在默认分支叫：**main**

所以本地和远程对不上，推送会失败。我给你**最简单、最安全、一步到位**的解决方法，直接复制命令就能用！

---

##### 一、最推荐方案：把本地分支重命名为 main（推荐）

直接在你的项目里执行：

```bash
git branch -M main
```

这条命令会把你当前的 `master` 分支**强制重命名为 main**。

然后再推送：
```bash
git push -u origin main
```

✅ **搞定！以后本地和 GitHub 都是 main，完全统一**

---

##### 二、如果你已经关联了远程仓库

直接执行下面 **2 行** 就能对齐：

```bash
git branch -M main
git push -u origin main
```

---

##### 三、以后新建项目永远默认用 main（一劳永逸）

让你本地 Git 以后初始化默认分支就是 main：

```bash
git config --global init.defaultBranch main
```

以后再 `git init` 就是 **main**，不会再出现 master！

---

##### 四、你现在只需要做这 2 步

你当前路径就是项目根目录，直接运行：

1. 重命名本地分支
```bash
git branch -M main
```

2. 推送到 GitHub
```bash
git push -u origin main
```

✅ **完美解决！**

---

##### 总结

- 本地 `master` → 改名为 `main`：`git branch -M main`
- 推送到 GitHub：`git push -u origin main`
- 永久默认 main：`git config --global init.defaultBranch main`

需要我帮你把整个提交+推送流程一次性跑完吗？

---

#### 三、日常开发标准流程（最常用）

以后你写代码，**永远按这个顺序来**：

##### 1. 拉取最新代码（避免冲突）

```bash
git pull
```

##### 2. 写代码/修改文件

##### 3. 提交三步曲

```bash
git add .                # 1. 添加所有修改
git commit -m "说明文字"  # 2. 提交到本地
git push                 # 3. 推送到GitHub
```

---

#### 四、必须掌握的高频命令

##### 查看状态

```bash
git status   # 看哪些文件修改了
```

##### 查看提交历史

```bash
git log
```

##### 切换分支（团队开发必用）

```bash
git checkout -b 分支名    # 创建并切换分支
git checkout 分支名       # 切换分支
git merge 分支名          # 合并分支
```

##### 取消修改

```bash
git checkout -- 文件名    # 撤销单个文件修改
git reset --hard          # 撤销所有修改（谨慎！）
```

---

#### 五、完整流程一句话总结

1. **git pull** 拉最新代码
2. 写代码
3. **git add .**
4. **git commit -m "说明"**
5. **git push** 推送到 GitHub

---

#### 总结

1. 第一次使用先配置 `user.name` 和 `user.email`
2. 本地项目 → GitHub：`init → add → commit → remote → push`
3. GitHub → 本地：`clone`
4. 日常开发：`pull → 写代码 → add → commit → push`

我可以给你做一份**超简洁速查表（一页纸）**，或者教你**解决提交冲突、回退版本、分支管理**这些进阶内容，你需要吗？

### GitHub HTTPS 与 SSH 完整对比+选择+配置

你说得对！GitHub 克隆/推送代码时，**HTTPS** 和 **SSH** 是两种完全不同的连接方式，我给你讲清楚区别、怎么选、怎么用，一步到位。

---

#### 一、核心区别（最简单版）

| 方式      | 优点                   | 缺点                          | 适合谁                     |
| --------- | ---------------------- | ----------------------------- | -------------------------- |
| **HTTPS** | 不用配置，直接用       | 每次推送要输**账号密码/令牌** | 新手、临时使用、多账号切换 |
| **SSH**   | 一次配置，**永久免密** | 第一次要生成密钥配置          | 长期开发、个人主力电脑     |

一句话总结：
- **临时用、不想折腾 → 选 HTTPS**
- **自己电脑、长期用 → 强烈推荐 SSH（一劳永逸）**

---

#### 二、HTTPS 怎么用（最简单，零配置）

##### 1. 直接复制链接

在 GitHub 仓库点击 **Code → HTTPS**，复制链接。

##### 2. 克隆

```bash
git clone https://github.com/用户名/仓库名.git
```

##### 3. 推送时

第一次 `git push` 会让你输入：
- 用户名：GitHub 用户名
- 密码：**必须用 GitHub 个人访问令牌（不是登录密码）**

> 现在 GitHub 已经**禁止直接用登录密码推送**了，HTTPS 必须用令牌。

---

#### 三、SSH 怎么用（推荐！一次配置永久免密）

##### 步骤1：生成 SSH 密钥（本地电脑）

打开终端，执行：
```bash
ssh-keygen -t ed25519 -C "你的GitHub邮箱"
```
一路回车，**不用设置密码**。

##### 步骤2：查看并复制公钥

###### Windows

```bash
cat ~/.ssh/id_ed25519.pub
```
###### Mac/Linux

```bash
cat ~/.ssh/id_ed25519.pub
```
把终端输出的**一长串内容全部复制**。

##### 步骤3：把公钥粘贴到 GitHub

1. 打开 GitHub → 右上角头像 → **Settings**
2. 左侧 **SSH and GPG keys** → **New SSH key**
3. 粘贴刚才复制的内容，保存

##### 步骤4：测试连接

```bash
ssh -T git@github.com
```
出现 `Hi xxx! You've successfully authenticated` 就成功了！

##### 步骤5：使用 SSH

克隆时用 SSH 链接：
```bash
git clone git@github.com:用户名/仓库名.git
```
以后 `git push` **完全不用输密码**！

---

#### 四、已经用了 HTTPS，想改成 SSH？

如果你的项目已经是 HTTPS，直接执行一行命令切换：
```bash
git remote set-url origin git@github.com:用户名/仓库名.git
```

查看当前用的是什么方式：
```bash
git remote -v
```

---

#### 五、我最推荐你用哪个？

✅ **自己的电脑 → 必须用 SSH**（一次配置，永远舒服）
✅ 公司/公共电脑 → 用 HTTPS（安全、不残留密钥）

---

#### 总结

1. **HTTPS**：不用配置，但推送要输令牌
2. **SSH**：配置一次，**永久免密，最舒服**
3. 现有项目可随时用 `git remote set-url` 切换

需要我教你**生成 GitHub 令牌（HTTPS 用）** 或者 **一步步带你配置 SSH** 吗？



### GitHub & Git 版本号（Commit-ID）查看 + 复制 + 回滚 

#### 核心概念

每次 `git commit` 都会生成唯一 **Commit 版本号（SHA-1哈希）**
- 完整：40 位十六进制字符串
- 短版本：前 7 位，日常通用
- `HEAD`：指代**当前最新提交版本**

---

#### 一、本地查看版本号（高频命令）

##### 1. 查看全部历史提交（含完整版本号）

```bash
git log
```
- 按 `q` 退出
- 第一行 `commit xxxx` 就是版本号

##### 2. 极简一行展示（推荐）

```bash
git log --oneline
```
格式：`短版本号 提交备注`

##### 3. 只查看当前最新版本号

```bash
# 完整40位
git rev-parse HEAD

# 短版本号（7位）
git rev-parse --short HEAD
```

##### 4. 查看所有分支全部提交

```bash
git log --oneline --all
```

---

#### 二、GitHub 网页端查看版本号

1. 打开仓库主页
2. 点击顶部 **Commits**，展示全部提交记录
3. 每条记录左侧：**短哈希版本号**
4. 点击版本号 → 进入提交详情页
   - 网址末尾 = 完整 40 位版本号
   - 可一键复制

---

#### 三、版本号复用：3 大核心场景

##### 场景1：复制指定版本号

1. 本地：`git log --oneline` 找到目标版本，复制 7 位短码即可使用
2. 网页：Commit 列表直接复制哈希值

---

##### 场景2：临时切换到历史版本（只读浏览）

适合：查看旧代码、对比内容，**不修改代码**
```bash
# 替换为你的目标版本号
git checkout 版本号
```
此时进入「游离 HEAD 状态」
切回最新主线：
```bash
git checkout main
```

---

##### 场景3：版本回滚（重点，两种方案）

###### 方案A：软回滚（推荐，保留代码修改）

只撤销提交，**代码文件不变**，可以重新提交
```bash
git reset --soft 目标版本号
```

###### 方案B：硬回滚（危险！丢弃所有修改）

强制还原到指定版本，**后续所有改动全部清空**
```bash
git reset --hard 目标版本号
```

> ⚠️ 硬回滚后，本地丢失的内容无法恢复，谨慎使用

---

###### 场景4：回滚后同步到远程 GitHub

本地回滚完成后，远程还是旧记录，需要强制推送：
```bash
git push -f origin main
```
> 团队多人协作禁止滥用 `-f`，仅个人仓库使用

---

###### 场景5：撤销某一次提交（保留其他版本）

不删除历史，新增一条「反向提交」修复代码
```bash
git revert 版本号
```
然后正常推送：
```bash
git push
```

---

#### 四、超精简速查清单

```bash
# 查看所有提交（简洁）
git log --oneline

# 查看当前完整版本号
git rev-parse HEAD

# 查看当前短版本号
git rev-parse --short HEAD

# 临时切换历史版本
git checkout 版本号

# 软回滚（保留代码）
git reset --soft 版本号

# 硬回滚（清空改动）
git reset --hard 版本号

# 强制覆盖远程仓库
git push -f
```

---

#### 五、补充常见问题

1. `git log` 卡住无法输入
👉 按 `q` 退出查看器
2. 版本号太长记不住
👉 统一使用**前7位短版本号**，Git 可自动识别
3. 误操作硬回滚怎么办
👉 用 `git reflog` 查看操作记录，找回丢失版本

### Git 回滚

**日常开发 90% 都用这 2 种**（最安全、最标准）

---

#### 一、**最常用、最推荐、最安全 → git revert（首选）**

##### 适用场景

你已经把代码 **push 到 GitHub 了**，想撤销某一次提交，**不破坏历史**。

##### 优点

✅ **安全之王**  
✅ **不会删除任何历史记录**  
✅ **团队协作绝对不会出事**  
✅ GitHub 上能看到你撤销了哪次提交（可追溯）

##### 命令

```bash
git revert 版本号
```

然后：
```bash
git push
```

##### 一句话总结

**想撤销某次提交 → 用 git revert**

---

#### 二、**个人项目最常用 → git reset --hard（干净彻底）**

##### 适用场景

**你自己的项目**、还没给别人用，想直接回到过去某版本，**后面的版本全部丢掉**。

##### 优点

✅ 干净  
✅ 彻底回到过去  
✅ 适合自己笔记、自己项目

##### 命令

```bash
git reset --hard 版本号
```

同步到 GitHub：
```bash
git push -f
```

##### ⚠ 注意

**团队项目千万别用！会删别人代码！**

---

#### 三、你应该用哪个？（直接给结论）

##### 1. 你现在是 **Java 面试笔记**（自己用）

👉 **用 git reset --hard 最方便**

##### 2. 公司/团队开发

👉 **必须用 git revert**

---

#### 四、最精简记忆口诀

```
自己项目回退版本 → reset --hard
公司项目撤销提交 → revert
```

---

#### 五、你现在最常用的回滚命令（直接抄）

##### 回退到某个历史版本（自己项目）

```bash
git log --oneline        # 查看版本号
git reset --hard 版本号   # 回退
git push -f               # 同步到GitHub
```

##### 撤销某一次提交（公司/安全）

```bash
git revert 版本号
git push
```

---

需要我教你**如何安全回退、不会弄丢笔记**吗？我可以一步步带你操作！

## Gitee

### Gitee使用教程

下面给你一份**从零到能日常使用的 Gitee 完整流程**（含注册→安装Git→配置SSH→建仓库→上传/更新代码→日常协作），直接照着做就能跑通。

---

#### 一、注册 Gitee 账号

1. 打开官网：https://gitee.com
2. 点击右上角「注册」，填手机号、密码、用户名（英文/数字）
3. 登录，进入个人主页

---

#### 二、本地安装 Git

- 下载：https://git-scm.com/download/win
- 安装：一路默认即可（安装后会有 Git Bash）

---

#### 三、配置本地 Git（只做一次）

打开 **Git Bash**：
```bash
# 用户名（和Gitee一致即可）
git config --global user.name "你的Gitee用户名"

# 邮箱（必须是注册Gitee的邮箱）
git config --global user.email "your_email@xxx.com"

# 查看配置
git config --global --list
```

---

#### 四、配置 SSH 免密码（强烈推荐，以后不用输账号密码）

##### 1）生成密钥

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@xxx.com"
```
- 出现路径：直接回车
- 密码：直接回车（不设密码）
- 确认：再回车

##### 2）查看并复制公钥

```bash
cat ~/.ssh/id_rsa.pub
```
- 复制全部内容（从 `ssh-rsa` 开头到邮箱结尾）

##### 3）添加到 Gitee

1. 网页右上角头像 → **设置**
2. 左侧 **SSH公钥** → 粘贴 → 标题随便写（如“PC”）→ 确定

##### 4）测试是否成功

```bash
ssh -T git@gitee.com
```
看到 `Hi xxx! You've successfully authenticated...` 就成功了

---

#### 五、在 Gitee 创建远程仓库

1. 右上角 **+** → **新建仓库**

2. 填写：
   - 仓库名称：英文/数字（如 `Java-Interview-Notes`）
   - 介绍：随便写
   - 可见性：**公开**（免费）或私有
   - ❌ **不要勾选“初始化仓库（README）”**（本地已有代码会冲突）
3. 点击 **创建**

---

#### 六、把本地代码上传到 Gitee（第一次）

假设你的代码在 `E:/Desktop/JAVA面试/`

##### 1）进入项目目录

```bash
cd /e/Desktop/JAVA面试/JAVA-Interview-Self
```

##### 2）初始化本地仓库（如果还不是 Git 项目）

```bash
git init
```

##### 3）关联远程仓库（复制你Gitee仓库的SSH地址）

```bash
git remote add origin git@gitee.com:你的用户名/仓库名.git

# 查看是否关联成功
git remote -v
```

##### 4）添加、提交、推送

```bash
# 添加所有文件
git add .

# 提交
git commit -m "第一次上传：Java面试笔记"

# 推送到main分支（-u 绑定，以后直接git push）
git push -u origin main
```

---

#### 七、日常使用流程（以后每天都这么干）

##### 1）修改代码后

```bash
# 查看改了哪些
git status
git diff   # 按q退出
```

##### 2）提交到本地

```bash
git add .
git commit -m "更新：新增并发八股笔记"
```

##### 3）推送到 Gitee

```bash
git push
```

##### 4）如果多人协作，先拉最新代码

```bash
git pull origin main
```
有冲突就手动改文件，然后：
```bash
git add .
git commit -m "解决冲突"
git push
```

---

#### 八、常用命令速查（直接复制用）

```bash
# 状态
git status
git diff

# 提交
git add .
git commit -m "备注"

# 推送/拉取
git push
git pull

# 查看日志
git log

# 撤销工作区修改（没add前）
git checkout -- 文件名

# 撤销暂存区（add后，没commit）
git reset HEAD 文件名
```

---

#### 九、和 GitHub 的区别（简单说）

- **GitHub**：国外，访问慢，公开免费
- **Gitee（码云）**：国内，速度快，中文界面，私有仓库免费（有容量限制）
- 操作命令完全一样，只是远程地址不同

---

如果你愿意，我可以直接帮你：
- 生成一份 **可直接复制的命令清单**（按你的用户名/仓库名写好）
- 或给你做一张 **Gitee 日常操作流程图**

你要哪种？

## Git 分支管理

### 一、分支核心概念

1. **主分支**
- `main / master`：线上稳定分支，保护、禁止直接提交
2. **开发分支**
- `develop`：日常开发合并分支（GitFlow 规范）
3. **临时分支**
- `feature/xxx`：新功能
- `bugfix/xxx`：测试bug修复
- `hotfix/xxx`：线上紧急bug
- `release/xxx`：版本预发布

---

### 二、分支基础管理命令

#### 1. 查看分支

```bash
git branch          # 查看本地分支
git branch -a       # 查看本地+远程所有分支
git branch -v       # 查看分支最新提交
```

#### 2. 创建分支

```bash
git branch 分支名                # 仅创建，不切换
git switch -c 分支名             # 创建并切换（新版推荐）
git checkout -b 分支名           # 创建并切换（旧版）
```

#### 3. 切换分支

```bash
git switch 分支名
git checkout 分支名
```

#### 4. 删除分支

```bash
git branch -d 分支名        # 安全删除（已合并）
git branch -D 分支名        # 强制删除（未合并）
git push origin --delete 远程分支名  # 删除远程分支
```

---

### 三、分支合并管理（两种核心方式）

#### 1. merge 合并（保留分支历史）

适合：业务迭代、保留开发记录
```bash
# 1. 切到目标分支（如 main）
git switch main
# 2. 合并功能分支
git merge feature/xxx
```
- 特点：生成**合并节点**，历史分叉清晰
- 缺点：提交记录杂乱

#### 2. rebase 变基（线性干净历史）

适合：开源项目、追求整洁提交日志
```bash
# 切到功能分支
git switch feature/xxx
# 变基到主干
git rebase main
```
- 特点：**线性提交**，日志干净
- 注意：**公共分支禁止随意rebase**

---

### 四、远程分支管理

```bash
git remote -v                  # 查看远程仓库
git fetch origin               # 拉取远程分支信息，不合并
git pull origin 分支名         # 拉取+合并远程分支
git push origin 分支名         # 推送本地分支到远程
git push -u origin 分支名      # 首次推送，绑定上下游
```

---

### 五、企业常用分支命名规范

统一规范，便于团队协作 & 自动部署
1. 功能分支：`feature/用户-需求名称`
2. 修复分支：`bugfix/问题描述`
3. 紧急修复：`hotfix/线上bug`
4. 版本分支：`release/v1.0.0`

示例：
`feature/login`、`hotfix/pay-error`

---

### 六、分支管理最佳规范

1. **禁止直接在 main 提交代码**，必须通过 PR/MR 审核合并
2. 功能开发统一从 `develop` / `main` 拉新分支
3. 分支用完及时删除，避免堆积无效分支
4. 多人协作频繁 `pull`，提前解决冲突
5. 线上紧急问题使用 `hotfix` 分支，修复完合并回主干

---

### 七、面试极简总结

1. 分支分类：**长期分支（main/develop）+ 临时分支（feature/hotfix/release）**
2. 合并方式：
- `merge`：保留分叉、适合团队常规开发
- `rebase`：线性历史、日志整洁
3. 协作规范：功能单独建分支、PR审核、主干保护、定期清理无效分支

我可以把**分支管理 + 之前工作流**整合为一页极简背诵版，要不要？

## Git 标签管理（Tag）

标签（Tag）就是给**某次重要提交打一个“版本号标记”**，比如 `v1.0.0`，方便以后快速找到发布版本，**主要用于发布版本**。

我给你整理成**最实用、面试/工作直接用**的版本：

---

### 一、标签是什么

- 给 commit 一个固定别名（如 v1.0、v2.1）
- 不会像分支一样移动，**永久指向某次提交**
- 用于：版本发布、里程碑标记

---

### 二、标签分类

#### 1. 轻量标签（Lightweight）

- 只是一个commit的指针
- 无额外信息，简单快速

#### 2. 附注标签（Annotated）**（推荐！企业常用）**

- 包含标签名、作者、日期、说明
- 可签名、可校验，更规范

---

### 三、标签常用命令

#### 1. 查看标签

```bash
git tag                # 查看所有标签
git tag -l "v1.*"      # 模糊查询（如v1开头）
git show v1.0.0        # 查看标签详情（提交+信息）
```

#### 2. 创建标签

```bash
# 轻量标签
git tag v1.0.0

# 附注标签（带信息，推荐）
git tag -a v1.0.0 -m "发布 v1.0.0 正式版"
```

#### 3. 给历史提交打标签

```bash
git tag -a v0.9 提交哈希
```

#### 4. 删除标签

```bash
# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0
```

#### 5. 推送标签到远程

```bash
# 推送单个标签
git push origin v1.0.0

# 一次性推送所有本地标签
git push origin --tags
```

#### 6. 切换到标签（查看历史版本）

```bash
git checkout v1.0.0
```

---

### 四、标签 vs 分支（必懂区别）

- **标签**：固定不动，用于**版本发布**
- **分支**：会不断前进，用于**开发迭代**

---

### 五、企业规范

- 版本命名：`v主版本.次版本.补丁版`
  - 例：`v1.0.0`、`v2.1.3`
- 正式发布必须用**附注标签**
- 发布后必须推送到远程仓库

---

### 六、极简背诵版

- 查看：`git tag`
- 创建：`git tag -a v1.0 -m "说明"`
- 推送：`git push origin --tags`
- 删除：`git tag -d 标签` + `git push origin --delete 标签`

需要我把 **分支管理 + 标签管理** 合并成一页终极速记表吗？

## Git 自定义（Git Config / 别名 / 忽略文件）

我给你整理**工作最常用、最实用**的 Git 自定义内容，直接照着用就行。

---

### 一、Git 配置（核心自定义）

Git 有 3 个配置级别：
1. **系统级**（所有用户）：`--system`
2. **全局级**（当前用户）：`--global` **最常用**
3. **仓库级**（当前项目）：`--local`

#### 1. 基础配置（必须）

```bash
# 配置用户名（全局）
git config --global user.name "你的名字"

# 配置邮箱
git config --global user.email "你的邮箱"

# 查看所有配置
git config --list
```

#### 2. 编辑器自定义

```bash
# 默认编辑器改为 VS Code
git config --global core.editor "code --wait"
```

#### 3. 换行符自定义（跨系统必备）

```bash
# Windows
git config --global core.autocrlf true

# Mac/Linux
git config --global core.autocrlf input
```

#### 4. 颜色自定义（好看）

```bash
git config --global color.ui auto
```

---

### 二、Git 命令别名（最爽自定义！）

把长命令变短，比如：
`git st` = `git status`
`git co` = `git checkout`
`git br` = `git branch`
`git ci` = `git commit`
`git last` = 看最后一次提交

#### 直接复制执行（一键配置）

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci "commit -m"
git config --global alias.last "log -1 HEAD"
git config --global alias.lg "log --color --graph --pretty=format:'%h %s %an %ad' --date=format:'%m-%d %H:%M'"
```

#### 用起来超爽：

```bash
git st      # 查看状态
git br      # 查看分支
git ci "修复bug"  # 提交
git lg      # 漂亮日志
```

---

### 三、.gitignore 忽略文件（必须自定义）

创建 `.gitignore` 文件，让 Git **不提交**无用文件。

#### 常用忽略规则：

```
# 系统文件
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.iml
*.iws

# 依赖
node_modules/
vendor/

# 构建产物
dist/
build/
*.log
.env
```

#### 快速生成：

```bash
touch .gitignore
```

---

### 四、Git 配置文件位置

- **全局配置**：`~/.gitconfig`
- **仓库配置**：`.git/config`

可以直接打开文件手动修改。

---

### 五、自定义总结（背诵版）

1. **配置身份**：`user.name` / `user.email`
2. **命令别名**：`st`、`co`、`br`、`ci`、`lg`
3. **忽略文件**：`.gitignore`
4. **编辑器**：可指定 VS Code
5. **换行符**：跨系统必须配置

---

需要我把 **Git 全部知识点（命令+分支+标签+自定义）合成一页终极面试速查表** 吗？