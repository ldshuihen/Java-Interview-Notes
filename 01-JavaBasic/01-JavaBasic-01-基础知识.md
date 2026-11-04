# Java 语言基础核心语法、内存模型与高频考点全景手册

> **文档定位与知识库联动说明**：
> - 本文档专注于 **Java 语言底座、编译与类加载生命周期、JVM 内存分布、八大基本数据类型与包装类、核心关键字、运算符与类型转换、流程控制、方法与值传递机制、基础数组/链表/系统类以及 Lambda 与方法引用**。
> - 同目录进阶专题请联动参阅：
>   - 面向对象深水区与设计原则：[`01-JavaBasic-02-面向对象.md`](./01-JavaBasic-02-面向对象.md)
>   - 集合容器源码与并发容器：[`01-JavaBasic-02-集合框架.md`](./01-JavaBasic-02-集合框架.md)
>   - 异常体系与 IO 流深潜：[`01-JavaBasic-03-异常处理-IO流.md`](./01-JavaBasic-03-异常处理-IO流.md)
>   - 字符串常量池、数组底层内存布局与标准 I/O：[`01-JavaBasic-04-字符串-数组-输入输出.md`](./01-JavaBasic-04-字符串-数组-输入输出.md)
>   - 枚举、泛型、注解与深浅拷贝：[`01-JavaBasic-04-枚举-泛型-注解-拷贝.md`](./01-JavaBasic-04-枚举-泛型-注解-拷贝.md)
>   - 反射、动态代理与虚拟线程：[`01-JavaBasic-04-反射-虚拟线程.md`](./01-JavaBasic-04-反射-虚拟线程.md)
>   - Stream 流式编程：[`01-JavaBasic-04-StreamAPI.md`](./01-JavaBasic-04-StreamAPI.md)

---

## 一、Java 语言特性、程序生命周期与内存基石

### 1.1 Java 语言核心特性与跨平台原理

#### 1. 为什么说 Java 是“平台无关的编程语言”？（Write Once, Run Anywhere）
- **核心机制**：Java 源代码（`.java`）并不直接编译成特定操作系统/CPU 架构的本地机器码，而是经由 `javac` 编译器编译为**平台中立的中间字节码文件（`.class`）**。
- **JVM 充当跨平台翻译官**：Windows、Linux、macOS 等不同操作系统只需安装各自平台专属的 **JVM（Java Virtual Machine）**，由 JVM 负责在运行期通过**解释器（Interpreter）+ JIT 即时编译器（Just-In-Time Compiler）**将同一份字节码翻译为当前宿主机的机器指令执行。
- **面试一句话总结**：**字节码（`.class`）是统一的跨平台通行证；不是 Java 语言本身跨平台，而是不同操作系统上的 JVM（自身不跨平台，由 C/C++ 针对各系统定制开发）托举起了 Java 程序的跨平台能力。**

#### 2. `JDK`、`JRE` 与 `JVM` 三者的包含关系与职责

```text
┌──────────────────────────────────────────────────────────┐
│ JDK (Java Development Kit - 开发者工具包)                  │
│   ├── 开发编译与诊断工具：javac, java, javap, jps, jstack...│
│   └── ┌──────────────────────────────────────────────┐   │
│       │ JRE (Java Runtime Environment - 运行时环境)    │   │
│       │   ├── 核心基础类库（rt.jar / JDK 9+ JMod 模块） │   │
│       │   └── ┌──────────────────────────────────┐   │   │
│       │       │ JVM (Java 虚拟机：类加载/内存/GC)  │   │   │
│       │       └──────────────────────────────────┘   │   │
│       └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

| 组件 | 英文全称 | 核心组成与职责 | 面向人群/场景 |
| :--- | :--- | :--- | :--- |
| **`JVM`** | Java Virtual Machine | 加载 `.class` 字节码、管理运行时数据区内存、执行垃圾回收（GC）并将字节码翻译为机器码 | 字节码执行底座 |
| **`JRE`** | Java Runtime Environment | **`JVM` + Java 核心标准类库**（如 `java.lang`、`java.util` 等），提供运行已编译 Java 程序的最小环境 | 仅运行程序（JDK 11+ 起官方不再单独分发独立 JRE，推荐用 `jlink` 按需裁剪运行时镜像） |
| **`JDK`** | Java Development Kit | **`JRE` + 编译与诊断工具链**（`javac` 编译器、`javap` 反编译、`jmap/jstat/jstack` 排障工具等） | Java 程序开发与生产排障 |

#### 3. Java 与 C++ 的七大核心区别（高频面试对比）

| 对比维度 | Java | C++ |
| :--- | :--- | :--- |
| **平台相关性** | 编译为 `.class` 字节码，依赖 JVM 实现跨平台 | 直接编译为目标平台本地机器码，平台强相关 |
| **内存管理** | **JVM 自动垃圾回收（GC）**，无需手动释放内存 | 需程序员手动管理（`new/delete`、`malloc/free`），易出现悬垂指针或内存泄漏 |
| **指针操作** | **无显式指针**（封装为安全的对象引用，禁止直接算术偏移操作内存） | 支持裸指针与指针算术运算，可直接读写物理虚拟内存地址 |
| **继承模型** | **类单继承 + 接口多实现**（彻底规避菱形继承二义性） | 支持类的多重继承 |
| **语法特性** | 语法纯净：无运算符重载、无析构函数、无全局变量（全封装在类中） | 支持运算符重载、析构函数、宏定义与多重编程范式 |
| **执行方式与效率** | 解释执行 + JIT 热点代码编译优化（启动预热后吞吐逼近原生代码） | AOT 直接编译为机器码，零虚拟机开销，极致贴近硬件性能 |
| **主流应用场景** | 大型分布式后端、微服务、大数据生态、Android 应用 | 操作系统内核、游戏引擎、音视频编解码、高频交易、嵌入式底层 |

#### 4. Java 是静态类型语言还是动态类型语言？
- **结论**：Java 是典型的**静态强类型语言（Statically & Strongly Typed Language）**。
- **依据**：所有变量必须先声明具体类型才能使用，**编译期即强制执行严格的类型安全检查**（不像 Python / JavaScript 在运行期才动态决定变量类型）；同时，Java 通过**反射（Reflection）、字节码增强（ASM/CGLIB）与 `invokedynamic` 指令**具备了强大的运行期动态扩展能力。

---

### 1.2 `javac` 编译与 `java` 运行命令实战（无包 vs 带包结构避坑）

#### 1. 无包结构（单文件极简模式）
```bash
# 1. 编译：将源文件编译为同目录下的 Hello.class（源文件含中文时务必指定 -encoding UTF-8）
javac -encoding UTF-8 Hello.java

# 2. 运行：启动 JVM 加载主类（⚠️ 切忌加 .class 后缀！只写类名 Hello）
java Hello
```

#### 2. 带 `package` 包结构的标准编译与运行（新手最易踩坑）
假设源文件 `src/com/demo/Hello.java` 首行声明了 `package com.demo;`：
```bash
# 1. 编译：用 -d 指定字节码输出根目录，javac 会自动在 ./bin 下按包名生成 bin/com/demo/Hello.class
javac -encoding UTF-8 -d ./bin src/com/demo/Hello.java

# 2. 运行：用 -cp（或 -classpath）指向包结构的【根目录 ./bin】，并传入【包名+类名（全限定类名）】
java -cp ./bin com.demo.Hello
```

#### 3. 常用命令参数与高频报错速查表

| 命令 / 报错现象 | 参数或根本原因 | 标准解决方案 |
| :--- | :--- | :--- |
| **`javac -d <dir>`** | 指定编译生成的 `.class` 文件输出根目录 | 自动按 `package` 声明创建子目录树 |
| **`javac -encoding UTF-8`** | 指定读取 `.java` 源文件的字符编码 | 解决 Windows 默认 GBK 环境下编译 UTF-8 源码报“编码 GBK 的不可映射字符”乱码问题 |
| **`java -cp <path>`** | 指定类搜索路径（ClassPath） | 多个路径拼接时：**Windows 用分号 `;` 分隔，Linux / macOS 用冒号 `:` 分隔** |
| **报错 `Could not find or load main class`** | ① 运行时误加了 `.class` 后缀；② 带包类未写全限定类名；③ `-cp` 未指向包结构根目录 | 运行命令只写全限定类名（如 `com.demo.Hello`），且 `-cp` 指向 `com` 所在的上级根目录 |

---

### 1.3 Java 程序完整生命周期（编译期 $\rightarrow$ 类加载期 $\rightarrow$ 运行期）

```text
.java 源码 ──(javac 编译期)──> .class 字节码 ──(java 启动 JVM 进程)──> 【类加载 5 阶段：加载→验证→准备→解析→初始化】
                                                                            ↓
JVM 进程退出（所有非守护线程终止） <── 多线程业务执行 & GC 守护回收 <── 主线程调用 main() 压入栈帧执行
```

#### 1. 三大核心时期拆解
1. **编译期（Compile Time）**：由 `javac` 执行词法/语法分析、语义检查、**泛型类型擦除**、**编译期常量折叠**与语法糖解糖（如自动拆装箱、`foreach`、Lambda），生成 `.class` 文件（此时 JVM 尚未启动）。
2. **类加载期（Class Loading，大厂必考 5 阶段）**：
   - **① 加载（Loading）**：通过类的全限定名读取 `.class` 二进制字节流，在**元空间（方法区）**构建类的底层元数据（`InstanceKlass`），并在 **Java 堆**中创建唯一对应的 `java.lang.Class` 镜像对象作为反射访问入口。
   - **② 验证（Verification）**：校验字节码格式（魔数 `0xCAFEBABE`、版本号）、元数据语义、字节码指令流与符号引用安全性。
   - **③ 准备（Preparation，⚠️ 高频坑点）**：为类的 **`static` 静态变量分配内存并赋类型的“零值”（默认初始值）**。
     - 例如 `public static int num = 10;` 在准备阶段结束后 `num` 的值是 **`0` 而不是 `10`**（赋值为 `10` 的指令要等到第 5 步初始化阶段才执行）！
     - **唯一例外**：若被 `static final` 修饰且为编译期常量（如 `public static final int MAX = 10;`），字段表中带有 `ConstantValue` 属性，**在准备阶段即直接赋值为 `10`**。
   - **④ 解析（Resolution）**：将常量池中的**符号引用**（字符串形式的类名、方法名、字段描述符）替换为指向内存真实地址或偏移量的**直接引用**。
   - **⑤ 初始化（Initialization）**：执行编译器自动收集所有 `static` 变量显式赋值语句与 `static {}` 静态代码块合并生成的类构造器 **`<clinit>()` 方法**（JVM 底层自动加锁保证多线程下 `<clinit>()` 只执行一次）。
3. **运行期与终止期（Runtime & Termination）**：
   - 启动 `main` 主线程，每次调用方法在当前线程的**虚拟机栈**压入一个栈帧，方法返回则弹出栈帧；
   - **JVM 进程退出铁律**：JVM 内的线程分为**用户线程（User Thread）**和**守护线程（Daemon Thread，如 GC 线程）**。**只要还有任意一个用户线程未结束，即便 `main` 主线程已经执行完毕，JVM 进程也不会退出**；唯有当所有用户线程执行完毕、或调用 `System.exit(status)`、或被 OS `kill -9` 强杀时，JVM 进程才终止并释放全部内存。

#### 2. 类加载器三层体系与双亲委派模型
- **三层类加载器（JDK 8 vs JDK 9+ 演进）**：
  1. **启动类加载器（`Bootstrap ClassLoader`）**：C++ 实现，Java 中获取为 `null`，负责加载 `<JAVA_HOME>/lib` 核心基础类（如 `java.lang.*`）；
  2. **扩展/平台类加载器（`Extension ClassLoader`，JDK 9+ 升级为 `Platform ClassLoader`）**：负责加载扩展模块类库；
  3. **应用类加载器（`Application ClassLoader / AppClassLoader`）**：`ClassLoader.getSystemClassLoader()` 的默认返回值，负责加载用户 `classpath` 下的应用类与第三方依赖包。
- **双亲委派模型（Parents Delegation Model）**：
  - **工作流程**：收到类加载请求时，当前类加载器**先不自己加载，而是递归委派给父类加载器**，直到顶层 `Bootstrap ClassLoader`；只有当父加载器在自己的搜索范围内找不到该类时，子加载器才尝试自己加载。
  - **两大核心价值**：① **防止核心 API 被篡改**（例如用户伪造 `java.lang.String`，委派到顶层会直接返回 JDK 核心库的 `String`，杜绝恶意代码注入）；② **避免类的重复加载**，保证同一个类在 JVM 中的全局唯一性（判断两个类是否相同，必须满足**同一 `.class` 文件 + 同一个 `ClassLoader` 实例加载**）。

#### 3. 生命周期高频面试追问
- **Q1：哪些操作会触发类的初始化（主动引用）？哪些不会（被动引用）？**
  - **主动引用（必定触发 `<clinit>`）**：① `new` 创建实例；② 访问或修改类的非编译期常量 `static` 字段或调用 `static` 方法；③ `Class.forName("...")` 反射调用；④ 初始化子类时若父类未初始化，先触发父类初始化；⑤ JVM 启动时标明的主类（含 `main()` 的类）。
  - **被动引用（绝不触发初始化）**：① **通过子类引用父类的静态变量**（只会触发父类初始化，子类不会初始化）；② **通过数组定义引用类**（如 `User[] arr = new User[10];` 不触发 `User` 初始化）；③ **访问类的 `static final` 编译期常量**（编译期已内联进调用方常量池，甚至不触发类加载）。
- **Q2：`<clinit>()` 和 `<init>()` 有什么本质区别？**
  - **`<clinit>()`（类构造器）**：由编译器合并所有 `static` 变量赋值与 `static {}` 块生成，在**类加载的初始化阶段执行，全局仅执行一次**；
  - **`<init>()`（实例构造器）**：由编译器合并非静态成员变量显式赋值、`{}` 实例构造代码块与构造方法代码生成，**每次 `new` 对象时都会执行一次**（且首行默认先调父类 `<init>()`）。

---

### 1.4 Java 类双层完整结构（源码组成 + `.class` 字节码 10 大结构）

#### 1. 源码层面：Java 类的 6 大成员要素
```java
public class OrderService {
    // 1. 成员变量（Field）：实例变量（随对象存堆） vs static 静态变量（属类，随 Class 对象存堆）
    private Long orderId;
    public static final String SERVICE_NAME = "ORDER_CORE";

    // 2. 静态代码块：类加载初始化阶段（<clinit>）执行，仅执行 1 次
    static { System.out.println("1. 静态代码块执行"); }

    // 3. 实例代码块：每次 new 对象时在构造方法体之前（内联进 <init>）执行
    { System.out.println("2. 实例代码块执行"); }

    // 4. 构造方法（Constructor）：与类同名、无返回类型（连 void 都不能写），用于初始化对象
    public OrderService() { System.out.println("3. 构造方法执行"); }

    // 5. 成员方法（Method）：实例方法 vs 静态方法
    public void createOrder() {}

    // 6. 内部类（Inner Class）：成员内部类、静态内部类、局部内部类、匿名内部类
    public static class OrderContext {}
}
```

#### 2. JVM 底层：`.class` 字节码文件 10 大紧凑二进制结构（手写/口述重点）

```text
ClassFile {
    u4             magic;                  // 1. 魔数：固定 0xCAFEBABE（4字节），识别合法字节码文件
    u2             minor_version;          // 2. 次版本号
    u2             major_version;          // 3. 主版本号（JDK8=52, JDK11=55, JDK17=61, JDK21=65；高版本JVM向下兼容）
    u2             constant_pool_count;    // 4. 常量池计数器（从 1 开始编号）
    cp_info        constant_pool[];        // 5. 【核心最大区】Class 常量池：存放字面量（文本/final常量）+ 符号引用（类/字段/方法描述符）
    u2             access_flags;           // 6. 类访问标志（public / final / abstract / interface / enum）
    u2             this_class;             // 7. 当前类索引（指向常量池）
    u2             super_class;            // 8. 父类索引（除 Object 为 0 外均指向常量池）
    u2             interfaces_count;       // 9. 接口计数与 interfaces[] 接口索引表
    field_info     fields[];               // 10. 字段表集合（声明的所有 static/实例字段定义，不含实例运行期值）
    method_info    methods[];              // 11. 方法表集合（含 <init>、<clinit> 及普通方法，内部 Code 属性存放真实字节码指令）
    attribute_info attributes[];           // 12. 附加属性表（Code、LineNumberTable 行号映射、SourceFile、Exceptions 等）
}
```

---

### 1.5 JVM 运行时内存划分与变量存储位置（含 `ThreadLocal` 内存泄漏）

#### 1. JVM 五大运行时数据区与变量存储全景对照表

| 内存区域 | 线程私有/共享 | 核心存储内容 | 变量存储归属与高频面试纠偏 |
| :--- | :---: | :--- | :--- |
| **虚拟机栈（JVM Stack）** | **线程私有** | 方法调用栈帧（局部变量表、操作数栈、动态链接、方法返回地址） | 存放**局部基本类型变量的值**（如 `int a = 10`）和**局部对象引用变量**（存堆中对象的内存地址，栈上绝不直接存 `new` 出的未逃逸外对象本体；生命周期随方法出栈立即销毁） |
| **本地方法栈（Native Method Stack）** | **线程私有** | 为 `native` 本地方法（C/C++ JNI 调用）服务 | HotSpot 虚拟机将本地方法栈与虚拟机栈合二为一 |
| **程序计数器（PC Register）** | **线程私有** | 当前线程正在执行的字节码指令行号地址 | **JVM 规范中唯一不会发生 `OutOfMemoryError (OOM)` 的区域** |
| **Java 堆（Heap）** | **线程共享** | **所有 `new` 出来的对象实例、数组本体、实例成员变量** | ⚠️ **两大权威纠偏（JDK 7/8+）**：<br>① **实例成员变量**随对象本体一起存放在**堆**中；<br>② 从 **JDK 7 起，类的 `static` 静态变量随 `java.lang.Class` 镜像对象一起存放在 Java 堆中**（不再存方法区）！<br>③ **字符串常量池（`StringTable`）**从 JDK 7 起也已迁移至 **Java 堆**中！ |
| **元空间（Metaspace / 方法区）** | **线程共享** | 类的元数据（`Klass` 结构、方法字节码、字段/方法定义）、运行时常量池 | JDK 8 彻底移除永久代（PermGen），改用基于本地内存（Native Memory）的**元空间（Metaspace）**实现方法区 |

#### 2. 成员变量（属性）与局部变量的 5 大本质区别

| 对比维度 | 成员变量（Field：实例变量 / 静态变量） | 局部变量（Local Variable：方法内 / 形参 / 代码块内） |
| :--- | :--- | :--- |
| **声明位置** | 类体内部、方法体外部 | 方法体内部、方法形参列表或代码块内部 |
| **内存位置** | 实例变量随对象存于**堆**；静态变量随 `Class` 对象存于**堆** | 存于当前线程**虚拟机栈**的局部变量槽（Slot）中 |
| **默认初始值** | ✅ **有默认初始值**（`int=0`, `boolean=false`, 引用=`null`） | ❌ **无默认值**，读取前必须显式赋值，否则**直接编译报错** |
| **可用修饰符** | 可用 `public/protected/private/static/final/volatile/transient` | **严禁加访问权限修饰符或 `static`**，仅允许用 `final` 修饰 |
| **生命周期** | 实例变量随对象创建而生、GC 回收而亡；静态变量随类卸载而亡 | 随方法调用压栈而生，方法执行完毕出栈立即销毁 |

#### 3. 经典内存泄漏剖析：`ThreadLocal` 弱引用 Key 与强引用 Value
```text
Thread 线程对象
  └── threadLocals (ThreadLocal.ThreadLocalMap)
        └── Entry[] table (Entry extends WeakReference<ThreadLocal<?>>)
              ├── key (弱引用 WeakReference) ──(虚线弱引用)──> ThreadLocal 对象
              └── value (强引用 Strong Ref)  ──(实线强引用)──> 业务存入的上下文数据 Object
```
- **泄漏发生四步链路**：
  1. `ThreadLocalMap` 的每个 `Entry` 继承自 `WeakReference<ThreadLocal<?>>`，即 **Key 对 `ThreadLocal` 实例是弱引用，而 Value 对业务数据是强引用**；
  2. 当外部指向 `ThreadLocal` 的强引用失效（或局部变量被回收）时，下一次 GC 会直接回收 `ThreadLocal` 对象，导致 `Entry` 中的 **`key` 变为 `null`**；
  3. 但此时存在强引用链：`当前 Thread -> ThreadLocalMap -> Entry -> value`，导致 **`key == null` 对应的 `value` 既无法再通过任何 `ThreadLocal` 访问，也无法被 GC 回收**；
  4. **高危重灾区（线程池）**：由于核心工作线程常驻复用永不销毁，若不清理，脏 `value` 不仅引发持久内存泄漏直至 OOM，还会造成下一个复用该线程的请求读到上一个用户的脏数据（串联事故）！
- **根治铁律**：声明 `private static final ThreadLocal<T>` 避免 Key 频繁失效，且**每次使用完毕必须在 `try-finally` 块中显式调用 `threadLocal.remove()`**（将该 `Entry` 的 key 和 value 一并清空移除）。

---

### 1.6 编译期常量（Compile-time Constant）vs 运行时常量

| 对比维度 | 编译期常量（常量折叠内联） | 运行时常量（运行期初始化） |
| :--- | :--- | :--- |
| **三大判定铁律** | 必须**同时满足**：<br>① `final` 修饰；<br>② 类型只能是 **8 种基本数据类型或 `String`**（包装类如 `final Integer` 绝不是！）；<br>③ **声明处直接用字面量或编译期常量表达式赋值** | 被 `final` 修饰，但：① 是包装类/普通对象（如 `new String("a")`）；或 ② 通过方法返回值、构造器参数、随机数等在运行期才确定值 |
| **字节码优化机制** | **编译期常量折叠（Constant Folding）**：`javac` 在编译阶段直接将所有引用该常量的地方**硬编码替换为字面量值**，运行时完全不访问原变量 | 编译期无法得知具体值，只能在运行期执行指令计算并赋值到内存 |

```java
// 高频考场 1：switch-case 条件必须是编译期常量
final int A = 1;
final int B = Integer.valueOf(2); // 包装类拆箱/方法调用 -> 运行时常量
switch (x) {
    case A: break; // ✅ 编译通过（A 是编译期常量）
    // case B: break; // ❌ 编译直接报错：constant expression required
}

// 高频考场 2：字符串 + 拼接常量池折叠
final String s1 = "ja";
final String s2 = "va";
String s3 = s1 + s2;              // s1、s2 均为编译期常量，javac 直接折叠为 String s3 = "java";
System.out.println(s3 == "java"); // ✅ true（直接指向字符串常量池同一个对象）

String v1 = "ja";                 // 无 final，是普通变量
String s4 = v1 + s2;              // 运行期通过 StringBuilder / invokedynamic 在堆上创建新对象
System.out.println(s4 == "java"); // ❌ false
```

---

## 二、八大基本数据类型与包装类深度剖析

### 2.1 八大基本数据类型全景表与赋值细节

#### 1. 八大基本数据类型核心参数速查表

| 分类 | 数据类型 | 字节数（位数） | 取值范围 | 默认初始值 | 对应包装类 | 字面量后缀与核心注意点 |
| :--- | :--- | :---: | :--- | :---: | :--- | :--- |
| **整型** | **`byte`** | 1 字节（8 bit） | `-128 ~ 127`（$-2^7 \sim 2^7-1$） | `(byte) 0` | `Byte` | 补码 `1000 0000` 规定为 `-128`，避免 `±0` 浪费 |
| **整型** | **`short`** | 2 字节（16 bit） | `-32,768 ~ 32,767`（$-2^{15} \sim 2^{15}-1$） | `(short) 0` | `Short` | 较少单独使用 |
| **整型** | **`int`** | 4 字节（32 bit） | $-2^{31} \sim 2^{31}-1$（约 `±21.47 亿`） | `0` | `Integer` | **Java 整数字面量默认类型** |
| **整型** | **`long`** | 8 字节（64 bit） | $-2^{63} \sim 2^{63}-1$ | `0L` | `Long` | 超出 `int` 范围字面量**必须加 `L`（强烈推荐大写 `L` 防与 `1` 混淆）** |
| **浮点型** | **`float`** | 4 字节（32 bit） | IEEE 754 单精度（有效十进制约 6~7 位） | `0.0f` | `Float` | 赋小数时**必须显式加 `F/f`**（否则默认是 `double` 编译报错） |
| **浮点型** | **`double`** | 8 字节（64 bit） | IEEE 754 双精度（有效十进制约 15~17 位） | `0.0d` | `Double` | **Java 小数字面量默认类型**（后缀 `D/d` 可省略） |
| **字符型** | **`char`** | 2 字节（16 bit） | `0 ~ 65,535`（`\u0000 ~ \uffff`，无符号） | `'\u0000'` | `Character` | 采用 UTF-16 编码，单引号包裹单个字符 `'A'` 或赋编码整数 |
| **布尔型** | **`boolean`** | 规范未定（通常 1 字节，数组元素 1 字节，单个变量栈上占 4 字节 Slot） | `true` / `false` | `false` | `Boolean` | **严禁与任何数值类型进行自动或强制转换**（`(int) true` 编译报错） |

#### 2. 赋值高频避坑指南
- **为什么 `byte` 范围是 `-128 ~ 127`？**：8 位二进制共 $2^8 = 256$ 种状态。最高位为符号位：`0000 0000` ~ `0111 1111` 表示 `0 ~ 127`（共 128 个非负数）；负数采用补码表示，`1111 1111` ~ `1000 0001` 表示 `-1 ~ -127`，而原本原码中的 `-0`（`1000 0000`）被规定用于表示 **`-128`**，从而不浪费任何一个比特位。
- **整数多进制字面量**：二进制以 `0b` 开头（`0b1010` = 10）、八进制以 `0` 开头（`012` = 10，业务慎用防误读）、十六进制以 `0x` 开头（`0x1A` = 26）；JDK 7+ 支持数字中间加下划线增强可读性（如 `long max = 10_000_000_000L;`）。
- **浮点数精度红线**：`0.1 + 0.2` 在二进制浮点数中是无限循环小数，计算结果为 `0.30000000000000004`。**凡涉及金钱、汇率、计费等敏感精度计算，严禁使用 `float/double`，必须使用 `new BigDecimal(String)` 或以“分”为单位的 `long` 整型存储！**

---

### 2.2 基本类型 vs 包装类 & 自动装箱/拆箱底层原理

#### 1. 基本类型与包装类的 5 大核心差异（以 `int` vs `Integer`、`long` vs `Long` 为例）

| 对比维度 | 基本数据类型（`int` / `long`） | 包装类引用类型（`Integer` / `Long`） |
| :--- | :--- | :--- |
| **本质与方法** | 原始数值，非对象，无法调用任何方法 | 封装了基本数值的 `final` 不可变类对象，提供 `parseInt/toBinaryString/compareTo` 等工具方法 |
| **内存与开销** | 局部变量直接存栈帧槽位，紧凑高效 | 引用存栈，对象本体存堆（含 12/16 字节对象头开销） |
| **默认初始值** | `0` / `0L`（无法区分“未传值”与“真实值为 0”） | **`null`**（阿里规约：**POJO 类属性、数据库 DO 字段、RPC 接口属性必须使用包装类**，以用 `null` 表达数据库 `NULL` 或未传参状态） |
| **泛型与集合支持** | ❌ 泛型擦除后为 `Object`，不支持 `List<int>` | ✅ 集合与泛型必须使用包装类 `List<Integer>` |
| **异常风险** | 无空指针风险 | 若引用为 `null`，触发自动拆箱时**立即抛出 `NullPointerException`** |

#### 2. 自动装箱（Autoboxing）与自动拆箱（Unboxing）字节码原理
JDK 5 引入的编译器语法糖，在编译阶段自动插入转换方法调用：
```java
// 1. 自动装箱：基本类型 -> 包装类，javac 编译后自动替换为 Integer.valueOf(int)
Integer a = 100;     // 字节码等价于：Integer a = Integer.valueOf(100);

// 2. 自动拆箱：包装类 -> 基本类型，javac 编译后自动替换为 a.intValue()
int b = a;           // 字节码等价于：int b = a.intValue();
```

#### 3. ⚠️ 笔试与面试致命编译陷阱：为什么 `Long sum = 0;` 直接编译失败？
```java
long prim = 0;       // ✅ 编译通过：int 字面量 0 自动拓宽（Widening）为 long 0L
Long wrap1 = 0L;     // ✅ 编译通过：long 字面量 0L 自动装箱（Autoboxing）为 Long.valueOf(0L)
// Long wrap2 = 0;   // ❌ 直接编译报错：incompatible types: int cannot be converted to java.lang.Long！
```
- **底层原因**：Java 编译器**不允许在一次赋值中同时叠加“基本类型自动拓宽（`int -> long`）”和“自动装箱（`long -> Long`）”**！字面量 `0` 默认是 `int`，要装箱只能装箱成 `Integer`，绝不会自动跨类型装箱成 `Long`。
- **性能反例警示**：严禁将循环累加变量声明为包装类（如 `Long sum = 0L; for (long i = 0; i < 1000000; i++) sum += i;`），因为每次 `+=` 都会触发“拆箱 $\rightarrow$ 相加 $\rightarrow$ `Long.valueOf` 重新 new 堆对象”，凭空制造上百万个短命包装类对象引爆 Young GC！

#### 4. `parseXxx(String)` 与 `valueOf(String)` 的区别
- **`Integer.parseInt("123")`**：返回基本类型 **`int`**，不涉及对象创建，算术运算首选；
- **`Integer.valueOf("123")`**：底层先调用 `parseInt("123")` 得到 `int`，再调用 `Integer.valueOf(int)` **优先从缓存池（`-128 ~ 127`）取对象返回 `Integer` 包装类实例**。

---

### 2.3 包装类高频大坑：缓存池机制（`-128 ~ 127`）、`==` vs `equals` 与拆箱 `NPE`

#### 1. 八大包装类缓存池（Cache Pool）全景对比表

| 包装类 | 是否内置缓存池 | 缓存范围 | 源码实现与 JVM 调优细节 |
| :--- | :---: | :---: | :--- |
| **`Integer`** | ✅ 是 | **`-128 ~ 127`** | 内部静态类 `IntegerCache`，**唯一支持通过 JVM 参数 `-XX:AutoBoxCacheMax=<size>`（或 `-Djava.lang.Integer.IntegerCache.high`）调大上限**（下限固定 `-128`） |
| **`Byte`** | ✅ 是 | **`-128 ~ 127`** | `ByteCache` 缓存了 `byte` 的全部 256 个可能值 |
| **`Short`** | ✅ 是 | **`-128 ~ 127`** | `ShortCache` 范围固定不可调 |
| **`Long`** | ✅ 是 | **`-128 ~ 127`** | `LongCache` 范围固定不可调 |
| **`Character`** | ✅ 是 | **`0 ~ 127`（`\u0000 ~ \u007f`）** | `CharacterCache` 固定缓存标准 ASCII 字符集 |
| **`Boolean`** | ✅ 是 | **`Boolean.TRUE` / `Boolean.FALSE`** | 直接返回两个静态常量单例对象 |
| **`Float` / `Double`** | ❌ **无** | 无缓存 | 区间内浮点数有无穷多个，缓存毫无意义，每次装箱必 `new` 新对象 |

> ⚠️ **注意**：只有通过**自动装箱**或显式调用 **`valueOf()`** 时才会走缓存池；若使用 `new Integer(100)`（JDK 9+ 已标记 `@Deprecated(forRemoval = true)`），则强制在堆上创建全新对象，绝不走缓存池！

#### 2. 包装类 `==` 与 `equals()` 比较的四大黄金判定铁律（笔试必考）
```java
Integer i1 = 127, i2 = 127;
Integer i3 = 128, i4 = 128;
int prim = 128;
Long l1 = 128L;

// 铁律 1：两边都是包装类引用时，== 比较的是【对象内存地址】！
System.out.println(i1 == i2);     // ✅ true（127 命中 IntegerCache 同一实例）
System.out.println(i3 == i4);     // ❌ false（128 超出缓存范围，分别 new 了两个不同对象）

// 铁律 2：只要 == 一侧是基本类型（或算术表达式），另一侧包装类必定【自动拆箱】后比较【纯数值】！
System.out.println(i3 == prim);   // ✅ true（i3 自动拆箱为 int 128，与 prim 比数值）

// 铁律 3：包装类的 equals() 方法会先用 instanceof 校验【类型是否一致】，再比较【内部数值】！
System.out.println(i3.equals(i4)); // ✅ true（同为 Integer 且值均为 128，生产环境包装类判等唯一标准做法！）
System.out.println(i3.equals(l1)); // ❌ false！（i3 是 Integer，l1 是 Long，类型不同直接返回 false！）

// 铁律 4：三元运算符自动拆箱 NPE 惊天大坑！
Integer nullInt = null;
boolean flag = true;
// int res = flag ? nullInt : 0;  // 💥 运行时直接抛 NullPointerException！
// 原因：第 3 操作数 0 是基本类型 int，导致第 2 操作数 nullInt 被编译器强制插入 nullInt.intValue() 自动拆箱！
```

---

### 2.4 多线程下包装类作为共享变量的并发安全陷阱

- **核心误区**：误以为 `Integer` 是不可变（Immutable）类，多线程并发操作 `Integer count` 就天然安全，甚至写出 `synchronized (count) { count++; }` 的严重 Bug 代码！
- **真相剖析**：
  1. `count++` 编译后等价于 **`count = Integer.valueOf(count.intValue() + 1)`**，包含“拆箱读取旧值 $\rightarrow$ 加 1 $\rightarrow$ 装箱返回新对象并篡改 `count` 引用指向”三步非原子操作，即使加 `volatile` 也无法保证原子性；
  2. 更致命的是，每次 `count++` 后 **`count` 指向了堆中一个全新的 `Integer` 对象**，若用 `synchronized (count)` 加锁，不同线程锁住的根本不是同一个对象，同步锁完全形同虚设！
- **生产标准解法**：高并发计数器一律使用 **`AtomicInteger` / `AtomicLong`**（CAS 无锁原子更新）或 **`LongAdder`**（分段 Cell 累加，超高并发吞吐最优）。

---

## 三、标识符规范与核心关键字全景

### 3.1 标识符命名规则、关键字与保留字

1. **编译器强制语法规则（违反直接编译报错）**：
   - 仅由 **字母（含 Unicode 字符）、数字（`0-9`）、下划线（`_`）、美元符（`$`）** 组成；
   - **不能以数字开头**，严格区分大小写；
   - **不能使用 Java 关键字、保留字（`goto`、`const`）以及特殊字面量（`true`、`false`、`null`）**。
   - *(版本演进细节：JDK 8 中单独一个 `_` 仅发警告，从 **JDK 9 起单独一个下划线 `_` 已正式升级为关键字**，写 `int _ = 10;` 直接编译报错；JDK 22+ 中 `_` 被赋予“未命名变量/模式”新语义。)*
2. **阿里《Java 开发手册》命名强制规约**：
   - **类名 / 接口名**：**大驼峰 `PascalCase`**（如 `UserOrderService`；抽象类推荐 `Abstract` 或 `Base` 开头，异常类 `Exception` 结尾，测试类 `Test` 结尾）；
   - **方法名 / 变量名**：**小驼峰 `camelCase`**（如 `getUserInfo`、`orderCount`；布尔成员变量**严禁以 `is` 开头**如 `isDeleted`，防止 Jackson/MyBatis 等框架 getter 序列化解析错乱，应命名为 `deleted`）；
   - **全局常量（`static final`）**：**全大写加下划线 `UPPER_SNAKE_CASE`**（如 `MAX_RETRY_COUNT`）；
   - **包名（`package`）**：**统一全小写**，点号分隔域名反转（如 `com.company.project.module`）。

---

### 3.2 `final` 关键字深度解析（类、方法、变量初始化时机与 `final/finally/finalize`）

#### 1. `final` 修饰三种目标的核心语义

| 修饰目标 | 核心语法约束 | 高频面试深水区细节 |
| :--- | :--- | :--- |
| **修饰类（`final class`）** | **该类绝不能被继承**（类中的所有方法隐式成为 `final` 方法） | 典型代表：`String`、`Integer` 等所有包装类、`System`。目的：保证不可变性与安全性，防止子类恶意篡改核心行为。**`final` 与 `abstract` 水火不容，绝不能同时修饰一个类或方法！** |
| **修饰方法（`final void f()`）** | **该方法绝不能被子类重写（Override）** | ⚠️ **注意**：`final` 方法虽然不能被子类重写，但**完全可以在本类或子类中被重载（Overload）**！此外，`private` 方法因其对子类不可见，隐式等同于 `final`。 |
| **修饰变量（`final Type v`）** | **变量一旦完成初始化赋值，便不可再被重新赋值（只读）** | - **修饰基本类型变量**：数值终生不可改变；<br>- **修饰引用类型变量（高频坑）**：**仅仅是栈上的引用变量保存的内存地址不可改变**（不能再指向别的新对象），但**其指向的堆对象内部的属性或集合元素完全可以自由修改**！ |

```java
final List<String> list = new ArrayList<>();
list.add("Java");        // ✅ 完全合法！修改的是堆对象内部元素，引用地址并未改变
// list = new ArrayList<>(); // ❌ 编译报错：cannot assign a value to final variable list
```

#### 2. `final` 变量的合法初始化时机（空白 `final` 规则）
`final` 变量并不强制要求在声明时立刻写死初始值（声明时不赋值的称为**空白 `final` 变量 Blank Final**），但必须严格遵守以下最晚初始化底线：
1. **实例 `final` 成员变量（非 `static`）**：必须在 **① 声明处直接赋值**、**② 实例构造代码块 `{}` 中赋值**、或 **③ 该类的每一个构造方法结束前完成赋值**（三选一，且只能赋值一次；若有多个重载构造器，未调用 `this(...)` 的每个构造器都必须为其赋值，否则编译报错）。
2. **静态 `static final` 常量**：必须在 **① 声明处直接赋值** 或 **② `static {}` 静态代码块中完成赋值**（二选一，绝不能拖到构造方法中赋值，因为类加载准备/初始化阶段时尚未创建任何实例对象）。
3. **局部 `final` 变量 / 方法 `final` 形参**：局部变量只需在**第一次读取使用前完成一次赋值**即可；`final` 形参由调用方传参时自动完成初始化，方法体内禁止对其重新赋值。

#### 3. 必考简答：`final`、`finally`、`finalize()` 三者区别

| 名称 | 性质 | 核心作用与生命周期 |
| :--- | :--- | :--- |
| **`final`** | 权限/状态修饰符关键字 | 修饰类（不可继承）、方法（不可重写）、变量（只能赋值一次的常量） |
| **`finally`** | 异常处理代码块关键字 | 配合 `try-catch` 使用，存放无论是否发生异常都**必定执行**的资源清理逻辑（唯一不执行特例：`System.exit()` 或进程被强杀） |
| **`finalize()`** | `Object` 类中的受保护方法 | GC 回收垃圾对象前可能调用的析构钩子。因触发时机不确定、严重拖慢 GC 且可导致对象复活，**已在 JDK 9 中被正式标记废弃（`@Deprecated`），JDK 18+ 默认禁用**，生产严禁使用 |

---

### 3.3 `static` 关键字与类初始化执行顺序

#### 1. `static` 四大应用场景与铁律
1. **`static` 静态变量（类变量）**：属于类本身，全类所有实例共享同一份内存；在类加载阶段初始化，直接通过 `类名.静态变量` 访问。
2. **`static` 静态方法（类方法）**：
   - **铁律 ①**：静态方法内部**只能直接访问静态变量和调用静态方法**，**严禁直接访问非静态（实例）成员**（因为静态方法调用时可能根本还没有 `new` 任何实例对象）；
   - **铁律 ②**：静态方法内部**严禁出现 `this` 或 `super` 关键字**；
   - **铁律 ③**：静态方法**不存在多态重写（Override）**！若子类定义了与父类相同签名的 `static` 方法，这称为**方法隐藏（Method Hiding）**，调用哪个方法完全由左边的**编译期引用类型**决定，与运行时对象无关。
3. **`static {}` 静态代码块**：类加载初始化阶段执行，**全生命周期只执行一次**，用于预加载驱动或初始化静态配置表。
4. **`static class` 静态内部类**：不持有外部类实例的隐式引用 `Outer.this`，可直接 `new Outer.StaticInner()` 创建，**不会引发外部类内存泄漏**（Builder 建造者模式、链表/树节点 `Node` 标配）。

#### 2. 父子类创建对象的完整执行顺序口诀（笔试必考代码题）
> **黄金口诀**：**先父后子，先静后动，先块后构造**！
> 完整 6 步顺序：**① 父类静态变量与静态代码块（按代码书写顺序，仅首次加载执行一次） $\rightarrow$ ② 子类静态变量与静态代码块（仅首次加载执行一次） $\rightarrow$ ③ 父类实例变量与实例代码块 $\rightarrow$ ④ 父类构造方法 $\rightarrow$ ⑤ 子类实例变量与实例代码块 $\rightarrow$ ⑥ 子类构造方法。**

```java
class Parent {
    static { System.out.print("1-父静 "); }
    { System.out.print("3-父实例块 "); }
    public Parent() { System.out.print("4-父构造 "); }
}
class Child extends Parent {
    static { System.out.print("2-子静 "); }
    { System.out.print("5-子实例块 "); }
    public Child() { System.out.print("6-子构造 "); }

    public static void main(String[] args) {
        new Child(); // 输出：1-父静 2-子静 3-父实例块 4-父构造 5-子实例块 6-子构造
        System.out.print("| 第二次 new：");
        new Child(); // 静态块不再执行！输出：| 第二次 new：3-父实例块 4-父构造 5-子实例块 6-子构造
    }
}
```

---

### 3.4 `this` 与 `super` 关键字对比

| 对比维度 | `this` 关键字 | `super` 关键字 |
| :--- | :--- | :--- |
| **本质含义** | 指向**当前对象实例自身**的引用 | 当前对象中指向**直接父类特征空间**的标识（注意：`super` 不是独立对象引用，不能作为返回值或赋给变量） |
| **访问属性** | `this.name`：访问本类成员变量（区分同名局部形参） | `super.name`：显式访问父类被子类同名字段隐藏的成员变量 |
| **调用方法** | `this.func()`：调用本类实例方法 | `super.func()`：调用父类被子类重写（Override）的原始方法逻辑 |
| **调用构造器** | `this(args)`：在本类构造器中调用**本类的其他重载构造器** | `super(args)`：在子类构造器中显式调用**父类的构造器** |
| **使用禁区** | ❌ **严禁在 `static` 静态方法或静态代码块中使用** | ❌ **严禁在 `static` 静态方法或静态代码块中使用** |

- **构造器调用两大死规矩**：
  1. **`this(...)` 和 `super(...)` 若出现在构造方法中，都必须写在构造方法的第一行，因此二者绝对不能在同一个构造器中同时出现！**
  2. **父类无参构造缺失陷阱**：子类所有构造方法（只要第一行没显式写 `this(...)` 或 `super(...)`），编译器都会在第一行**自动隐式插入无参的 `super();`**。如果父类显式定义了有参构造器而没写无参构造器，子类构造器会直接报编译错误，此时子类必须显式写 `super(参数)` 调用父类的有参构造器。

---

### 3.5 `abstract` 与其它核心关键字全景速查

#### 1. `abstract` 抽象类与抽象方法核心规则
- **抽象方法（`abstract void work();`）**：只有方法签名、没有方法体大括号 `{}`；**包含抽象方法的类必须声明为 `abstract` 抽象类**。
- **抽象类（`abstract class`）**：
  1. **不能直接 `new` 实例化**，但**一定有构造方法**（供子类初始化时通过 `super()` 调用以初始化父类公共成员变量）；
  2. **抽象类中可以没有抽象方法**（例如仅为了禁止外部直接 `new` 该基类），也可以包含普通具体方法、静态方法、实例变量和常量；
  3. 非抽象子类继承抽象类时，**必须重写实现父类的全部抽象方法**；
  4. **修饰符冲突禁区（必考选择题）**：`abstract` **绝对不能与 `final`（禁止继承/重写）、`private`（子类不可见无法重写）、`static`（属于类无多态重写）、`native`（无方法体冲突）同时修饰同一个方法！**

#### 2. Java 其余高频关键字速查矩阵

| 关键字 | 核心作用与底层原理 | 面试高频避坑与最佳实践 |
| :--- | :--- | :--- |
| **`native`** | 声明一个方法由非 Java 语言（通常是 C/C++）通过 **JNI（Java Native Interface）** 在本地动态库中实现，仅有方法声明无方法体 | 典型代表：`Object.hashCode()/clone()`、`Thread.start0()`、`System.arraycopy()`、**`String.intern()`**（将字符串放入堆中字符串常量池并返回池中引用） |
| **`synchronized`** | JVM 内置互斥同步锁，保证代码块或方法的**原子性、可见性与有序性** | 修饰实例方法锁 `this`；修饰静态方法锁当前类的 `Class` 对象；修饰代码块锁括号内指定的对象 |
| **`volatile`** | 轻量级同步关键字：① **保证多线程内存可见性**（写操作强制刷主存，读操作失效本地缓存）；② **插入内存屏障禁止指令重排序** | ⚠️ **不能保证复合操作（如 `i++`）的原子性**！仅适用于“一写多读”状态标志位或 **DCL 双重检查锁单例模式** |
| **`transient`** | 修饰成员变量，指示 JDK 原生序列化（`Serializable`）时**忽略该字段，不写入字节流** | 常用于密码等敏感字段或可实时计算的派生缓存字段；反序列化后该字段恢复为类型默认值（如 `null` 或 `0`）；**注意：`static` 静态变量无论是否加 `transient` 都不会被实例序列化** |
| **`instanceof`** | 运行期动态判断左边对象是否为右边类/接口的实例或子类实例（若左操作数为 `null`，直接返回 `false` 不会抛 NPE） | **Java 16+ 模式匹配（Pattern Matching）转正语法**：<br>`if (obj instanceof String s && !s.isEmpty()) { System.out.println(s.length()); }`（自动完成类型判断与变量强转绑定） |
| **`package` / `import`** | `package` 声明命名空间（源文件首行）；`import` 导入外部类，避免手写冗长全限定类名（`import static` 可静态导入工具类方法） | ⚠️ **辟谣纠偏**：使用通配符 `import java.util.*;` **绝对不会降低运行期性能或增加类加载开销**！`import` 仅在编译期辅助 `javac` 解析符号，编译后的 `.class` 中全是全限定名；但阿里规约不推荐 `*` 是为了防止多包同名类（如 `java.util.Date` 与 `java.sql.Date`）命名冲突影响可读性 |
| **`strictfp`** | `strict floating point`，强制类、接口或方法内的所有浮点运算严格遵循 IEEE 754 标准，消除不同 CPU 80 位扩展精度寄存器带来的跨平台细微误差 | **Java 17（JEP 306）起 JVM 已将严格浮点语义设为全局默认行为**，`strictfp` 关键字已无需再显式书写 |
| **`assert`** | 调试期断言：`assert condition : "错误提示";`，若布尔表达式为 `false` 则抛出 `AssertionError` | **JVM 默认关闭断言**（生产环境不执行），需加启动参数 `-ea`（`-enableassertions`）才生效；**严禁在生产代码中用 `assert` 代替业务参数校验**（生产校验请用 `if + IllegalArgumentException` 或 `Objects.requireNonNull`） |
| **`new`** | 在 Java 堆中分配内存、调用 `<init>` 构造方法初始化实例或数组，并返回堆内存地址引用 | 与反射创建对比：`new` 在编译期静态绑定具体类且极速；反射 `Class.getDeclaredConstructor().newInstance()` 在运行期动态解析加载类 |

---

## 四、运算符优先级、位运算与类型转换体系

### 4.1 八大类运算符全景与高频陷阱

#### 1. 算术运算符与自增/自减（`++` / `--`）字节码真相
- **整数除法与取余规则**：
  - 两个整数相除（`/`）**直接向零截断小数部分**（如 `5 / 2 == 2`，`-5 / 2 == -2`）；若需保留小数，至少将一侧强转为浮点型（`5.0 / 2 == 2.5`）；
  - 取余运算（`%`）结果的**正负号永远与被除数（左操作数）一致**（如 `-7 % 3 == -1`，`7 % -3 == 1`）。
- **前缀 `++i` vs 后缀 `i++`**：
  - **后缀 `i++`（先用后加）**：先将 `i` 的原始值压入操作数栈参与表达式运算，再将局部变量表中的 `i` 加 1；
  - **前缀 `++i`（先加后用）**：先将局部变量表中的 `i` 加 1，再将更新后的新值压入操作数栈参与运算。

```java
// 经典笔试题 1：混合自增表达式
int x = 5;
int res = x++ * 2 + ++x;
// 拆解：① x++ 先取旧值 5 算 5 * 2 = 10（此时 x 变为 6）；② ++x 先自增变为 7 再参与加法；③ res = 10 + 7 = 17，最终 x = 7

// 经典笔试题 2：臭名昭著的 i = i++; 陷阱
int i = 1;
i = i++;
System.out.println(i); // 输出：1（而不是 2！）
// 字节码级原理解析：
// 1. iload_1      -> 先把 i 的旧值 1 压入操作数栈暂存；
// 2. iinc 1, 1    -> 直接将局部变量表槽位 1 中的 i 自增变为 2；
// 3. istore_1     -> 将操作数栈顶暂存的旧值 1 弹出，重新写回局部变量表槽位 1，把刚算出的 2 给覆盖掉了！
```

#### 2. 逻辑运算符：短路与（`&&`）vs 逻辑与（`&`）、短路或（`||`）vs 逻辑或（`|`）

| 运算符 | 名称 | 运算规则 | 短路特性（Short-Circuit）与实战作用 |
| :--- | :--- | :--- | :--- |
| **`&&`** | **短路与** | 两边同为 `true` 才返回 `true` | ✅ **左操作数为 `false` 时，右侧表达式直接跳过不执行**（常用于防空指针保护：`if (str != null && !str.isEmpty())`） |
| **`||`** | **短路或** | 任一为 `true` 即返回 `true` | ✅ **左操作数为 `true` 时，右侧表达式直接跳过不执行** |
| **`&`** | 按位与 / 逻辑与 | 两侧为 `boolean` 时作逻辑与；两侧为整数时作二进制按位与 | ❌ **不短路**：即便左侧已经能决定结果，右侧表达式依然会强行执行完毕 |
| **`|`** | 按位或 / 逻辑或 | 两侧为 `boolean` 时作逻辑或；两侧为整数时作二进制按位或 | ❌ **不短路**：左右两侧表达式必定全部执行 |
| **`^`** | 逻辑异或 / 按位异或 | 两边**不同**返回 `true`（或二进制位 `1`），**相同**返回 `false`（或 `0`） | ❌ 必须计算完左右两侧才能得知是否不同，天然无法短路 |

#### 3. 位运算符与移位运算符高频实战技巧
```java
// 技巧 1：高效乘除 2 的幂次方（算术移位 vs 无符号逻辑右移）
int a = 16;
int mul4 = a << 2;   // 左移 2 位：等价于 16 * 2^2 = 64（低位补 0）
int div4 = a >> 2;   // 带符号算术右移 2 位：等价于 16 / 2^2 = 4（正数高位补 0，负数高位补 1）
int unsigned = -16 >>> 2; // 无符号逻辑右移：无论正负，高位一律补 0（将负数迅速变为正整数）

// 技巧 2：位与 (&) 判断奇偶性（比 n % 2 != 0 快且免疫负数取余坑：-3 % 2 == -1 != 1）
boolean isOdd = (n & 1) == 1;

// 技巧 3：位与 (&) 替代取模（前提：容量 cap 必须是 2 的幂次方，HashMap / ArrayDeque 核心寻址算法）
int index = hash & (cap - 1);

// 技巧 4：异或 (^) 不借助临时变量交换两个整数（基于性质：x ^ x == 0 且 x ^ 0 == x）
int m = 10, n = 20;
m = m ^ n;
n = m ^ n; // 等价于 (m ^ n) ^ n = m
m = m ^ n; // 等价于 (m ^ n) ^ m = n
```

---

### 4.2 运算符优先级 14 级速记表与口诀

> **记忆口诀**：**括号一元乘除模，加减移位比较符；等位与或三目赋，单目三目赋右左。**
> - **核心链路**：`()` `[]` `.` **>** 单目（`! ++ -- ~`） **>** 算术（`* / %` > `+ -`） **>** 移位（`<< >> >>>`） **>** 关系比较（`< <= > >= instanceof` > `== !=`） **>** 按位（`&` > `^` > `|`） **>** 短路逻辑（**`&&` 高于 `||`**） **>** 三目（`? :`） **>** 赋值（`= +=`）。
> - **结合性特例**：除**单目运算符、三目条件运算符、赋值运算符**这三类是**从右向左**结合外，其余所有运算符均为**从左向右**结合。

```java
// 高频笔试易错题：&& 的优先级高于 || ！
boolean flag = true || false && false;
// 解析：先算高优先级的 (false && false) 得 false，再算 true || false -> 最终结果为 true！
```

---

### 4.3 类型转换全景体系（隐式拓宽、显式强转与复合赋值 `+=` 隐式强转语法糖）

#### 1. 基本数据类型转换规则
```text
自动拓宽方向（隐式转换，安全无损，仅 long -> float / double -> float 可能存在有效位数精度损失）：
  byte (1B) ──> short (2B) ──> int (4B) ──> long (8B) ──> float (4B,指数范围大) ──> double (8B)
                 char (2B) ──↗
```
- **三大铁律与高频坑点**：
  1. **`byte / short` 与 `char` 之间绝不能自动互转**：因为 `byte`（`-128~127`）和 `short` 是有符号数，而 `char`（`0~65535`）是无符号数，彼此取值范围存在互不包含的区间，互转必须显式强转；
  2. **`byte`、`short`、`char` 参与算术运算时自动提升为 `int`**：
     ```java
     byte b1 = 10, b2 = 20;
     // byte b3 = b1 + b2;       // ❌ 编译报错！b1 + b2 在运算瞬间已自动提升为 int 类型
     byte b3 = (byte) (b1 + b2); // ✅ 必须显式强转回 byte
     ```
  3. **强制类型转换（`(TargetType) val`）的二进制截断溢出本质**：
     ```java
     int num = 130; // 二进制后 8 位为 1000 0010
     byte b = (byte) num;
     System.out.println(b); // 输出：-126（高位直接截断，1000 0010 作为补码正是 -128 + 2 = -126）
     ```

#### 2. 【面试超高频必考】复合赋值运算符（`+=` / `-=` / `*=` / `/=`）的隐式强转语法糖
- **经典拷问**：为什么下面两行看似完全等价的代码，第一行编译报错，第二行却能正常编译通过？
  ```java
  short s = 10;
  // s = s + 1; // ❌ 编译报错：incompatible types: possible lossy conversion from int to short
  s += 1;       // ✅ 编译通过！
  ```
- **底层原因**：根据《Java 语言规范（JLS §15.26.2）》，复合赋值表达式 `E1 op= E2` 在编译期会被 `javac` 自动等价重写为 **`E1 = (T) ((E1) op (E2))`**（其中 `T` 是 `E1` 的类型）。因此 **`s += 1;` 编译后等价于 `s = (short) (s + 1);`，编译器自动隐式插入了强制类型转换**！
- **生产风险警示**：复合赋值的隐式强转是静默发生的，若数值超界会发生无警告的静默溢出（如 `short s = 32767; s += 2;` 会静默变成 `-32767`）。

---

## 五、流程控制语句与现代语法演进

### 5.1 分支语句 `if-else` 避坑与卫语句（Guard Clause）扁平化重构

> **什么是卫语句（Guard Clause）？**
> 阿里《Java 开发手册》强烈推崇的代码重构范式：**将空值判断、参数非法、权限不足等异常/边界分支全部前置到方法最上方，一旦命中立即通过 `return` 或 `throw` 提前退出（Early Return）**，从而彻底消除多层向右缩进的“箭头型 `if-else` 嵌套代码”，让主干正常逻辑保持在最外层扁平展开。

```java
// ❌ 反例：多层 if-else 嵌套（“箭头型代码”，阅读时需层层配对大括号）
public String getScoreLevelBad(Integer score) {
    if (score != null) {
        if (score >= 0 && score <= 100) {
            if (score >= 90) return "优秀";
            else if (score >= 60) return "及格";
            else return "不及格";
        } else {
            return "分数越界";
        }
    } else {
        return "分数为空";
    }
}

// ✅ 正例：使用卫语句（Guard Clause）前置拦截异常边界，主干逻辑零嵌套
public String getScoreLevelGood(Integer score) {
    if (score == null) return "分数为空";               // 卫语句 1：空值拦截
    if (score < 0 || score > 100) return "分数越界";    // 卫语句 2：边界拦截

    // 走到此处必为 [0, 100] 的合法分数，直接扁平处理主干业务
    if (score >= 90) return "优秀";
    if (score >= 60) return "及格";
    return "不及格";
}
```

---

### 5.2 `switch` 语法全景演进（从 JDK 5/7 到 JDK 14/17/21 模式匹配）

#### 1. `switch` 支持的数据类型历史演进与限制
- **允许的类型**：
  1. **JDK 1.0 ~ 1.4**：仅支持 4 种整数兼容基本类型 —— **`byte`、`short`、`char`、`int`**（⚠️ **严禁传入 `long`、`float`、`double`、`boolean`！**）；
  2. **JDK 5**：新增 **枚举 `enum`**（底层调用 `enum.ordinal()` 转为 `int`）以及 **`Byte`、`Short`、`Character`、`Integer`** 四种包装类（依赖自动拆箱）；
  3. **JDK 7**：新增 **`String` 字符串**；
     - **底层实现揭秘（面试常问）**：JVM 并没有新增处理字符串的字节码指令，而是由 `javac` 编译器在编译期解糖为两步 —— **先对 `str.hashCode()` 的 `int` 值做 `switch` 跳转，命中对应 hash 分支后再自动插入 `str.equals("常量")` 进行二次比对**（完美解决哈希碰撞问题）！
  4. **JDK 14 / 17 / 21 现代演进**：新增 **`->` 箭头分支（无穿透）**、**`switch` 返回值表达式（配合 `yield`）**、以及 **对象类型模式匹配与 `case null` 分支**。

#### 2. 传统 `switch` 穿透坑 vs 现代 Java 14~21 箭头表达式与模式匹配

```java
// 1. Java 14+ 箭头 switch 表达式（天然无 case 穿透，无需写 break，支持多值逗号合并与直接返回值）
String season = switch (month) {
    case 12, 1, 2 -> "冬季";
    case 3, 4, 5  -> "春季";
    case 6, 7, 8  -> "夏季";
    case 9, 10, 11 -> {
        System.out.println("进入秋季多行代码块");
        yield "秋季"; // 代码块分支使用 yield 关键字产出 switch 表达式的返回值（注意不是 return）
    }
    default -> throw new IllegalArgumentException("非法月份: " + month);
};

// 2. Java 21 正式转正：switch 类型模式匹配 + case null 防空指针（替代繁琐的 if-else instanceof）
static String formatValue(Object obj) {
    return switch (obj) {
        case null                -> "空对象 (null)"; // 传统 switch 传 null 直接抛 NPE，现代模式匹配可直接捕获！
        case Integer i when i > 0 -> "正整数: " + i; // 支持 when 守卫条件
        case Integer i           -> "非正整数: " + i;
        case String s            -> "字符串(长" + s.length() + "): " + s;
        default                  -> "未知类型: " + obj.toString();
    };
}
```

#### 3. `switch` vs `if-else` 的底层性能差异
- 当分支较多时，`javac` 会根据 `case` 常量的紧凑程度生成两种跳转表字节码指令：
  1. **`tableswitch`（紧凑连续值，如 `1, 2, 3, 4`）**：底层直接生成连续数组索引表，通过下标计算一次直接跳转，时间复杂度为 **$O(1)$**；
  2. **`lookupswitch`（稀疏离散值，如 `1, 100, 10000`）**：底层生成排序后的键值映射表，通过**二分查找**定位分支，时间复杂度为 **$O(\log N)$**，均优于多层 `if-else` 的 $O(N)$ 顺序扫描。

---

### 5.3 四大循环语句（`for` / `while` / `do-while` / `for-each`）与跳转控制

| 循环类型 | 执行顺序与核心特征 | 底层原理与高频避坑 |
| :--- | :--- | :--- |
| **`for (init; cond; update)`** | 先初始化 1 次 $\rightarrow$ 判条件 $\rightarrow$ 执行循环体 $\rightarrow$ 更新步进 | 已知循环次数或需要数组下标时首选；`for (;;)` 等价于 `while (true)` 无限循环 |
| **`while (cond)`** | **先判断条件，后执行循环体** | 条件初始为 `false` 时，循环体**执行 0 次** |
| **`do { ... } while (cond);`** | **先执行一次循环体，再判断条件**（注意末尾分号 `;` 不能漏） | 无论条件是否成立，循环体**至少执行 1 次** |
| **增强 `for-each`<br>`for (T item : coll)`** | JDK 5 语法糖，专用于只读遍历数组或 `Iterable` 集合 | ① **遍历数组**：编译后还原为普通下标 `for` 循环；<br>② **遍历集合**：编译后还原为 `Iterator` 迭代器（`hasNext/next`）。**严禁在 `for-each` 内直接调 `list.remove/add`，否则必抛 `ConcurrentModificationException`！** |

- **跳转控制关键字（`break` / `continue` / 带标签跳出多层循环）**：
  - `break`：立即终止并跳出当前层循环；`continue`：跳过本轮剩余语句，直接进入下一轮循环判定；
  - **跳出多层嵌套循环（Label 标签语法）**：在最外层循环前定义 `outer:`，内层直接写 **`break outer;`** 即可一次性跳出所有嵌套循环。

---

## 六、Java 方法体系、参数传递铁律与面向对象核心速查

> *(关于面向对象设计原则 SOLID、动态绑定虚方法表 vtable、接口多继承冲突等深水区内容，请联动参阅 [`01-JavaBasic-02-面向对象.md`](./01-JavaBasic-02-面向对象.md)。)*

### 6.1 方法签名（Method Signature）与可变参数（`Varargs`）

1. **方法签名（Method Signature）的严格定义**：
   - 在 Java 语言规范中，**方法签名仅包含两项：`方法名称` + `形参类型列表（参数个数、类型、顺序）`**。
   - ⚠️ **易错点**：**返回值类型、访问权限修饰符、形参变量名、`throws` 异常列表均不属于方法签名**！因此同一个类中绝不允许存在方法名与参数列表完全相同、仅返回值不同的两个方法。
2. **可变参数（`Type... varName`，JDK 5+）**：
   - **底层本质**：编译期语法糖，底层完全等价于**一维数组 `Type[]`**（因此同一个类中不能同时定义 `void foo(int... a)` 和 `void foo(int[] a)`，会报方法签名重复编译错误）；
   - **语法限制**：一个方法**最多只能有 1 个可变参数，且必须放在形参列表的最后一位**；
   - **重载匹配优先级**：定长参数方法的匹配优先级**高于**可变参数方法（因为可变参数每次调用需隐式创建数组，属于兜底匹配）。

---

### 6.2 【面试必考铁律】值传递 vs 引用传递：为什么 Java 只有值传递？

#### 1. 权威概念界定
- **值传递（Pass by Value）**：方法调用时，将实际参数（实参）中存储的值**复制一份副本**传递给形式参数（形参）。方法内部对形参本身的重新赋值，仅修改栈上的副本，**绝对不会影响外部实参变量本身**。
- **引用传递（Pass by Reference，如 C++ 中的 `void swap(int &a, int &b)`）**：方法调用时，传递的是实参变量本身的内存地址别名（形参就是外部实参本身），在方法内对形参重新赋值会直接改变外部实参的指向。
- **核心定调**：**Java 语言中只有值传递，绝对不存在引用传递！** 之所以很多人产生错觉，是因为当参数是对象类型时，**传递的这个“值”恰好是“对象在堆中内存地址的副本（引用副本）”**。

#### 2. 三种传参场景栈堆内存流转对比（代码铁证）

```java
public class PassByValueProof {
    public static void main(String[] args) {
        // 场景 1：基本类型 -> 传递数值副本，外部毫发无损
        int num = 10;
        changePrimitive(num);
        System.out.println(num); // 输出：10

        // 场景 2：可变对象引用 -> 传递地址副本
        StringBuilder sb = new StringBuilder("Hello");
        changeObject(sb);
        System.out.println(sb.toString()); // 输出："Hello World"（而不是 "New Builder"！）
    }

    static void changePrimitive(int x) {
        x = 999; // 仅修改栈上形参副本 x
    }

    static void changeObject(StringBuilder builderRefCopy) {
        // 操作 A：顺着地址副本找到堆里同一个 StringBuilder 对象并修改其内部状态 -> 外部实参可见！
        builderRefCopy.append(" World");

        // 操作 B：【证明非引用传递的铁证】将形参引用重新指向一个新对象 -> 仅改变了栈上副本的指向，外部实参 sb 仍指向原对象！
        builderRefCopy = new StringBuilder("New Builder");
    }
}
```

| 传递参数类型 | 栈上拷贝给形参的内容 | 方法内修改**堆对象内部属性** | 方法内对**形参变量重新赋值（`= new ...`）** |
| :--- | :--- | :---: | :---: |
| **基本数据类型（`int/double`）** | 基本类型**数值的副本** | 不涉及堆对象 | ❌ **不影响**外部实参 |
| **可变引用类型（自定义类/`List`/`Map`/数组）** | **对象堆内存地址的副本** | ✅ **影响**（形参与实参指向同一个堆对象） | ❌ **不影响**外部实参原有指向 |
| **不可变引用类型（`String` / 包装类）** | **对象堆内存地址的副本** | 不可变类无修改内部属性的方法 | ❌ **不影响**（拼接或自增本质都是生成新对象并改写形参副本指向） |

---

### 6.3 方法重载（Overload）vs 方法重写（Override）全方位对比

> **重写（Override）核心口诀**：“**两同、两小、一大**”
> - **两同**：**方法名相同**、**形参列表完全相同**；
> - **两小**：子类方法的**返回值类型**必须小于或等于父类方法返回值类型（即相同或是其子类型，称为**协变返回类型**；若为基本类型或 `void` 则必须完全相同）；子类方法声明抛出的**受检异常（Checked Exception）**必须小于或等于父类方法声明的受检异常（或干脆不抛）；
> - **一大**：子类方法的**访问权限修饰符**必须大于或等于父类方法的访问权限（如父类 `protected` $\rightarrow$ 子类可为 `protected` 或 `public`，绝不能缩小为 `private`）。

| 对比维度 | 方法重载（`Overload`） | 方法重写（`Override`） |
| :--- | :--- | :--- |
| **发生范围** | **同一个类中**（或含继承自父类的同名不同参方法） | **有继承/实现关系的父子类之间** |
| **方法名称** | 必须相同 | 必须相同 |
| **参数列表** | **必须不同**（参数个数、类型或顺序至少一项不同） | **必须完全相同** |
| **返回值类型** | 无限制（但**仅返回值不同绝不能构成重载**，会编译报错） | 必须相同，或是父类返回类型的**子类型（协变返回类型）** |
| **访问权限** | 无限制 | **严禁比父类更严格**（只允许扩大或持平） |
| **抛出受检异常** | 无限制 | **严禁抛出比父类更宽泛或全新的受检异常**（运行时异常无限制） |
| **`private/static/final`** | 均可在同类内正常重载 | **严禁重写**（`private` 不可见、`final` 禁止重写、`static` 同名属于方法隐藏） |
| **多态与绑定时机** | **编译期多态（静态绑定）**：编译时根据实参静态类型决定 | **运行期多态（动态绑定）**：运行时根据堆对象的实际类型动态分派 |

```java
// 协变返回类型（Covariant Return Type）标准示例
class Animal {
    protected Animal reproduce() throws IOException { return new Animal(); }
}
class Dog extends Animal {
    @Override
    public Dog reproduce() throws FileNotFoundException { // 权限扩大(public)、返回子类(Dog)、异常缩小(FileNotFoundException)
        return new Dog();
    }
}
```

---

### 6.4 协变（Covariance）、逆变（Contravariance）与泛型 PECS 原则

假设 `Dog` 是 `Animal` 的子类（`Dog extends Animal`）：

| 型变类型 | Java 语法表现 | 核心含义与读写安全边界 | 典型工程应用 |
| :--- | :--- | :--- | :--- |
| **协变（Covariance）** | ① 数组 `Dog[] -> Animal[]`<br>② 泛型上界 **`List<? extends Animal>`** | 保持子类型方向一致（可将 `List<Dog>` 赋给 `List<? extends Animal>`）。<br>⚠️ **泛型协变只读禁写**：只能从中安全读取为 `Animal`，**严禁向其 `add` 除 `null` 外的任何元素**（因编译器无法得知底层究竟是 `List<Dog>` 还是 `List<Cat>`） | **Producer-Extends（生产者只读）**：从集合中遍历读取元素 |
| **逆变（Contravariance）** | 泛型下界 **`List<? super Dog>`** | 反转子类型方向（可将 `List<Animal>` 或 `List<Object>` 赋给 `List<? super Dog>`）。<br>✅ **泛型逆变可写受限读**：可安全向其 `add(new Dog())`，但从中 `get()` 读取出的元素类型只能兜底用 `Object` 接收 | **Consumer-Super（消费者只写）**：向集合中批量写入/消费 `Dog` 元素 |
| **不变（Invariance）** | 普通泛型 **`List<Animal>`** | 无型变关系：**`List<Dog>` 绝不是 `List<Animal>` 的子类型**，直接赋值编译报错 | 既读又写的常规集合 |

```java
// ⚠️ 数组协变运行时陷阱 vs 泛型编译期保护
Dog[] dogs = new Dog[2];
Animal[] animals = dogs;     // ✅ 数组支持协变，编译通过
// animals[0] = new Cat();   // 💥 编译不报错，但运行时直接抛出 ArrayStoreException！（数组在对象头记录了实际元素类型 Dog）

// PECS 原则经典源码体现：Collections.copy(List<? super T> dest, List<? extends T> src)
// src 是数据源（Producer）用 ? extends T 安全读取；dest 是目的地（Consumer）用 ? super T 安全写入！
```

---

### 6.5 接口 vs 抽象类 & 四大权限修饰符 & 四大内部类速查

#### 1. 接口（`interface`）vs 抽象类（`abstract class`）核心对比

| 对比维度 | 接口（`interface`） | 抽象类（`abstract class`） |
| :--- | :--- | :--- |
| **设计语义** | **“能不能做（`can-do` / `like-a`）”的行为契约规范**（如 `Comparable`、`Serializable`） | **“是不是（`is-a`）”的血缘模板共性抽取**（如 `AbstractList`、`HttpServlet`） |
| **多继承/实现** | ✅ 一个类可 `implements` **多个接口**；接口可 `extends` 多个接口 | ❌ Java 类只能 `extends` **单个抽象类** |
| **成员变量** | 只能是 **`public static final` 全局静态常量**（不可保存实例状态） | 可拥有任意访问权限的**普通实例变量**、静态变量与常量（可保存状态） |
| **构造方法** | ❌ **绝对没有构造方法**，无初始化块 | ✅ **有构造方法**（供子类 `super()` 调用以初始化公共属性） |
| **方法实现演进** | - JDK 7 及以前：全为 `public abstract` 抽象方法；<br>- **JDK 8+**：新增 **`default` 默认方法**（向下兼容扩展接口）与 **`static` 静态方法**；<br>- **JDK 9+**：新增 **`private` 私有方法**（复用多个 `default` 方法间的公共逻辑） | 可包含抽象方法（`abstract`），也可包含任意完整实现的实例方法、静态方法、`final` 方法 |

#### 2. 四大访问权限修饰符可见性矩阵

| 修饰符 | 当前类内部 | 同一个包内（Same Package） | 不同包的子类（Subclass） | 不同包的非子类（Global） |
| :--- | :---: | :---: | :---: | :---: |
| **`public`** | ✅ | ✅ | ✅ | ✅ |
| **`protected`** | ✅ | ✅ | ✅ | ❌ |
| **`default`（缺省不写）** | ✅ | ✅ | ❌ | ❌ |
| **`private`** | ✅ | ❌ | ❌ | ❌ |

#### 3. Java 四种内部类对比速查

| 内部类类型 | 核心特征与外部类引用关系 | 实战应用场景与避坑 |
| :--- | :--- | :--- |
| **静态内部类（`static class`）** | 用 `static` 修饰，**不持有外部类实例引用**，仅能直接访问外部类静态成员 | **生产首选**（如 `HashMap.Node`、`Builder` 模式），无外部类内存泄漏风险 |
| **成员内部类（非 `static`）** | 隐式持有外部类实例引用 `Outer.this`，可访问外部类一切私有成员 | 需依附外部类实例创建（`outer.new Inner()`）；若生命周期长于外部类易引发**外部类内存泄漏** |
| **局部内部类** | 定义在方法或代码块内，仅在该作用域内可见 | 极少使用；访问方法局部变量要求该变量为 **`effectively final`** |
| **匿名内部类** | 无类名，声明的同时直接 `new 接口/父类() { ... }` 创建唯一实例 | 单次回调或重写抽象类方法；若实现的是单抽象方法函数式接口，**JDK 8+ 一律推荐用 `Lambda` 表达式替代** |

---

## 七、基础核心类（`Object` / `System`）与基础线性结构（数组 / 链表 / 异常速查）

### 7.1 根类 `Object` 核心方法与 `equals()` + `hashCode()` 铁律契约

#### 1. `java.lang.Object` 核心方法全景速查表

| 方法签名 | 核心作用与默认实现 | 面试高频注意点 |
| :--- | :--- | :--- |
| **`boolean equals(Object obj)`** | 判断两个对象是否“逻辑相等”；**`Object` 默认实现就是 `return (this == obj);`（比较内存地址）** | `String`、包装类、集合类均已重写为按**内容**比较；自定义业务实体按需重写 |
| **`int hashCode()`** | `native` 方法，返回对象的 32 位整型哈希码 | 支撑 `HashMap` / `HashSet` 的桶槽位计算；**重写 `equals()` 必须同步重写 `hashCode()`** |
| **`String toString()`** | 返回对象的字符串描述；默认返回 `类名@十六进制hashCode` | 建议所有 POJO/DTO 重写（或加 `@ToString`），便于日志排查 |
| **`final Class<?> getClass()`** | `native` 方法，返回对象运行时的真实 `Class` 元数据对象 | **反射（Reflection）的核心入口之一**，注意该方法为 `final` 不可重写 |
| **`protected Object clone()`** | `native` 方法，按字节复制并返回当前对象的**浅拷贝（Shallow Copy）**副本 | 调用类必须显式实现 `Cloneable` 标记接口，否则抛 `CloneNotSupportedException` |
| **`void wait() / notify() / notifyAll()`** | 传统管程线程等待与唤醒方法（`wait` 另有带超时参数的重载，共 5 个线程协作方法） | **必须在 `synchronized` 同步块内由锁对象调用**，否则抛 `IllegalMonitorStateException`；**`wait()` 会立即释放持有的对象锁** |
| **`protected void finalize()`** | GC 回收对象前调用的析构方法 | **JDK 9 已废弃，JDK 18+ 默认禁用**，严禁用于释放资源 |

#### 2. 【面试必考】为什么重写 `equals()` 时必须同时重写 `hashCode()`？
- **Java 官方三大约定（Contract）**：
  1. 如果两个对象通过 `equals()` 比较返回 **`true`**，那么它们的 `hashCode()` **必须返回完全相同的整数**；
  2. 如果两个对象的 `hashCode()` 相同，它们通过 `equals()` 比较**不一定为 `true`**（即存在**哈希碰撞 Hash Collision**）；
  3. 若两个对象的 `hashCode()` 不同，则它们 `equals()` 比较**必定为 `false`**。
- **只重写 `equals` 不重写 `hashCode` 的灾难后果**：
  - 当把自定义对象（如 `new User(1L, "张三")`）作为 Key 存入 `HashMap` 或 `HashSet` 时，`HashMap` **第一步是先调用 `key.hashCode()` 计算数组桶下标**；
  - 若未重写 `hashCode()`，直接继承了 `Object` 默认基于内存地址生成的哈希值，导致两个属性完全相同的 `User` 实例算出不同的 `hashCode`，被散列到不同的桶槽位中。此时根本走不到第二步 `equals()` 比对，造成 **`map.get(new User(1L, "张三"))` 查不到数据**、以及 `HashSet` 中存入大量重复对象的严重线上 Bug！

---

### 7.2 `java.lang.System` 核心系统方法与标准 I/O 速查

> *(关于 `Scanner` vs `BufferedReader` 算法竞赛快读、`PrintStream` 格式化与关闭标准流陷阱，详见 [`01-JavaBasic-04-字符串-数组-输入输出.md` 第八章](./01-JavaBasic-04-字符串-数组-输入输出.md)。)*

| 功能分类 | 核心 API / 静态字段 | 底层原理与工程实战要点 |
| :--- | :--- | :--- |
| **标准输入/输出/错误流** | `System.in`（`InputStream`）<br>`System.out`（`PrintStream`）<br>`System.err`（`PrintStream`）<br>`setIn / setOut / setErr`（流重定向） | - **`System.out` vs `System.err`**：`out` 对应操作系统文件描述符 `fd=1`（带行缓冲），`err` 对应 `fd=2`（无缓冲立即刷新，因此在 IDE 控制台中二者混打时顺序可能交错）；<br>- **生产红线**：`System.out.println` 内部包含全局 `synchronized (this)` 同步锁且同步阻塞磁盘/终端 I/O，**生产环境严禁使用，一律改用 SLF4J + Logback 异步日志**！ |
| **本地极速数组拷贝** | `System.arraycopy(src, srcPos, dest, destPos, length)` | **`native` 本地方法**，底层直接调用 C/C++ 的 `memmove` 在连续虚拟内存上做整块字节复制，比手写 `for` 循环快数倍，且安全支持**同一数组内部重叠区间自拷贝**（`ArrayList` 增删扩容核心底座） |
| **时间度量双雄** | `System.currentTimeMillis()`<br>`System.nanoTime()` | - `currentTimeMillis()`：返回自 UTC `1970-01-01 00:00:00` 起的毫秒时间戳，适合打业务绝对时间，但**受服务器 NTP 时间同步回拨影响**；<br>- `nanoTime()`：基于 CPU 硬件单调时钟源的纳秒计数，**不受系统时间修改影响，专用于精确测量一段代码的执行耗时差（`end - start`）** |
| **环境变量与系统属性** | `System.getenv("PATH")`<br>`System.getProperty("user.dir")` | `getenv` 读取操作系统环境变量；`getProperty` 读取 JVM 启动 `-Dkey=val` 参数与运行环境属性（如 `os.name`、`java.version`、`user.dir` 当前工作目录） |
| **原始哈希与进程控制** | `System.identityHashCode(obj)`<br>`System.exit(int status)`<br>`System.gc()` | - `identityHashCode(obj)`：无视对象重写的 `hashCode()`，强制返回 `Object` 默认的原始哈希码；<br>- `exit(0)`：触发 ShutdownHook 并立即终止整个 JVM 进程（唯一跳过 `finally` 块的操作）；<br>- `gc()`：仅向 JVM 发送垃圾回收“建议”，不保证立即执行 |

---

### 7.3 数组核心语法、内存模型与 `Arrays` 工具类避坑（含排序算法纠偏）

> *(关于 HotSpot 数组对象头 `Mark Word + Klass Word + 4字节配列长度`、CPU Cache Line 空间局部性及 `Arrays.asList()` 三大生产连环雷，详见 [`01-JavaBasic-04-字符串-数组-输入输出.md`](./01-JavaBasic-04-字符串-数组-输入输出.md)。)*

#### 1. 数组初始化与内存布局核心规则
```java
// 1. 一维数组 3 种合法初始化方式
int[] arr1 = new int[5];               // 动态初始化：指定长度，元素自动赋类型默认值 0
int[] arr2 = new int[]{10, 20, 30};    // 静态标准初始化（⚠️ 右侧中括号 [] 内严禁再写长度数字！）
int[] arr3 = {10, 20, 30};             // 静态简写语法糖（⚠️ 只能在声明变量的同时一行完成！拆成两行 int[] arr3; arr3 = {1, 2}; 直接编译报错！）

// 2. 二维数组本质：“数组的数组”（外层一维数组存放内层各行一维数组的堆地址，各行在堆内存中不一定连续，且每行长度可不同即“锯齿数组”）
int[][] jagged = new int[2][];         // 外层行数必填，内层列数可留空后续动态分配
jagged[0] = new int[]{1, 2};
jagged[1] = new int[]{3, 4, 5, 6};
```

#### 2. 数组 vs `String` vs `ArrayList` 核心对比表

| 对比维度 | 数组（`T[]` / `int[]`） | 字符串（`String`） | 动态数组集合（`ArrayList<E>`） |
| :--- | :--- | :--- | :--- |
| **可变性与长度** | **长度创建后固定不可变**；内部元素值可自由修改 | **对象内容与长度均绝对不可变**（拼接/替换均生成全新的 `String` 对象） | **长度动态可变**（底层数组满时自动按 `1.5` 倍 `oldCap + (oldCap >> 1)` 扩容） |
| **获取长度语法** | **`arr.length`**（数组对象头的只读**属性**，无括号） | **`str.length()`**（封装内部数组长度的成员**方法**，有括号） | **`list.size()`**（集合接口方法，返回实际存入的元素个数） |
| **支持元素类型** | 既支持**基本类型**（连续存值），也支持**引用类型**（连续存地址） | 底层存字符序列（JDK 8 `final char[]`，JDK 9+ 紧凑字符串 `final byte[]`） | **仅支持引用类型/包装类**（不支持基本类型） |
| **打印与判等** | 直接打印输出 `[I@哈希码`；需用 **`Arrays.toString(arr)`** 打印内容、**`Arrays.equals(a, b)`** 判等（多维数组用 `deepToString / deepEquals`） | 已重写 `toString()` 和 `equals()` | 已重写 `toString()` 和 `equals()` |

#### 3. 数组拷贝三剑客与 `Arrays.sort / binarySearch` 底层机制（⚠️ 权威纠偏）
- **数组拷贝三种方式对比（注意：对引用类型数组均为“浅拷贝 Shallow Copy”，仅拷贝元素引用地址）**：
  1. **`System.arraycopy(src, srcPos, dest, destPos, len)`**：最底层 `native` 内存复制，需调用方预先建好目标数组 `dest`，支持局部区间复制与自复制，性能最高；
  2. **`Arrays.copyOf(src, newLength)`**：内部自动新建长度为 `newLength` 的新数组，然后直接调用 `System.arraycopy()` 完成复制，常用于**数组动态扩容或截断**；
  3. **`arr.clone()`**：返回与原数组同类型、同长度的完整浅拷贝副本。
- **⚠️ 纠偏 1：`Arrays.sort()` 对基本类型数组和对象数组采用完全不同的排序算法！**
  - **基本数据类型数组（`int[]`、`double[]` 等）**：采用 **`DualPivotQuicksort`（双轴快速排序）**（结合小数组 `< 47` 退化为插入排序、超大重复度数组采用计数排序）。因为相同的两个基本类型整数（如两个 `5`）没有任何身份区别，无需保证排序稳定性，双轴快排具备极致的 CPU 缓存局部性与原地零额外空间开销；
  - **对象引用数组（`Object[]`）**：采用 **`TimSort`（归并排序 MergeSort + 二分插入排序 Binary Insertion Sort 的混合稳定排序算法）**。因为对象排序**必须保证稳定性（Stability）**（相同权重的两个订单排序后不能打乱原有的创建时间先后顺序），且 `TimSort` 对业务中常见的“部分已近乎有序数据”最快可达 $O(N)$。
- **⚠️ 纠偏 2：`Arrays.binarySearch(arr, key)` 的前提条件**：
  - 调用二分查找前，**目标数组必须已经按升序（Ascending）排好序**（若为降序数组且未传入配套的逆序 `Comparator`，二分查找方向相反会直接返回错误负数结果）！
  - 若未找到目标值，返回值为 **`-(insertionPoint + 1)`**（之所以减 1，是为了区分插入点在下标 `0` 时的 `-1` 与命中下标 `0` 时的 `0`）。

---

### 7.4 手写单链表（`ListNode`）与标准库 `LinkedList` 对比

```java
// 算法笔试/面试手写单链表标准节点模板
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; }
    public ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

| 对比维度 | 手写单向链表（`ListNode`） | JDK 标准库 `java.util.LinkedList<E>` |
| :--- | :--- | :--- |
| **底层指针结构** | **单向链表**（仅含 `val` + 后继指针 `next`） | **双向链表**（内部静态类 `Node<E>` 含 `item` + 前驱 `prev` + 后继 `next`，维护 `first` 与 `last` 头尾引用） |
| **核心操作复杂度** | 头插 $O(1)$；无尾指针时尾插需遍历 $O(N)$；按索引查找 $O(N)$ | 头尾增删（`addFirst/addLast/pollFirst/pollLast`）均为 **$O(1)$**；按索引 `get(i)` 采用折半遍历（前半从头找、后半从尾找），仍为 **$O(N)$** |
| **适用场景** | LeetCode 算法题（反转链表、快慢指针判环、合并有序链表） | 实现双端队列 `Deque` 或栈；常规顺序列表优先用缓存友好的 `ArrayList` 或 `ArrayDeque` |

---

### 7.5 异常处理核心速查：`try-catch-finally` 执行顺序与 `try-with-resources`

> *(关于自定义异常、异常链包装、Spring 全局异常处理 `@RestControllerAdvice` 详见 [`01-JavaBasic-03-异常处理-IO流.md`](./01-JavaBasic-03-异常处理-IO流.md)。)*

#### 1. `try-catch-finally` 遇到 `return` 时的字节码执行铁律（笔试高频坑题）
```java
public static int testBasicReturn() {
    int num = 10;
    try {
        return num; // ① 先将返回值 10 拷贝暂存到局部变量表的专用 returnValue 槽位，然后转去执行 finally
    } finally {
        num = 20;   // ② 修改的是局部变量 num，完全不影响已暂存的 returnValue 槽位！
    }
} // 调用结果返回：10（绝不是 20！）

public static int testFinallyReturnAntiPattern() {
    try {
        int a = 1 / 0; // 抛出 ArithmeticException
        return 10;
    } finally {
        return 99;     // ❌ 【阿里开发手册严禁写法】finally 中写 return 会强行覆盖 try/catch 的返回值，且将算术异常静默吞没！
    }
} // 调用结果：不抛任何异常，直接返回 99！
```

#### 2. `try-with-resources`（JDK 7+）vs 传统 `try-catch-finally` 全方位对比

| 对比维度 | `try-with-resources`（生产强制推荐） | 传统 `try-catch-finally` |
| :--- | :--- | :--- |
| **资源接口要求** | 资源类必须实现 **`java.lang.AutoCloseable`**（或子接口 `Closeable`） | 无接口限制，需手动在 `finally` 中判空并调 `close()` |
| **关闭顺序与原理** | `javac` 编译期自动生成嵌套 `finally` 块；括号内声明多个资源时，**按声明的相反顺序（后声明的先关闭）自动调用 `close()`** | 需手写多层嵌套 `try-catch` 关闭，代码极其臃肿易漏关 |
| **异常屏蔽问题（Suppressed Exception）** | ✅ **完美保留原始主异常**：若 `try` 块抛出业务异常 A，且自动 `close()` 时又抛出关闭异常 B，**向外抛出的是主异常 A**，而异常 B 被自动通过 `A.addSuppressed(B)` 挂载到主异常的抑制列表中（可通过 `e.getSuppressed()` 取出） | ❌ **原始异常被吞没**：若 `try` 块抛出异常 A，而 `finally` 中 `close()` 又抛出异常 B，最终向外抛出的是异常 B，**导致真正引发故障的第一现场异常 A 彻底丢失**！ |

```java
// JDK 7+ / JDK 9+（支持直接传入外部已定义的 effectively final 资源变量）标准写法
try (BufferedInputStream in = new BufferedInputStream(new FileInputStream("input.txt"));
     BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream("output.txt"))) {
    in.transferTo(out); // JDK 9+ 零样板流拷贝
} catch (IOException e) {
    log.error("文件流拷贝失败", e);
}
```

---

## 八、Java 8+ 函数式编程基石（`Lambda` 表达式与方法引用 `::`）

### 8.1 函数式接口（`@FunctionalInterface`）与 JDK 四大核心内置接口

> **什么是函数式接口（Functional Interface）？**
> **有且仅有一个抽象方法（Single Abstract Method, SAM）**的接口（接口中可以包含任意多个 `default` 默认方法、`static` 静态方法、以及重写自 `Object` 类的公开方法如 `boolean equals(Object)`）。添加 `@FunctionalInterface` 注解可让编译器强制校验该契约。**Lambda 表达式的本质就是函数式接口的实例！**

#### `java.util.function` 四大核心函数式接口与常用变体速查表

| 接口名称 | 核心抽象方法签名 | 函数描述符 | 核心语义与 Stream 典型应用场景 |
| :--- | :--- | :---: | :--- |
| **`Consumer<T>`**<br>（消费型接口） | `void accept(T t)` | `T -> void` | 接收 1 个参数并消费处理，无返回值。<br>典型应用：`list.forEach(item -> ...)`、`Optional.ifPresent(...)` |
| **`Supplier<T>`**<br>（供给型/工厂接口） | `T get()` | `() -> T` | 无入参，按需生产并返回一个结果（常用于懒加载）。<br>典型应用：`Optional.orElseGet(() -> createDefault())`、`Stream.generate(...)` |
| **`Function<T, R>`**<br>（函数转换型接口） | `R apply(T t)` | `T -> R` | 接收类型 `T` 参数，转换映射后返回类型 `R` 结果。<br>典型应用：`stream.map(User::getName)`、`map.computeIfAbsent(k, key -> ...)` |
| **`Predicate<T>`**<br>（断言判断型接口） | `boolean test(T t)` | `T -> boolean` | 接收类型 `T` 参数，返回布尔判断结果（支持 `and / or / negate` 组合）。<br>典型应用：`stream.filter(u -> u.getAge() >= 18)`、`list.removeIf(...)` |
| **双参扩展与线程接口** | `BiFunction<T,U,R>` / `BiPredicate<T,U>` / `BiConsumer<T,U>` / `Runnable` / `Callable<V>` | `(T, U) -> R`<br>`() -> void / V` | `map.forEach((k, v) -> ...)`、线程池提交异步任务 |

---

### 8.2 `Lambda` 表达式语法简写规则与底层 `invokedynamic` 原理

#### 1. 语法简写三原则
```java
// 标准形式：(参数列表) -> { 方法体; }
// 原则 1：参数类型全部可省略（由编译器根据目标函数式接口泛型自动推导）
// 原则 2：若【有且仅有 1 个参数】，小括号 () 可省略（0 个参数或多个参数时小括号绝不能省！）
// 原则 3：若方法体【只有 1 条语句】，大括号 {}、分号 ; 以及 return 关键字必须同时省略！
Comparator<Integer> cmp1 = (Integer a, Integer b) -> { return Integer.compare(a, b); }; // 完整版
Comparator<Integer> cmp2 = (a, b) -> Integer.compare(a, b);                             // 极简版
```

#### 2. 【面试深水区】`Lambda` 表达式与匿名内部类的 4 大本质区别

| 对比维度 | 匿名内部类（Anonymous Inner Class） | `Lambda` 表达式 |
| :--- | :--- | :--- |
| **适用目标范围** | 可实现**任意接口**（无论多少个抽象方法），也可继承**抽象类或普通类** | **仅限于实现函数式接口**（仅含 1 个抽象方法的接口，绝不可用于抽象类） |
| **编译产物与底层机制** | 编译期物理生成独立的 **`外部类$1.class`** 字节码文件，每次执行直接 `new` 对象 | **不会生成独立的 `.class` 文件**！编译期在当前类中生成一个私有静态方法，并植入 **`invokedynamic` 字节码指令**，运行期首次调用时通过 `LambdaMetafactory.metafactory()` 动态在内存中生成调用点字节码 |
| **`this` 关键字指向（极易踩坑！）** | `this` 指向**匿名内部类自身创建的那个实例对象** | Lambda 不开辟独立类作用域，**Lambda 内部的 `this` 直接指向外层包围类的实例对象**！ |
| **作用域与变量遮蔽** | 开辟独立作用域，内部可定义与外层局部变量同名的变量（遮蔽外层） | 与外层方法共用同一词法作用域，**严禁声明与外层方法同名的局部变量或形参** |

#### 3. 为什么 `Lambda`（及匿名内部类）访问外部局部变量必须是 `effectively final`？
```java
int base = 10;
// base = 20; // ❌ 若取消此行注释，下一行 Lambda 直接报编译错误：Variable used in lambda expression should be final or effectively final
Function<Integer, Integer> adder = x -> x + base;
```
- **底层内存生命周期矛盾**：
  1. **局部变量 `base` 存储在当前线程的虚拟机栈帧中**，一旦外层方法执行完毕出栈，栈上的 `base` 就立刻灰飞烟灭了；
  2. 而 **`Lambda` 表达式实例对象存储在 Java 堆中**，它可能被返回给调用方、或提交给异步线程池，在外层方法结束后很久才被执行；
  3. 为了让堆上的 `Lambda` 在未来仍能读到 `base`，Java 底层采用的是**“值捕获（Capture by Value）”**——在创建 `Lambda` 实例时，直接将 `base` 的值**拷贝一份副本**存进 `Lambda` 内部持有；
  4. **设计铁律**：如果允许外层方法或 `Lambda` 内部修改 `base`，就会出现“栈上原变量改了，而堆上拷贝副本没变”的**诡异数据不一致分裂现象**。因此 Java 语言规范从编译期强制规定：被捕获的局部变量必须是 `final` 或 **`effectively final`（有效只读，即初始化后从未发生过二次赋值）**！
  *(注：类的实例成员变量或 `static` 静态变量因为本身就存储在堆中，无需栈变量拷贝，所以在 Lambda 中可以随意读写，不受此限制。)*

---

### 8.3 四种方法引用（`::`）与参数自动传递规则对照表（难点彻底打通）

当 `Lambda` 表达式的方法体**仅仅是直接调用一个已经存在的方法，不做任何额外逻辑**时，可用方法引用 `::` 进一步化简。

#### 四种方法引用语法与参数自动推导全景对照表

| 引用类型 | 语法格式 | 参数自动传递核心规则（一眼看穿怎么传参！） | 等价 `Lambda` 表达式对照 | 标准方法引用写法 |
| :--- | :--- | :--- | :--- | :--- |
| **1. 静态方法引用** | **`类名::静态方法名`** | 函数式接口抽象方法的**全部参数**，原封不动全部传给该静态方法 | `(String s) -> Integer.parseInt(s)` | `Integer::parseInt` |
| **2. 特定对象的实例方法引用** | **`现成对象引用::实例方法名`** | 函数式接口抽象方法的**全部参数**，原封不动全部传给这个现成对象的实例方法 | `(String s) -> System.out.println(s)`<br>`(String s) -> sb.append(s)` | `System.out::println`<br>`sb::append` |
| **3. 特定类型的任意对象实例方法引用** ⭐ | **`类名::实例方法名`**<br>*(最易混淆，笔试重点！)* | 接口抽象方法的 **【第 1 个参数】自动作为调用该实例方法的目标对象（`this`）**，**【剩余的第 2~N 个参数】依次作为该实例方法的入参**！ | ① 单参接口 `Function<String, Integer>`：<br>`(String s) -> s.length()`<br>② 双参接口 `BiPredicate<String, String>`：<br>`(String s1, String s2) -> s1.equals(s2)`<br>③ 双参接口 `BiFunction<String, String, String>`：<br>`(String s1, String s2) -> s1.concat(s2)` | ① `String::length`<br>② `String::equals`<br>③ `String::concat` |
| **4. 构造器 / 数组构造引用** | **`类名::new`**<br>**`类型[]::new`** | 函数式接口抽象方法的**全部参数**，原封不动传给匹配参数列表的构造函数（或作为新建数组的长度） | ① 无参 `Supplier<User>`：`() -> new User()`<br>② 单参 `Function<String, User>`：`name -> new User(name)`<br>③ 数组 `IntFunction<String[]>`：`len -> new String[len]` | ① `User::new`<br>② `User::new`<br>③ `String[]::new` |

```java
// 实战秒懂验证：为什么 String::equals 和 String::length 虽然都是【实例方法】，却能用【类名::实例方法】引用？
// 1. String::length 是无参实例方法 -> 消费接口的第 1 个参数 s 作为调用者：s.length()
Function<String, Integer> lenFunc = String::length;
System.out.println(lenFunc.apply("Antigravity")); // 等价于 "Antigravity".length() -> 11

// 2. String::equals 是单参实例方法 -> 第 1 个参数 s1 作为调用者，第 2 个参数 s2 作为 equals 入参：s1.equals(s2)
BiPredicate<String, String> eqPred = String::equals;
System.out.println(eqPred.test("Java", "Java"));  // 等价于 "Java".equals("Java") -> true
```
