## VisualVM

### **Heap Dump**

#### 一、翻译

**Heap Dump**

- 中文标准翻译：**堆转储文件**
- 通俗理解：**堆内存快照 / 堆内存镜像**

---
#### 二、是什么意思（大白话）

1. **Heap**：JVM 堆内存，所有对象、集合、字符串都存在这里
2. **Dump**：导出、抓取、镜像备份

✅ **Heap Dump 含义：**
**某一时刻，把当前 JVM 堆里「所有对象、对象数量、占用内存、引用关系、泄漏对象」全部完整抓取保存下来的文件。**

文件后缀：`.hprof`

---
#### 三、作用（面试+实战必记）

用来分析：
1. **内存泄漏**（静态集合强引用、无效对象无法回收）
2. **OOM 内存溢出**
3. 哪个对象占用内存最大
4. 谁持有引用导致 GC 回收不掉
5. 排查大对象、不合理缓存

---
#### 四、VisualVM 里操作

在 VisualVM 中：
- 对应按钮：**堆 Dump / Heap Dump**
- 点击瞬间：立刻冻结当前堆内存，生成快照文件
- 打开后可查看：
  - 类实例数量
  - 对象占用大小
  - 引用链（排查泄漏核心）

---
#### 五、区分两个易混概念

1. **Heap Dump**：堆快照 → 查内存泄漏、OOM
2. **Thread Dump**：线程快照 → 查死锁、线程阻塞、CPU 飙高

---
#### 六、极简背诵版

Heap Dump：**堆内存快照，记录当前所有对象与引用，用于分析内存泄漏、OOM 问题。**

### Perform GC

结论：**VisualVM 里的 Perform GC ≈ 代码里的 System.gc()，会触发 Full GC**。

#### 1. 按钮位置

打开 VisualVM → 选进程 → Monitor 标签 → 有个 **Perform GC** 按钮（中文叫“执行垃圾回收”）。

#### 2. 等价代码

点这个按钮，本质是通过 JMX 调用：
```java
System.gc();
```
- **官方说明**：建议 JVM 做一次“全堆收集”（包括新生代+老年代+元空间）。
- **日志表现**：GC 日志里会看到：
  ```
  Full GC (System.gc())
  ```

#### 3. 为什么是 Full GC

- `System.gc()` **不强制**，但几乎所有 JVM 实现（HotSpot）都会**做 Full GC**。
- 它会：
  - 回收新生代（Young）
  - 回收老年代（Old）
  - 回收元空间（Metaspace/PermGen）
  - 可能做压缩（取决于 GC 算法）

#### 4. 和 Young GC 的区别

- Young GC（Minor GC）：只清新生代，**不碰老年代**，速度快。
- Perform GC → Full GC：**全堆扫描**，暂停时间长，生产环境慎用。

#### 5. 生产环境建议

- **不要随便点**：会引发长时间 STW（Stop-The-World），影响业务。
- 想手动触发又怕 Full GC：
  ```java
  // 只建议 Young GC（HotSpot 不一定完全遵守）
  System.runFinalization();
  ```

#### 6. 怎么验证

1. 开启 GC 日志：
   ```
   -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
   ```
2. 点 Perform GC
3. 看日志：出现 `Full GC (System.gc())` → 确认是 Full GC。

要不要我帮你整理一份VisualVM监控与GC日志分析的快速参考清单？

## Jstack

### `jstack <pid>` 到底是什么？

#### 一句话总结

`jstack <pid>` 是 **JDK 自带的命令行工具**，作用是：
**打印指定 Java 进程（PID）的所有线程当前时刻的堆栈快照**。
主要用来 **排查 Java 程序卡死、CPU 高、死锁、线程阻塞** 等问题。

---

#### 它能做什么？（最常用场景）

1. **查死锁**（线程互相等待，程序卡住不动）
2. **查 CPU 100%**（哪个线程疯狂占用资源）
3. **查线程阻塞/等待**（程序慢、假死）
4. **看所有线程在干什么**（运行、等待、锁等待）

---

#### 怎么用？

##### 1. 先找到 Java 进程 PID

```bash
jps -l
```

会输出：

```
12345 com.example.Application
```

`12345` 就是 PID。

##### 2. 执行 jstack

```bash
jstack 12345
```

直接输出**当前所有线程的调用栈**。

##### 3. 保存到文件方便分析

```bash
jstack 12345 > thread.log
```

---

#### 输出里能看到什么？

你会看到类似这样的信息：

```
"main" #1 prio=5 os_prio=0 tid=0x00007f8... nid=0x123 waiting on condition
java.lang.Thread.sleep(Native Method)
com.example.Test.main(Test.java:10)

"线程-1" #2 waiting for monitor entry  <-- 等待锁
```

##### 关键状态你要认识

- `RUNNABLE`：正在运行
- `WAITING` / `TIMED_WAITING`：在等待
- `BLOCKED`：**被阻塞，等锁**（大概率性能问题）
- `waiting for monitor entry`：**死锁/锁竞争高危信号**

---

#### 最经典用法：排查死锁

执行后，jstack 会**自动在底部标出死锁**：

```
Found one Java-level deadlock:
=============================
"线程A": waiting to lock <0x123>
"线程B": waiting to lock <0x456>
```

直接告诉你**哪两个线程、哪两行代码死锁**。

---

#### 小总结

- **jstack = Java 线程堆栈查看工具**
- **用途**：排查死锁、CPU 高、程序卡死、线程阻塞
- **命令**：`jstack <Java进程PID>`
- **输出**：所有线程当前正在执行的方法调用栈

需要我教你 **一步步排查 CPU 100% 或死锁** 的实战流程吗？

## jmap

### jmap 常用命令（面试必背+实战）

`jmap`：JDK 自带，**分析堆内存、对象、堆快照**，排查内存溢出、内存泄漏。

#### 1. jmap -heap PID

查看**堆内存整体信息**
- 新生代、老年代、元空间大小/使用率
- GC收集器类型
- Xms/Xmx 配置
> 作用：快速看堆内存现状、内存是否爆满

#### 2. jmap -dump:format=b,file=heap.hprof PID

**导出堆快照文件**（最重要）
```bash
jmap -dump:format=b,file=heap.hprof 12345
```
- 生成 hprof 文件
- 后续用 MAT、JProfiler、IDEA 分析
> 作用：排查**内存泄漏、OOM、大对象**

#### 3. jmap -histo PID

查看**堆内对象统计**
- 类名、实例数量、占用内存大小
- 按内存排序
```bash
# 只看存活对象
jmap -histo:live PID
```
> 作用：快速定位哪个类对象过多、内存暴涨

#### 4. jmap -clstats PID

打印**类加载器统计**
- 各类加载器加载类数量、内存占用
> 作用：排查元空间溢出、类加载泄漏

#### 5. jmap -finalizerinfo PID

查看**等待finalize的对象**
> 作用：排查finalize阻塞导致内存回收异常

---

#### 高频面试总结（背诵版）

1. `jmap -heap`：查看堆内存配置与各代使用情况，静态快照。
2. `jmap -histo`：查看堆中对象数量与内存占用，快速定位大对象。
3. `jmap -dump`：导出堆dump文件，线下分析内存泄漏、OOM。
4. `jmap -clstats`：类加载器与元空间分析。

---

#### 组合排查套路（线上实操）

1. `jstat -gc` 发现 FullGC 频繁
2. `jmap -heap` 看老年代是否占满
3. `jmap -histo:live` 找异常大对象
4. `jmap -dump` 导出文件，MAT 深度分析

## jstat

### jstat 常用命令（面试+线上实战 精简背诵版）

`jstat`：JDK 轻量实时监控，**无侵入、看GC、类加载、编译**。
格式：`jstat [选项] PID 间隔ms 次数`

---

#### 1. 最常用：jstat -gc PID 1000

**核心：全程GC监控（面试必考）**
输出：
- S0C/S1C、EC、OC、MC：各区容量
- S0U/S1U、EU、OU、MU：各区已用
- YGC/YGT：年轻GC次数、耗时
- FGC/FGCT：FullGC次数、耗时
- GCT：总GC耗时
用途：排查 **GC频繁、FullGC过多、内存压力**

---

#### 2. jstat -gcutil PID 1000

**极简百分比视图，线上最常用**
只展示：
各内存区域**使用率** + YGC/FGC + 耗时
适合快速看：老年代是否爆满、是否频繁FullGC

---

#### 3. jstat -gccapacity PID 1000

查看**堆内存各代容量大小**
新生代、老年代、元空间 最大/最小/当前容量
适合分析：堆分区大小配置是否合理

---

#### 4. jstat -gcnew PID 1000

专注**新生代GC**
查看Eden、Survivor 变化、Minor GC频率
排查：新生代回收过快、对象晋升频繁

---

#### 5. jstat -gcold PID 1000

专注**老年代**
观察老年代增长速度，预判OOM、内存泄漏

---

#### 6. jstat -class PID 1000

监控**类加载**
加载数量、卸载数量、耗时
排查：元空间溢出、动态类加载泄漏

---

#### 7. jstat -compiler PID 1000

JIT即时编译耗时、编译任务数
排查：代码编译过慢、线上卡顿

---

#### 面试极简背诵

1. `-gc`：完整GC明细，看各代内存、GC次数与耗时
2. `-gcutil`：使用率百分比，线上快速排查首选
3. `-gcnew / -gcold`：分别监控新生代、老年代
4. `-class`：类加载统计，排查元空间问题
5. 用法：`jstat -gcutil pid 1000` 每秒刷新一次

---

#### 线上排查固定组合

- 看GC频率：`jstat -gcutil pid 1000`
- 看内存细节：`jstat -gc pid 1000`
- 配合：jmap 堆快照、jstack 线程堆栈 联合定位OOM/卡顿

## Arthas

### Arthas 常用命令（面试+线上排查 必背精简版）

Arthas：阿里开源 Java 线上诊断神器，**无需重启、无侵入、实时排查**。

#### 一、基础环境

```bash
# 启动
java -jar arthas-boot.jar
```

---

#### 二、系统全局查看

1. **dashboard**
全局面板：CPU、内存、线程、GC、堆栈 总览，实时刷新。

2. **thread**
```bash
thread         # 查看所有线程
thread -n 5    # 打印 top5 高CPU线程
thread -b      # 一键检测死锁
```

3. **jvm**
查看 JVM 参数、堆配置、GC收集器、系统环境。

---

#### 三、线上调试（高频）

1. **watch**
**监听方法入参、出参、异常**，线上排错第一神器
```bash
watch 全类名 方法名 '{params,returnObj,throwExp}'
```

2. **trace**
追踪方法调用链路，**看耗时**，定位慢接口、慢SQL
```bash
trace 全类名 方法名
```

3. **stack**
打印方法调用堆栈，定位代码调用来源。

4. **tt**
时间隧道：记录方法每次调用记录，可回放、查看入参出参。

---

#### 四、热更新 / 反编译

1. **jad**
反编译线上运行代码，快速核对线上版本是否和本地一致
```bash
jad 全类名
```

2. **mc / redefine**
- `mc`：内存编译Java文件
- `redefine`：**热更新代码**，不用重启服务

---

#### 五、内存 & GC 排查

1. **memory**
查看堆、非堆、直接内存使用。
2. **heapdump**
导出堆快照，等同于 jmap dump。
3. **gc**
手动触发 FullGC（谨慎生产）。

---

#### 六、耗时&性能

1. **monitor**
统计方法 调用次数、成功率、平均耗时、QPS。
2. **profiler**
火焰图分析，精准定位 CPU 热点代码。

---

#### 面试极简背诵版

1. **dashboard**：系统全局监控
2. **thread**：查高CPU线程、死锁
3. **watch**：监听方法入参/返回值/异常
4. **trace**：追踪方法链路、排查慢接口
5. **jad**：线上反编译看代码
6. **heapdump**：导出堆文件分析内存泄漏
7. **profiler**：火焰图分析CPU耗时

---

#### 线上排查万能组合

- 接口慢：`trace`
- 报错未知：`watch`
- CPU飙高：`thread -n`
- 代码不一致：`jad`
- 内存泄漏：`heapdump`