### 一、 线上故障排查链路（核心必考，定薪关键）

这部分是 20k+ 面试的重灾区，面试官通常会给你一个线上场景（如 CPU 飙高、内存 OOM、请求变慢），让你说排查步骤。

#### 1. CPU 飙高排查链路
*   **核心命令：** `top` -> `top -Hp <pid>` -> `printf "%x\n" <tid>` -> `jstack <pid> | grep <hex_tid>`
*   **高分考点：**
    *   明确 `top` 命令中 `us`（用户态 CPU）、`sy`（内核态 CPU）、`wa`（IO 等待）的含义。如果 `wa` 高，说明是磁盘 IO 瓶颈，不是 CPU 瓶颈。
    *   理解 Load Average（1/5/15分钟负载）不仅与 CPU 有关，还包含等待磁盘 IO 和网络的进程（D 状态进程）。
    *   能说出死循环（us 高）和频繁 GC（sy 高，上下文切换多）在 CPU 表现上的区别。

#### 2. 内存异常排查链路
*   **核心命令：** `free -m` -> `top` -> `jmap -heap/-histo` -> `jmap -dump`
*   **高分考点：**
    *   看懂 `free` 命令中的 `buff/cache` 和 `available`，不要把 cache 占用高误认为是内存泄漏（Linux 会尽量利用内存做缓存）。
    *   理解 Java 进程的 RSS（实际物理内存占用）远大于 Heap 大小，因为还包括 Metaspace、线程栈、Direct Memory（堆外内存）、JIT 编译缓存等。
    *   **进阶：** 讲清楚 Linux 的 OOM Killer 机制（`dmesg -T | grep -i oom`），当系统内存耗尽时，内核如何根据 `oom_score` 杀进程。

#### 3. 磁盘与 IO 排查
*   **核心命令：** `df -h`（看容量） -> `iostat -x 1`（看 IO 延迟和吞吐） -> `iotop`（找具体进程）
*   **高分考点：**
    *   **Inode 耗尽故障：** 磁盘空间 (`df -h`) 还有剩余，但无法创建新文件，因为 inode (`df -i`) 被大量小文件耗尽（常见于大量碎片化日志或缓存文件）。
    *   `iostat` 中的 `await`（响应时间）和 `%util`（使用率）的判断标准。

#### 4. 网络异常排查
*   **核心命令：** `netstat` / `ss` -> `tcpdump` -> `traceroute` / `mtr`
*   **高分考点：**
    *   能用 `ss -s` 快速查看 TCP 连接状态分布。大量 `CLOSE_WAIT` 说明程序 Bug（被动关闭方没调用 close），大量 `TIME_WAIT` 说明并发短连接太多（需调优内核参数或改用长连接）。
    *   抓包分析：`tcpdump -i eth0 port 8080 -nn -A` 结合 Wireshark 分析三次握手和数据包。

---

### 二、 进程管理与信号机制（关联 JVM 与容器）
Java 应用本质上是 Linux 上的一个进程，理解进程机制对理解 JVM 和 Docker 至关重要。

*   **进程状态：** 理解 R（运行）、S（睡眠）、D（不可中断睡眠，通常是 IO 等待）、Z（僵尸进程）。
*   **信号机制：**
    *   `kill -9` (SIGKILL) 与 `kill -15` (SIGTERM) 的本质区别。**重点：** JVM 是如何捕获 `-15` 信号执行 `ShutdownHook`（优雅停机）的？而 `-9` 是内核直接强制回收，Java 应用无法拦截，会导致数据丢失。
    *   `kill -3` (SIGQUIT) 可以让 JVM 打印当前的线程快照（等同于 `jstack`）。
*   **守护进程：** 了解 `nohup` 和 `&` 的原理，以及为什么脱离终端后进程不会随着 SSH 断开而退出（SIGHUP 信号被忽略）。

---

### 三、 IO 模型与网络底层（关联 Netty / Redis）
20k 面试必问底层 IO，需要结合 Linux 内核机制来讲。

*   **文件描述符：** 什么是 FD？`ulimit -n` 控制单进程最大文件句柄数，著名的 `Too many open files` 报错原因（可能是连接数太多或文件未关闭）。
*   **五大 IO 模型：** 阻塞 IO、非阻塞 IO、IO 多路复用、信号驱动 IO、异步 IO。
*   **IO 多路复用底层：**
    *   `select`：有 1024 个 FD 限制，轮询效率低。
    *   `poll`：打破 1024 限制，但依然是轮询。
    *   `epoll`：**核心考点！** 事件驱动（回调机制），O(1) 效率。讲清楚 `epoll_create`、`epoll_ctl`、`epoll_wait` 三个核心系统调用，以及红黑树和双向链表的数据结构。
    *   **关联：** Netty 底层为什么默认用 Epoll？Redis 为什么是单线程却很快？

---

### 四、 容器化底层与隔离机制（K8s/Docker 环境）
现在的 Java 部署几乎都在 Docker/K8s 中，面试官非常看重你对容器底层 Linux 技术的理解。

*   **Linux Namespaces（隔离）：**
    *   PID Namespace：为什么在容器内看 Java 进程 PID 是 1，而在宿主机看是几十万？
    *   Net Namespace：容器拥有独立的虚拟网卡（veth pair）和 IP。
*   **Cgroups（资源限制）：**
    *   Docker 是如何限制容器 CPU 和内存的？（底层就是 Cgroups）。
    *   **踩坑经验（20k 加分项）：** 讲讲早期 JDK 8 在 Docker 中不识别 Cgroups 限制，导致 JVM 按宿主机总内存计算 MaxHeapSize，从而引发 OOM Killed 的问题。以及 `XX:+UseContainerSupport` 参数的作用。
*   **OverlayFS（联合文件系统）：** Docker 镜像分层技术的底层原理。

---

### 五、 内核参数调优
能说出几个生产环境中常用的内核参数优化，会让面试官觉得你有实战经验。

*   **网络相关：**
    *   `net.core.somaxconn`：全连接队列大小。如果 Tomcat / Nginx 并发高但建立连接慢，可能是这个值太小（配合应用端的 `backlog`）。
    *   `net.ipv4.tcp_max_tw_buckets` 和 `tcp_tw_reuse`：解决高并发短连接下的 `TIME_WAIT` 过多问题。
    *   `net.ipv4.ip_local_port_range`：向外发起连接时的本地端口范围。如果作为高并发网关，端口耗尽无法新建连接需调大此范围。
*   **内存相关：**
    *   `vm.swappiness`：控制 swap 倾向。Java 服务强烈建议设为 0 或 1，禁止使用 Swap，否则 JVM GC 停顿会导致灾难性后果。

---

### 六、 高级文本处理与 Shell 脚本（基本功体现）
不需要你写几百行的复杂脚本，但必须能熟练组合命令处理日志。

*   **高频组合：**
    *   找出访问量 Top 10 的 IP：`awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 10`
    *   统计 ERROR 出现次数：`grep "ERROR" app.log | wc -l`
    *   找出占用空间最大的目录：`du -sh /* | sort -nr`
*   **核心命令原理：** 知道 `grep` 配合正则、`awk` 按列处理、`sed` 流编辑、`xargs` 管道传参。

### 💡 总结备考建议

对于 20k+ 的面试，准备 Linux 知识时**不要去背命令参数**，而是要建立**“场景 -> 命令 -> 原理”**的脑回路。
例如：面试官问“线上接口大面积超时怎么排查？”
你的思路应该是：先 `top` 看系统负载和 CPU -> 再 `iostat` 看是不是磁盘 IO 瓶颈导致 DB 慢 -> 再 `ss` 看是不是连接数打满 -> 再 `tcpdump` 抓包看网络延迟。把 Linux 命令当作你排查问题的“手术刀”，而不是孤立的知识点。