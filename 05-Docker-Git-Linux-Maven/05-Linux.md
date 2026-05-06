# Linux

## **Linux / Docker 命令的标准语法规则**

### 核心原因

- **`-v` 是短参数** → 只用 **一个横杠 `-`**
- **`--mount` 是长参数** → 必须用 **两个横杠 `--`**

---

### 1. 短参数：-v

- 格式：**`-` + 单个字母**
- 例子：`-v`、`-d`、`-p`、`-i`、`-t`
- 特点：**简写、快速、方便敲**
```bash
-v  # volume 缩写
-d  # detach 缩写
-p  # port 缩写
```

---

### 2. 长参数：--mount

- 格式：**`--` + 完整单词**
- 例子：`--mount`、`--name`、`--help`、`--version`
- 特点：**语义清晰、不容易写错**
```bash
--mount   # 完整单词
--name    # 完整单词
--help    # 完整单词
```

---

### 3. 它们功能完全一样！

只是**写法不同**，不是功能不同。

```bash
-v my-vol:/data           # 短参数（一个横杠）
--mount src=my-vol,dst=/data  # 长参数（两个横杠）
```

**效果 100% 相同**。

---

### 4. 超级好记的规则

- **一个字母 → 一个横杠 `-`**
  `-v` `-d` `-p`

- **多个字母/单词 → 两个横杠 `--`**
  `--mount` `--name` `--help`

---

### 总结

- `-v` = 短参数 → **`-`**
- `--mount` = 长参数 → **`--`**
- 功能一样，只是写法规范不同！

##  一、Linux 基础与常用命令

1. Linux 文件系统的层次结构（/bin、/etc、/usr、/var 等）分别有什么作用？
2. 如何查看当前目录、切换目录、返回上一级？
3. `ls` 命令有哪些常用参数？如何按时间或大小排序文件？
4. 如何查看当前路径？如何查找某个文件？
5. 如何查看文件内容（`cat`、`more`、`less`、`tail`、`head`）？
6. 如何复制、移动、删除文件或目录？
7. 如何创建、删除用户和用户组？
8. 如何查看系统中的环境变量？如何临时与永久设置环境变量？
9. 如何查看系统时间与时区？如何修改？
10. 如何查看磁盘空间与目录占用情况？（`df -h`、`du -sh`）



------

### **1. Linux 文件系统的层次结构（/bin、/etc、/usr、/var 等）分别有什么作用？**

| 目录             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| `/`              | 根目录，所有文件的起点。                                     |
| `/bin`           | 存放基本命令（如 `ls`、`cp`、`mv`、`cat`），单用户模式下也可用。 |
| `/sbin`          | 系统管理命令（如 `ifconfig`、`reboot`）。                    |
| `/etc`           | 系统配置文件目录（如 `passwd`、`hosts`、`profile`）。        |
| `/home`          | 普通用户的家目录。                                           |
| `/root`          | 超级用户（root）的家目录。                                   |
| `/usr`           | 用户安装的软件与共享资源，类似 Windows 的 Program Files。    |
| `/var`           | 存放动态数据，如日志文件（`/var/log`）、缓存、邮件等。       |
| `/tmp`           | 临时文件存放目录（系统重启后会清空）。                       |
| `/lib`           | 系统共享库文件（类似于 Windows 的 DLL）。                    |
| `/dev`           | 设备文件目录（硬盘、U盘、终端等）。                          |
| `/mnt`、`/media` | 临时挂载点。                                                 |

------

### **2. 如何查看当前目录、切换目录、返回上一级？**

```bash
pwd                # 查看当前目录
cd /home/user      # 切换到指定目录
cd ..              # 返回上一级目录
cd -               # 返回上一个访问的目录
cd ~               # 回到当前用户的 home 目录
```

> ✅ 小技巧：`cd -` 会快速在两个目录间来回切换，非常常用。

------

### **3. `ls` 命令有哪些常用参数？如何按时间或大小排序文件？**

```bash
ls -l        # 详细列表显示
ls -a        # 显示所有文件（包含隐藏文件）
ls -lh       # 以人类可读方式显示文件大小（KB、MB）
ls -lt       # 按修改时间排序
ls -lS       # 按文件大小排序
```

> 常用组合：`ls -lhtr` → 按时间升序排列，显示详细信息。

------

### **4. 如何查看当前路径？如何查找某个文件？**

```bash
pwd                              # 当前路径
find / -name "app.log"           # 从根目录开始查找文件
find /var/log -type f -name "*.log"   # 查找所有日志文件
locate filename                  # 使用数据库快速查找（需运行过 updatedb）
which java                       # 查看命令的绝对路径
whereis java                     # 查找命令、源代码和帮助文档
```

> ⚠️ `find` 实时查找最精确，`locate` 查询快但依赖数据库。

------

### **5. 如何查看文件内容（`cat`、`more`、`less`、`tail`、`head`）？**

| 命令              | 用法                 | 说明                   |
| ----------------- | -------------------- | ---------------------- |
| `cat file`        | 显示全部内容         | 小文件快速查看         |
| `more file`       | 分页显示             | 按空格翻页             |
| `less file`       | 分页显示             | 可上下滚动，支持搜索   |
| `head -n 10 file` | 显示前 10 行         |                        |
| `tail -n 10 file` | 显示最后 10 行       |                        |
| `tail -f file`    | 实时查看日志追加内容 | 常用于查看应用日志输出 |

------

### **6. 如何复制、移动、删除文件或目录？**

```bash
cp source.txt dest.txt              # 复制文件
cp -r dir1 dir2                     # 复制目录
mv oldname newname                  # 重命名或移动文件
mv file /home/user/                 # 移动文件
rm file                             # 删除文件
rm -r dir                           # 删除目录
rm -rf dir                          # 强制删除目录（⚠️ 慎用）
```

> 🚨 注意：`rm -rf /` 会删除整个系统。

------

### **7. 如何创建、删除用户和用户组？**

```bash
useradd username              # 创建用户
passwd username               # 设置密码
userdel username              # 删除用户
groupadd groupname            # 创建用户组
groupdel groupname            # 删除用户组
usermod -aG groupname user    # 将用户添加到组
id username                   # 查看用户信息
whoami                        # 当前登录用户
```

------

### **8. 如何查看系统中的环境变量？如何临时与永久设置环境变量？**

```bash
env                           # 查看所有环境变量
echo $PATH                    # 查看 PATH
export JAVA_HOME=/usr/java/jdk1.8   # 临时设置变量（当前会话有效）
echo 'export JAVA_HOME=/usr/java/jdk1.8' >> ~/.bashrc
source ~/.bashrc              # 永久生效（追加到用户配置文件）
```

> 系统级配置：`/etc/profile`、`/etc/environment`
>  用户级配置：`~/.bashrc`、`~/.bash_profile`

------

### **9. 如何查看系统时间与时区？如何修改？**

```bash
date                          # 查看当前时间
timedatectl                   # 查看时间、时区、NTP状态
timedatectl list-timezones    # 查看可用时区
timedatectl set-timezone Asia/Shanghai   # 修改时区
date -s "2025-10-17 15:00:00" # 手动设置时间
hwclock -w                    # 同步系统时间到硬件时钟
```

------

### **10. 如何查看磁盘空间与目录占用情况？**

```bash
df -h                         # 查看磁盘空间使用情况（按挂载点）
du -sh *                      # 查看当前目录下每个文件夹大小
du -sh /var/log               # 查看某个目录的占用大小
```

> 常用组合：
>  `du -h --max-depth=1 /home` → 按层级查看各目录大小

------

##  二、文件与权限管理

1. Linux 权限的三组 rwx 分别代表什么？
2. 如何修改文件或目录的权限？（`chmod`、`chown`、`chgrp`）
3. 什么是 umask？如何影响新建文件的默认权限？
4. 如何查看文件的 inode 信息？
5. `ln` 命令的软链接与硬链接有什么区别？
6. 如何使用 `find` 按权限、用户、时间等条件查找文件？
7. 如何查看或修改文件的修改时间（mtime）和访问时间（atime）？



------

### **1. Linux 权限的三组 rwx 分别代表什么？**

在 Linux 中，每个文件或目录都有三组权限：

| 权限对象       | 含义           | 示例                   |
| -------------- | -------------- | ---------------------- |
| **User (u)**   | 文件所有者权限 | 拥有者对文件的权限     |
| **Group (g)**  | 同组用户权限   | 属于该文件组的用户权限 |
| **Others (o)** | 其他用户权限   | 所有非拥有者或非组用户 |

权限标识：

```bash
-rwxr-xr--
│││ │ │ │
│││ │ │ └─ 其他用户（r-）
│││ │ └─── 所属组用户（r-x）
│││ └───── 拥有者用户（rwx）
│└──────── 文件类型（-：普通文件，d：目录）
```

| 符号 | 含义            | 数值表示 |
| ---- | --------------- | -------- |
| r    | read（读）      | 4        |
| w    | write（写）     | 2        |
| x    | execute（执行） | 1        |

> 示例：`-rwxr-xr--` → 数字表示为 `754`

------

### **2. 如何修改文件或目录的权限？（`chmod`、`chown`、`chgrp`）**

#### 🔹 修改权限：`chmod`

```bash
chmod 755 file.txt             # 数字法：rwxr-xr-x
chmod u+x file.txt             # 符号法：给文件所有者添加执行权限
chmod g-w file.txt             # 去掉组用户的写权限
chmod a+r file.txt             # 所有人添加读权限
```

#### 🔹 修改所有者：`chown`

```bash
chown user1 file.txt           # 改变文件所有者
chown user1:group1 file.txt    # 同时修改所属组
chown -R user1 /opt/app        # 递归修改整个目录
```

#### 🔹 修改用户组：`chgrp`

```bash
chgrp developers file.txt
```

> 💡 数字法是面试中常考的，比如 `chmod 644`、`chmod 700`、`chmod 777` 各代表什么权限。

------

### **3. 什么是 umask？如何影响新建文件的默认权限？**

**umask（用户掩码）**：定义新建文件时默认权限中“去掉哪些权限”。

#### 🔹 查看与设置

```bash
umask                # 查看当前掩码（如 0022）
umask 0027           # 临时修改
```

#### 🔹 计算规则

- 文件默认权限：`666 - umask`
- 目录默认权限：`777 - umask`

例如：

```
umask = 0022
→ 文件权限：644 (-rw-r--r--)
→ 目录权限：755 (drwxr-xr-x)
```

> ✅ 文件不会默认带执行权限（x），除非手动添加。

------

### **4. 如何查看文件的 inode 信息？**

每个文件在 Linux 中都有一个唯一的 **inode 号**，存储文件的元数据（权限、时间、大小、位置等）。

```bash
ls -i file.txt                # 查看文件的 inode 号
stat file.txt                 # 查看详细 inode 信息
```

输出示例：

```
File: file.txt
Inode: 1234567   Links: 1   Access: (0644/-rw-r--r--)
Access: 2025-10-17 10:00:00
Modify: 2025-10-17 11:00:00
Change: 2025-10-17 11:05:00
```

> 面试延伸：
>
> - inode 不存储文件名，文件名由目录项（directory entry）维护。
> - 删除文件后，inode 释放。

------

### **5. `ln` 命令的软链接与硬链接有什么区别？**

| 类型                   | 命令                | 特点                                                         |
| ---------------------- | ------------------- | ------------------------------------------------------------ |
| **硬链接**             | `ln file1 file2`    | 指向同一个 inode；删除原文件不影响链接文件。不能跨分区或链接目录。 |
| **软链接（符号链接）** | `ln -s file1 file2` | 类似 Windows 快捷方式，保存原文件路径。删除原文件后链接失效。可跨分区。 |

#### 示例：

```bash
ln file.txt hard.txt        # 创建硬链接
ln -s file.txt soft.txt     # 创建软链接
ls -li                      # 查看 inode，可见硬链接 inode 相同
```

> ✅ 硬链接共享同一个 inode；软链接是独立文件（不同 inode）。

------

### **6. 如何使用 `find` 按权限、用户、时间等条件查找文件？**

```bash
find /etc -user root             # 查找属主为 root 的文件
find /var -group mysql           # 查找属组为 mysql 的文件
find /home -perm 755             # 查找权限为 755 的文件
find /var/log -type f -name "*.log"   # 查找日志文件
find /tmp -atime -7              # 最近 7 天访问过的文件
find /tmp -mtime +30             # 30 天前修改的文件
find /home -size +100M           # 大于 100MB 的文件
```

> 💡 实战组合：
>  删除 30 天前的日志文件：
>
> ```bash
> find /var/log -type f -mtime +30 -exec rm -f {} \;
> ```

------

### **7. 如何查看或修改文件的修改时间（mtime）和访问时间（atime）？**

| 时间类型  | 含义                                               |
| --------- | -------------------------------------------------- |
| **atime** | 文件最近访问时间（access time）                    |
| **mtime** | 文件内容最后修改时间（modify time）                |
| **ctime** | 文件元数据（权限/所有者等）修改时间（change time） |

#### 查看时间：

```bash
stat file.txt
ls -l --time=atime file.txt
ls -l --time=ctime file.txt
```

#### 修改时间：

```bash
touch file.txt                     # 更新 atime 与 mtime 为当前时间
touch -t 202510171500 file.txt     # 设置指定时间（YYYYMMDDhhmm）
```

------

✅ **面试延伸问题**

- “删除文件不释放空间的原因？”
   → 文件被进程占用（打开文件描述符未关闭），需使用 `lsof` 定位。
- “如何防止误删系统文件？”
   → 不要在 root 下执行 `rm -rf`，使用 `alias rm='rm -i'`。



------

## 三、进程与任务管理

1. 如何查看当前系统运行的进程？（`ps`、`top`、`htop`）
2. `ps aux | grep java` 输出各列的含义是什么？
3. 如何结束一个进程？（`kill`、`kill -9`、`pkill`、`killall`）
4. 如何查看进程树？（`pstree`）
5. `nice` 和 `renice` 有什么作用？
6. 如何让任务在后台运行？（`&`、`nohup`、`screen`、`tmux`）
7. 如何定时执行任务？（`cron`、`crontab`）
8. 如何查看某个端口对应的进程？（`lsof -i`、`netstat -tunlp`）
9. 如何查看一个 Java 服务的内存占用和 CPU 使用情况？



------

### **1. 如何查看当前系统运行的进程？（`ps`、`top`、`htop`）**

#### 🔹 基本命令：

```bash
ps aux                     # 查看所有进程（系统常用）
ps -ef                     # 标准格式显示所有进程
```

| 参数 | 说明                              |
| ---- | --------------------------------- |
| a    | 显示所有用户的进程                |
| u    | 显示详细信息（用户、CPU、内存等） |
| x    | 显示无终端的后台进程              |

#### 🔹 动态查看进程：

```bash
top                        # 实时监控系统 CPU、内存、进程状态
```

> `htop`（增强版 top）：支持颜色、筛选、交互式管理进程（↑↓ 查找、F9 结束进程）。





### **2. `ps aux | grep java` 输出各列的含义是什么？**

示例输出：

```
user   12345  10.3  5.2 2123456 267892 ?  Sl  10:12  1:32 java -jar app.jar
```

| 列名    | 含义                                                      |
| ------- | --------------------------------------------------------- |
| USER    | 启动该进程的用户                                          |
| PID     | 进程 ID                                                   |
| %CPU    | 占用的 CPU 百分比                                         |
| %MEM    | 占用的内存百分比                                          |
| VSZ     | 虚拟内存大小（KB）                                        |
| RSS     | 常驻内存大小（实际使用物理内存，KB）                      |
| TTY     | 终端名称（? 表示无终端）                                  |
| STAT    | 进程状态（S：休眠，R：运行，Z：僵尸，T：停止，l：多线程） |
| START   | 进程启动时间                                              |
| TIME    | 占用 CPU 的累计时间                                       |
| COMMAND | 执行的命令                                                |

> 💡 常用过滤：
>
> ```bash
> ps aux | grep java | grep -v grep
> ```



### `ps -ef | grep xxx` 

#### 一、命令拆解

##### 1. `ps -ef`

- `ps`：查看**当前系统进程快照**
- `-e`：显示**所有进程**（全部用户、全部后台进程）
- `-f`：**完整格式化输出**，展示全字段（UID、PID、PPID、CMD 完整命令）

##### 2. `|` 管道符

把前一个命令的**输出结果**，交给后一个命令处理。

##### 3. `grep 关键词`

过滤、只展示包含该关键词的行。

---

#### 二、完整列含义（重点）

执行：
```bash
ps -ef | grep java
```
输出7列：
```
UID  PID  PPID  C  STIME  TTY       TIME     CMD
```
1. **UID**：运行进程的用户
2. **PID**：当前进程ID（核心，排查必用）
3. **PPID**：父进程ID
4. **C**：CPU占用百分比
5. **STIME**：进程启动时间
6. **TTY**：终端，`?` 代表后台守护进程
7. **TIME**：进程累计占用CPU时间
8. **CMD**：启动命令/程序路径

---

#### 三、常见用法

```bash
# 查java进程
ps -ef | grep java

# 查tomcat
ps -ef | grep tomcat

# 查nginx
ps -ef | grep nginx

# 查指定jar
ps -ef | grep app.jar
```

---

#### 四、常见坑：自带一条 grep 进程

执行后总会多一行：
```
grep --color=auto java
```
原因：
`grep` 命令本身也是一个进程，刚好包含关键词。

##### 过滤掉自身

```bash
ps -ef | grep java | grep -v grep
```
`-v`：反向过滤，排除含 `grep` 的行。

---

#### 五、精简只看PID（排查常用）

```bash
ps -ef | grep java | grep -v grep | awk '{print $2}'
```
直接输出所有 Java 进程 PID，方便后续 `kill、jstack、top -p`。

---

#### 六、补充：`ps aux` 和 `ps -ef` 区别

1. **ps -ef**
   - 标准System V风格
   - 看 PID、父进程、启动用户、完整启动命令
   **查进程归属、父子进程、启动参数 首选**

2. **ps aux**
   - BSD风格
   - 直观展示 **CPU、内存占用百分比**
   **看资源占用首选**

---

#### 七、面试一句话总结

`ps -ef` 列出系统所有完整进程，管道结合 `grep` 过滤指定服务进程，快速获取 PID，用于线上排查、进程启停、故障分析；搭配 `grep -v grep` 可过滤掉 grep 自身进程干扰。

------

### **3. 如何结束一个进程？（`kill`、`kill -9`、`pkill`、`killall`）**

#### 🔹 常见命令：

```bash
kill PID                    # 发送默认信号（SIGTERM）请求进程结束
kill -9 PID                 # 强制杀死（SIGKILL，不可捕获）
pkill java                  # 按名称杀死进程
killall java                # 杀死所有 java 名称进程
```

#### 🔹 查看信号：

```bash
kill -l                     # 查看所有信号（如 9：SIGKILL，15：SIGTERM）
```

> ⚠️ 面试重点：
>  `kill` 默认发送信号 15（可被拦截），而 `kill -9` 是立即终止，无法保存数据。

------

### **4. 如何查看进程树？（`pstree`）**

```bash
pstree -p                   # 显示进程树及 PID
pstree -u                   # 显示用户信息
pstree -a                   # 显示完整命令行
```

> 应用场景：
>
> - 查找父子进程关系（如 Tomcat 主进程与 worker）。
> - 排查僵尸进程或多重启动问题。

------

### **5. `nice` 和 `renice` 有什么作用？**

用于控制进程的 **优先级（priority）**。

| 值      | 范围      | 含义                           |
| ------- | --------- | ------------------------------ |
| nice 值 | -20 ~ +19 | 越小优先级越高，越大优先级越低 |

#### 示例：

```bash
nice -n 10 java -jar app.jar      # 启动时指定优先级（默认 0）
renice -n 5 -p 12345              # 调整已运行进程的优先级
```

> ⚙️ Root 用户可设置负值（高优先级）。
>  普通用户只能提高（降低）优先级。

------

### **6. 如何让任务在后台运行？（`&`、`nohup`、`screen`、`tmux`）**

| 方法     | 示例                                     | 特点                                |
| -------- | ---------------------------------------- | ----------------------------------- |
| `&`      | `java -jar app.jar &`                    | 简单后台运行，退出终端会被中断      |
| `nohup`  | `nohup java -jar app.jar &`              | 忽略挂起信号（HUP），常用于长期运行 |
| `screen` | `screen -S app` / `screen -r app`        | 可创建持久会话，退出后可重新连接    |
| `tmux`   | `tmux new -s app` / `tmux attach -t app` | 多窗口终端复用，更现代、更常用      |

> ✅ 生产环境推荐：`nohup` + `tmux`
>  结合使用可实现稳定后台运行和日志输出。
>  例如：
>
> ```bash
> nohup java -jar app.jar > app.log 2>&1 &
> ```

------

### **7. 如何定时执行任务？（`cron`、`crontab`）**

#### 🔹 定时任务编辑：

```bash
crontab -e                   # 编辑当前用户的定时任务
crontab -l                   # 查看定时任务
```

#### 🔹 格式说明：

```
分钟 小时 日 月 星期 命令
```

示例：

```bash
0 2 * * * /usr/local/bin/backup.sh     # 每天凌晨2点执行备份脚本
*/5 * * * * /home/user/check.sh        # 每5分钟执行一次
```

#### 🔹 查看系统服务：

```bash
systemctl status crond                 # 查看 cron 服务状态
```

> ⚡ 面试延伸：
>
> - Java 项目中可用 **Spring Task / Quartz** 实现同样功能。
> - 系统级任务推荐使用 `cron`。

------

### **8. 如何查看某个端口对应的进程？（`lsof -i`、`netstat -tunlp`）**

```bash
lsof -i :8080                    # 查看占用 8080 端口的进程
netstat -tunlp | grep 8080       # 查看端口、协议、PID
ss -tunlp | grep 8080            # 更现代替代 netstat
```

| 参数 | 说明         |
| ---- | ------------ |
| -t   | TCP 连接     |
| -u   | UDP 连接     |
| -n   | 不解析主机名 |
| -l   | 仅监听端口   |
| -p   | 显示进程 PID |

> 💡 常见问题：
>  “端口被占用” → 查 PID → `kill PID` 即可释放。

------

### **9. 如何查看一个 Java 服务的内存占用和 CPU 使用情况？**

#### 🔹 系统级方法：

```bash
top -p <pid>                    # 监控特定进程
ps -p <pid> -o %cpu,%mem,cmd    # 查看占用率
```

#### 🔹 Java 专用工具：

```bash
jps -l                          # 查看所有 Java 进程
jstat -gc <pid>                 # 查看 GC 与堆内存使用
jmap -heap <pid>                # 查看堆结构
jmap -histo:live <pid> | head   # 查看对象占用前几名
jstack <pid>                    # 查看线程堆栈（排查死锁）
```

#### 🔹 常用组合命令：

```bash
ps aux | grep java
top -Hp <pid>                   # 查看 Java 内部线程占用
```

> ✅ 实战技巧：
>
> - 高 CPU → 用 `top -Hp` 找线程 → 转十六进制 → `jstack` 分析。
> - 高内存 → 用 `jmap` + `MAT` 分析内存泄漏。

------

##  四、网络与远程操作

1. 如何查看服务器的 IP 地址与网络接口？（`ifconfig`、`ip addr`）
2. 如何测试网络连通性？（`ping`、`telnet`、`curl`、`wget`）
3. 如何查看当前系统的端口占用情况？
4. 如何查看路由表和默认网关？（`route`、`ip route`）
5. 如何配置防火墙？（`firewalld`、`iptables`）
6. 如何查看和修改 DNS 配置？
7. 如何使用 `scp` 或 `rsync` 进行文件传输？
8. SSH 登录慢可能有哪些原因？
9. 如何配置 SSH 免密登录？



### **1.如何查看服务器的 IP 地址与网络接口？**

- `ifconfig`（旧版命令，需安装 net-tools）
- `ip addr show` 或 `ip a`（推荐）
- 查看网络接口信息：`ip link show`
- 查看某网卡详情：`ethtool eth0`

### **2.如何测试网络连通性？**

- `ping <ip>`：检测连通性、延迟
- `telnet <ip> <port>`：测试端口是否开放
- `curl <url>`：测试 HTTP 响应（可查看状态码）
- `wget <url>`：下载测试
- `traceroute <ip>`：查看路由路径

### **3.如何查看当前系统的端口占用情况？**

- `netstat -tunlp`（需安装 net-tools）
- `ss -tunlp`（推荐）
- `lsof -i:<port>`：查看某端口对应进程
- `nmap localhost`：扫描本机端口

### **4.如何查看路由表和默认网关？**

- `route -n`（旧版）
- `ip route show`（推荐）
- 查看默认网关：`ip route | grep default`

### **5.如何配置防火墙？**

- **Firewalld**：
  - 查看状态：`systemctl status firewalld`
  - 查看规则：`firewall-cmd --list-all`
  - 开放端口：`firewall-cmd --permanent --add-port=8080/tcp`
  - 重载规则：`firewall-cmd --reload`
- **iptables**（较底层）：
  - 查看规则：`iptables -L -n`
  - 添加规则：`iptables -A INPUT -p tcp --dport 8080 -j ACCEPT`

### **6.如何查看和修改 DNS 配置？**

- 查看文件：`cat /etc/resolv.conf`
- 临时修改：直接编辑该文件
- 永久修改（CentOS）：`/etc/sysconfig/network-scripts/ifcfg-eth0` 中增加 `DNS1=`、`DNS2=`
- 通过 `nmcli` 命令配置（NetworkManager）

### **7.如何使用 `scp` 或 `rsync` 进行文件传输？**

- **scp：**
  - 上传文件：`scp local.txt user@remote:/path/`
  - 下载文件：`scp user@remote:/path/file ./`
  - 目录传输：`scp -r dir user@remote:/path/`
- **rsync：**
  - `rsync -avz local/ user@remote:/path/`
  - 支持断点续传与增量同步

### **8.SSH 登录慢可能有哪些原因？**

- DNS 反向解析慢（解决：在 `/etc/ssh/sshd_config` 中设置 `UseDNS no`）
- GSSAPI 认证慢（解决：`GSSAPIAuthentication no`）
- 网络延迟或 MTU 设置不当
- 防火墙或代理导致握手慢

### **9.如何配置 SSH 免密登录？**

- 本地生成密钥：`ssh-keygen -t rsa`
- 将公钥传到目标主机：
  - `ssh-copy-id user@remote`
  - 或手动追加到远程主机的 `~/.ssh/authorized_keys`
- 确保权限正确：`chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`

### netstat 

#### 一、核心作用

`netstat`：Linux 查看**端口、网络连接、TCP状态、监听、连接数、套接字** 神器，排查接口慢、连接泄漏、端口占用、TCP异常必备。

---

#### 二、常用核心参数（必记）

```bash
-a   # 显示所有连接、监听、套接字（通用参数）
-n   # 纯数字展示，不解析域名/端口名，速度快（通用参数）
-t   # 只看 TCP 协议 （Linux独有）
-u   # 只看 UDP 协议 （Linux独有）
-p   # 显示进程PID/程序名（需要root ， Linux独有）
-l   # 只看 LISTEN 监听状态 （Linux独有）
-o ：显示进程 PID（Windows 独有）

grep = Linux 里的「搜索 / 过滤」命令，Windows 里对应的是 findstr
```

✅ 工作万能组合：
```bash
netstat -antlp
```

---

#### 三、5大高频组合命令

##### 1. 查看所有TCP连接（最常用）

```bash
netstat -ant
```

##### 2. 查看当前机器所有监听端口（查端口占用）

```bash
netstat -lntp
```
作用：查 8080、3306、6379 是否被占用

##### 3. 查看指定端口占用（比如Tomcat 8080）

```bash
netstat -ant | grep 8080 （linux）
netatat -ano | findstr 8080 （windows）
两者功能等价，效果一样
```

##### 4. 统计已建立正常连接数

```bash
netstat -ant | grep ESTABLISHED | wc -l
```

##### 5. 排查连接泄漏（CLOSE_WAIT 过多）

```bash
netstat -ant | grep CLOSE_WAIT
```

##### 6.查看并杀死指定端口的进程

```bash
netatat -ano | findstr 8080
TCP    127.0.0.1:8080    0.0.0.0:0    LISTENING    18520
127.0.0.1:8080：监听地址 + 端口
LISTENING：正在监听（有服务占用）
末尾数字 18520 = PID 进程 ID

杀掉占用 8080 端口进程
taskkill /PID 18520 /F
```



---

#### 四、TCP 七大核心状态（面试必考）

1. **LISTEN**：监听中，等待别人连我（服务端口）
2. **ESTABLISHED**：连接已建立，正常传输数据
3. **TIME_WAIT**：主动关闭方，等待释放端口（短连接多就爆满）
4. **CLOSE_WAIT**：对方断开，我方没关连接 → **连接泄漏**
5. **SYN_SENT**：主动发起连接
6. **SYN_RECV**：接收连接握手
7. **FIN_WAIT**：连接关闭中

---

#### 五、线上经典问题排查

1. **接口很慢、超时**
- ESTABLISHED 过多：Tomcat/容器连接数打满
- CLOSE_WAIT 很多：代码未关闭数据库、Redis、Http连接

2. **启动项目提示：端口被占用**
```bash
netstat -lntp | grep 8080
# 找到PID，kill -9 pid
```

3. **服务器对外请求大量积压**
查看 TIME_WAIT 数量，过高需优化内核参数

---

#### 六、面试精简背诵版

1. `netstat -ant` 查看全部TCP网络连接；
2. 重点关注 `ESTABLISHED` 正常连接、`CLOSE_WAIT` 连接泄漏、`LISTEN` 监听端口；
3. 配合 grep 过滤端口、连接状态，用来排查端口占用、连接数耗尽、接口响应慢、连接不释放等问题。

---

#### 七、补充（新系统替代命令）

netstat 已逐步弃用，新版 Linux 推荐更快的 `ss`
```bash
ss -antlp
```
功能一致，性能更强。

##  五、系统与性能监控

1. 如何查看系统负载与平均负载（load average）？
2. 如何分析 CPU 使用率？（`top`、`mpstat`、`sar`）
3. 如何查看内存使用情况？（`free`、`vmstat`）
4. 如何查看磁盘 IO 情况？（`iostat`、`iotop`）
5. 如何查看网络流量？（`iftop`、`nload`）
6. 如何查看系统启动时间与运行时长？
7. 如何查看系统日志？（`/var/log/messages`、`dmesg`）
8. 如何定位系统卡顿或高负载的原因？
9. 如何查看历史命令？如何防止命令记录泄露敏感信息？



### **1.如何查看系统负载与平均负载（load average）？**

- 命令：`uptime` 或 `top`

- 示例输出：

  ```
  load average: 0.15, 0.30, 0.25
  ```

  分别表示过去 **1 分钟、5 分钟、15 分钟** 的平均负载。

- 解释：

  - 值 ≈ CPU 核数 → 系统运行正常
  - 值 ≫ CPU 核数 → 系统过载（可能 CPU、IO、内存瓶颈）

### **2.如何分析 CPU 使用率？**

- `top`：实时查看 CPU、内存、进程占用
  - `%us` 用户态CPU、`%sy` 内核态、`%id` 空闲、`%wa` I/O等待
- `mpstat -P ALL 1`：查看每个 CPU 核的使用率
- `sar -u 1 5`：每秒统计一次 CPU 使用情况，共采样5次
- 常见瓶颈：
  - `us` 高 → 程序计算密集
  - `sy` 高 → 系统调用频繁
  - `wa` 高 → I/O 等待严重

### **3.如何查看内存使用情况？**

- `free -h`：查看总内存、已用、可用、缓存
  - `-/+ buffers/cache` 行反映真实可用内存
- `vmstat 1`：动态查看内存、swap、CPU 状态
- `cat /proc/meminfo`：更详细的内存统计
- 判断方法：
  - Swap 使用频繁 → 内存不足
  - `buffers/cache` 高但可用内存多 → Linux 缓存机制正常

### **4.如何查看磁盘 IO 情况？**

- `iostat -x 1`：显示磁盘读写速率和使用率
  - `util` 接近 100% 表示磁盘繁忙
  - `await` 表示平均I/O等待时间
- `iotop`：实时查看哪个进程占用 IO
- `df -h`：查看磁盘空间占用
- `du -sh *`：查看各目录空间占用

### **5.如何查看网络流量？**

- `iftop`：实时显示各连接的带宽使用情况
- `nload`：展示网络流量变化图
- `sar -n DEV 1`：查看每个网卡的流量
- `ss -s`：查看当前 TCP/UDP 连接数量统计

### **6.如何查看系统启动时间与运行时长？**

- `uptime`：显示运行时长
- `who -b`：显示最近启动时间
- `last reboot`：查看系统重启记录
- `dmesg | head`：查看开机日志时间戳

### **7.如何查看系统日志？**

- 系统日志路径：`/var/log/`
  - 核心系统日志：`/var/log/messages`
  - 内核日志：`dmesg`
  - 登录日志：`/var/log/secure`
  - 系统启动：`/var/log/boot.log`
- 实时查看：`tail -f /var/log/messages`
- 过滤关键字：`grep error /var/log/messages`

### **8.如何定位系统卡顿或高负载的原因？**

 排查思路：

- CPU：`top` → 查看占用高的进程（如 Java）
- 内存：`free -h` / `top` → 是否 OOM
- IO：`iostat` / `iotop` → 是否磁盘繁忙
- 网络：`iftop` / `ss -s` → 是否连接过多
- 日志：`/var/log/messages` / `dmesg` → 是否有异常
- 如果是 Java 服务：`jstack` + `top -Hp <pid>` 联合分析线程占用

### **9.如何查看历史命令？如何防止命令记录泄露敏感信息？**

- 查看历史命令：`history`
- 清空历史：`history -c`
- 临时禁止记录：
  - 执行命令前加空格（若 `HISTCONTROL=ignorespace`）
  - 或 `unset HISTFILE` 临时关闭记录
- 永久删除：`> ~/.bash_history`
- 敏感命令（如密码操作）建议：
  - 使用 `read -s` 输入隐藏参数
  - 或通过安全脚本读取配置文件中的凭据



### vmstat\mpstat

两个都是 **Linux 系统级性能监控命令**，秒级实时刷新，专门排查：CPU瓶颈、上下文切换、IO阻塞、系统负载。

---

#### 一、vmstat 1 详解

##### 作用

`vmstat 1`：**每秒刷新1次**，监控：进程、内存、交换分区、IO、系统、CPU 全局状态。

##### 命令

```bash
vmstat 1
```

##### 输出字段拆解（重点看这几个）

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
```

1. **procs 进程**

- `r`：**运行队列数**（正在运行/等待CPU的进程数）
  - 大于CPU核心数 → **CPU瓶颈**
- `b`：阻塞进程数（等待磁盘IO、网络、锁）

2. **swap 交换分区（内存不足标志）**

- `si`：换入磁盘
- `so`：换出磁盘
  👉 **si/so 持续不为0 = 内存不足，严重卡顿**

3. **io 磁盘IO**

- `bi`：磁盘读
- `bo`：磁盘写
  👉 数值大 = 磁盘IO瓶颈

4. **system 系统**

- `in`：每秒中断数
- `cs`：**上下文切换次数**
  👉 cs 过高 → 线程竞争、频繁切换，CPU 利用率变差

5. **cpu 核心（面试必看）**

- `us`：用户态CPU（业务代码、Java应用）
- `sy`：内核态CPU
- `id`：空闲CPU
- `wa`：IO等待（磁盘/数据库慢）
- `st`：虚拟化抢占

---

#### 二、关键阈值（排障标准）

1. `r > CPU核数`：CPU 满载，业务跑不动
2. `us 长期 > 80%`：Java 代码死循环、复杂计算、线程自旋
3. `wa 长期 > 20%`：磁盘IO慢、MySQL慢SQL、文件读写瓶颈
4. `cs 暴涨`：大量线程争抢锁、频繁上下文切换
5. `si/so > 0`：物理内存不足，用了磁盘交换，服务卡死

---

#### 三、mpstat 1 详解

##### 作用

`vmstat` 是**全局平均**
`mpstat 1`：**按每个CPU核心单独监控**，精准定位是否单核打爆、多核不均。

##### 命令

```bash
# 每秒打印所有CPU
mpstat 1

# 只看平均
mpstat 1 -A
```

##### 核心字段

- `%usr`：用户态CPU（Java业务）
- `%system`：内核态
- `%iowait`：IO等待
- `%idle`：空闲
- `%irq / %soft`：软硬中断

##### 实战价值

- 发现 **单核100%、其他核空闲**：
  典型：**单线程死循环、锁竞争、单任务串行**
- 所有核心均匀高负载：
  多线程并发、定时任务、批量计算

---

#### 四、组合排查场景（Java线上故障）

##### 场景1：CPU 100%

1. `vmstat 1` → `us` 很高、id 很低
2. `mpstat 1` → 看是单核还是多核高
3. 再用 `arthas thread` / `jstack` 定位死循环代码

##### 场景2：接口响应慢、但CPU不高

1. `vmstat 1` → `wa` 很高
2. 结论：**IO瓶颈**（慢SQL、磁盘慢、Redis/DB阻塞）

##### 场景3：服务卡顿、上下文切换频繁

1. `vmstat 1` → `cs` 巨大
2. 原因：线程池过多、锁竞争激烈、CAS自旋

##### 场景4：内存不足卡顿

1. `vmstat 1` → `si so` 持续上涨
2. 结论：内存泄漏、堆外内存溢出

---

#### 五、面试极简背诵版

1. `vmstat 1`：全局实时监控，重点看 **r运行队列、us用户CPU、waIO等待、cs上下文切换、si/so内存交换**；
2. `mpstat 1`：查看**每个CPU核心**占用，区分单核打爆还是整体CPU瓶颈；
3. us高查代码死循环，wa高查磁盘/数据库IO，cs高查线程锁竞争，si/so高查内存不足。

---

#### 六、补充：高频搭配命令

```bash
# 看整机CPU占用
top

# 看Java进程CPU/内存
ps aux | grep java

# 看线程CPU（配合排查100%）
top -H -p pid
```

需要我把 **top / vmstat / mpstat / netstat / arthas** 整套 Java 线上排查流程，给你浓缩成一页面试速记吗？

------

##

------

## 六、部署与服务管理

1. 如何在 Linux 上部署 Java 项目（JAR、WAR）？
2. 如何使用 `systemctl` 或 `service` 管理后台服务？
3. 如何配置 Java 环境变量（JAVA_HOME、PATH）？
4. 如何使用 `nohup`、`screen` 或 `tmux` 让服务后台运行并保持日志？
5. 如何排查端口占用或服务无法启动问题？
6. 如何查看并优化服务器启动脚本（如 `/etc/systemd/system`）？



### **1.如何在 Linux 上部署 Java 项目（JAR、WAR）？**

 **① JAR 部署方式：**

- 直接运行：

  ```bash
  java -jar app.jar
  ```

- 后台运行并输出日志：

  ```bash
  nohup java -jar app.jar > app.log 2>&1 &
  ```

- 指定端口或配置文件：

  ```bash
  java -jar app.jar --server.port=8081 --spring.profiles.active=prod
  ```

**② WAR 部署方式（Tomcat）：**

- 将 `xxx.war` 拷贝到 `tomcat/webapps/` 目录

- 启动 Tomcat：

  ```bash
  ./bin/startup.sh
  ```

- 查看日志：

  ```bash
  tail -f logs/catalina.out
  ```

**③ 优化建议：**

- 使用 `-Xms` / `-Xmx` 设置 JVM 内存
- 使用 `-Dspring.config.location=` 指定外部配置文件
- 使用 `systemd` 管理服务保证开机自启

------

### **2.如何使用 `systemctl` 或 `service` 管理后台服务？**

- **启动/停止/重启服务：**

  ```bash
  systemctl start nginx
  systemctl stop nginx
  systemctl restart nginx
  ```

- **查看状态：**

  ```bash
  systemctl status nginx
  ```

- **开机自启：**

  ```bash
  systemctl enable nginx
  ```

- **关闭自启：**

  ```bash
  systemctl disable nginx
  ```

- **旧版命令（兼容模式）：**

  ```bash
  service nginx start
  service nginx status
  ```

💡 Java 应用也可注册为 systemd 服务，便于统一管理（见第6题）。

------

### **3.如何配置 Java 环境变量（JAVA_HOME、PATH）？**

- 查看 Java 安装路径：

  ```bash
  which java
  readlink -f $(which java)
  ```

- 编辑配置文件（推荐 `/etc/profile` 或 `~/.bashrc`）：

  ```bash
  export JAVA_HOME=/usr/local/java/jdk1.8.0_361
  export PATH=$JAVA_HOME/bin:$PATH
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  ```

- 立即生效：

  ```bash
  source /etc/profile
  ```

- 验证：

  ```bash
  java -version
  echo $JAVA_HOME
  ```

------

### **4.如何使用 `nohup`、`screen` 或 `tmux` 让服务后台运行并保持日志？**

**① nohup**（最常用）

```bash
nohup java -jar app.jar > app.log 2>&1 &
tail -f app.log
```

- 退出终端不会中断进程
- `jobs -l` 查看后台任务，`kill` 结束

**② screen**

```bash
screen -S myapp
java -jar app.jar
# 退出保留会话：Ctrl + A + D
screen -r myapp  # 重新连接
```

**③ tmux**

- 启动：`tmux new -s myapp`
- 分离：`Ctrl + B, D`
- 查看：`tmux ls`
- 重新连接：`tmux attach -t myapp`

✅ 适用于长时间运行的后端服务或训练脚本。

------

### **5.如何排查端口占用或服务无法启动问题？**

- 查看端口是否被占用：

  ```bash
  netstat -tunlp | grep 8080
  ss -tunlp | grep 8080
  lsof -i:8080
  ```

- 杀掉占用进程：

  ```bash
  kill -9 <pid>
  ```

- 检查防火墙：

  ```bash
  firewall-cmd --list-ports
  firewall-cmd --add-port=8080/tcp --permanent
  firewall-cmd --reload
  ```

- 查看 Java 日志：

  ```bash
  tail -f app.log
  ```

- 检查系统限制：

  ```bash
  ulimit -n  # 打开文件数
  df -h      # 磁盘是否满
  ```

------

### **6.如何查看并优化服务器启动脚本（如 `/etc/systemd/system`）？**

- 示例：创建 Java 服务脚本

  ```bash
  sudo vim /etc/systemd/system/app.service
  ```

  内容示例：

  ```ini
  [Unit]
  Description=My Java Application
  After=network.target
  
  [Service]
  User=root
  ExecStart=/usr/bin/java -jar /opt/app/app.jar
  Restart=always
  StandardOutput=append:/var/log/app.log
  StandardError=append:/var/log/app.err
  
  [Install]
  WantedBy=multi-user.target
  ```

- 启动并启用：

  ```bash
  systemctl daemon-reload
  systemctl start app
  systemctl enable app
  ```

- 查看日志：

  ```bash
  journalctl -u app -f
  ```

✅ 优点：可自启、自动重启、集中日志管理、统一进程控制。

------

##  七、日志与排错分析

**1.Linux 系统日志主要存放在哪里？常见日志文件有哪些？**

- 默认日志目录：`/var/log/`

- 常见系统日志：

  | 日志文件                             | 说明                               |
  | ------------------------------------ | ---------------------------------- |
  | `/var/log/messages`                  | 系统级通用日志（内核、服务启动等） |
  | `/var/log/secure`                    | 登录、认证、安全相关               |
  | `/var/log/boot.log`                  | 启动信息                           |
  | `/var/log/cron`                      | 定时任务                           |
  | `/var/log/dmesg`                     | 内核缓冲区信息                     |
  | `/var/log/syslog`                    | Ubuntu/Debian 系统常见系统日志     |
  | `/var/log/nginx/`、`/var/log/httpd/` | Web 服务日志                       |
  | `/var/log/audit/`                    | SELinux、访问审计                  |

------

**2.如何实时查看日志输出？**

- **实时追踪最新日志：**

  ```bash
  tail -f /var/log/messages
  ```

- **查看最后 N 行：**

  ```bash
  tail -n 100 /var/log/secure
  ```

- **动态过滤输出（查关键字）：**

  ```bash
  tail -f app.log | grep ERROR
  ```

- **组合使用（日志排查高频）：**

  ```bash
  journalctl -u app -f | grep Exception
  ```

------

**3.如何查看系统启动和运行日志？**

- 查看开机日志：

  ```bash
  dmesg | less
  ```

- 过滤硬件相关错误：

  ```bash
  dmesg | grep -i error
  ```

- 查看系统服务启动情况：

  ```bash
  journalctl -b
  ```

  （`-b` 表示本次启动）

------

**4.如何查看特定服务的日志？（例如 Java 应用、Nginx、MySQL）**

- **Java 应用（systemd 管理）：**

  ```bash
  journalctl -u app -f
  ```

- **Nginx：**

  ```
  /var/log/nginx/access.log
  /var/log/nginx/error.log
  ```

- **MySQL：**

  ```
  /var/log/mysqld.log
  ```

- **Tomcat：**

  ```
  /opt/tomcat/logs/catalina.out
  ```

------

**5.如何过滤、统计或分析日志内容？**

- 按关键字过滤：

  ```bash
  grep "ERROR" app.log
  grep -i "exception" app.log
  ```

- 按日期过滤：

  ```bash
  grep "2025-10-17" app.log
  ```

- 统计出现次数：

  ```bash
  grep "NullPointerException" app.log | wc -l
  ```

- 去重统计 IP：

  ```bash
  awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
  ```

- 结合管道快速分析：

  ```bash
  grep "ERROR" app.log | awk '{print $5}' | sort | uniq -c | sort -nr
  ```

------

### 6.**何使用 `journalctl` 查看系统日志？（CentOS 7+）**

- 查看所有日志：

  ```bash
  journalctl
  ```

- 查看某个服务：

  ```bash
  journalctl -u app.service
  ```

- 实时输出：

  ```bash
  journalctl -u app -f
  ```

- 查看指定时间段：

  ```bash
  journalctl --since "2025-10-17 09:00" --until "2025-10-17 10:00"
  ```

- 查看某次开机日志：

  ```bash
  journalctl -b -1
  ```

------

**7.如何使用 `strace` 调试无法启动的程序？**

- 跟踪系统调用（定位卡点、文件缺失等问题）：

  ```bash
  strace -f -o trace.log java -jar app.jar
  ```

- 查看错误行：

  ```bash
  grep -i "ENOENT" trace.log
  ```

  🔍 常见于文件路径、配置文件读取错误。

- 跟踪已运行进程：

  ```bash
  strace -p <pid>
  ```

------

**8.如何使用 `lsof` 排查文件与端口占用？**

- 查看某端口被哪个进程占用：

  ```bash
  lsof -i:8080
  ```

- 查看进程打开的文件：

  ```bash
  lsof -p <pid>
  ```

- 查看删除但仍占用的文件（导致磁盘满）：

  ```bash
  lsof | grep deleted
  ```

- 查找文件被哪个进程锁定：

  ```bash
  fuser -v /path/to/file
  ```

------

**9.常见线上问题排查思路（综合题）**

- **服务无法启动：**
  1. `journalctl -u app -xe` 查看报错
  2. 检查端口占用：`ss -tunlp | grep 8080`
  3. 检查磁盘空间：`df -h`
  4. 检查权限或文件路径
- **CPU 或内存过高：**
  1. `top` 或 `htop` 找出进程 PID
  2. 对 Java：`top -Hp <pid>` 找线程 → `jstack <pid>` 打印堆栈分析热点
- **磁盘写满：**
  1. `df -h` 查看分区
  2. `du -sh * | sort -h` 定位大文件
  3. `lsof | grep deleted` 检查已删除但仍占用的句柄
- **日志暴增 / IO 过高：**
  1. `iotop`、`dstat` 检查写入频率
  2. 结合 `grep -E "ERROR|WARN"` 定位异常模块



------

##  八、安全与权限控制（防火墙、SELinux、sudo、SSH）

1. **Linux 的权限模型由哪三部分组成？文件权限 `rwx` 是如何解释的？**
   - ✅ 用户（owner）、用户组（group）、其他人（others）；
      `r`=读(4)，`w`=写(2)，`x`=执行(1)。
2. **如何使用 `chmod`、`chown` 和 `chgrp` 修改文件权限和所有者？举例说明。**
   - ✅ `chmod 755 app.sh`、`chown tomcat:tomcat /opt/app`、`chgrp developers /opt/code`
3. **`sudo` 的原理是什么？如何给普通用户授权使用 `sudo`？**
   - ✅ `sudo` 临时以 root 权限执行命令；
      修改 `/etc/sudoers` 或用 `visudo` 添加 `user ALL=(ALL) NOPASSWD:ALL`
4. **如何限制用户只能使用特定命令（例如只允许执行 `docker ps`）？**
   - ✅ 在 `/etc/sudoers` 添加精确命令限制；
      或使用 `rbash`（受限 bash）。
5. **如何配置 SSH 公钥免密登录？**
   - ✅ `ssh-keygen` → 将公钥追加到服务器的 `~/.ssh/authorized_keys`。
6. **如何修改 SSH 默认端口？如何禁止 root 远程登录？**
   - ✅ 修改 `/etc/ssh/sshd_config` 中的 `Port` 和 `PermitRootLogin no`，再执行 `systemctl restart sshd`。
7. **防火墙（firewalld/iptables）如何放行某个端口（如 8080）？**
   - ✅ `firewall-cmd --zone=public --add-port=8080/tcp --permanent && firewall-cmd --reload`。
8. **如何查看 SELinux 状态？如果阻止了某个服务该如何处理？**
   - ✅ `getenforce` 查看状态；
      临时关闭：`setenforce 0`；永久关闭：修改 `/etc/selinux/config`。
9. **如何查看哪些端口正在监听？如何检测是否有可疑进程？**
   - ✅ `netstat -tulnp`、`ss -tulnp`、`lsof -i`、`ps aux | grep ssh`。
10. **如何通过 SSH 配置多台服务器的免密批量管理？（简述 Ansible / SSH Config 用法）**

- ✅ 在 `~/.ssh/config` 配置别名与密钥路径；
   或用 Ansible 通过 SSH 管理服务器，实现一键部署与任务执行。



## 九、性能监控与资源分析

1. **如何使用 `top` 命令查看系统资源使用情况？**

   - ✅ 实时查看 CPU、内存、负载和进程信息
   - 重要字段：
     - `%CPU`、`%MEM`：进程占用资源
     - `load average`：系统平均负载（1/5/15 分钟）
     - `VIRT/RES/SHR`：虚拟内存、常驻内存、共享内存
   - 快捷操作：
     - `P` 按 CPU 排序
     - `M` 按内存排序
     - `k` 杀进程

2. **如何使用 `htop` 监控系统资源？**

   - ✅ 交互式工具，图形化显示 CPU 核、内存、进程
   - 支持筛选、排序、搜索进程、发送信号
   - 安装：`yum install htop` 或 `apt install htop`

3. **如何使用 `vmstat` 查看系统性能？**

   - ✅ 命令：`vmstat 1 5`（每秒刷新 5 次）
   - 重要字段：
     - `r`：运行队列长度
     - `free`：空闲内存
     - `si/so`：swap 读写
     - `us/sy/wa`：用户态/内核态/IO等待 CPU 占用

4. **如何使用 `iostat` 查看磁盘 I/O？**

   - ✅ 命令：`iostat -x 1`（每秒刷新）
   - 关键指标：
     - `r/s` / `w/s`：每秒读写次数
     - `await`：平均等待时间
     - `util`：磁盘使用率（接近 100% 表示瓶颈）

5. **如何使用 `iotop` 实时查看进程 I/O？**

   - ✅ 命令：`iotop -o`（只显示有 I/O 的进程）
   - 可查看进程的磁盘读写速率
   - 常用于定位磁盘瓶颈或日志写入过快问题

6. **如何使用 `sar` 收集历史性能数据？**

   - ✅ 安装：`yum install sysstat`
   - 收集系统信息：
     - CPU：`sar -u 1 5`
     - 内存：`sar -r 1 5`
     - IO：`sar -b 1 5`
     - 网络：`sar -n DEV 1 5`
   - 可生成历史报表，便于性能趋势分析

7. **如何查看系统负载及平均负载（load average）？**

   - ✅ 命令：

     ```bash
     uptime
     top
     ```

   - 判断标准：

     - load ≈ CPU 核数 → 正常
     - load ≫ CPU 核数 → 系统过载

8. **如何定位高 CPU 或内存的 Java 进程？**

   - ✅ 使用 `top -Hp <pid>` 查看线程占用
   - `jstack <pid>` 打印线程栈
   - `jmap -heap <pid>` 查看堆内存占用
   - 排查思路：CPU 高 → 线程热点分析，内存高 → 堆快照分析

9. **如何分析磁盘 I/O 瓶颈？**

   - ✅ 组合使用：
     - `iostat -x 1` → 查看磁盘总体性能
     - `iotop` → 查看单进程 I/O
     - `df -h` / `du -sh *` → 查看磁盘空间
   - 常见原因：
     - 日志过多或写入过快
     - 磁盘满或老化
     - 大量临时文件

10. **如何分析网络性能？**

    - ✅ 实时工具：
      - `iftop`：带宽使用情况
      - `nload`：流量变化图
      - `ss -s` / `netstat -s`：连接数、协议统计
    - 历史分析：
      - `sar -n DEV 1 5` → 网卡流量
    - 常见问题：
      - TCP 连接过多
      - 网络带宽不足或丢包



## 十、其他

**加分方向（常问实战）**：

- 如何编写启动脚本自动拉起 Spring Boot 项目？
- 如何查看 Java 服务日志实时输出？
- 如何排查“服务器 CPU 飙高”的问题？
- 如何查看内存泄漏的进程和占用？
- 如何定位“端口被占用”是哪一个 Java 服务？

我把你列的五个问题整理成 **实战型 Linux + Java 后端运维技巧**，每个问题都给出可直接使用的命令与分析思路，便于面试和线上排查。

------

### 1️⃣ 如何编写启动脚本自动拉起 Spring Boot 项目？

**思路：** 用 shell 脚本 + `nohup` 或 `systemd` 实现自动拉起、后台运行、日志记录。

**① 简单 shell 脚本（`start_app.sh`）**

```bash
#!/bin/bash
APP_NAME="app.jar"
LOG_FILE="app.log"

# 杀掉旧进程
PID=$(ps -ef | grep $APP_NAME | grep -v grep | awk '{print $2}')
if [ -n "$PID" ]; then
    echo "Killing existing process $PID"
    kill -9 $PID
fi

# 启动新进程
echo "Starting $APP_NAME ..."
nohup java -jar $APP_NAME > $LOG_FILE 2>&1 &
echo "Started with PID $!"
```

- 保存后 `chmod +x start_app.sh`
- 执行：`./start_app.sh`

**② 使用 systemd 管理（推荐生产环境）**

```ini
[Unit]
Description=Spring Boot Application
After=network.target

[Service]
User=tomcat
ExecStart=/usr/bin/java -jar /opt/app/app.jar
SuccessExitStatus=143
Restart=always
StandardOutput=append:/var/log/app.log
StandardError=append:/var/log/app.err

[Install]
WantedBy=multi-user.target
```

- 保存为 `/etc/systemd/system/app.service`
- 启动/启用：

```bash
systemctl daemon-reload
systemctl start app
systemctl enable app
```

------

### 2️⃣ 如何查看 Java 服务日志实时输出？

**命令：**

```bash
# 如果是 nohup 启动
tail -f app.log

# systemd 管理的服务
journalctl -u app -f

# 过滤异常信息
tail -f app.log | grep -i "ERROR\|Exception"
```

- `-f` 表示实时追踪
- 可结合 `grep` 分析错误、异常堆栈

------

### 3️⃣ 如何排查“服务器 CPU 飙高”的问题？

**步骤：**

1. 查看整体 CPU 使用：

```bash
top
htop   # 图形化
mpstat -P ALL 1
```

1. 找到占用高的进程：

```bash
top -b -n 1 | head -20
```

1. 针对 Java 进程分析线程：

```bash
TOP_PID=$(pidof java)
top -Hp $TOP_PID
jstack $TOP_PID > thread_dump.log
```

- `top -Hp` 可以看到每个线程 CPU 使用情况
- `jstack` 打印线程栈 → 分析死循环或热点方法

------

### 4️⃣ 如何查看内存泄漏的进程和占用？

**步骤：**

1. 查看内存占用高的进程：

```bash
ps aux --sort=-%mem | head -20
top -o %MEM
```

1. 针对 Java 进程：

```bash
jmap -heap <pid>       # 查看堆内存使用
jstat -gc <pid> 1000   # 查看 GC 统计
```

1. 分析堆快照：

```bash
jmap -dump:live,format=b,file=heap.hprof <pid>
# 然后用 Eclipse MAT 或 VisualVM 分析泄漏对象
```

1. 可结合 `free -h`、`vmstat` 查看系统内存整体状态

------

### 5️⃣ 如何定位“端口被占用”是哪一个 Java 服务？

**命令：**

1. 查看端口占用：

```bash
netstat -tunlp | grep 8080
ss -tunlp | grep 8080
lsof -i:8080
```

1. 输出示例：

```
java    12345  tomcat   50u  IPv6  0x...  TCP *:8080 (LISTEN)
```

- `12345` → PID
- `tomcat` → 用户
- `java` → 进程名称

1. 查看进程详细信息：

```bash
ps -p 12345 -o pid,user,cmd
```

1. 杀掉占用进程（如果确认可终止）：

```bash
kill -9 12345
```

------

# 面试

### 面试回答：Linux 内存管理机制

Linux 内存管理是操作系统核心模块之一，核心目标是「高效利用物理内存、优化内存访问速度、支持多进程安全隔离」，通过 **虚拟内存、分页机制、内存分配与回收、缓存策略** 四大核心技术实现。以下从底层原理到实际机制展开，结合面试高频考点详细说明：

#### 一、核心基础：虚拟内存（Virtual Memory）

##### 1. 定义与核心价值

虚拟内存是 Linux 内存管理的基石，它为每个进程提供了「独立的、连续的虚拟地址空间」，进程操作的是虚拟地址，而非直接访问物理内存。  
**核心价值**：
- 进程隔离：每个进程的虚拟地址空间独立，互不干扰（进程看不到其他进程的内存数据）；
- 突破物理内存限制：允许进程使用的内存超过实际物理内存大小（通过硬盘交换分区实现）；
- 简化内存管理：进程无需关心物理内存的实际分布，只需按虚拟地址编程。

##### 2. 虚拟地址与物理地址的映射

- 虚拟地址空间大小：32 位系统为 4GB（2³²），64 位系统为 128TB（理论值，实际受内核限制）；
- 映射机制：通过 **页表（Page Table）** 实现虚拟地址到物理地址的转换，页表由内核维护，存储在物理内存中；
- 地址转换流程：
  1. 进程访问虚拟地址 → CPU 的 MMU（内存管理单元）解析虚拟地址；
  2. MMU 查找页表，将虚拟地址转换为物理地址；
  3. 若页表中无对应映射（缺页异常），内核触发「缺页中断」，加载对应数据到物理内存后更新页表。

#### 二、内存组织：分页与分段机制

Linux 采用「分页机制」管理内存（摒弃了纯分段，仅保留分段用于权限隔离），核心是将虚拟内存和物理内存划分为固定大小的「页（Page）」：
- 页大小：默认 4KB（可配置为 8KB、16KB 等），小页可减少内存浪费，大页适合高内存需求场景；
- 分页优势：
  1. 解决内存碎片问题（物理内存按页分配，无需连续空间）；
  2. 便于内存交换（仅交换需要的页，而非整个进程地址空间）；
- 多级页表：为避免页表占用过多内存（32 位系统纯页表需 4MB），Linux 采用「二级页表」（32 位）或「四级页表」（64 位），仅加载当前需要的页表项，节省内存。

#### 三、内存分配机制：从内核到用户态

Linux 内存分配分为「内核态分配」和「用户态分配」，底层依赖统一的内存管理框架：

##### 1. 内核态内存分配

内核自身需要内存时（如创建进程、分配内核数据结构），通过以下两种方式分配：
- **伙伴系统（Buddy System）**：
  - 核心逻辑：将物理内存按 2 的幂次方（1KB、2KB、4KB、8KB...）划分为「块」，分配时寻找最小的合适块，释放时合并相邻块，避免外部碎片；
  - 适用场景：分配大内存（如页框），高效管理物理内存。
- **slab 分配器**：
  - 核心逻辑：基于伙伴系统，为内核常用数据结构（如 inode、task_struct）预分配「缓存池」，分配时直接从缓存池取，释放时归还给缓存池，避免频繁分配/释放导致的碎片；
  - 优势：提升内核内存分配效率，减少页表操作开销。

##### 2. 用户态内存分配

用户进程申请内存时（如 `malloc()` 函数），底层通过「虚拟内存区域（VMA）」管理，核心流程：
- 进程虚拟地址空间划分：
  - 代码段（.text）：存储程序指令；
  - 数据段（.data/.bss）：存储全局变量、静态变量；
  - 堆（Heap）：动态分配内存（`malloc()`/`new`），从低地址向高地址增长；
  - 栈（Stack）：存储函数栈帧、局部变量，从高地址向低地址增长；
  - 共享库/文件映射区：加载共享库、内存映射文件（`mmap()`）。
- 分配流程：
  1. 用户调用 `malloc()` → 底层调用 `brk()` 或 `mmap()` 向内核申请虚拟内存；
  2. 内核为进程分配虚拟地址空间（VMA），但不立即分配物理内存；
  3. 进程首次访问虚拟地址时，触发「缺页中断」，内核才分配物理内存并建立页表映射；
  4. 小内存分配（<128KB）通过 `brk()` 扩展堆，大内存分配（≥128KB）通过 `mmap()` 映射匿名页。

#### 四、内存回收与交换机制

Linux 为避免物理内存耗尽，通过「内存回收」和「交换分区（Swap）」主动释放空闲内存，核心机制：

##### 1. 内存回收（Page Reclaim）

当物理内存紧张时，内核的「页框回收器（PFRA）」会回收空闲内存，主要回收两类页：
- **可回收页**：
  - 缓存页（如文件页、目录项缓存、inode 缓存）：通过 `page cache` 缓存磁盘文件数据，回收时直接释放（数据可从磁盘重新加载）；
  - 匿名页（如堆、栈数据）：无磁盘备份，需先写入交换分区（Swap）再回收。
- 回收策略：
  - 惰性回收（kswapd 进程）：后台异步回收，避免内存突然耗尽；
  - 直接回收：当内存分配时无空闲页，进程触发直接回收（可能导致进程阻塞）。

##### 2. 交换分区（Swap）

- 定义：磁盘上的一块特殊区域，作为物理内存的「备份空间」；
- 作用：当物理内存不足时，内核将不常用的匿名页写入 Swap，释放物理内存给活跃进程；
- 注意事项：
  - Swap 速度远慢于物理内存，过度使用会导致系统卡顿（Swap Thrashing）；
  - 建议 Swap 大小为物理内存的 1~2 倍（内存较大时可适当减小）。

#### 五、缓存策略：Page Cache 与 Buffer Cache

Linux 利用「缓存」提升 I/O 效率，核心缓存类型：
- **Page Cache（页缓存）**：
  - 缓存磁盘文件的内容（如读取的文件数据），进程再次访问时直接从内存读取，避免磁盘 I/O；
  - 写操作时采用「写回（Write Back）」策略：数据先写入 Page Cache，内核异步刷盘（`pdflush` 进程），提升写效率。
- **Buffer Cache（缓冲区缓存）**：
  - 缓存磁盘块数据（如文件系统元数据、磁盘扇区数据），与 Page Cache 在 Linux 2.4 后已合并，统一由页缓存管理；
- 缓存回收：当物理内存紧张时，内核优先回收访问频率低的缓存页（LRU 算法：最近最少使用）。

#### 六、面试高频考点总结

##### 1. 核心机制题

- 虚拟内存的作用？（进程隔离、突破物理内存限制、简化编程）；
- 分页机制的优势？（解决内存碎片、便于交换和共享）；
- 内核态内存分配用什么？（伙伴系统 + slab 分配器）；
- 用户态 `malloc()` 底层实现？（`brk()`/`mmap()` + 缺页中断）。

##### 2. 实践问题

- 如何查看 Linux 内存使用情况？（`free -h`、`top`、`vmstat`、`cat /proc/meminfo`）；
-  Swap 使用率过高怎么办？（增加物理内存、优化应用减少内存占用、调整 `swappiness` 参数降低交换倾向）；
- 内存泄漏如何排查？（`ps` 查看进程内存增长、`pstack` 分析栈、`valgrind` 检测内存泄漏）。

##### 3. 关键参数

- `swappiness`：控制内存交换倾向（0~100，值越小越倾向于使用物理内存，默认 60）；
- `overcommit_memory`：内存过度分配策略（0=默认，1=允许过度分配，2=禁止过度分配）。

#### 总结

Linux 内存管理的核心是「虚拟内存 + 分页机制」，通过分层分配（伙伴系统、slab、VMA）、高效回收（LRU、kswapd）、缓存优化（Page Cache）实现内存的高效利用。面试中需重点掌握「虚拟内存原理、内存分配流程、回收与交换机制」，并结合工具使用和实际问题排查，体现对底层逻辑的理解。