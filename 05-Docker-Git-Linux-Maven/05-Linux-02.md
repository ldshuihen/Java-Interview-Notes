## Linux三种模式

在 Linux 系统中，提到“三种模式”，通常是指 **Vi/Vim 编辑器的三种工作模式**。这是初学者学习 Linux 时经常接触的重要概念。当然，在更广义的系统层面，有时也会指系统的运行级别（runlevel）或操作模式（如用户态/内核态），但最常见、最标准的“Linux 的三种模式”指的是 **Vim 编辑器的三种模式**。

下面分别介绍：

------

### 一、Vim 编辑器的三种模式（最常见含义）

Vim（Vi Improved）是 Linux 下最常用的文本编辑器之一，它有三种主要操作模式：

#### 1. **命令模式（Normal Mode / Command Mode）**

- **默认进入的模式**。
- 用于执行光标移动、删除、复制、粘贴、进入其他模式等**命令操作**。
- 按 `Esc` 键可随时返回此模式。
- 示例操作：
  - `h/j/k/l`：左/下/上/右移动
  - `dd`：删除当前行
  - `yy`：复制当前行
  - `p`：粘贴
  - `:wq`：保存并退出（需先按 `:` 进入末行模式）

#### 2. **插入模式（Insert Mode）**

- 用于**输入和编辑文本**。
- 从命令模式按以下键之一进入：
  - `i`：在光标前插入
  - `a`：在光标后插入
  - `o`：在当前行下方新建一行并插入
- 按 `Esc` 返回命令模式。

#### 3. **末行模式（Ex Mode / Command-Line Mode）**

- 用于执行**保存、退出、查找替换、设置选项**等高级命令。
- 在命令模式下按 `:`（冒号）进入。
- 常见命令：
  - `:w` — 保存
  - `:q` — 退出
  - `:wq` 或 `:x` — 保存并退出
  - `:q!` — 强制退出不保存
  - `:set nu` — 显示行号
  - `:/keyword` — 向下搜索关键词

> 💡 提示：Vim 实际还有**可视模式（Visual Mode）**、**选择模式**等，但“三种基本模式”通常指上述三种。

------

### 其他可能的“三种模式”解释（较少见）

### 二、系统运行级别（Runlevel）— 已逐渐被 systemd 取代

传统 SysV init 系统中有多个运行级别（0~6），其中常被简称为几种“模式”：

- **单用户模式**（runlevel 1）：用于系统维护，只有 root 用户
- **多用户模式（无图形）**（runlevel 3）
- **多用户图形模式**（runlevel 5）

> 注：现代 Linux（如 Ubuntu 16.04+、CentOS 7+）使用 **systemd**，用 **target** 替代 runlevel，如：
>
> - `rescue.target` ≈ 单用户模式
> - `multi-user.target` ≈ runlevel 3
> - `graphical.target` ≈ runlevel 5

但这通常不被称为“Linux 的三种模式”。

------

### 三、CPU/系统特权级别（Ring Levels）

- x86 架构有 

  Ring 0 ~ Ring 3

   四个特权级：

  - **Ring 0**：内核模式（Kernel Mode）— 最高权限
  - **Ring 3**：用户模式（User Mode）— 普通程序运行

- 虽然有“内核态 vs 用户态”的说法，但这是**两种模式**，不是三种。

------

### 总结

| 语境                   | “三种模式”指                                                 |
| ---------------------- | ------------------------------------------------------------ |
| **最常见（推荐理解）** | **Vim 编辑器的三种模式**： 1. 命令模式 2. 插入模式 3. 末行模式 |
| 系统管理（旧系统）     | 运行级别：单用户、多用户文本、多用户图形                     |
| 底层架构               | 不适用（通常是两种模式：用户态/内核态）                      |

> 📌 **对于 Linux 初学者，“Linux 的三种模式”几乎总是指 Vim 的三种工作模式。**

如果你是在学习 Vim 或被要求掌握“三种模式”，请重点掌握上面第一部分的内容。

## 快捷键

在 Linux（特别是使用 Bash、Zsh 等 Shell 的终端环境）中，**Tab 键**和其他一些按键具有非常实用的**特殊功能**，能极大提升操作效率。下面详细介绍：

------

### 🔑 一、Tab 键：自动补全（Auto-completion）

这是 **Linux 终端中最强大、最常用的功能之一**。

#### ✅ 功能说明：

- **按一次 Tab**：尝试自动补全命令、文件名、目录名、变量名等。
- **按两次 Tab**：列出所有可能的补全选项（如果存在多个匹配项）。

#### 🌰 示例：

```bash
# 假设当前目录有文件：report.pdf, report_final.docx, README.md

$ cat rep<Tab>
# 按一次 Tab → 自动补全为 "report"
# 再按一次 Tab → 显示：
# report.pdf        report_final.docx

$ cat report<Tab><Tab>
# 此时会列出两个以 "report" 开头的文件供选择
```

#### 💡 支持补全的内容包括：

| 类型                        | 示例                                   |
| --------------------------- | -------------------------------------- |
| 命令名                      | `ls`、`grep`、`systemctl`              |
| 文件/目录路径               | `/etc/pa<Tab>` → `/etc/passwd`         |
| 用户名（在 `~username` 中） | `ls ~ro<Tab>` → `~root/`               |
| 主机名（配合 SSH）          | `ssh user@ser<Tab>` → 自动补全已知主机 |
| Git 分支名（在 Git 仓库中） | `git checkout fea<Tab>`                |
| 选项参数（部分命令支持）    | `docker run --rm<Tab>`                 |

> ⚠️ 注意：补全功能依赖于 Shell（如 Bash 的 `bash-completion` 包）。若未安装，某些高级补全（如 systemctl、docker）可能不可用。可通过以下命令安装：
>
> ```bash
> # Ubuntu/Debian
> sudo apt install bash-completion
> # CentOS/RHEL
> sudo yum install bash-completion
> ```

------

### 🔑 二、其他常用特殊功能键

#### 1. **Ctrl + C**

- **中断当前运行的程序**（发送 SIGINT 信号）。
- 例如：停止一个卡住的 `ping` 或 `top`。

#### 2. **Ctrl + Z**

- **将当前任务挂起到后台**（暂停，非终止），返回 Shell。

- 可通过 `fg`（前台恢复）或 `bg`（后台继续）管理。

- 示例：

  ```bash
  $ sleep 100
  ^Z
  [1]+  Stopped    sleep 100
  $ fg   # 恢复运行
  ```

#### 3. **Ctrl + D**

- **发送 EOF（End of File）信号**。
- 在交互式 Shell 中：**退出当前 Shell 会话**（等效于 `exit`）。
- 在输入时：表示“输入结束”（如 `cat > file` 后按 Ctrl+D 保存）。

#### 4. **Ctrl + L**

- **清屏**（效果同 `clear` 命令），但历史记录仍在，可滚动查看。

#### 5. **方向键（↑ ↓ ← →）**

- **↑ / ↓**：浏览命令历史（等效于 `history` + 选择）
- **← / →**：在当前命令行中左右移动光标

#### 6. **Ctrl + A / Ctrl + E**

- **Ctrl + A**：光标跳到**行首**
- **Ctrl + E**：光标跳到**行尾**

#### 7. **Ctrl + U / Ctrl + K**

- **Ctrl + U**：删除**从光标到行首**的内容
- **Ctrl + K**：删除**从光标到行尾**的内容

#### 8. **Ctrl + R**

- **反向搜索命令历史**（Reverse Search）。
- 输入关键词即可查找之前执行过的命令。
- 多次按 Ctrl+R 可循环匹配更早的命令。

#### 9. **Alt + . （或 Esc + .）**

- 插入**上一条命令的最后一个参数**。

- 极其高效！例如：

  ```bash
  $ cp file.txt /backup/
  $ cd <Alt+.>   # 自动变成 cd /backup/
  ```

#### 10. **Tab + 其他组合（高级）**

- **Alt + Tab**：在图形界面中切换窗口（非终端功能）
- **Shift + Tab**：在支持的程序中反向补全（较少用）

------

### 🧩 三、Shell 特性补充

- 通配符配合 Tab：

  ```bash
  $ ls *.log<Tab>   # 可补全匹配的日志文件
  ```

- 变量补全：

  ```bash
  $ echo $HO<Tab>   # 补全为 $HOME
  ```



------

### Linux 中查看/操作命令历史的高效方法

虽然没有“图形化弹窗”，但 Linux 提供了以下**更实用、可搜索、可脚本化**的方式：

------

#### 1. **`history` 命令 — 查看完整历史**

```bash
$ history
```

- 显示带编号的命令历史列表（默认保存在 `~/.bash_history`）。

- 可配合 

  ```
  grep
  ```

   搜索：

  ```bash
  $ history | grep git
  ```

------

#### 2. **方向键 ↑ / ↓ — 快速浏览历史**

- 最常用！按 **↑** 键逐条回溯历史命令。
- 支持在输入部分命令后按 ↑ 进行**前缀匹配**（需配置，见下文）。

------

#### 3. **Ctrl + R — 交互式反向搜索（强烈推荐！）**

- 按 `Ctrl + R`，然后输入关键词（如 `ssh`），Shell 会**实时反向搜索**包含该词的历史命令。
- 多次按 `Ctrl + R` 可循环查找更早的匹配项。
- 按 `Enter` 执行，按 `Esc` 或方向键退出搜索并编辑。

> 💡 这是 Linux 下比 F7 更强大的“搜索式历史”功能！

------

#### 4. **增强版：启用“增量历史搜索”或“模糊搜索”**

##### 方法一：配置 Bash 支持前缀搜索（按 ↑ 自动匹配）

在 `~/.inputrc` 文件中添加：

```bash
# ~/.inputrc
"\e[A": history-search-backward
"\e[B": history-search-forward
```

然后重启终端。
效果：输入 `git` 后按 ↑，只会显示以 `git` 开头的历史命令。

##### 方法二：使用 **fzf + Ctrl+R**（超强大！）

安装 [`fzf`](https://github.com/junegunn/fzf)（模糊查找工具）后，`Ctrl + R` 会变成**图形化模糊搜索界面**（带高亮、预览）：

```bash
# Ubuntu/Debian
sudo apt install fzf

# CentOS/RHEL
sudo yum install fzf

# 或通过 GitHub 安装最新版
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

效果如下（类似 F7 的交互体验，但更强大）：

```
> ssh
  1567 ssh user@server1
  1423 ssh admin@192.168.1.100
  1302 ssh -i key.pem ubuntu@cloud
```

------

#### 5. **图形化终端模拟器插件（接近 F7 体验）**

某些高级终端支持插件或内置历史浏览器：

| 终端                 | 功能                                            |
| -------------------- | ----------------------------------------------- |
| **Konsole**（KDE）   | `View → Show Command History`（部分版本支持）   |
| **Terminator**       | 右键 → "Show command history"（需插件）         |
| **Alacritty + fzf**  | 通过快捷键调用 fzf 历史搜索                     |
| **VS Code 内置终端** | 支持 `Ctrl + Shift + H`（需配置）或使用命令面板 |

但这些都不是标准功能，**主流做法仍是 `Ctrl+R` 或 `history`**。

------

#### 🔧 总结：Linux 如何替代 Windows 的 F7？

| Windows CMD                | Linux 替代方案                                            |
| -------------------------- | --------------------------------------------------------- |
| **F7：图形化命令历史窗口** | ❌ 无直接等效                                              |
|                            | ✅ **`Ctrl + R`**：交互式反向搜索（最推荐）                |
|                            | ✅ **`history`**：查看全部历史                             |
|                            | ✅ **↑ / ↓**：逐条浏览                                     |
|                            | ✅ **`fzf` + `Ctrl+R`**：模糊搜索 + 图形化预览（最佳体验） |
|                            | ✅ 配置 `~/.inputrc` 实现前缀历史搜索                      |

------

#### 🚀 推荐操作

1. **立即尝试 `Ctrl + R`** — 你会爱上它！
2. **安装 `fzf`** — 获得接近甚至超越 F7 的体验。
3. **配置 `~/.inputrc`** — 让 ↑/↓ 更智能。

> 💬 小知识：Linux 的设计哲学是“组合小工具完成复杂任务”，所以它不提供单一图形窗口，而是给你**可组合、可自动化、可定制**的强大工具链。

需要我帮你写一个 `~/.inputrc` 配置模板，或者 `fzf` 的一键安装脚本吗？

------

### ✅ 总结：高频快捷键速查表

| 快捷键           | 功能                               |
| ---------------- | ---------------------------------- |
| **Tab**          | 自动补全（单次）/ 列出选项（双次） |
| **Ctrl + C**     | 终止当前程序                       |
| **Ctrl + Z**     | 暂停程序并放入后台                 |
| **Ctrl + D**     | 退出 Shell / 表示输入结束          |
| **Ctrl + L**     | 清屏                               |
| **↑ / ↓**        | 浏览命令历史                       |
| **Ctrl + A / E** | 跳到行首 / 行尾                    |
| **Ctrl + U / K** | 删除到行首 / 行尾                  |
| **Ctrl + R**     | 反向搜索历史命令                   |
| **Alt + .**      | 插入上条命令最后一个参数           |

------

## Linux命令参数

在 Linux 中，**命令参数**（也称为选项、标志或 arguments）用于控制命令的行为。理解参数的格式、类型和使用方式，是高效使用 Linux 的关键。

------

### 一、命令的基本结构

```bash
command [options] [arguments]
```

- **`command`**：要执行的命令（如 `ls`, `cp`, `grep`）
- **`[options]`**：可选的**选项/参数**，用于修改命令行为（通常以 `-` 或 `--` 开头）
- **`[arguments]`**：命令操作的**目标对象**（如文件名、目录名、字符串等）

> 📌 注意：方括号 `[ ]` 表示“可选”，实际使用时不需要输入。

------

### 二、参数的常见格式

#### 1. **短选项（Short Options）**

- 以单个连字符 `-` 开头，后跟**一个字母**
- 可合并多个短选项（只要它们不带参数）

✅ 示例：

```bash
ls -l -a        # 等价于
ls -la          # 合并写法

tar -x -z -f archive.tar.gz   # 等价于
tar -xzf archive.tar.gz
```

#### 2. **长选项（Long Options）**

- 以双连字符 `--` 开头，后跟**完整单词**（更易读）
- 不能合并

✅ 示例：

```bash
ls --long --all
grep --ignore-case "hello" file.txt
```

> 💡 很多命令同时支持短/长选项，例如：
>
> - `-h` ≡ `--help`
> - `-r` ≡ `--recursive`
> - `-v` ≡ `--verbose`

#### 3. **带参数的选项**

某些选项需要额外的值（称为“选项参数”）：

```bash
# -n 后面需要一个数字
head -n 10 file.txt

# --output 后面需要文件名
gcc --output myprog source.c

# 等价写法（短选项可连写）
head -n10 file.txt    # 注意：无空格也可（但非所有命令都支持）
```

#### 4. **位置参数（Positional Arguments）**

指不带 `-` 或 `--` 的参数，顺序很重要：

```bash
cp source.txt dest.txt
#     ↑         ↑
#   源文件    目标文件（顺序不能颠倒！）
```

------

### 三、特殊符号与约定

| 符号           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| `-`            | 单独使用时，常表示“标准输入”或“标准输出” 例：`tar -cf - dir | gzip > dir.tar.gz` |
| `--`           | **选项结束标记**，后面的内容即使以 `-` 开头也被视为普通参数 例：`rm -- -file.txt`（删除名为 `-file.txt` 的文件） |
| `*`, `?`, `[]` | Shell 通配符（由 Shell 展开，不是命令参数本身）              |

------

### 四、如何查看命令支持哪些参数？

#### 1. **`--help`（最常用）**

```bash
ls --help
grep --help
```

#### 2. **`man` 手册页**

```bash
man ls
man grep
```

- 查看 **SYNOPSIS** 部分了解参数格式
- **OPTIONS** 部分详细说明每个选项

#### 3. **`info`（GNU 软件更详细文档）**

```bash
info coreutils 'ls invocation'
```

------

### 五、常见参数类型示例

| 功能       | 短选项      | 长选项          | 说明                   |
| ---------- | ----------- | --------------- | ---------------------- |
| 帮助       | `-h`        | `--help`        | 显示帮助信息           |
| 递归       | `-r` / `-R` | `--recursive`   | 递归处理子目录         |
| 强制       | `-f`        | `--force`       | 忽略不存在文件，不提示 |
| 详细输出   | `-v`        | `--verbose`     | 显示详细过程           |
| 静默       | `-q`        | `--quiet`       | 不输出信息             |
| 版本       | `-V`        | `--version`     | 显示版本号             |
| 输出到文件 | `-o file`   | `--output=file` | 指定输出文件           |

> ⚠️ 注意：不同命令的选项可能含义不同！
> 例如：`-v` 在 `grep` 中是“反向匹配”（`--invert-match`），而在 `cp` 中是“详细模式”。

------

### 六、高级用法技巧

#### 1. **组合短选项**

```bash
# 创建父目录并显示过程
mkdir -pv /a/b/c
# -p: parents, -v: verbose
```

#### 2. **使用 `--` 处理特殊文件名**

```bash
# 删除名为 "-rf" 的文件
rm -- -rf

# 或用路径避免歧义
rm ./-rf
```

#### 3. **参数来自文件（`xargs` 或 `@file`）**

某些命令支持从文件读取参数：

```bash
# GCC 支持 @file 语法
gcc @options.txt main.c

# 或用 xargs
cat files.txt | xargs rm
```

------

### 七、POSIX 与 GNU 参数风格

| 风格      | 特点                         | 示例              |
| --------- | ---------------------------- | ----------------- |
| **POSIX** | 传统 Unix 风格，仅支持短选项 | `ls -l -a`        |
| **GNU**   | 扩展支持长选项、更友好       | `ls --long --all` |

现代 Linux 命令大多遵循 **GNU 风格**。

------

### ✅ 总结

| 概念            | 说明                             |
| --------------- | -------------------------------- |
| **短选项**      | `-l`, `-a`, 可合并为 `-la`       |
| **长选项**      | `--long`, `--all`, 更清晰        |
| **带参选项**    | `-n 10`, `--output=file`         |
| **位置参数**    | `cp src dest`，顺序敏感          |
| **`--` 的作用** | 标记选项结束，防止文件名被误解析 |
| **查帮助**      | 优先用 `--help`，再用 `man`      |

------

掌握参数的使用，能让你从“会用命令”进阶到“**精准控制命令**”。
如果你有某个具体命令（如 `find`、`rsync`、`awk`）想深入学习其参数，我可以为你详解！

## Linux内置命令

在 Linux 中，“**内置命令**”（Built-in Commands）是指由 **Shell 本身直接提供和执行的命令**，而不是通过调用外部可执行文件（如 `/bin/ls`、`/usr/bin/grep` 等）来运行的。

------

### 🔍 为什么需要内置命令？

1. **效率更高**：无需创建新进程，直接在当前 Shell 进程中执行。
2. **控制 Shell 状态**：有些操作（如改变目录、设置变量）必须在当前 Shell 中完成，不能交给子进程。
3. **保证基本功能可用**：即使系统路径损坏或外部命令丢失，Shell 仍能工作。

------

### ✅ 常见的 Shell 内置命令（以 Bash 为例）

> 💡 可通过 `type command` 或 `help` 判断是否为内置命令：
>
> ```bash
> $ type cd
> cd is a shell builtin
> $ help cd    # 查看内置命令帮助
> ```

### 一、环境与变量控制

| 命令       | 说明                                                 |
| ---------- | ---------------------------------------------------- |
| `cd`       | 切换当前工作目录（必须是内置，否则只在子进程中生效） |
| `export`   | 导出环境变量                                         |
| `unset`    | 删除变量或函数                                       |
| `readonly` | 设置只读变量                                         |
| `local`    | 在函数中定义局部变量（Bash 特有）                    |

### 二、流程控制与逻辑

| 命令            | 说明                                    |
| --------------- | --------------------------------------- |
| `.` 或 `source` | 在当前 Shell 中执行脚本（非新建子进程） |
| `eval`          | 将参数作为命令执行                      |
| `exec`          | 用指定命令替换当前 Shell 进程           |
| `exit`          | 退出当前 Shell                          |
| `return`        | 从函数或 sourced 脚本中返回             |

### 三、作业控制（Job Control）

| 命令   | 说明                                                 |
| ------ | ---------------------------------------------------- |
| `jobs` | 列出后台作业                                         |
| `fg`   | 将作业放到前台                                       |
| `bg`   | 将暂停的作业放到后台继续运行                         |
| `kill` | 终止作业或进程（Shell 内置版，也可调用 `/bin/kill`） |

### 四、历史与别名

| 命令      | 说明               |
| --------- | ------------------ |
| `history` | 显示或操作命令历史 |
| `alias`   | 定义命令别名       |
| `unalias` | 删除别名           |

### 五、其他实用内置命令

| 命令         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| `pwd`        | 打印当前工作目录（多数 Shell 提供内置版，比 `/bin/pwd` 更快） |
| `test` / `[` | 条件测试（如 `[ -f file ]`）                                 |
| `getopts`    | 解析脚本选项（用于 Shell 脚本）                              |
| `shift`      | 移动位置参数（用于脚本）                                     |
| `times`      | 显示 Shell 及子进程的累计 CPU 时间                           |

------

### ⚠️ 注意：有些命令既是内置，又有外部版本

例如：

```bash
$ type -a echo
echo is a shell builtin
echo is /bin/echo

$ type -a kill
kill is a shell builtin
kill is /bin/kill
```

- Shell 会优先使用**内置版本**。

- 若想强制使用外部命令，可加路径：

  ```bash
  /bin/echo "Hello"
  ```

------

### 🧩 如何查看所有 Bash 内置命令？

1. **使用 `help` 命令**：

   ```bash
   $ help
   ```

   会列出所有内置命令及其简要说明。

2. **查看 Bash 手册**：

   ```bash
   $ man bash
   ```

   搜索 “SHELL BUILTIN COMMANDS” 部分。

3. **编程方式获取（Bash 特有）**：

   ```bash
   $ enable -p    # 列出所有启用的内置命令
   ```

------

### 📌 不同 Shell 的内置命令可能不同

- **Bash**：最常用，内置命令丰富。
- **Zsh**：兼容 Bash，但有额外内置（如 `zle` 编辑行）。
- **Dash**（Debian 的 `/bin/sh`）：精简，内置命令较少。
- **Fish**：设计不同，很多命令行为与 POSIX 不同。

> ✅ 在编写可移植 Shell 脚本时，建议使用 `#!/bin/sh` 并避免依赖 Bash 特有内置命令。

------

### ✅ 总结

| 特性                 | 说明                                                |
| -------------------- | --------------------------------------------------- |
| **什么是内置命令？** | 由 Shell 自身实现，无需调用外部程序                 |
| **如何识别？**       | 用 `type command` 或 `help command`                 |
| **典型例子**         | `cd`, `export`, `source`, `alias`, `history`, `pwd` |
| **优势**             | 高效、能修改当前 Shell 环境                         |
| **注意**             | 某些命令有内置+外部两个版本                         |

------

如果你正在写 Shell 脚本，理解哪些是内置命令非常重要——比如你**永远不能用 `cd` 作为管道的一部分**，因为它不会在子 Shell 中生效：

```bash
# 错误！cd 在子 Shell 中执行，主 Shell 目录不变
echo /tmp | cd

# 正确
cd /tmp
```

需要一份 **Bash 内置命令速查表** 或 **常见陷阱示例** 吗？我可以为你整理。

## Linux 命令

可以根据其用途和功能大致分为以下几类。这种分类有助于系统学习和高效使用 Linux 系统：

------

### 1. **文件与目录操作命令**

用于创建、查看、修改、删除文件和目录。

- `ls`：列出目录内容  
- `cd`：切换目录  
- `pwd`：显示当前工作目录  
- `mkdir`：创建目录  
- `rmdir` / `rm`：删除空目录 / 删除文件或目录  
- `cp`：复制文件或目录  
- `mv`：移动或重命名文件/目录  
- `touch`：创建空文件或更新文件时间戳  
- `find`：查找文件  
- `locate`：快速定位文件（基于数据库）

------

#### 📁 1. `ls`：列出目录内容（最常用命令之一）

##### ✅ 基础用法

```bash
ls                # 列出非隐藏文件
ls -l             # 长格式（权限、所有者、大小、时间）
ls -a             # 显示隐藏文件（以 . 开头）
ls -lh            # 人类可读的文件大小（KB/MB/GB）
ls -lt            # 按修改时间排序（最新在前）
ls -S             # 按文件大小排序（大文件在前）
```

##### 💡 组合技巧

```bash
# 查看目录本身的信息（而非内容）
ls -ld /etc

# 彩色输出（多数系统默认开启）
ls --color=auto

# 递归列出子目录（慎用于大目录！）
ls -R /path
```

> ⚠️ 注意：`ls` 不显示文件真实磁盘占用（用 `du`）。

------

#### 🧭 2. `cd`：切换目录

##### ✅ 常用形式

```bash
cd /home/user     # 绝对路径
cd documents      # 相对路径
cd ..             # 返回上一级
cd ~              # 回到家目录（等价于 cd）
cd -              # 切换到上一个工作目录（非常实用！）
```

> 💡 **小知识**：`cd` 是 shell 内建命令（不是外部程序），所以 `which cd` 找不到。

------

#### 📍 3. `pwd`：显示当前工作目录

```bash
pwd               # 输出绝对路径，如 /home/alice/project
```

> ✅ 用途：脚本中确认当前位置，或复制路径。

------

#### ➕ 4. `mkdir`：创建目录

###### ✅ 基础与进阶

```bash
mkdir dir1                    # 创建单个目录
mkdir -p a/b/c                # 递归创建（父目录不存在也创建）
mkdir -v project/{src,bin,doc} # 详细模式 + 花括号展开（创建多个子目录）
```

> 💡 **最佳实践**：总是用 `mkdir -p`，避免“目录不存在”错误。

------

#### 🗑️ 5. `rmdir` / `rm`：删除目录或文件

| 命令    | 用途                   | 限制         |
| ------- | ---------------------- | ------------ |
| `rmdir` | 删除**空目录**         | 目录必须为空 |
| `rm`    | 删除文件或**非空目录** | 需加 `-r`    |

##### ✅ 安全删除示例

```bash
rmdir empty_dir               # 仅删空目录

rm file.txt                   # 删除文件
rm -i file.txt                # 交互确认（防误删）
rm -r dir/                    # 递归删除目录及内容
rm -rf dir/                   # 强制删除（无确认！危险！）
```

> ##### ⚠️ **致命警告**：
>
> - `rm -rf /` 或 `rm -rf *` 可能导致系统崩溃！
> - 永远不要在 root 下盲目执行 `rm -rf`
> - 建议 alias `rm='rm -i'`（但脚本中不生效）

> ✅ **替代方案**：使用 `trash-cli` 工具（类似回收站）：
>
> ```bash
> sudo apt install trash-cli
> trash file.txt    # 移入回收站
> trash-list        # 查看
> trash-restore     # 恢复
> ```

------

#### 📤 6. `cp`：复制文件或目录

##### ✅ 常用选项

```bash
cp file1.txt file2.txt        # 复制文件
cp -r dir1/ dir2/             # 递归复制目录
cp -v file.txt /backup/       # 显示过程（verbose）
cp -p file.txt copy.txt       # 保留权限、时间戳等属性
cp -a src/ dest/              # 归档模式（等价于 -dR --preserve=all）
```

> 💡 **注意**：
>
> - `cp dir1 dir2`：若 `dir2` 不存在，则创建 `dir2` 并复制内容进去
> - `cp dir1/ dir2/`：复制 `dir1` 的**内容**到 `dir2`（末尾 `/` 很关键！）

------

#### 🔄 7. `mv`：移动或重命名

```bash
mv oldname.txt newname.txt    # 重命名
mv file.txt /backup/          # 移动文件
mv dir1/ dir2/                # 移动目录（或重命名）
```

> ✅ **优势**：`mv` 在同一文件系统内是**元数据操作**，极快（不复制数据）。

> ⚠️ 注意：跨文件系统移动 = 复制 + 删除，速度慢且占空间。

------

#### ✍️ 8. `touch`：创建空文件或更新时间戳

##### ✅ 用途

```bash
touch newfile.txt             # 创建空文件（若不存在）
touch existing.log            # 更新访问/修改时间为当前时间
touch -t 202301010000 file    # 设置指定时间（格式：YYYYMMDDHHMM）
```

> 💡 **典型场景**：
>
> - 创建占位文件
> - 触发基于时间戳的构建系统（如 make）
> - 初始化日志文件

------

#### 🔍 9. `find`：强大而灵活的文件查找

##### ✅ 基本语法

```bash
find /path -name "*.log"      # 按名称查找（区分大小写）
find . -iname "*.txt"         # 忽略大小写
find /var/log -mtime -7       # 7 天内修改过的文件
find . -size +100M            # 大于 100MB 的文件
find . -type f -executable    # 查找可执行文件
```

##### 🌰 实用组合

```bash
# 删除 30 天前的日志
find /logs -name "*.log" -mtime +30 -delete

# 查找并压缩
find . -name "*.txt" -exec gzip {} \;

# 查找空文件
find . -empty
```

> ⚠️ 注意：`find` 速度较慢（实时遍历），但功能全面。

------

#### ⚡ 10. `locate`：极速文件定位（基于数据库）

##### ✅ 使用方法

```bash
locate nginx.conf             # 快速查找（毫秒级）
```

##### 🔧 管理数据库

```bash
sudo updatedb                 # 手动更新数据库（通常由 cron 每天自动运行）
```

> ✅ **优势**：比 `find` 快 100 倍以上
> ❌ **缺点**：结果可能不是最新的（依赖数据库更新频率）

> 💡 **建议**：先用 `locate` 快速定位，再用 `find` 精确操作。

------

#### 🛡️ 安全与效率最佳实践

| 场景         | 建议                                                         |
| ------------ | ------------------------------------------------------------ |
| 删除重要文件 | 先 `ls` 确认路径，再 `rm`；或用 `trash`                      |
| 复制大量数据 | 用 `rsync`（支持断点续传、校验）                             |
| 防止误操作   | 在 `~/.bashrc` 中设置： `alias rm='rm -i'` `alias cp='cp -i'` `alias mv='mv -i'` |
| 查找文件     | 日常用 `locate`，精确控制用 `find`                           |
| 创建深层目录 | 总是用 `mkdir -p`                                            |

------

#### 📋 快速参考表

| 操作         | 命令                                            |
| ------------ | ----------------------------------------------- |
| 列出详细文件 | `ls -lh`                                        |
| 创建多级目录 | `mkdir -p a/b/c`                                |
| 安全删除     | `rm -i file` 或 `trash file`                    |
| 复制整个目录 | `cp -r src/ dest/`                              |
| 重命名文件   | `mv old new`                                    |
| 查找大文件   | `find / -size +1G`                              |
| 快速找文件   | `locate filename` → `sudo updatedb`（若未找到） |

------

如果你需要：

- **`rsync` 替代 `cp` 的高级用法**
- 如何用 `find` + `xargs` 批量处理文件
- **防止 `rm -rf /\*` 的系统级保护措施**

欢迎继续提问！

------

### 2. **文件内容查看与编辑命令**

用于查看、搜索、编辑文件内容。

- `cat`：连接并显示文件内容  
- `more` / `less`：分页查看文件内容  
- `head` / `tail`：显示文件开头/末尾若干行  
- `grep`：在文件中搜索指定模式  
- `sed`：流编辑器，用于文本替换、删除等  
- `awk`：强大的文本处理语言  
- `nano` / `vim` / `emacs`：文本编辑器

------

#### 📄 一、快速查看文件内容

##### 1. **`cat`：全屏输出（适合小文件）**

```bash
cat file.txt          # 显示全部内容
cat file1.txt file2.txt > combined.txt  # 合并文件
cat -n script.sh      # 显示行号
```

> ⚠️ **警告**：不要用 `cat` 查看大文件（如日志），会刷屏！

------

##### 2. **`more` / `less`：分页查看（推荐 `less`）**

| 工具   | 特点                                   |
| ------ | -------------------------------------- |
| `more` | 只能向下翻页，功能简单                 |
| `less` | ✅ **双向滚动、搜索、高亮、支持大文件** |

```bash
less /var/log/syslog
```

**常用快捷键（less 中）**：

- `Space` / `b`：下/上一页  
- `/keyword`：向下搜索  
- `?keyword`：向上搜索  
- `n` / `N`：下一个/上一个匹配  
- `q`：退出  
- `&pattern`：只显示匹配行（过滤模式）

> 💡 **黄金组合**：`journalctl | less` 查看系统日志。

------

##### 3. **`head` / `tail`：查看开头/结尾**

```bash
head -n 10 file.log     # 前 10 行
tail -n 20 file.log     # 后 20 行
tail -f /var/log/nginx/access.log  # 实时追踪新日志（Ctrl+C 退出）
```

> ✅ **`tail -f` 是运维神器**！常用于监控服务日志。

> 🔧 进阶：`tail -F`（大写 F）可跟踪**轮转日志**（如 logrotate 后仍有效）。

------

#### 🔍 二、文本搜索：`grep`（必学！）

##### ✅ 基本用法

```bash
grep "error" /var/log/syslog        # 搜索包含 "error" 的行
grep -i "Error" file.txt            # 忽略大小写
grep -v "debug" file.log            # 反向匹配（排除 debug 行）
grep -r "TODO" ~/project/           # 递归搜索目录
```

##### 🔑 常用选项

| 选项        | 作用                           |
| ----------- | ------------------------------ |
| `-i`        | 忽略大小写                     |
| `-v`        | 反向匹配                       |
| `-r` / `-R` | 递归搜索子目录                 |
| `-n`        | 显示行号                       |
| `-l`        | 只输出匹配的文件名             |
| `-E`        | 使用扩展正则（等价于 `egrep`） |

##### 🌰 实用示例

```bash
# 搜索 IP 地址
grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log

# 统计错误数量
grep -c "ERROR" app.log

# 高亮显示（配合 --color=auto）
grep --color=always "warning" file.txt | less -R
```

> 💡 **技巧**：`grep` + `less` = 强大日志分析组合。

------

#### ✂️ 三、流编辑器：`sed`（文本替换利器）

##### ✅ 基本语法

```bash
sed 's/old/new/g' file.txt    # 替换每行所有 "old" 为 "new"
```

##### 🌰 常见用法

```bash
# 替换并保存到新文件
sed 's/foo/bar/g' input.txt > output.txt

# 直接修改原文件（慎用！）
sed -i 's/foo/bar/g' file.txt

# 删除空行
sed '/^$/d' file.txt

# 打印第 5-10 行
sed -n '5,10p' file.txt

# 注释掉配置项
sed -i 's/^port/port/' config.conf  # 假设原行为 "port=8080"
```

> ⚠️ **警告**：`-i` 会直接修改原文件，建议先备份或测试。

------

#### 📊 四、文本处理语言：`awk`（数据分析首选）

`awk` 擅长**按列处理结构化文本**（如日志、CSV）。

##### ✅ 基本语法

```bash
awk '{print $1}' file.txt    # 打印每行第 1 列（默认空格分隔）
```

##### 🌰 实用示例

```bash
# 统计 Nginx 日志中访问最多的 IP
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head

# 计算某列总和（如第 3 列）
awk '{sum += $3} END {print sum}' data.csv

# 条件过滤：打印第 2 列 > 100 的行
awk '$2 > 100 {print}' file.txt

# 指定分隔符（如 CSV）
awk -F',' '{print $1, $3}' data.csv
```

> 💡 **`awk` vs `cut`**：  
>
> - 简单列提取 → `cut -d' ' -f1`  
> - 复杂逻辑（计算、条件）→ `awk`

------

#### ✏️ 五、文本编辑器对比

| 编辑器      | 特点                             | 适用场景                       |
| ----------- | -------------------------------- | ------------------------------ |
| **`nano`**  | 简单直观，底部有快捷键提示       | 新手、快速编辑配置文件         |
| **`vim`**   | 功能强大、高度可定制、资源占用低 | 开发者、系统管理员（必备技能） |
| **`emacs`** | “操作系统中的操作系统”，插件丰富 | 资深用户、Lisp 爱好者          |

##### ✅ 快速上手 `nano`

```bash
nano config.txt
# Ctrl+O → 保存  
# Ctrl+X → 退出  
# Ctrl+K → 剪切行  
# Ctrl+U → 粘贴
```

##### ✅ `vim` 最小生存集

```bash
vim file.txt
# i → 进入插入模式  
# Esc → 回到命令模式  
# :wq → 保存退出  
# :q! → 强制退出不保存  
# /keyword → 搜索
```

> 💡 **建议**：至少掌握 `nano` 和 `vim` 基础，服务器环境通常无 GUI。

------

#### 🧩 六、高级组合技巧（管道艺术）

Linux 的威力在于**命令组合**：

```bash
# 实时监控并高亮错误
tail -f app.log | grep --color=always "ERROR"

# 提取日志中的 URL 并去重
awk '{print $7}' access.log | sort | uniq -c | sort -nr

# 批量替换多个文件中的字符串
grep -rl "old_string" ./project | xargs sed -i 's/old_string/new_string/g'
```

------

#### ✅ 最佳实践总结

| 场景              | 推荐命令                    |
| ----------------- | --------------------------- |
| 查看小文件        | `cat`                       |
| 浏览大文件/日志   | `less`                      |
| 实时监控日志      | `tail -f`                   |
| 搜索关键词        | `grep -r "pattern" dir/`    |
| 批量替换文本      | `sed -i 's/old/new/g' file` |
| 分析结构化数据    | `awk`                       |
| 快速编辑配置      | `nano`                      |
| 长期开发/远程编辑 | `vim`                       |

------

#### ⚠️ 安全提醒

1. **避免 `sed -i` 无备份操作**：重要文件先 `cp file file.bak`。
2. **`grep -r` 可能遍历大量文件**：在大型项目中限定范围（如 `grep -r "TODO" src/`）。
3. **不要用 `cat` + `grep` 代替 `grep file`**：前者效率低且浪费内存。

------

如果你需要：

- **`vim` 速查表**（常用命令）
- **`awk` 处理 JSON 的技巧**（配合 `jq`）
- 一个 **日志分析实战案例**（Nginx/Apache）

欢迎继续提问！

------

### 3. **系统信息与状态命令**

用于获取系统运行状态、硬件信息等。

- `uname -a`：显示系统内核信息  
- `hostname`：显示或设置主机名  
- `df`：显示磁盘空间使用情况  
- `du`：显示目录或文件的磁盘使用量  
- `free`：显示内存使用情况  
- `top` / `htop`：实时查看进程和资源占用  
- `ps`：显示当前运行的进程  
- `lscpu` / `lsmem` / `lsblk` / `lspci`：查看 CPU、内存、块设备、PCI 设备等硬件信息

------

#### 🖥️ 一、系统与内核信息

##### ✅ `uname -a`：查看内核与系统架构

```bash
$ uname -a
Linux myhost 5.15.0-91-generic #101-Ubuntu SMP Fri Dec 1 14:03:57 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

| 字段                | 含义              |
| ------------------- | ----------------- |
| `Linux`             | 内核名称          |
| `myhost`            | 主机名            |
| `5.15.0-91-generic` | 内核版本          |
| `x86_64`            | 硬件架构（64 位） |

> 💡 其他常用选项：
>
> - `uname -r`：仅显示内核版本（用于驱动兼容性检查）
> - `uname -m`：仅显示架构（如 `x86_64`, `aarch64`）

------

##### ✅ `hostname`：查看或设置主机名

```bash
hostname                # 查看当前主机名
sudo hostnamectl set-hostname newname   # 永久修改（systemd 系统）
```

> ⚠️ 注意：直接 `hostname newname` 仅临时生效（重启丢失）。

------

#### 💾 二、磁盘空间管理

##### 1. **`df`：查看文件系统磁盘使用**

```bash
df -h    # -h = human-readable（以 GB/MB 显示）
```

输出示例：

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   20G   28G  42% /
tmpfs           3.2G     0  3.2G   0% /run
```

> ✅ **关键列**：
>
> - `Use%`：使用率超过 80% 需警惕（日志、缓存堆积）
> - `Mounted on`：挂载点

> 🔍 进阶用法：
>
> ```bash
> df -T    # 显示文件系统类型（ext4, xfs, btrfs...）
> df /home # 只看 /home 分区
> ```

------

##### 2. **`du`：查看目录/文件实际占用**

```bash
du -sh /var/log        # -s = summary, -h = human-readable
du -h --max-depth=1 /  # 查看根目录下各子目录大小
```

> 💡 **常见场景**：
>
> - 找出大目录：`du -ah /path | sort -rh | head -n 20`
> - 排除挂载点：`du -x /`（不跨越文件系统）

> ⚠️ 注意：`du` 统计的是**磁盘块占用**，可能大于文件实际大小（因 block 对齐）。

------

#### 🧠 三、内存使用情况

##### ✅ `free`：查看物理内存与交换空间

```bash
$ free -h
               total    used    free  shared  buff/cache   available
Mem:            15Gi   2.1Gi   8.0Gi   120Mi        5.0Gi        12Gi
Swap:          2.0Gi      0B   2.0Gi
```

| 列           | 含义                               |
| ------------ | ---------------------------------- |
| `total`      | 总内存                             |
| `used`       | 已使用（含缓存）                   |
| `available`  | **真正可用内存**（推荐关注此列！） |
| `buff/cache` | 内核缓冲区 + 页面缓存（可回收）    |

> 💡 **Linux 内存哲学**：
> “Free memory is wasted memory.” → 空闲内存会被用作缓存，**高 cache 不代表内存不足**！

> 🔍 强制释放缓存（调试用，生产慎用）：
>
> ```bash
> echo 3 | sudo tee /proc/sys/vm/drop_caches
> ```

------

#### 📊 四、进程与资源监控

##### 1. **`top`：实时进程监控（经典）**

- 按 `P`：按 CPU 排序  
- 按 `M`：按内存排序  
- 按 `k`：杀死进程  
- 按 `q`：退出

> ⚠️ 缺点：界面简陋，不支持鼠标。

------

##### 2. **`htop`：增强版 top（强烈推荐安装）**

```bash
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL/Fedora
```

✅ 优势：

- 彩色界面、支持鼠标
- 树状进程视图（按 `t`）
- 垂直/水平滚动
- 直接 F9 杀进程

------

##### 3. **`ps`：静态进程快照（脚本友好）**

```bash
ps aux --sort=-%mem | head    # 按内存降序列出前 10 进程
ps -ef | grep nginx           # 查找 nginx 进程
```

> 💡 与 `top` 互补：`ps` 适合一次性查询，`top`/`htop` 适合持续观察。

------

#### 🖨️ 五、硬件信息探测（`ls*` 系列）

这些命令来自 `util-linux` 包，**无需 root 即可安全运行**。

| 命令        | 用途                         | 示例                       |
| ----------- | ---------------------------- | -------------------------- |
| **`lscpu`** | CPU 架构、核心数、线程、缓存 | `lscpu | grep "CPU(s)"`    |
| **`lsmem`** | 物理内存布局（需较新内核）   | `lsmem`                    |
| **`lsblk`** | 块设备（磁盘、分区、LVM）    | `lsblk -f`（显示文件系统） |
| **`lspci`** | PCI 设备（显卡、网卡等）     | `lspci | grep -i network`  |
| **`lsusb`** | USB 设备                     | `lsusb -t`（树状显示）     |

#### 🌰 实用组合示例：

##### 查看磁盘分区与挂载

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,LABEL
```

输出：

```text
NAME        SIZE FSTYPE LABEL MOUNTPOINT
sda        100G
├─sda1       1G ext4         /boot
└─sda2      99G ext4         /
nvme0n1    500G
└─nvme0n1p1 500G ext4         /data
```

##### 查看网卡型号

```bash
lspci -v | grep -A 10 -i ethernet
```

##### 查看 CPU 核心与线程

```bash
lscpu | grep -E "CPU\(s\)|Thread"
# 输出：
# CPU(s):              8
# Thread(s) per core:  2  → 表示 4 核 8 线程
```

------

#### 🧰 六、其他重要系统信息命令（补充）

| 命令                  | 用途                                         |
| --------------------- | -------------------------------------------- |
| `cat /etc/os-release` | 查看发行版信息（跨平台标准）                 |
| `uptime`              | 系统运行时间 + 平均负载                      |
| `dmidecode`           | 查看 BIOS、主板、内存条等硬件详情（需 root） |
| `inxi -Fxz`           | 全面硬件报告（需安装 `inxi`）                |

------

#### ✅ 最佳实践总结

| 场景                 | 推荐命令                       |
| -------------------- | ------------------------------ |
| 快速检查磁盘是否满   | `df -h`                        |
| 找出哪个目录占空间大 | `du -sh /path/* | sort -hr`    |
| 查看真实可用内存     | `free -h`（看 `available` 列） |
| 监控高 CPU 进程      | `htop` 或 `top` → 按 `P`       |
| 确认服务器 CPU 规格  | `lscpu`                        |
| 列出所有磁盘分区     | `lsblk`                        |

------

#### ⚠️ 安全与性能提醒

1. **不要盲目清理 `/tmp`**：某些服务依赖其中的 socket 或 lock 文件。
2. **高负载 ≠ 高 CPU**：`uptime` 中的 load average 包含不可中断进程（如 I/O 等待）。
3. **`dmidecode` 需谨慎**：输出包含序列号等敏感信息，避免泄露。

------

如果你需要：

- 一个 **系统健康检查脚本模板**（整合 df/free/top）
- 如何解读 **平均负载（load average）**
- **`/proc` 和 `/sys` 文件系统** 中的关键信息

欢迎继续提问！



------

### 4. **用户与权限管理命令**

用于管理用户账户、组以及文件权限。

- `whoami` / `id`：显示当前用户信息  
- `who` / `w`：显示当前登录用户  
- `useradd` / `userdel` / `usermod`：添加/删除/修改用户  
- `groupadd` / `groupdel`：管理用户组  
- `passwd`：修改密码  
- `chmod`：修改文件权限（如 755）  
- `chown` / `chgrp`：修改文件所有者/所属组

------

#### 👤 一、查看用户信息

##### ✅ `whoami`

```bash
whoami        # 仅输出当前用户名（如 alice）
```

##### ✅ `id`（更全面）

```bash
id            # 显示 UID、GID、所属组
# 输出示例：uid=1001(alice) gid=1001(alice) groups=1001(alice),27(sudo),100(users)

id alice      # 查看指定用户信息
```

> 💡 **用途**：脚本中判断当前用户身份。

------

##### ✅ `who` / `w`：查看登录用户

| 命令  | 特点                                   |
| ----- | -------------------------------------- |
| `who` | 显示登录用户、终端、登录时间           |
| `w`   | 更详细：包含正在执行的命令、CPU 时间等 |

```bash
$ w
 14:30:01 up 10 days,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
alice    pts/0    192.168.1.100    14:25    0.00s  0.01s  0.00s w
bob      tty1     -                10:00    4:30m  0.02s  0.02s -bash
```

> ⚠️ 注意：这些命令读取 `/var/run/utmp`，**不显示通过 `su` 切换的用户**。

------

#### 👥 二、用户账户管理

> 🔐 **需要 root 权限**（或 sudo）

##### 1. **`useradd`：创建用户**

```bash
# 基本创建（不创建家目录）
sudo useradd alice

# 推荐方式：自动创建家目录、设置 shell
sudo useradd -m -s /bin/bash alice

# 指定 UID 和主组
sudo useradd -u 2000 -g developers -m alice
```

> 📁 默认行为：
>
> - 不加 `-m` → **不创建 `/home/alice`**
> - 不加 `-s` → 使用 `/bin/sh`（不是 bash！）

------

##### 2. **`userdel`：删除用户**

```bash
sudo userdel alice          # 仅删用户，保留家目录
sudo userdel -r alice       # 同时删除家目录和邮件池
```

> ⚠️ **警告**：确保用户未登录，否则可能失败。

------

##### 3. **`usermod`：修改用户属性**

```bash
# 添加到附加组（如 sudo）
sudo usermod -aG sudo alice    # -aG = append to Group

# 修改家目录
sudo usermod -d /new/home/alice -m alice  # -m 表示移动旧目录

# 锁定/解锁账户
sudo usermod -L alice   # 锁定（密码前加 !）
sudo usermod -U alice   # 解锁
```

> ✅ **关键技巧**：  
>
> - 修改组必须用 `-aG`，否则会**覆盖原有附加组**！
> - 锁定账户 ≠ 禁止 SSH 密钥登录（仍可通过密钥登录）。

------

#### 👥 三、用户组管理

##### ✅ `groupadd` / `groupdel`

```bash
sudo groupadd developers
sudo groupdel developers    # 要求组内无用户
```

##### ✅ 查看组信息

```bash
getent group developers    # 标准方式（支持 LDAP/NIS）
cat /etc/group | grep dev  # 仅本地组
```

------

#### 🔑 四、密码管理

##### ✅ `passwd`：修改密码

```bash
passwd              # 修改自己的密码
sudo passwd alice   # root 修改他人密码（无需旧密码）
```

##### ✅ 其他密码操作

```bash
# 设置密码过期
sudo chage -M 90 alice     # 90 天后过期
sudo chage -l alice        # 查看过期信息

# 立即过期（强制下次登录改密码）
sudo passwd -e alice
```

> ⚠️ 安全建议：生产环境应启用密码策略（如 `pam_pwquality`）。

------

#### 🔒 五、文件权限管理（核心！）

Linux 权限模型：**用户（u）、组（g）、其他（o）** × **读（r=4）、写（w=2）、执行（x=1）**

##### 1. **`chmod`：修改权限**

###### 📌 数字模式（常用）

```bash
chmod 755 script.sh    # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
chmod 600 secret.key   # rw-------（私钥常用）
```

###### 📌 符号模式（灵活）

```bash
chmod u+x script.sh      # 给所有者加执行权
chmod g-w file           # 移除组的写权限
chmod o=r file           # 设置其他人为只读
chmod +x script.sh       # 所有类别加执行权（等价于 a+x）
```

> 💡 **目录 vs 文件**：
>
> - 目录需要 `x` 才能 `cd` 进入
> - 文件需要 `x` 才能执行

------

##### 2. **`chown` / `chgrp`：修改所有者**

```bash
# 修改所有者
sudo chown alice file.txt

# 修改所有者 + 组
sudo chown alice:developers file.txt

# 递归修改目录
sudo chown -R alice:developers /project

# 仅修改组（等价于 chgrp）
sudo chown :developers file.txt
```

> ✅ **注意**：
>
> - 普通用户**不能**将文件所有权转给他人（防止权限提升）
> - 只有 root 或文件当前所有者可修改权限

------

#### 🧩 六、高级权限概念（补充）

| 概念           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| **umask**      | 控制新建文件的默认权限（如 `umask 022` → 文件 644，目录 755） |
| **ACL**        | 扩展权限（`setfacl`/`getfacl`），支持多用户精细控制          |
| **sticky bit** | 目录上设 `chmod +t`（如 `/tmp`），用户只能删自己文件         |
| **SUID/SGID**  | `chmod u+s`：以文件所有者身份运行（如 `passwd` 命令）        |

------

#### ✅ 最佳实践与安全建议

| 场景           | 建议                                                   |
| -------------- | ------------------------------------------------------ |
| Web 服务器文件 | 所有者为部署用户，组为 `www-data`，权限 `644`/`755`    |
| 私钥文件       | `chmod 600`，所有者为自己                              |
| 共享目录       | `chmod 2775`（SGID 位确保新文件继承组）                |
| 禁止用户登录   | 创建用户时指定 shell 为 `/usr/sbin/nologin`            |
| 审计权限       | 定期用 `find / -perm -4000 2>/dev/null` 查找 SUID 文件 |

------

#### 🚫 常见错误

1. **`chmod 777` 滥用** → 严重安全风险！
2. **忘记递归（-R）** → 目录权限未生效
3. **混淆 `chown user.group` 和 `chown user:group`** → 两者都有效，但 `:` 更通用
4. **用 `usermod -G` 而非 `-aG`** → 意外清空用户附加组

------

#### 📋 快速参考表

| 操作                 | 命令                                 |
| -------------------- | ------------------------------------ |
| 创建带家目录的用户   | `sudo useradd -m -s /bin/bash alice` |
| 将用户加入 sudo 组   | `sudo usermod -aG sudo alice`        |
| 设置文件仅自己可读写 | `chmod 600 file`                     |
| 递归修改目录所有者   | `sudo chown -R alice:group dir/`     |
| 查看用户所属组       | `groups alice` 或 `id alice`         |

------

如果你需要：

- 一个 **自动化创建开发用户的脚本模板**
- 如何配置 **sudo 免密码**（安全方式）
- **ACL 权限实战示例**（如多人协作目录）

欢迎继续提问！

------

### 5. **网络相关命令**

用于网络配置、测试和诊断。

- `ping`：测试网络连通性  
- `ifconfig` / `ip`：查看或配置网络接口（`ip` 是现代推荐命令）  
- `netstat` / `ss`：查看网络连接、监听端口（`ss` 更快）  
- `dig` / `nslookup` / `host`：DNS 查询工具  
- `wget` / `curl`：下载文件或与 Web 服务交互  
- `ssh` / `scp` / `sftp`：远程登录与文件传输（基于 SSH）  
- `traceroute` / `mtr`：追踪数据包路径

------

#### 🌐 一、连通性测试

##### ✅ `ping`：测试主机是否可达

```bash
ping google.com          # 持续 ping（Ctrl+C 停止）
ping -c 4 8.8.8.8        # 只发 4 个包
```

> ⚠️ 注意：某些服务器禁 ping（ICMP），但服务仍可能正常。

------

#### 📡 二、网络接口配置（**优先用 `ip`**）

##### ❌ 旧命令：`ifconfig`（已过时）

- 来自 `net-tools` 包，**不再维护**
- 无法处理现代网络特性（如多 IP、VLAN、策略路由）

##### ✅ 新标准：`ip`（来自 `iproute2`，功能强大）

```bash
# 查看所有接口
ip addr show        # 或简写 ip a

# 启用/禁用接口
ip link set eth0 up
ip link set eth0 down

# 添加 IP 地址
ip addr add 192.168.1.100/24 dev eth0

# 查看路由表
ip route show       # 或 ip r

# 添加默认网关
ip route add default via 192.168.1.1
```

> 💡 **建议**：在脚本和日常操作中**完全使用 `ip` 替代 `ifconfig`/`route`**。

------

#### 🔌 三、网络连接与端口监听（**优先用 `ss`**）

##### ❌ 旧命令：`netstat`

- 依赖 `/proc/net/`，速度慢
- 已被 `ss` 取代

##### ✅ 新命令：`ss`（Socket Statistics）

```bash
# 查看所有监听端口（TCP+UDP）
ss -tuln

# 查看所有连接（含进程）
ss -tunlp

# 只看 TCP 连接
ss -tn

# 查看 ESTABLISHED 连接
ss -o state established
```

| 选项 | 含义                         |
| ---- | ---------------------------- |
| `-t` | TCP                          |
| `-u` | UDP                          |
| `-l` | Listening                    |
| `-n` | 不解析服务名（显示数字端口） |
| `-p` | 显示进程 PID/名称            |

> ✅ **优势**：`ss` 直接从内核获取数据，比 `netstat` 快 10 倍以上。

------

#### 🧭 四、DNS 查询工具（推荐 `dig`）

| 工具       | 特点                                   | 推荐度   |
| ---------- | -------------------------------------- | -------- |
| `dig`      | 功能最全、输出清晰、支持 ANY/AXFR 等   | ✅✅✅ 首选 |
| `host`     | 简洁，适合脚本                         | ✅✅       |
| `nslookup` | 交互式，但行为不一致（不同系统差异大） | ⚠️ 不推荐 |

##### ✅ `dig` 示例

```bash
dig example.com                 # 默认查 A 记录
dig example.com MX              # 查邮件记录
dig @8.8.8.8 example.com        # 指定 DNS 服务器
dig +short example.com          # 简洁输出（适合脚本）
dig -x 8.8.8.8                  # 反向解析（PTR）
```

##### ✅ `host` 示例

```bash
host example.com
host -t AAAA ipv6.google.com
```

------

#### 📥 五、下载与 HTTP 交互（`curl` vs `wget`）

| 工具       | 优势                                                         | 典型场景                 |
| ---------- | ------------------------------------------------------------ | ------------------------ |
| **`curl`** | 支持 **30+ 协议**（HTTP/HTTPS/FTP/SFTP/...）、可发送 POST/JSON、调试请求 | API 测试、上传、复杂请求 |
| **`wget`** | 支持**断点续传**、递归下载整个网站、后台下载                 | 下载大文件、镜像站点     |

##### ✅ `curl` 示例

```bash
# GET 请求
curl https://api.example.com/data

# POST JSON
curl -X POST -H "Content-Type: application/json" -d '{"name":"Alice"}' https://api.example.com/users

# 下载文件
curl -o file.zip https://example.com/file.zip

# 跟随重定向
curl -L https://bit.ly/xxx
```

##### ✅ `wget` 示例

```bash
# 下载文件
wget https://example.com/file.zip

# 断点续传
wget -c https://example.com/large.iso

# 递归下载网站（谨慎！）
wget -r -p -np -k https://docs.example.com/
```

> 💡 **建议**：  
>
> - 脚本中调用 API → 用 `curl`  
> - 下载大文件或整站 → 用 `wget`

------

#### 🔒 六、安全远程访问（SSH 生态）

##### 1. **`ssh`：远程登录**

```bash
ssh user@192.168.1.100
ssh -p 2222 user@host    # 指定端口
ssh -i ~/.ssh/id_rsa user@host  # 指定私钥
```

##### 2. **`scp`：安全拷贝（已逐渐被 `sftp`/`rsync` 替代）**

```bash
# 上传
scp file.txt user@host:/remote/path/

# 下载
scp user@host:/remote/file.txt ./
```

##### 3. **`sftp`：交互式安全文件传输**

```bash
sftp user@host
sftp> get remote_file .
sftp> put local_file /remote/
```

> ✅ **现代替代方案**：  
>
> - 大量文件同步 → 用 `rsync -e ssh`（增量、高效）  
> - 自动化脚本 → 用 `ssh` + 命令管道，或 `scp`（简单场景）

------

#### 🗺️ 七、路径追踪（`traceroute` vs `mtr`）

##### 1. **`traceroute`**：显示到目标的路由跳数

```bash
traceroute google.com
# 使用 UDP（Linux 默认），也可用 -I（ICMP）或 -T（TCP）
```

##### 2. **`mtr`（My TraceRoute）**：`ping` + `traceroute` 结合

```bash
mtr google.com          # 实时动态显示每跳的延迟和丢包
mtr --report google.com # 生成报告（适合发给网络管理员）
```

> ✅ **优势**：`mtr` 能持续监控，快速定位哪一跳网络不稳定。

------

#### 🛠️ 八、其他重要网络命令（补充）

| 命令               | 用途                                    |
| ------------------ | --------------------------------------- |
| `ipcalc`           | 计算子网、广播地址等（需安装）          |
| `arp` / `ip neigh` | 查看 ARP 缓存（`ip neigh show` 更现代） |
| `tcpdump`          | 抓包分析（高级诊断）                    |
| `nmap`             | 端口扫描、服务探测（安全审计）          |
| `hostnamectl`      | 查看/设置主机名（systemd 系统）         |

------

#### ✅ 最佳实践总结

| 场景         | 推荐命令                  |
| ------------ | ------------------------- |
| 查看 IP 地址 | `ip addr`                 |
| 查看监听端口 | `ss -tuln`                |
| DNS 查询     | `dig +short example.com`  |
| 测试 API     | `curl -v https://...`     |
| 下载大文件   | `wget -c url`             |
| 远程执行命令 | `ssh user@host 'command'` |
| 网络路径诊断 | `mtr --report target`     |

------

#### ⚠️ 安全提醒

1. **避免在公网暴露 `sshd` 默认端口**（改端口 + 密钥登录）。

2. **不要用 `telnet`/`ftp`**（明文传输），始终用 `ssh`/`sftp`/`scp`。

3. `curl`/`wget` 执行远程脚本要极度谨慎

   ：

   ```bash
   # 危险！可能执行恶意代码
   curl https://malicious.site/install.sh | bash
   ```

------

如果你需要：

- 一个 **网络故障排查流程图**
- **`curl` 调用 REST API 的完整示例**
- 如何用 `tcpdump` 抓取 HTTPS 以外的明文流量

欢迎继续提问！



------

### 6. **压缩与归档命令**

用于打包和压缩文件。

- `tar`：打包/解包文件（常配合 gzip/bzip2 使用）  
- `gzip` / `gunzip`：压缩/解压 .gz 文件  
- `bzip2` / `bunzip2`：压缩/解压 .bz2 文件  
- `zip` / `unzip`：处理 .zip 格式  
- `xz` / `unxz`：高压缩率格式



------

#### 📦 一、`tar`：归档（打包）的核心工具

> 💡 `tar` 本身**不压缩**，只把多个文件合并成一个 `.tar` 文件（称为“归档”）。
> 通常配合 `gzip`、`bzip2`、`xz` 实现“打包 + 压缩”。

##### ✅ 基本语法

```bash
tar [选项] [归档文件名] [要打包的文件/目录]
```

##### 🔑 核心选项（必须记住）

| 选项 | 含义                                           |
| ---- | ---------------------------------------------- |
| `-c` | **create** 创建新归档                          |
| `-x` | **extract** 解包                               |
| `-t` | **list** 列出内容                              |
| `-f` | 指定归档文件名（**必须放在最后**）             |
| `-z` | 通过 **gzip** 压缩/解压（`.tar.gz` 或 `.tgz`） |
| `-j` | 通过 **bzip2** 压缩/解压（`.tar.bz2`）         |
| `-J` | 通过 **xz** 压缩/解压（`.tar.xz`）             |
| `-v` | verbose，显示过程                              |

------

##### 🌰 常用示例

###### 1. **创建压缩包**

```bash
# 打包并 gzip 压缩
tar -czvf archive.tar.gz /path/to/dir

# 打包并 xz 压缩（高压缩率）
tar -cJvf archive.tar.xz /path/to/dir
```

###### 2. **解压**

```bash
# 自动识别格式（GNU tar 支持）
tar -xvf archive.tar.gz

# 显式指定
tar -xzvf archive.tar.gz      # gzip
tar -xjvf archive.tar.bz2     # bzip2
tar -xJvf archive.tar.xz      # xz
```

###### 3. **查看内容（不解压）**

```bash
tar -tzvf archive.tar.gz
```

###### 4. **只解压部分文件**

```bash
tar -xzvf archive.tar.gz path/to/file.txt
```

> ✅ **黄金法则**：  
>
> - 创建用 `-c`，解压用 `-x`，查看用 `-t`  
> - 总是加 `-v`（看进度），`-f` 必须在最后

------

#### 🗜️ 二、专用压缩工具对比

| 工具      | 扩展名 | 压缩率 | 速度 | 跨平台              | 命令示例                  |
| --------- | ------ | ------ | ---- | ------------------- | ------------------------- |
| **gzip**  | `.gz`  | 中     | ⚡ 快 | ✅ 广泛支持          | `gzip file` → `file.gz`   |
| **bzip2** | `.bz2` | 高     | 慢   | ✅                   | `bzip2 file`              |
| **xz**    | `.xz`  | 🔝 最高 | 很慢 | ✅（现代系统）       | `xz file`                 |
| **zip**   | `.zip` | 中     | 快   | ✅✅ Windows 原生支持 | `zip -r archive.zip dir/` |

------

##### 1. **gzip / gunzip**

```bash
gzip file.txt          # 压缩 → file.txt.gz（原文件删除）
gunzip file.txt.gz     # 解压 → file.txt
# 或
gzip -d file.txt.gz
```

> 💡 通常不单独用，而是和 `tar` 配合：`tar.gz`

------

##### 2. **bzip2 / bunzip2**

```bash
bzip2 file.txt         # → file.txt.bz2
bunzip2 file.txt.bz2   # → file.txt
```

> ⚠️ 注意：`bzip2` 压缩比 `gzip` 高约 10~~15%，但慢 2~~3 倍。

------

##### 3. **xz / unxz**

```bash
xz file.txt            # → file.txt.xz
unxz file.txt.xz       # → file.txt
# 或控制压缩级别（-0 到 -9，默认 -6）
xz -9 file.txt         # 最高压缩率（更慢）
```

> 💡 适合分发大型软件包（如 Linux 内核源码常用 `.tar.xz`）。

------

##### 4. **zip / unzip**（跨平台首选）

```bash
# 压缩目录
zip -r archive.zip mydir/

# 解压
unzip archive.zip

# 列出内容
unzip -l archive.zip

# 解压到指定目录
unzip archive.zip -d /target/dir
```

> ✅ 优势：Windows/macOS/Linux 通用，支持**单个文件压缩**（无需先 tar）。

------

#### 🔧 三、高级技巧 & 最佳实践

##### 1. **不解压直接查看内容**

```bash
# 查看 gz 文件内容
zcat file.txt.gz
zless file.log.gz

# 查看 zip 内容
unzip -p archive.zip file.txt | head
```

##### 2. **管道压缩（避免中间文件）**

```bash
# 备份数据库并直接压缩
mysqldump db | gzip > backup.sql.gz

# 从网络流解压
curl -s https://example.com/data.tar.gz | tar -xz
```

##### 3. **排除文件打包**

```bash
tar --exclude='*.log' --exclude='.git' -czvf code.tar.gz project/
```

##### 4. **保持权限与所有权（备份用）**

```bash
tar -czvpf backup.tar.gz /important/dir   # -p 保留权限
```

------

#### 📊 四、如何选择压缩格式？

| 场景                    | 推荐格式                    |
| ----------------------- | --------------------------- |
| 日常快速压缩            | `.tar.gz`（平衡速度与体积） |
| 网络传输给 Windows 用户 | `.zip`                      |
| 长期存档、节省空间      | `.tar.xz`                   |
| 脚本日志轮转            | `.gz`（配合 logrotate）     |
| 内核/大型源码分发       | `.tar.xz`                   |

------

#### ⚠️ 五、常见陷阱

1. **`tar` 不加 `-f` 会输出到 stdout**（可能乱屏）：

   ```bash
   tar -czv dir/   # ❌ 错误！会把二进制数据打到终端
   tar -czvf out.tar.gz dir/  # ✅ 正确
   ```

2. **`zip` 默认不压缩目录**，必须加 `-r`。

3. **`gzip`/`bzip2` 只能压缩单个文件**，不能直接压缩目录（需先 `tar`）。

4. **解压时注意路径安全**：
   恶意 tar 包可能包含 `../../../etc/passwd`，建议先 `tar -tvf` 查看内容。

------

#### ✅ 总结速查表

| 操作               | 命令                            |
| ------------------ | ------------------------------- |
| 打包压缩为 .tar.gz | `tar -czvf name.tar.gz dir/`    |
| 解压 .tar.gz       | `tar -xzvf name.tar.gz`         |
| 压缩单个文件为 .gz | `gzip file`                     |
| 创建 .zip          | `zip -r archive.zip dir/`       |
| 解压 .zip          | `unzip archive.zip`             |
| 高压缩归档         | `tar -cJvf archive.tar.xz dir/` |

------

如果你需要：

- 一个 **自动备份脚本模板**（含压缩+日期命名）
- 如何用 `pigz`（多线程 gzip）加速压缩
- **比较不同压缩算法的实际效果**

欢迎继续提问！

------

### 7. **进程管理命令**

用于控制和监控系统进程。

- `ps`：列出进程  
- `kill` / `killall`：终止进程  
- `pkill`：按名称杀进程  
- `nice` / `renice`：调整进程优先级  
- `nohup`：使进程在退出终端后继续运行  
- `jobs` / `fg` / `bg`：管理后台任务

------

#### 🔍 一、`ps`：查看当前进程快照

##### ✅ 基本用法

```bash
ps              # 仅显示当前终端的进程（基本不用）
ps aux          # BSD 风格：显示所有进程（最常用！）
ps -ef          # System V 风格：显示完整格式
```

##### 📊 `ps aux` 输出字段含义：

| 列          | 含义                      |
| ----------- | ------------------------- |
| USER        | 进程所有者                |
| PID         | 进程 ID（唯一标识）       |
| %CPU / %MEM | CPU 和内存占用            |
| VSZ / RSS   | 虚拟内存 / 物理内存（KB） |
| TTY         | 控制终端（? 表示无终端）  |
| STAT        | 进程状态（见下表）        |
| COMMAND     | 启动命令                  |

##### 🧾 进程状态（STAT）常见值：

- `R` = Running（运行中）
- `S` = Interruptible Sleep（可中断睡眠）
- `D` = Uninterruptible Sleep（不可中断，通常 I/O 阻塞）
- `Z` = Zombie（僵尸进程）
- `T` = Stopped（被暂停）
- `<` = 高优先级
- `N` = 低优先级
- `+` = 前台进程组

##### 💡 实用技巧

```bash
# 查找特定进程
ps aux | grep nginx

# 按 CPU 使用率排序
ps aux --sort=-%cpu | head

# 只显示某用户的进程
ps -u username
```

> ⚠️ 注意：`ps` 是**瞬时快照**，如需动态监控，请用 `top` 或 `htop`。

------

#### 🛑 二、`kill` / `killall` / `pkill`：终止进程

##### 1. **`kill`：通过 PID 发送信号**

```bash
kill 1234        # 默认发送 SIGTERM（优雅终止）
kill -9 1234     # 强制终止（SIGKILL，无法被捕获）
kill -l          # 列出所有信号名称
```

> ✅ **最佳实践**：先尝试 `SIGTERM`（`kill PID`），无效再用 `SIGKILL`（`kill -9 PID`）。

------

##### 2. **`killall`：通过进程名终止（注意：行为因系统而异）**

```bash
killall firefox      # 终止所有名为 firefox 的进程
killall -9 nginx     # 强制终止
```

> ⚠️ **警告**：在 Debian/Ubuntu 中，`killall` 真正按名字杀；但在 RHEL/CentOS 中，`killall` 会**杀死所有进程**（危险！）。
> **替代方案**：优先用 `pkill`（更安全、跨平台一致）。

------

##### 3. **`pkill`：按名字/用户/其他属性杀进程（推荐！）**

```bash
pkill firefox                # 杀 firefox
pkill -u alice               # 杀用户 alice 的所有进程
pkill -f "python script.py"  # 按完整命令行匹配（-f）
pkill -9 -f myapp            # 强制杀包含 "myapp" 的进程
```

> ✅ `pkill` 支持正则表达式，且行为一致，**比 `killall` 更可靠**。

------

#### ⚖️ 三、`nice` / `renice`：调整进程优先级

Linux 使用 **Nice 值**（-20 ~ +19）表示优先级：

- **数值越小，优先级越高**（-20 最高，+19 最低）
- 普通用户只能**降低**优先级（即增加 nice 值）

##### ✅ 用法示例

```bash
# 启动时设置低优先级（后台任务）
nice -n 10 ./heavy_task.sh

# 修改已运行进程的优先级（PID=1234）
renice -n 5 -p 1234

# 修改某用户所有进程的优先级
renice -n 10 -u alice
```

> 💡 适用场景：让备份、编译等任务不干扰前台操作。

------

#### 🌙 四、`nohup`：让进程在退出终端后继续运行

##### ✅ 基本用法

```bash
nohup ./server.sh &
```

- `nohup` 忽略 **SIGHUP 信号**（终端关闭时发送）
- 默认输出重定向到 `nohup.out`
- 结合 `&` 放入后台

##### 🔧 更完整的写法（推荐）

```bash
nohup ./app > app.log 2>&1 &
echo $! > app.pid   # 保存 PID 便于后续管理
```

> ⚠️ 注意：`nohup` 不能防止进程被 `kill`，只是避免终端退出导致终止。

------

#### 🔄 五、`jobs` / `fg` / `bg`：Shell 内部作业控制

这些命令管理**当前 Shell 启动的后台任务**（不是系统级进程）。

##### ✅ 基本流程

```bash
# 1. 启动一个长时间任务
sleep 1000

# 2. 按 Ctrl+Z 暂停 → 进入后台（Stopped）
^Z
[1]+  Stopped    sleep 1000

# 3. 查看作业
jobs
# [1]+ Stopped    sleep 1000

# 4. 放入后台继续运行
bg %1

# 5. 切回前台
fg %1

# 6. 直接启动后台任务
sleep 2000 &
```

##### 📌 符号说明

- `%1`, `%2`：作业编号（由 `jobs` 显示）
- `%+` 或 `%%`：最近一个作业
- `%-`：倒数第二个作业

> 💡 适用场景：临时切换任务、调试时暂停程序。

------

#### 🧩 六、补充：其他重要进程命令

| 命令           | 用途                                     |
| -------------- | ---------------------------------------- |
| `top` / `htop` | 实时动态监控进程（`htop` 更友好）        |
| `pgrep`        | 按名字查 PID（`pgrep nginx`）            |
| `systemctl`    | 管理 systemd 服务（现代系统首选）        |
| `lsof`         | 查看进程打开的文件/端口（`lsof -i :80`） |

------

#### ✅ 最佳实践总结

| 场景               | 推荐命令                                   |
| ------------------ | ------------------------------------------ |
| 查看所有进程       | `ps aux` 或 `htop`                         |
| 终止进程           | 优先 `pkill -f name`，其次 `kill PID`      |
| 后台长期运行       | `nohup cmd > log 2>&1 &`                   |
| 临时挂起任务       | `Ctrl+Z` → `bg` / `fg`                     |
| 降低任务优先级     | `nice -n 10 task`                          |
| 查找占用端口的进程 | `lsof -i :8080` 或 `ss -tulnp | grep 8080` |

------

#### ⚠️ 安全提醒

1. **不要随意 `kill -9`**：可能导致数据丢失（如数据库）。
2. **僵尸进程（Z）无需处理**：它已死亡，只是父进程未回收，通常无害。
3. **D 状态进程（不可中断）**：通常是磁盘 I/O 卡住，**无法 kill**，需修复硬件或等待。

------

如果你需要：

- 一个 **进程监控脚本模板**
- 如何用 `systemctl` 替代 `nohup`（更现代的方式）
- **自动重启崩溃进程** 的方案（如用 `supervisor`）

欢迎继续提问！

------

### 8. **软件包管理命令**

不同发行版使用不同的包管理器。

- **Debian/Ubuntu**：
  - `apt` / `apt-get`：安装、更新、删除软件包  
  - `dpkg`：底层 .deb 包管理
- **RHEL/CentOS/Fedora**：
  - `yum`（旧） / `dnf`（新）：包管理  
  - `rpm`：底层 .rpm 包管理
- **Arch Linux**：
  - `pacman`
- **通用**：
  - `snap` / `flatpak`：跨发行版软件包格式



------

#### 📦 一、Debian/Ubuntu 系列（.deb 包）

##### 1. **`apt`（推荐） vs `apt-get`**

- `apt` 是面向用户的**友好版**（带进度条、彩色输出），适合交互式使用。
- `apt-get` 更稳定、脚本兼容性更好，适合自动化脚本。

###### ✅ 常用命令对比

| 功能                 | `apt`                    | `apt-get`                    |
| -------------------- | ------------------------ | ---------------------------- |
| 更新软件源列表       | `sudo apt update`        | `sudo apt-get update`        |
| 升级已安装软件       | `sudo apt upgrade`       | `sudo apt-get upgrade`       |
| 安装软件             | `sudo apt install nginx` | `sudo apt-get install nginx` |
| 删除软件（保留配置） | `sudo apt remove nginx`  | `sudo apt-get remove nginx`  |
| 彻底删除（含配置）   | `sudo apt purge nginx`   | `sudo apt-get purge nginx`   |
| 自动清理无用依赖     | `sudo apt autoremove`    | `sudo apt-get autoremove`    |
| 搜索软件包           | `apt search keyword`     | ❌（需用 `apt-cache search`） |
| 查看包信息           | `apt show package`       | `apt-cache show package`     |

> 💡 **建议**：日常使用 `apt`，脚本中用 `apt-get`（行为更可预测）。

------

##### 2. **`dpkg`：底层 .deb 包操作**

用于直接安装/查询 `.deb` 文件（不处理依赖）。

```bash
# 安装本地 .deb 包
sudo dpkg -i package.deb

# 列出所有已安装包
dpkg -l

# 查询某文件属于哪个包
dpkg -S /usr/bin/vim

# 卸载包（不自动解决依赖）
sudo dpkg -r package_name

# 修复依赖问题（常在 dpkg 安装失败后使用）
sudo apt install -f
```

> ⚠️ 注意：`dpkg` 不会自动安装依赖，通常配合 `apt install -f` 使用。

------

#### 📦 二、RHEL/CentOS/Fedora 系列（.rpm 包）

##### 1. **`dnf`（Fedora ≥22, RHEL/CentOS 8+） vs `yum`（旧版）**

- `dnf` 是 `yum` 的下一代替代品（更快、内存占用更低）。
- 在 CentOS 7 及更早版本中仍用 `yum`。

###### ✅ 常用命令（`dnf` / `yum` 通用）

```bash
# 更新软件源缓存
sudo dnf check-update      # 或 yum check-update

# 安装软件
sudo dnf install nginx     # yum install nginx

# 卸载软件
sudo dnf remove nginx

# 搜索软件包
dnf search keyword

# 列出已安装包
dnf list installed

# 查看包信息
dnf info package_name

# 清理缓存
sudo dnf clean all
```

> 💡 在 RHEL/CentOS 8+ 中，`yum` 实际是 `dnf` 的软链接，命令完全兼容。

------

###### 2. **`rpm`：底层 .rpm 包操作**

类似 `dpkg`，直接操作 `.rpm` 文件，**不解决依赖**。

```bash
# 安装本地 rpm 包
sudo rpm -ivh package.rpm

# 查询已安装包
rpm -qa

# 查询某文件属于哪个包
rpm -qf /usr/bin/python3

# 卸载包
sudo rpm -e package_name

# 验证包完整性
rpm -V package_name
```

> ⚠️ 同样需注意：`rpm` 不处理依赖，建议优先用 `dnf`/`yum`。

------

#### 📦 三、Arch Linux：`pacman`

Arch 的包管理器简洁强大，**同时处理安装、升级、依赖和仓库同步**。

##### ✅ 核心命令

```bash
# 同步仓库并升级系统（Arch 必做！）
sudo pacman -Syu

# 安装软件
sudo pacman -S firefox

# 搜索软件包
pacman -Ss keyword

# 查看包信息
pacman -Si package_name

# 卸载软件（保留依赖）
sudo pacman -R package_name

# 卸载 + 删除无用依赖
sudo pacman -Rs package_name

# 清理未使用的包缓存
sudo pacman -Sc
```

> 💡 Arch 用户需养成定期 `pacman -Syu` 的习惯（滚动更新）。

------

#### 🌐 四、跨发行版通用包格式

##### 1. **Snap（由 Canonical 推出）**

- 应用自带依赖，沙箱运行。
- 适用于 Ubuntu、Debian、Fedora、Arch 等。

```bash
# 安装 snapd（如未安装）
sudo apt install snapd        # Debian/Ubuntu
sudo dnf install snapd        # Fedora

# 安装应用
sudo snap install code --classic

# 列出已安装 snap
snap list

# 更新所有 snap
sudo snap refresh

# 卸载
sudo snap remove code
```

> ⚠️ 缺点：启动稍慢、磁盘占用大；优点：版本新、无需编译。

------

##### 2. **Flatpak**

- 类似 Snap，但权限模型更精细，由社区驱动。
- 需先添加仓库（如 Flathub）。

```bash
# 安装 Flatpak（多数现代发行版预装）
sudo apt install flatpak      # Debian/Ubuntu

# 添加 Flathub 仓库
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# 安装应用
flatpak install flathub org.gimp.GIMP

# 运行
flatpak run org.gimp.GIMP

# 更新
flatpak update
```

> 💡 GNOME 软件中心默认支持 Flatpak，体验更集成。

------

#### 🔍 五、如何选择包管理方式？

| 场景                                  | 推荐方式                                                 |
| ------------------------------------- | -------------------------------------------------------- |
| 系统核心软件（如 nginx, python）      | 使用发行版原生包管理器（`apt`/`dnf`/`pacman`）           |
| 最新版桌面应用（如 VS Code, Discord） | `snap` 或 `flatpak`                                      |
| 需要自定义编译选项                    | 从源码编译（`./configure && make && sudo make install`） |
| 企业环境稳定性要求高                  | 优先使用官方仓库，避免第三方源                           |

------

#### ⚠️ 六、重要注意事项

1. **不要混用包管理器**
   例如：用 `apt` 安装了 `nginx`，就不要再用 `snap install nginx`，否则端口冲突、配置混乱。

2. **第三方仓库谨慎添加**
   如 Ubuntu 的 `ppa`、CentOS 的 `EPEL`，确保来源可信。

3. **定期清理缓存**  

   ```bash
   # Debian/Ubuntu
   sudo apt clean
   # Fedora
   sudo dnf clean all
   ```

4. **查看软件来源**  

   ```bash
   apt policy nginx      # Debian/Ubuntu
   dnf info nginx        # RHEL/Fedora
   ```

------

#### ✅ 总结速查表

| 发行版             | 高层命令      | 底层命令 | 跨平台方案    |
| ------------------ | ------------- | -------- | ------------- |
| Debian/Ubuntu      | `apt`         | `dpkg`   | Snap, Flatpak |
| RHEL/CentOS/Fedora | `dnf` / `yum` | `rpm`    | Snap, Flatpak |
| Arch Linux         | `pacman`      | —        | Snap, Flatpak |

------

如果你告诉我你使用的具体 Linux 发行版（如 Ubuntu 22.04、CentOS 7 等），我可以提供**针对性的包管理最佳实践**或**常见软件安装命令示例**（如 Docker、Node.js、Python 环境等）。欢迎继续提问！

------

### 9. **定时任务与计划命令**

用于自动化任务调度。

- `crontab`：设置周期性任务  
- `at`：执行一次性计划任务

------

#### ✅ 一、`crontab`：周期性任务（重复执行）

##### 🔧 1. 基本概念

- **Cron** 是 Linux 的守护进程（`crond`），用于按**固定时间间隔**自动执行任务。
- 每个用户都有自己的 **crontab 文件**（不直接编辑 `/etc/crontab`，除非是系统级任务）。

##### 📝 2. 基本命令

```bash
crontab -e    # 编辑当前用户的定时任务
crontab -l    # 列出当前用户的定时任务
crontab -r    # 删除当前用户的所有定时任务（慎用！）
```

> 💡 系统级任务可放在：
>
> - `/etc/crontab`（需指定用户）
> - `/etc/cron.d/`（推荐，模块化）
> - `/etc/cron.hourly/`, `/etc/cron.daily/` 等（按周期目录）

------

##### 🕒 3. Cron 时间格式（5 个星号）

```text
# ┌───────────── 分钟 (0 - 59)
# │ ┌──────────── 小时 (0 - 23)
# │ │ ┌───────────── 日 (1 - 31)
# │ │ │ ┌────────────── 月 (1 - 12)
# │ │ │ │ ┌─────────────── 星期 (0 - 6, 0=周日)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command_to_run
```

###### 🌰 示例

| 表达式                  | 含义                        |
| ----------------------- | --------------------------- |
| `0 2 * * * /backup.sh`  | 每天凌晨 2 点执行           |
| `*/5 * * * * /check.sh` | 每 5 分钟执行一次           |
| `0 0 1 * * /monthly.sh` | 每月 1 号 0 点执行          |
| `0 0 * * 0 /weekly.sh`  | 每周日 0 点执行             |
| `30 8 1,15 * * /pay.sh` | 每月 1 号和 15 号 8:30 执行 |

> ✅ 提示：可用在线工具 [crontab.guru](https://crontab.guru/) 验证表达式。

------

##### ⚠️ 4. 注意事项（常见坑！）

1. **环境变量问题**
   Cron 的环境非常精简（`PATH` 可能只有 `/usr/bin:/bin`），建议：

   - 使用**绝对路径**：`/usr/bin/python3 /opt/script.py`

   - 或在 crontab 开头定义环境：

     ```bash
     SHELL=/bin/bash
     PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
     ```

2. **输出处理**
   默认会邮件发送给用户（若配置了 mail）。通常重定向到日志或丢弃：

   ```bash
   0 2 * * * /backup.sh >> /var/log/backup.log 2>&1
   # 或静默
   */10 * * * * /check.sh >/dev/null 2>&1
   ```

3. **% 符号转义**
   `%` 在 cron 中有特殊含义（表示换行），如需使用需转义为 `\%`：

   ```bash
   # 错误 ❌
   0 0 * * * echo "Today is $(date +%Y-%m-%d)"
   # 正确 ✅
   0 0 * * * echo "Today is $(date +\%Y-\%m-\%d)"
   ```

------

#### ✅ 二、`at`：一次性任务（只执行一次）

##### 🔧 1. 基本概念

- 用于**在指定时间点执行一次任务**（如“今晚 11 点重启服务”）。
- 依赖 `atd` 守护进程（需确保已安装并启动）。

###### 📦 安装（如未安装）

```bash
# Ubuntu/Debian
sudo apt install at

# CentOS/RHEL
sudo yum install at

# 启动服务
sudo systemctl enable --now atd
```

------

##### 🕒 2. 基本用法

###### 方式一：交互式输入

```bash
$ at 23:00
warning: commands will be executed using /bin/sh
at> /restart-app.sh
at> <EOT>          # 按 Ctrl+D 结束输入
```

###### 方式二：从文件或管道读取

```bash
echo "/cleanup.sh" | at now + 5 minutes
at -f /path/to/script.sh 10:30 tomorrow
```

###### 🕐 时间格式示例

| 表达式            | 含义       |
| ----------------- | ---------- |
| `now + 5 minutes` | 5 分钟后   |
| `10:30 PM`        | 今晚 10:30 |
| `tomorrow`        | 明天此时   |
| `next week`       | 下周此时   |
| `noon + 1 hour`   | 中午 1 点  |

------

##### 🔍 3. 管理 `at` 任务

```bash
atq          # 查看待执行任务队列
atrm <job_id> # 删除任务（job_id 来自 atq 输出）
```

------

##### ⚠️ 4. 注意事项

1. **脚本环境同样受限** → 使用绝对路径！

2. 默认使用 `/bin/sh`

    → 若脚本是 Bash 特有语法，需显式调用：

   ```bash
   echo "/bin/bash /my-script.sh" | at now + 1 hour
   ```

3. 输出默认通过 mail 发送

    → 可重定向：

   ```bash
   echo "/task.sh >> /tmp/at.log 2>&1" | at now + 10 min
   ```

------

#### 🔁 三、`cron` vs `at` 对比总结

| 特性         | `cron`                   | `at`                                  |
| ------------ | ------------------------ | ------------------------------------- |
| **执行次数** | 周期性（重复）           | 一次性                                |
| **适用场景** | 日常备份、日志轮转、监控 | 临时任务、延迟操作                    |
| **配置方式** | `crontab -e`             | `at` 命令交互或管道                   |
| **时间精度** | 最小 1 分钟              | 支持秒级延迟（如 `now + 30 seconds`） |
| **依赖服务** | `crond`                  | `atd`                                 |

------

#### 🛠️ 四、高级替代方案（现代推荐）

对于复杂任务，可考虑：

| 工具                                           | 优势                                       |
| ---------------------------------------------- | ------------------------------------------ |
| **systemd timer**                              | 更强大、集成日志、依赖管理（适合系统服务） |
| **Python APScheduler** / **Node.js node-cron** | 应用内调度                                 |
| **Airflow** / **Celery Beat**                  | 分布式任务调度（大数据/企业级）            |

但对大多数运维场景，`cron` + `at` 依然简洁高效。

------

#### ✅ 最佳实践建议

1. **测试 cron 任务**：先用 `* * * * *` 每分钟执行，验证后再改正式时间。

2. **记录日志**：始终重定向 stdout/stderr 到日志文件。

3. **避免长任务阻塞**：cron 不会并发执行同一任务（除非用锁机制）。

4. 使用 `flock` 防止重复运行

   ：

   ```bash
   */10 * * * * flock -n /tmp/mylock.lock /my-script.sh
   ```

------

需要我提供：

- 一个 **完整的备份脚本 + cron 配置示例**
- **systemd timer 的入门指南**
- 如何用 `at` 实现“5 分钟后自动关机”？

欢迎继续提问！

------

### 10. **Shell 与脚本相关命令**

用于编写和运行 Shell 脚本。

- `echo`：输出文本  
- `read`：读取用户输入  
- `test` / `[ ]`：条件判断  
- `source` / `.`：在当前 shell 中执行脚本  
- `export`：设置环境变量

------

#### 1. `echo`：输出文本

##### ✅ 功能

向终端或文件输出字符串（常用于调试、提示信息）。

##### 🔧 基本用法

```bash
echo "Hello, World!"
echo -e "Line1\nLine2"   # -e 启用转义字符（如 \n, \t）
echo -n "No newline"     # -n 不换行
```

##### ⚠️ 注意

- 在脚本中，若需兼容 POSIX，避免使用 `-e`/`-n`（不同系统行为不一）。

- 更可移植的方式是使用 

  ```
  printf
  ```

  ：

  ```bash
  printf "Name: %s\n" "Alice"
  ```

------

#### 2. `read`：读取用户输入

##### ✅ 功能

从标准输入（通常是键盘）读取一行，并赋值给变量。

##### 🔧 基本用法

```bash
read name
echo "You entered: $name"

# 带提示（Bash 特有）
read -p "Enter your name: " name

# 静默输入（如密码）
read -s -p "Password: " passwd
echo  # 换行
```

##### 🛡️ 安全建议

- 使用 

  ```
  -r
  ```

   选项防止反斜杠转义（推荐总是加）：

  ```bash
  read -r line   # 避免 \n 被解释为换行
  ```

------

#### 3. `test` / `[ ]`：条件判断

##### ✅ 功能

测试文件属性、字符串、数值等条件，返回退出状态（0 = true, 非0 = false）。

##### 🔧 等价写法

```bash
test -f file.txt
[ -f file.txt ]          # 注意：[ 是 test 的别名，两边必须有空格！
[[ -f file.txt ]]        # Bash 扩展（更强大，支持正则、逻辑组合）
```

##### 📌 常见测试条件

| 类型         | 示例                   | 说明                                  |
| ------------ | ---------------------- | ------------------------------------- |
| **文件测试** | `[ -f file ]`          | 是否为普通文件                        |
|              | `[ -d dir ]`           | 是否为目录                            |
|              | `[ -e path ]`          | 路径是否存在                          |
|              | `[ -r file ]`          | 是否可读                              |
| **字符串**   | `[ "$str" = "hello" ]` | 字符串相等（注意引号防空）            |
|              | `[ -z "$str" ]`        | 字符串是否为空                        |
| **数值**     | `[ $a -eq $b ]`        | 相等（`-eq`, `-ne`, `-lt`, `-gt` 等） |

##### ⚠️ 重要规则

- **变量务必加双引号**：`[ "$var" = "value" ]`，避免空值导致语法错误。
- `[` 和 `]` **前后必须有空格**！

------

#### 4. `source` / `.`：在当前 Shell 中执行脚本

##### ✅ 功能

**不创建子进程**，直接在当前 Shell 环境中运行脚本，因此可以：

- 修改当前 Shell 的变量
- 改变当前工作目录（`cd` 生效）
- 加载配置文件（如 `.bashrc`）

##### 🔧 用法对比

```bash
source ./config.sh    # Bash/Zsh 推荐写法
. ./config.sh         # POSIX 标准写法（所有 Shell 支持）
```

##### 🌰 典型场景

```bash
# config.sh
export API_KEY="secret123"
cd /project

# main.sh
source config.sh
echo $API_KEY   # 能打印出 secret123
pwd             # 当前目录已变为 /project
```

> ❌ 如果用 `./config.sh`，则变量和目录变更只在子进程中生效，主脚本不受影响。

------

#### 5. `export`：设置环境变量

##### ✅ 功能

将 Shell 变量**导出为环境变量**，使其对**子进程可见**。

##### 🔧 用法

```bash
MY_VAR="local"        # 普通变量（仅当前 Shell 可见）
export MY_VAR         # 导出为环境变量
export PATH="$PATH:/new/bin"  # 追加路径

# 一行写法
export DB_HOST=localhost
```

#### 🌐 环境变量 vs 普通变量

| 类型     | 子进程能否访问？ | 示例              |
| -------- | ---------------- | ----------------- |
| 普通变量 | ❌                | `msg="hi"`        |
| 环境变量 | ✅                | `export msg="hi"` |

#### 💡 查看环境变量

```bash
env        # 列出所有环境变量
printenv PATH
```

------

#### 🧩 补充：其他重要的 Shell 脚本相关命令

| 命令      | 用途                                     |
| --------- | ---------------------------------------- |
| `set`     | 设置 Shell 选项（如 `set -e`：遇错退出） |
| `shift`   | 移动位置参数（`$1←$2, $2←$3...`）        |
| `getopts` | 解析脚本命令行选项（如 `-f file`）       |
| `return`  | 从函数返回（不能用于主脚本）             |
| `exit`    | 退出整个脚本（可指定退出码）             |

------

#### ✅ 最佳实践建议

1. 脚本开头加 Shebang：

   ```bash
   #!/bin/bash
   ```

2. 用 `set -euo pipefail` 提高脚本健壮性

   ：

   - `-e`：任何命令失败则退出
   - `-u`：使用未定义变量时报错
   - `-o pipefail`：管道中任一命令失败则整体失败

3. **变量始终加引号**：`"$var"` 而不是 `$var`

4. **优先用 `[[ ]]` 而非 `[ ]`（Bash 脚本中）**：支持模式匹配、逻辑组合更安全

------

如果你正在编写 Shell 脚本，我可以提供：

- 完整的 **Shell 脚本模板**
- **参数解析**（`getopts` 示例）
- **日志记录**、**错误处理** 的通用函数

欢迎继续深入探讨！

------

### 补充说明

- 很多命令支持 **选项（options）** 和 **参数（arguments）**，可通过 `man command` 查看手册。

- 使用 `--help` 可快速查看命令用法（如 `ls --help`）。

- 组合命令（管道 

  ```
  |
  ```

  、重定向 

  ```
  >
  ```

  ```
  <
  ```

  ）可实现强大功能，例如：  

  ```bash
  ps aux | grep nginx | wc -l
  ```

------

### Linux / Shell 中调用脚本

在 Windows 的 `cmd.exe` 中，**`call` 命令**主要用于：

1. **从一个批处理文件（.bat）中调用另一个批处理文件**，并**在被调用脚本执行完后返回到原脚本继续执行**（而不是终止当前脚本）。
2. 调用带有标签（label）的子程序（较少用）。

Linux 的 Shell（如 Bash、Zsh）**不需要专门的 `call` 命令**，因为它的脚本调用行为**天然就支持“调用后返回”**。但根据使用场景，有以下几种对应方式：

------

#### 场景 1：**在一个脚本中调用另一个脚本，并返回继续执行**

##### ✅ 直接执行（最常用）

```bash
#!/bin/bash
echo "Before calling"
./other_script.sh      # ← 相当于 Windows 的 call other_script.bat
echo "After calling"
```

> 🔍 **说明**：
>
> - 在 Shell 中，直接运行 `./script.sh` 就会在**子 Shell 中执行**，执行完自动返回当前脚本。
> - 这与 Windows 中 **`call script.bat`** 的行为完全一致。
> - 如果不加 `call` 而直接写 `script.bat`，Windows 会**终止当前脚本**，而 Linux 不会——这是关键区别。

✅ 所以：**Linux 中直接写脚本名 = Windows 中的 `call`**

------

#### 场景 2：**在当前 Shell 环境中执行另一个脚本（类似“包含”）**

有时你希望被调用的脚本能**修改当前 Shell 的环境**（如设置变量、改变目录），这时不能用子进程。

##### ✅ 使用 `source` 或 `.`（点命令）

```bash
#!/bin/bash
echo "Before sourcing"
source ./config.sh     # 或 . ./config.sh
echo "After sourcing, VAR=$VAR"
```

> 🔍 **说明**：
>
> - `source` 和 `.` 是 **Shell 内置命令**，会在**当前 Shell 进程中执行脚本**（不是子进程）。
> - 被调用脚本中的 `cd`、`export VAR=xxx` 等操作会影响当前脚本。
> - 这类似于 Windows 中 `call` 的某些高级用法（但 Windows 无法直接继承变量，除非用特殊技巧）。

| 功能               | Windows (CMD)     | Linux (Bash)                        |
| ------------------ | ----------------- | ----------------------------------- |
| 调用子脚本并返回   | `call script.bat` | `./script.sh`                       |
| 在当前环境执行脚本 | ❌（难以实现）     | `source script.sh` 或 `. script.sh` |

------

#### 场景 3：**调用函数（替代批处理中的标签子程序）**

Windows 批处理可用 `call :label` 调用内部子程序。
Linux Shell 使用 **函数（function）** 实现类似功能：

```bash
#!/bin/bash

my_func() {
    echo "Inside function"
    return 0
}

echo "Before function"
my_func          # ← 直接调用函数（无需 call）
echo "After function"
```

> ✅ 函数是 Shell 原生支持的，比 Windows 的标签更清晰、可传参、可返回值。

------

#### ⚠️ 重要区别总结

| 行为              | Windows CMD                      | Linux Shell                                |
| ----------------- | -------------------------------- | ------------------------------------------ |
| `script.bat`      | **终止当前脚本**，转去执行新脚本 | **在子 Shell 中运行**，结束后**返回**      |
| `call script.bat` | 在当前上下文中调用，结束后返回   | **不需要 `call`**，直接 `./script.sh` 即可 |
| 修改父环境变量    | 非常困难（子进程无法影响父进程） | 用 `source` 可以                           |

> 💡 因此，**Linux 的默认行为更接近 Windows 的 `call`**，而 Windows 默认行为反而需要 `call` 来“修复”。

------

#### ✅ 最佳实践建议

1. **普通调用其他脚本** → 直接写 `./script.sh`
2. **需要继承变量/目录变更** → 用 `source script.sh`
3. **模块化代码** → 使用 **函数** 而不是外部小脚本

------

#### 🧪 示例对比

##### Windows (`main.bat`)

```bat
@echo off
echo Before
call helper.bat
echo After
```

##### Linux (`main.sh`)

```bash
#!/bin/bash
echo "Before"
./helper.sh    # ← 这就是 call 的等价写法！
echo "After"
```

两者行为完全一致。

------

如有需要，我也可以提供：

- 如何将 `.bat` 脚本迁移到 `.sh`
- `source` 与直接执行的详细对比表
- 函数 vs 外部脚本的性能与适用场景分析

欢迎继续提问！