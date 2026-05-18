### Java 语言基础

#### Java编译和运行命令

Java 的编译和运行核心依赖两个命令：`javac`（编译命令）和 `java`（运行命令）。下面我会从「基础用法」「带包结构的用法」「常见问题」三个维度讲解，确保你能直接上手，同时避开新手容易踩的坑。

##### 一、基础用法（无包结构）

适用于简单的单个 Java 文件（文件中没有 `package` 声明）。

###### 1. 编译：`javac` 命令

**作用**：将 `.java` 源文件编译成 JVM 能识别的 `.class` 字节码文件。
**语法**：

```bash
javac [选项] 源文件名.java
```

**示例**：
假设你有一个 `Hello.java` 文件，内容如下：
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

编译步骤：
1. 打开终端/命令提示符，进入 `Hello.java` 所在的目录；
2. 执行编译命令：
```bash
javac Hello.java
```
- 执行成功后，目录下会生成 `Hello.class` 文件（字节码文件）；
- 如果代码有语法错误，`javac` 会输出错误提示（比如少分号、拼写错误），需修复后重新编译。

###### 2. 运行：`java` 命令

**作用**：启动 JVM，加载并执行 `.class` 字节码文件。
**语法**：
```bash
java [选项] 类名
```

**注意**：运行时**不需要加 `.class` 后缀**，只需要写类名（比如 `Hello` 而非 `Hello.class`）。

**示例**：
编译成功后，执行运行命令：
```bash
java Hello
```
输出结果：
```
Hello, Java!
```

##### 二、带包结构的用法（新手重点避坑）

实际开发中，Java 文件都会有 `package` 声明（包结构），此时编译和运行的路径规则非常关键。

###### 示例场景

假设你有如下目录结构：
```
src/
  └── com/
      └── demo/
          └── Hello.java
```
`Hello.java` 内容：
```java
package com.demo; // 包声明

public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Package!");
    }
}
```

###### 1. 编译（指定输出目录）

使用 `-d` 选项指定编译后的 `.class` 文件输出目录（推荐做法），避免手动创建目录：
```bash
# 进入 src 目录的上级目录（假设当前在 src 同级目录）
javac -d ./bin src/com/demo/Hello.java
```
- `-d ./bin`：表示将编译后的类文件输出到当前目录下的 `bin` 文件夹；
- 执行后，`bin` 目录会自动生成和包结构一致的目录：`bin/com/demo/Hello.class`。

###### 2. 运行（指定类路径）

使用 `-cp`（或 `-classpath`）选项指定类路径，然后执行**全限定类名**（包名+类名）：
```bash
# 进入 bin 目录的上级目录
java -cp ./bin com.demo.Hello
```
输出结果：
```
Hello, Package!
```

**核心规则**：
- 运行时的类路径（`-cp` 指定）要指向包结构的「根目录」（比如 `bin`）；
- 类名必须写**全限定名**（比如 `com.demo.Hello`），而非文件路径。

##### 三、常见选项与问题解决

###### 1. 常用命令选项

| 命令    | 选项              | 作用                                                     |
| ------- | ----------------- | -------------------------------------------------------- |
| `javac` | `-d <目录>`       | 指定生成的.class文件输出目录                             |
| `javac` | `-encoding UTF-8` | 指定源文件编码（解决中文乱码）                           |
| `java`  | `-cp <路径>`      | 指定类路径（多个路径用;（Windows）或:（Linux/Mac）分隔） |
| `java`  | `-version`        | 查看JDK版本（验证环境是否配置）                          |

###### 2. 新手常见问题

- **问题1**：运行时提示 `Error: Could not find or load main class Hello`
  原因：① 类名写错（比如加了 `.class` 后缀）；② 当前目录不在类路径中；③ 带包结构时没写全限定类名。
  解决：运行时只写类名（无后缀），带包则写全限定名，用 `-cp` 指定类路径。

- **问题2**：编译时中文乱码
  原因：源文件编码和 `javac` 默认编码不一致。
  解决：编译时指定编码：
  ```bash
  javac -encoding UTF-8 Hello.java
  ```

- **问题3**：`javac`/`java` 命令不识别
  原因：JDK 环境变量（`JAVA_HOME`、`PATH`）未配置。
  解决：检查环境变量，确保 `%JAVA_HOME%\bin`（Windows）或 `$JAVA_HOME/bin`（Linux/Mac）加入 `PATH`。

##### 总结

1. 核心命令：`javac 源文件.java` 编译生成 `.class`，`java 类名` 运行（无 `.class` 后缀）；
2. 带包结构时，编译用 `-d` 指定输出目录，运行用 `-cp` 指定类路径 + 全限定类名；
3. 常见坑：运行时加后缀、包结构类名写错、编码不一致，对应用 `-encoding`/`-cp` 解决。

### 一、Java 核心概念与环境

#### **1.Java 语言的跨平台特性是如何实现的？其核心原理是什么？**

Java 的跨平台特性通过 **JVM（Java Virtual Machine）** 实现。Java 程序编译后生成字节码文件（.class），这些字节码可以在任何安装了 JVM 的操作系统上运行，而不需要重新编译。JVM 将字节码翻译为对应平台的机器码，因此实现了“编写一次，到处运行”的跨平台特性。

#### **2.JDK、JRE 和 JVM 三者之间有什么区别与联系？**

- **JVM (Java Virtual Machine)**：是运行 Java 字节码的虚拟机，负责将字节码转换成本地机器码。
- **JRE (Java Runtime Environment)**：是 Java 运行环境，包含了 JVM 和 Java 类库，使得 Java 程序可以在某个平台上运行。
- **JDK (Java Development Kit)**：是 Java 开发工具包，包含 JRE 以及用于开发 Java 应用的编译器（javac）和调试工具等。



---

#### 2、为什么 Java 被称作是“平台无关的编程语言”？

Java 实现**一次编写，到处运行（Write Once, Run Anywhere）**，原因：
1. Java 源码不直接编译成操作系统/CPU 专属的机器码，而是编译成**字节码（.class）**。
2. 不同平台（Windows、macOS、Linux）只需要安装对应版本的 **JVM（Java 虚拟机）**。
3. JVM 负责把字节码翻译成当前平台能识别的机器码，**源码和字节码完全不用修改**。

一句话总结：**不是 Java 语言跨平台，是 JVM 跨平台，让 Java 实现了平台无关。**

---

#### 3、Java 和 C++ 的区别？（高频面试）

| 对比项     | Java                             | C++                            |
| ---------- | -------------------------------- | ------------------------------ |
| 平台性     | 平台无关（依赖 JVM）             | 平台相关（直接编译成机器码）   |
| 内存管理   | 自动垃圾回收（GC），无需手动释放 | 手动 malloc/free / new/delete  |
| 指针       | 不支持直接操作指针               | 支持指针，可直接操作内存       |
| 多继承     | 只支持**接口多继承**，类单继承   | 支持类的多继承                 |
| 语法复杂度 | 简洁，无运算符重载、无析构函数   | 复杂，功能更强大               |
| 运行效率   | 稍低（虚拟机解释/编译）          | 更高（直接运行）               |
| 适用场景   | 企业开发、Web、大数据、安卓      | 底层、游戏、嵌入式、高性能程序 |

核心区别：
**Java 更安全、简单、自动管理内存；C++ 更底层、更高效、可控性更强。**

---

#### 4、什么是字节码？采用字节码的最大好处是什么？

##### 什么是字节码？

Java 源码（.java）经过 `javac` 编译后生成的 **.class 文件** 就是字节码。
它是一种**中间代码**，不是机器码，人类不可读，只能由 JVM 识别。

##### 采用字节码的最大好处

1. **实现平台无关性**：一份字节码可以在所有装有 JVM 的系统上运行。
2. **提升执行效率**：比直接解释源码快得多，同时支持 JIT 即时编译优化。
3. **安全可控**：字节码由 JVM 加载执行，有严格的安全校验，避免直接操作内存。

**核心好处：实现 Java 的跨平台特性。**

---

#### 5、Java 运行的过程？

标准流程（4 步）：
1. **编写源码**：.java 源代码文件。
2. **编译**：使用 `javac` 编译成 **字节码 .class 文件**。
3. **类加载**：JVM 的类加载器加载 .class 文件到内存。
4. **执行**：
   - 解释器：逐行解释字节码运行
   - JIT 编译器：将热点代码编译成机器码，加速执行

简化记忆：
**源码 → 编译成字节码 → JVM 加载 → 解释/编译执行**

---

#### 6、Java 是动态类型语言还是静态类型语言？

**Java 是典型的静态类型语言（强类型语言）。**

判断依据：
1. **编译时检查类型**：变量必须先声明类型，且类型不能随意改变。
2. **类型安全**：编译期就会检查类型错误，不会等到运行时才报错。
3. 虽然 Java 有反射、泛型擦除等动态特性，但**从语言分类上属于静态类型**。

对比：
- 静态类型：Java、C、C++、Go
- 动态类型：Python、JavaScript、PHP

---

##### 总结

1. Java 平台无关 = **JVM 跨平台 + 字节码**
2. Java vs C++ = **自动GC、无指针、单继承、简单安全**
3. 字节码 = .class 文件，**核心价值：跨平台**
4. Java 运行 = 源码→字节码→JVM加载→执行
5. Java = **静态类型语言**

这些都是 Java 最基础、面试必问的知识点，记熟就够用啦！

#### **3.Java 中的面向对象三大特征是什么？分别简要说明其含义。**

- **封装**：将对象的状态和行为打包在一起，外界无法直接访问对象的内部状态，必须通过公共方法访问。
  - **状态 = 成员变量（属性 / 字段）**
  - **行为 = 成员方法（方法 / 函数）**
- **继承**：通过继承机制，子类可以继承父类的属性和方法，促进代码复用。
- **多态**：同一方法或操作作用于不同的对象时，能够产生不同的行为。通过方法重载和方法重写实现。

#### **4.什么是值传递和引用传递？Java 中是值传递还是引用传递？**

- **值传递**：传递的是变量的值，调用方法时修改参数的值不会影响原始变量。
- **引用传递**：传递的是对象的引用，调用方法时**修改对象的属性**会影响原始对象，而不会改变原始引用。例如Map,Set,LIst等可变对象可以被修改。String是不可变对象。
- Java 是 **值传递**，即所有的参数传递都是值传递。对于对象，传递的是对象的引用值，但引用本身是传递的副本。

这是一个 Java 面试中最高频、也最容易产生误解的问题。简单直接的回答是：**Java 中只有值传递（Pass by Value），不存在引用传递（Pass by Reference）**[[source_group_web_1]]。

之所以会产生困惑，是因为 Java 在传递对象时，传递的这个“值”恰好是一个**引用的副本**[[source_group_web_2]]。

为了彻底搞懂这个问题，我们需要先明确两个概念，然后分情况讨论 Java 的行为。

##### 1. 什么是值传递 vs. 引用传递？

- 值传递 (Pass by Value)：
  - **定义**：调用方法时，会将实际参数的**值**（或副本）传递给形式参数。
  - **后果**：在方法内部对形参的任何修改（无论是修改值还是重新赋值），都只影响副本，**永远不会影响**原始的实际参数。
- 引用传递 (Pass by Reference)：
  - **定义**：调用方法时，传递的是实际参数的**内存地址**（引用本身，而不是副本）。
  - **后果**：形参和实参指向同一块内存地址。在方法内部修改形参指向的内容，或者直接给形参重新赋值（改变指向），都会**直接影响**原始的实际参数。

##### 2. Java 中是如何传递的？

Java 坚定地选择了**值传递**[[source_group_web_4]]。但是，根据传递参数的类型不同（基本类型 vs. 引用类型），表现出来的现象有所不同。

###### 📏 情况一：基本数据类型 (Primitive Types)

当传递 `int`, `double`, `boolean` 等基本类型时，传递的是**变量值的副本**。

- **现象**：方法内修改形参，**不影响**外部实参。

- 代码示例：

  ```java
  public class Test {
      public static void main(String[] args) {
          int a = 10;
          changePrimitive(a);
          System.out.println(a); // 输出 10，a 的值没有改变
      }
      
      public static void changePrimitive(int x) {
          x = 20; // 修改的是副本 x，不影响原来的 a
      }
  }
  ```

  内存解释：`a`的值 10 被复制了一份给 `x`。`x`的改变与 a 无关。

###### 🏗️ 情况二：引用数据类型 (Object/Reference Types)

当传递对象（如 `List`, `Map`, 自定义类等）时，传递的是**对象引用的副本**。

这里需要区分两个操作：

1. **修改对象的内容（属性/元素）**：**会影响**外部对象。
2. **修改引用的指向（重新赋值）**：**不会影响**外部引用。

- **代码示例**：

  ```java
  public class Test {
      public static void main(String[] args) {
          Person p = new Person("Alice");
          changeObject(p);
          System.out.println(p.getName()); // 输出 "Bob"，名字被改了
          
          List<String> list = new ArrayList<>();
          list.add("A");
          modifyList(list);
          System.out.println(list.size()); // 输出 2，列表内容变了
      }
      
      public static void changeObject(Person person) {
          person.setName("Bob"); // 1. 修改对象内容 -> 影响外部
          
          person = new Person("Charlie"); // 2. 重新赋值引用
          // 此时 person 指向了新对象，但这不影响 main 方法里的 p
          // main 方法里的 p 依然指向 "Bob"
      }
      
      public static void modifyList(List<String> lst) {
          lst.add("B"); // 修改集合内容 -> 影响外部
          lst = new ArrayList<>(); // 重新赋值引用 -> 不影响外部的 list
      }
  }
  ```

- **内存解释（关键）**：

  - 外部的引用变量（如 `p`）存储在栈中，它指向堆中的对象。
  - 调用方法时，**引用变量 `p` 的值（即堆中对象的地址）被复制了一份**传给了形参 `person`。
  - 此时，`p` 和 `person` 是两个不同的变量（都在栈中），但它们存储的值（地址）相同，都指向堆中的同一个对象。
  - 当你执行 `person.setName("Bob")` 时，是顺着地址找到堆中的对象并修改它，所以外部能看到。
  - 当你执行 `person = new Person(...)` 时，是把副本 `person` 指向了一个新地址，这并不会改变原始引用 `p` 的指向[[source_group_web_7]]。

##### 3. 特殊情况：String 类型

`String` 是一个经典的“坑”。它是引用类型，但表现得像基本类型。

- **现象**：修改 `String` 形参，**不影响**外部 `String`。
- 原因：不可变性。
  - `String` 类被设计为不可变的（`final` 修饰）。
  - 当你尝试修改字符串内容（如 `s = s + "new"`）时，JVM 实际上是创建了一个**新的** `String` 对象，并将形参指向这个新对象。
  - 这本质上就是上面提到的“修改引用的指向”，所以不影响外部的原始引用。

##### 📌 总结

| 传递类型     | 传递的内容     | 修改内容的影响         | 重新赋值的影响     |
| ------------ | -------------- | ---------------------- | ------------------ |
| **基本类型** | **值的副本**   | 无（修改副本）         | 无                 |
| **引用类型** | **引用的副本** | **有**（指向同一对象） | 无（修改副本指向） |

**一句话总结**：Java 只有值传递。对于对象，传递的是“引用的副本”。你可以通过副本去修改它指向的对象（内容变了），但你不能通过副本去改变原引用变量本身（指向没变）。

##### 什么是引用类型？

除了 **8 种基本类型** 以外，都是**引用类型**。

包括：

- 类（String、自定义类）
- 接口
- 数组

特点：

- 变量中存的**不是值本身，而是对象的地址（引用）**
- 对象本身在**堆内存**，引用在栈

------

#### **5.解释 Java 中的多态，实现多态需要满足哪些条件？**

**多态**：同一操作作用于不同对象时，表现出不同的行为。多态有两种形式：方法重载（编译时多态）和方法重写（运行时多态）。
实现多态需要满足以下条件：

- 继承：子类继承父类。
- 方法重写：子类重写父类方法。
- 向上转型：将子类对象赋值给父类引用。

##### 💡 核心定义

多态（Polymorphism）是面向对象编程的三大核心特性之一（另外两个是封装和继承）。简单来说，它允许我们**将子类对象当作父类对象来使用**，在执行相同的操作时，不同的子类对象会表现出不同的行为。

##### 📜 实现运行时多态的三大条件

你提到的三点是实现**运行时多态**（即通过方法重写实现的动态绑定）的核心条件，我们可以对它们稍作展开：

\1. **继承关系 (Inheritance)**
  \*  必须存在父类和子类的关系，或者类与接口的关系。这是多态的基础，为“统一调用”提供了前提[[source_group_web_1]]。
\2. **方法重写 (Method Overriding)**
  \*  子类必须重写父类的非私有、非静态、非 final 的方法。这是多态的核心，为“不同行为”提供了保障。
\3. **向上转型 (Upcasting)**
  \*  必须使用父类的引用指向子类的对象。这是多态的表现形式，为“统一接口”提供了载体[[source_group_web_4]]。
  \*  语法：`父类类型 变量名 = new 子类类型();`

##### ⚖️ 两种多态的对比

虽然“多态”一词通常指代运行时多态，但严格来说，Java 中存在两种多态形式。你的回答已经点明了这一点，这里做一个更系统的对比：

| 特性         | 编译时多态 (静态多态)                                    | 运行时多态 (动态多态)                                       |
| ------------ | -------------------------------------------------------- | ----------------------------------------------------------- |
| **实现方式** | **方法重载** (Overloading)                               | **方法重写** (Overriding)                                   |
| **绑定时机** | 在**编译期**由编译器根据参数类型、数量决定调用哪个方法。 | 在**运行期**由 JVM 根据对象的实际类型动态决定调用哪个方法。 |
| **核心机制** | 参数列表必须不同 (参数个数、类型或顺序)。                | 继承 + 重写 + 向上转型。                                    |
| **灵活性**   | 灵活性较低，方法调用关系在代码编译后就已确定。           | 灵活性高，新增子类无需修改原有调用逻辑，符合开闭原则。      |

##### ⚠️ 注意事项

在讨论多态时，有几个常见的误区需要避免：

\*  **成员变量没有多态性**：变量的访问只看引用类型（编译时类型），而不是对象的实际类型。即“**变量看左边，方法看右边**”。
\*  **静态方法没有多态性**：静态方法属于类，不属于实例，调用时由编译器根据引用类型决定，不存在动态绑定。
\*  **`final` 和 `private` 方法**：`final` 方法不能被重写，`private` 方法对子类不可见，因此它们都不具备多态特性。

##### 🚀 总结

总而言之，多态的核心价值在于**解耦**和**扩展性**。它让我们的代码可以**面向抽象（父类或接口）编程**，而不是面向具体实现编程。当你需要添加新的功能时，只需添加新的子类并重写方法，而无需修改已有的调用代码。

我直接给你**Java 最标准、秋招笔试面试必背的三大阶段**，完全对应你这句：

##### Java程序生命周期

也常被称为：
**编译阶段 → 类加载阶段 → 运行阶段**

###### 1. 编译期（Compile Time）

- `.java` → 由 **javac 编译器** 编译成 `.class` 字节码
- 做的事：语法检查、类型检查、泛型擦除、常量优化
- 错误：**编译错误**（语法错、类型不匹配）

###### 2. 类加载期（Class Loading）

- `.class` → 由 **JVM 类加载器** 加载到内存
- 过程：**加载 → 验证 → 准备 → 解析 → 初始化**
- 这是**编译和运行之间的关键阶段**，银行、大厂必考

###### 3. 运行期（Run Time）

- JVM 执行字节码，由 **解释器 + JIT 即时编译器** 执行
- 错误：**运行时异常**（空指针、数组越界等）

------

#### **6.什么是类和对象？类的成员通常包括哪些部分？**

- **类**：是对象的模板或蓝图，定义了对象的属性和行为。
- **对象**：类的实例化，代表类的具体存在。
   类的成员通常包括：字段（属性）、方法、构造方法、内部类、静态块、实例块。

#### **7.Java 中的构造方法有什么作用？它有哪些特点？**

构造方法用于创建对象并初始化对象的状态。特点：

- 名称与类名相同。
- 没有返回类型。
- 可以重载。
- 每当创建对象时，构造方法会自动调用。

#### **8.什么是方法的重载？方法重载需要满足哪些条件？**

**方法重载**：同一个类中，方法名相同，但参数类型、数量或顺序不同。
条件：

- 方法名相同。
- 参数列表不同（参数类型不同、参数个数不同或参数顺序不同）。
- 返回类型可以相同或不同，但不作为区分重载的依据。

#### **9.什么是方法的重写？方法重写需要满足哪些条件？**

**方法重写**：子类重新定义父类的方法，方法签名相同，目的是改变父类方法的实现。
条件：

- 方法名相同。
- 参数列表相同。
- 返回类型相同或协变（Java 5 及以上支持返回类型协变）。
- 重写方法不能低于父类方法的访问权限。

**方法签名**指的是：**方法名 + 参数列表（参数个数、类型、顺序）**

##### 方法重写（Method Overriding）

**核心概念**：子类提供父类方法的**特定实现**，实现运行时多态。

---

##### ✅ 完整条件清单

| 条件                | 说明                                   | 示例                                                   |
| ------------------- | -------------------------------------- | ------------------------------------------------------ |
| **方法名相同**      | 必须完全一致                           | `void show()` ↔ `void show()`                          |
| **参数列表相同**    | 类型、个数、顺序一致                   | `(int a, String b)` 必须相同                           |
| **返回类型**        | 相同或为子类型（协变返回）             | 父类返回 `Animal`，子类可返回 `Dog`                    |
| **访问权限**        | 不能比父类更严格                       | 父类 `protected` → 子类可以是 `public`                 |
| **异常声明**        | 不能比父类声明更多更广的受检异常       | 父类抛 `IOException`，子类可抛 `FileNotFoundException` |
| **非静态方法**      | 静态方法不算重写，是隐藏               | `@Override` 会报错                                     |
| **非final/private** | final 方法不可重写，private 方法不可见 | 编译错误                                               |

**必须遵守的「禁止性规则」（容易忽略的点）**

- 父类方法是`final`：子类**不能重写**（final 修饰的方法是「最终方法」，禁止修改）
- 父类方法是`static`：子类不能重写（静态方法属于类，不属于实例，子类定义同名静态方法只是「隐藏」而非「重写」）
- 异常声明：子类重写方法抛出的异常不能比父类方法更宽泛（比如父类抛`IOException`，子类不能抛`Exception`，但可以抛`FileNotFoundException`）

---

##### 📝 代码示例

```java
class Animal {
    protected String speak() throws Exception {
        return "Some sound";
    }
}

class Dog extends Animal {
    @Override
    public String speak() {  // 权限扩大：protected → public
        return "Woof!";      // 返回类型协变：String 是 Object 的子类
    }                        // 异常：可以不抛或抛子异常
}
```

---

##### ⚡ 关键特性

1. **`@Override` 注解**：强制编译器检查是否为有效重写，**强烈推荐使用**
2. **动态绑定**：运行时根据实际对象类型调用方法，而非引用类型
3. **`super` 关键字**：子类可通过 `super.method()` 调用父类版本

```java
class Cat extends Animal {
    @Override
    public String speak() {
        // 先执行父类逻辑，再扩展
        return super.speak() + " → Meow!";
    }
}
```

---

##### ❌ 常见误区

| 情况               | 结果         | 说明                              |
| ------------------ | ------------ | --------------------------------- |
| 静态方法同名同参数 | **方法隐藏** | 属于类，不发生多态                |
| 子类方法更严格权限 | **编译错误** | 如父类 `public`，子类 `protected` |
| 返回类型不兼容     | **编译错误** | 必须是相同类型或子类型            |

#### **协变（Covariance）** 

是面向对象编程中的重要概念，指**子类型关系在特定方向上保持一致**。让我用最直观的方式解释：

---

##### 核心定义

> 如果 `Dog` 是 `Animal` 的子类，那么 `Dog[]` 也是 `Animal[]` 的子类型，这就是**协变**。

简单说：**"子类型的东西" 可以替代 "父类型的东西" 的位置**。

##### 1. 协变返回类型（方法重写）✅ 重点

```java
class Animal {
    public Animal getOffspring() {  // 父类返回 Animal
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    public Dog getOffspring() {     // 子类返回更具体的 Dog（协变）
        return new Dog();
    }
}
```

**逻辑**：既然 `Dog` **是** `Animal`，那返回 `Dog` 的地方当然可以用 `Animal` 接收。

```java
Animal animal = new Dog();
Animal baby = animal.getOffspring();  // 实际得到 Dog 对象，安全！
```

---

##### 2. 数组协变（有陷阱 ⚠️）

```java
Dog[] dogs = new Dog[3];
Animal[] animals = dogs;  // 编译通过！数组是协变的

animals[0] = new Cat();   // 💥 运行时 ArrayStoreException！
```

**问题**：数组记住实际类型，编译时检查不足，运行时才报错。

---

##### 3. 泛型协变（用 `? extends`）

```java
List<Dog> dogs = new ArrayList<>();
// List<Animal> animals = dogs;  // ❌ 编译错误！泛型不支持协变

List<? extends Animal> animals = dogs;  // ✅ 协变通配符
```

| 特性                     | 能否添加元素               | 能否读取元素          |
| ------------------------ | -------------------------- | --------------------- |
| `List<? extends Animal>` | ❌ 不能（不知道具体子类型） | ✅ 可以（当作 Animal） |

---

###### 对比：协变 vs 逆变 vs 不变

| 类型     | 符号          | 含义                | 应用场景           |
| -------- | ------------- | ------------------- | ------------------ |
| **协变** | `? extends T` | 子类型 → 父类型方向 | 生产者（读取数据） |
| **逆变** | `? super T`   | 父类型 → 子类型方向 | 消费者（写入数据） |
| **不变** | `T`           | 必须完全一致        | 默认情况           |

###### PECS 原则（Producer-Extends, Consumer-Super）

```java
// 协变：只读（生产者）
void feedAnimals(List<? extends Animal> animals) {
    for (Animal a : animals) a.eat();  // 安全读取
    // animals.add(new Dog());  // ❌ 编译错误！
}

// 逆变：只写（消费者）
void addDogs(List<? super Dog> list) {
    list.add(new Dog());       // ✅ 安全写入
    // Dog d = list.get(0);    // ❌ 只能拿到 Object
}
```

---

##### 一句话总结

> **协变 = 用"更具体的子类"替代"较抽象的父类"，保证"是一个"（is-a）关系的传递性。**

在方法重写中，协变返回类型让你能返回更精确的类型，既保证多态，又获得更好的类型安全。

#### **10.简述 Java 的内存模型，包括程序计数器、虚拟机栈、本地方法栈、堆和方法区的作用。**

- **程序计数器**：存储当前线程所执行的字节码指令地址。
- **虚拟机栈**：每个线程都有一个虚拟机栈，用于存储方法的局部变量、操作数栈、动态链接和方法返回地址等信息。
- **本地方法栈**：为 JNI（Java Native Interface）调用提供支持，存储本地方法调用的栈帧。
- **堆**：用于存放对象实例和数组，所有的对象都在堆中分配内存。
- **方法区**：存储类信息、常量、静态变量、即时编译器编译后的代码等。

#### **11.什么是垃圾回收机制？Java 的垃圾回收有什么优势？**

  **垃圾回收机制**：JVM 会自动回收不再使用的对象，释放其占用的内存空间。
  Java 的优势：自动内存管理，开发者无需手动管理内存，减少内存泄漏和指针错误。

#### **12.常见的垃圾回收算法有哪些？简要说明其工作原理。**

  常见的垃圾回收算法有：

 - **标记清除算法**：标记所有可达对象，然后清除不可达对象。
 - **复制算法**：将内存划分为两个区域，使用一个区域时，回收时复制存活对象到另一区域。
 - **标记整理算法**：类似标记清除，但清除时将存活对象移动到内存的一端，避免产生碎片。
 - **分代收集算法**：将堆分为年轻代、老年代，分别使用不同的回收算法。

#### **13.如何判断一个对象是否为垃圾对象？有哪些判断方法？**

  **垃圾对象**是指没有任何活跃引用指向它的对象。可以通过：

 - **引用计数法**：对象的引用计数为 0，表示没有任何引用指向该对象。
 - **可达性分析法**：通过 GC 根（如栈中的局部变量、静态变量等）开始，遍历对象图，判断哪些对象不可达。

#### **14.Java 中的异常体系结构是怎样的？Checked 异常和 Unchecked 异常有什么区别？**

 - **异常体系结构**：所有异常类都继承自 `Throwable`，其中包括 `Error`（错误）和 `Exception`（异常）。
 - **Checked 异常**：编译时异常，必须显式捕获或声明抛出（如 `IOException`）。
 - **Unchecked 异常**：运行时异常，不强制要求捕获或声明（如 `NullPointerException`）。

#### **15.什么是类加载器？Java 中的类加载器有哪些种类？它们的职责是什么？**

  **类加载器**：用于将类的字节码加载到内存中。
  常见的类加载器：

 - **Bootstrap ClassLoader**：加载 JDK 自带的核心类库。
 - **Extension ClassLoader**：加载 `ext` 目录下的类库。
 - **System ClassLoader**：加载 `classpath` 路径下的类。

#### **16.类的加载过程包括哪几个阶段？每个阶段主要做什么？**

 - **加载**：查找并加载类的字节码文件。
 - **链接**：包括验证、准备（为类变量分配内存并赋予默认值）、解析（将符号引用转为直接引用）。
 - **初始化**：执行类的静态代码块和静态变量初始化。

#### **17.什么是双亲委派模型？它有什么作用？**

  **双亲委派模型**：类加载器先委派父类加载器进行类的加载，只有父类加载器无法加载时，子类加载器才会尝试加载。
  作用：避免类的重复加载，确保类的唯一性和安全性。

#### **18.Java 中的接口和抽象类有什么区别？在什么场景下使用接口，什么场景下使用抽象类？**

 - **接口**：只能包含常量、抽象方法、默认方法等，不能包含实现。
 - **抽象类**：可以包含实现和抽象方法，可以有构造方法、成员变量等。
    使用场景：
 - 接口：适用于多种不同类型的类可以实现同一个行为时。
 - 抽象类：适用于有共享行为和状态的类，但又需要在子类中提供不同实现的场景。

  您的回答基本正确，但需要补充一些 Java 8+ 的重要变化和使用策略。让我完善这个答案：

---

##### 接口 vs 抽象类 对比表

| 特性           | 接口（Interface）                                            | 抽象类（Abstract Class）             |
| -------------- | ------------------------------------------------------------ | ------------------------------------ |
| **方法实现**   | Java 8+ 支持 `default`/`static` 方法，Java 9+ 支持 `private` 方法 | 可以包含完整实现                     |
| **抽象方法**   | ✅ 隐式 `public abstract`                                     | ✅ 必须显式声明 `abstract`            |
| **变量**       | 只能是 `public static final` 常量                            | 任意类型：实例变量、常量             |
| **构造方法**   | ❌ 不能有                                                     | ✅ 可以有（用于子类调用）             |
| **多继承**     | ✅ 一个类可实现多个接口                                       | ❌ 只能单继承                         |
| **访问修饰符** | 方法默认 `public`，Java 9+ 可有 `private` 方法               | 任意：`public`/`protected`/`private` |
| **状态存储**   | ❌ 不能保存状态（无实例变量）                                 | ✅ 可以保存状态                       |
| **演化性**     | 添加默认方法不破坏已有实现类                                 | 添加方法必须修改所有子类             |

---

##### 代码示例对比

###### 接口（Java 8+ 新特性）

```java
public interface Flyable {
    // 1. 常量（隐式 public static final）
    int MAX_HEIGHT = 10000;
    
    // 2. 抽象方法（隐式 public abstract）
    void fly();
    
    // 3. 默认方法（有实现，不强制重写）⭐Java 8
    default void land() {
        System.out.println("安全着陆");
        brake(); // 调用私有方法
    }
    
    // 4. 静态方法（工具方法）⭐Java 8
    static void checkWeather() {
        System.out.println("检查天气...");
    }
    
    // 5. 私有方法（内部复用）⭐Java 9
    private void brake() {
        System.out.println("刹车减速");
    }
}
```

###### 抽象类

```java
public abstract class Animal {
    // 1. 实例变量（保存状态）
    protected String name;
    protected int age;
    
    // 2. 构造方法（子类必须调用）
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // 3. 具体方法（共享实现）
    public void breathe() {
        System.out.println(name + "在呼吸");
    }
    
    // 4. 抽象方法（子类必须实现）
    public abstract void makeSound();
    
    // 5. 模板方法模式
    public final void dailyRoutine() {
        eat();      // 具体
        makeSound(); // 抽象（钩子）
        sleep();    // 具体
    }
    
    private void eat() { System.out.println("吃饭"); }
    private void sleep() { System.out.println("睡觉"); }
}
```

---

##### 使用场景决策树

```
需要多继承能力？ ──→ 接口（Java类只能单继承）
    │
    └── 需要保存状态/有实例变量？ ──→ 抽象类
            │
            └── 需要为不相关类提供统一行为？ ──→ 接口
                    │
                    └── 需要定义契约+提供默认实现？ ──→ 接口（default方法）
                            │
                            └── 需要代码复用+控制继承层次？ ──→ 抽象类
```

##### 具体场景

| 场景                                             | 选择               | 原因                         |
| ------------------------------------------------ | ------------------ | ---------------------------- |
| **定义能力契约**（`Comparable`、`Serializable`） | 接口               | 表示"能做什么"，与类层次无关 |
| **跨类共享代码**（工具方法、通用逻辑）           | 抽象类             | 避免重复代码，控制实现细节   |
| **需要演化API**（向后兼容）                      | 接口 + default方法 | 新增方法不破坏现有实现       |
| **is-a关系 + 代码复用**（`AbstractList`）        | 抽象类             | 模板方法模式，定义算法骨架   |
| **回调/策略模式**（`Runnable`、`Comparator`）    | 接口               | 轻量级，Lambda表达式友好     |
| **菱形继承问题**                                 | 抽象类优先         | 避免多接口默认方法冲突       |

---

##### 现代 Java 最佳实践（Java 8+）

###### 优先使用接口 + 默认方法

```java
// 接口提供默认实现，兼顾灵活性和演化性
public interface Logger {
    void log(String msg);  // 抽象
    
    default void logError(String msg) {  // 默认实现
        log("[ERROR] " + msg);
    }
}
```

###### 接口 + 抽象类组合（骨架类模式）

```java
// 接口定义契约
public interface List<E> {
    int size();
    E get(int index);
    default void sort() { /* 默认实现 */ }
}

// 抽象类提供骨架实现
public abstract class AbstractList<E> implements List<E> {
    protected int modCount = 0;  // 共享状态
    
    public boolean isEmpty() {
        return size() == 0;  // 基于抽象方法的具体实现
    }
    
    // 子类只需实现核心方法
}
```

---

##### 一句话总结

> **"接口是契约，抽象类是骨架；接口定义能做什么，抽象类定义是什么；优先用接口，必要时用抽象类提供代码复用。"**

您的原始答案在 Java 7 时代完全正确，但 Java 8+ 中接口已支持默认方法，两者的界限变得模糊，**设计意图**比语法差异更重要。

#### **19.简述 Java 中的权限修饰符，包括 public、protected、default 和 private 的作用范围。**

- **public**：表示该成员可以被任何类访问，没有访问限制。
- **protected**：表示该成员只能在同一包中或子类中访问（包括不同包中的子类）。
- **default**（包私有）：没有修饰符时，默认只能在同一包内访问。
- **private**：表示该成员只能在当前类内部访问，外部无法访问。

#### **20.什么是内部类？Java 中有哪些类型的内部类？它们的特点是什么？**

 **内部类**：是定义在另一个类内部的类，通常用于封装类的一部分功能或简化代码。
 Java 中的内部类有几种类型：

- **成员内部类**：定义在外部类的成员位置，可以访问外部类的所有成员（包括私有成员）。
- **静态内部类**：用 `static` 修饰，属于外部类的一部分，可以访问外部类的静态成员。
- **局部内部类**：定义在方法内，只能在该方法内使用。
- **匿名内部类**：没有类名的类，用于简化代码，通常用于实例化接口或抽象类的对象。



#### 21.classpath

在 Java 中，**classpath（类路径）** 是 JVM 用来查找 **类文件（`.class`）** 和 **资源文件（如 `.properties`、图片、XML 等）** 的一组目录、JAR 文件或 ZIP 文件的路径集合。

---

##### 🧠 一、什么是 classpath？

###### 简单理解：

> **classpath 告诉 JVM：“去哪里找你运行程序需要的 .class 文件和资源文件。”**

- 类文件：由 Java 源码编译生成（如 `MyClass.class`）
- 资源文件：非代码文件，但程序运行时需要（如 `config/app.properties`、`logo.png`）

---

##### 📁 二、classpath 包含哪些内容？

###### 1. **源码编译后的 `.class` 文件目录**

- 例如：`target/classes`（Maven）或 `build/classes/java/main`（Gradle）
- 开发时，IDE（如 IntelliJ、Eclipse）会自动将 `src/main/java` 编译到这些目录，并加入 classpath

###### 2. **资源文件目录**

- Maven/Gradle 默认将 `src/main/resources` **复制到 `target/classes`**（或等效目录）
- 所以 `src/main/resources/config/app.properties` 在运行时可通过 classpath 路径 `config/app.properties` 访问

###### 3. **依赖的 JAR 文件**

- 所有通过 Maven/Gradle 引入的第三方库（如 `spring-core-5.x.jar`）都会被加入 classpath

###### 4. **手动指定的目录或 JAR**

- 通过 `-cp` 或 `-classpath` 参数指定：
  ```bash
  java -cp ".;lib/*" com.example.Main
  ```

---

##### 🔍 三、如何从 classpath 加载资源？

Java 提供两种主要方式（如前所述）：

###### ✅ 方式 1：通过 `ClassLoader`

```java
// 路径相对于 classpath 根，不能以 / 开头
InputStream is = MyClass.class.getClassLoader()
                              .getResourceAsStream("config/app.properties");
```

###### ✅ 方式 2：通过 `Class`

```java
// 以 / 开头：从 classpath 根开始
InputStream is1 = MyClass.class.getResourceAsStream("/config/app.properties");

// 不以 / 开头：从 MyClass 所在包开始
InputStream is2 = MyClass.class.getResourceAsStream("helper.txt");
// 假设 MyClass 在 com.example 包，则实际路径是：com/example/helper.txt
```

> 💡 资源文件最终都位于 classpath 的某个位置（如 `target/classes/config/app.properties`），JVM 通过上述 API 查找。

---

##### 🏗️ 四、典型项目结构（Maven）

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/MyApp.java
│   │   └── resources/
│   │       ├── config/
│   │       │   └── app.properties
│   │       └── assets/
│   │           └── logo.png
│   └── test/
│       └── ...
├── target/
│   └── classes/                 ← 这就是运行时的 classpath 根！
│       ├── com/
│       │   └── example/
│       │       └── MyApp.class
│       ├── config/
│       │   └── app.properties
│       └── assets/
│           └── logo.png
└── pom.xml
```

✅ 所以：
- `MyApp.class` 的 classpath 路径是：`com/example/MyApp.class`
- `app.properties` 的 classpath 路径是：`config/app.properties`

---

##### ⚙️ 五、如何查看当前 classpath？

###### 方法 1：代码中打印

```java
String classpath = System.getProperty("java.class.path");
System.out.println("Classpath: " + classpath);
```

###### 方法 2：命令行运行时指定

```bash
# Linux/macOS
java -cp "lib/*:target/classes" com.example.Main

# Windows
java -cp "lib/*;target/classes" com.example.Main
```

---

##### ❗ 六、常见误区

| 误区                                                         | 正确理解                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| “resources 目录就是 classpath”                               | ✅ 编译后 `resources` 内容被复制到 `classes` 目录，**`classes` 才是 classpath 根** |
| 路径写成 `src/main/resources/xxx`                            | ❌ 运行时没有 `src` 目录！应使用 `xxx`（相对于 classpath 根） |
| `getResourceAsStream("/xxx")` 和 `getClassLoader().getResourceAsStream("xxx")` 总是等价 | ✅ 大多数情况下等价，但 `Class.getResourceAsStream` 支持相对路径（不带 `/`） |

---

##### ✅ 总结

- **classpath = 运行时查找类和资源的根目录集合**
- 开发时放在 `src/main/resources` 的文件，**运行时位于 classpath 根下**
- 使用 `ClassLoader.getResourceAsStream("path")` 是最清晰、推荐的方式
- 路径**不要包含 `src`、`resources` 等开发目录名**，只写逻辑路径（如 `config/app.properties`）

---

如果你正在处理资源加载问题（如找不到文件、`null` 输入流等），可以贴出你的项目结构和代码，我可以帮你精准定位路径写法！

#### 22.jar和war

`.jar` 和 `.war` 是 Java 生态中两种常见的**打包格式**，它们都基于 ZIP 格式，但用途和结构不同。下面从多个维度详细对比和介绍：

---

##### 📦 一、JAR（Java Archive）

###### ✅ 用途

- **通用 Java 库或可执行应用程序**的打包格式。
- 用于：
  - 第三方依赖库（如 `guava-32.0.jar`）
  - 独立运行的 Java 应用（带 `Main-Class`）

###### 🗂️ 结构示例

```
my-app.jar
├── META-INF/
│   └── MANIFEST.MF        ← 清单文件（可指定 Main-Class）
├── com/
│   └── example/
│       └── MyApp.class
└── config/
    └── app.properties
```

###### ▶️ 如何运行？

```bash
# 如果 MANIFEST.MF 中定义了 Main-Class
java -jar my-app.jar

# 或指定主类
java -cp my-app.jar com.example.MyApp
```

###### 🔧 构建工具支持

- Maven: `<packaging>jar</packaging>`（默认）
- Gradle: `jar` 任务

---

##### 🌐 二、WAR（Web Application Archive）

###### ✅ 用途

- **Java Web 应用**的标准打包格式，部署到 **Servlet 容器**（如 Tomcat、Jetty、WebLogic）。
- 遵循 Java EE / Jakarta EE 的 Web 应用规范。

###### 🗂️ 标准结构

```
my-webapp.war
├── WEB-INF/
│   ├── web.xml             ← Web 配置文件（可选，Servlet 3.0+ 可省略）
│   ├── classes/            ← 编译后的 .class 文件和资源
│   │   ├── com/example/MyServlet.class
│   │   └── config/app.properties
│   └── lib/                ← Web 应用私有依赖 JAR
│       ├── spring-web-5.x.jar
│       └── ...
├── index.html
├── css/
│   └── style.css
└── WEB-INF/lib/...         ← 注意：lib 在 WEB-INF 下！
```

###### ▶️ 如何部署？

- 将 `.war` 文件复制到 Tomcat 的 `webapps/` 目录
- Tomcat 会自动解压并部署为 Web 应用（如 `http://localhost:8080/my-webapp`）

###### 🔧 构建工具支持

- Maven: `<packaging>war</packaging>`
- Gradle: `war` 插件

---

##### 🔁 三、核心区别对比

| 特性         | JAR                         | WAR                                                          |
| ------------ | --------------------------- | ------------------------------------------------------------ |
| **全称**     | Java Archive                | Web Application Archive                                      |
| **用途**     | 普通 Java 程序 / 库         | Web 应用（Servlet/JSP）                                      |
| **部署目标** | JVM（直接运行）             | Web 容器（如 Tomcat）                                        |
| **入口点**   | `Main-Class`（MANIFEST.MF） | `web.xml` 或注解（如 `@WebServlet`）                         |
| **标准目录** | 无强制结构                  | 必须有 `WEB-INF/`                                            |
| **依赖位置** | 通常由外部 classpath 提供   | 私有依赖放在 `WEB-INF/lib/`                                  |
| **资源访问** | 通过 classpath              | `WEB-INF/classes` 属于 classpath；其他资源（如 HTML）可通过 HTTP 访问 |

---

##### 💡 四、常见问题

###### Q1：Spring Boot 用的是 JAR 还是 WAR？

- **默认是可执行 JAR**（内嵌 Tomcat/Jetty），直接 `java -jar app.jar` 运行。
- 也可打包成 WAR，用于部署到外部 Tomcat（需继承 `SpringBootServletInitializer`）。

###### Q2：WAR 能当 JAR 用吗？

- **不能直接运行**，因为没有 `Main-Class`，且结构不符合。
- 但 WAR 中的 `WEB-INF/classes` 和 `WEB-INF/lib` 会被 Web 容器加入 classpath。

###### Q3：JAR 能包含 Web 内容吗？

- 技术上可以，但**不符合规范**。Web 容器不会识别普通 JAR 为 Web 应用。

---

##### 🛠️ 五、如何选择？

| 场景                                  | 推荐格式      |
| ------------------------------------- | ------------- |
| 开发微服务、命令行工具、工具库        | ✅ JAR         |
| 传统 Java Web 项目（需部署到 Tomcat） | ✅ WAR         |
| Spring Boot 应用（内嵌容器）          | ✅ JAR（默认） |
| 需要与旧版 Web 容器集成               | ✅ WAR         |

---

##### 🔚 总结

- **JAR**：通用、轻量，适合独立应用或库。
- **WAR**：专为 Web 应用设计，必须部署到 Servlet 容器。
- 两者都是 ZIP，但**结构和用途完全不同**，不可混用。

> 💡 现代开发趋势：**Spring Boot 推动了“可执行 JAR”成为主流**，减少对 WAR 和外部容器的依赖。

如果你有具体的项目类型（如 Spring Boot、传统 Servlet），我可以给出更针对性的打包建议！



#### 23.jar的使用流程

下面详细介绍 **JAR（Java Archive）文件的完整使用流程**，涵盖从 **编写代码 → 编译 → 打包 → 运行/分发** 的全过程，并包含 **可执行 JAR** 与 **普通库 JAR** 两种常见场景。

---

##### 🧩 一、JAR 是什么？

- JAR 是 Java 的标准打包格式，基于 ZIP。
- 用途：
  - **可执行程序**（带主类）
  - **类库**（供其他项目依赖）

---

##### 📁 二、项目结构示例（Maven 风格）

```bash
my-java-app/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/
│       │       ├── Main.java        # 主类
│       │       └── Utils.java
│       └── resources/
│           └── config.properties    # 资源文件
└── manifest.txt (可选，用于自定义 MANIFEST.MF)
```

###### `Main.java` 示例：

```java
package com.example;

import java.io.InputStream;
import java.util.Properties;

public class Main {
    public static void main(String[] args) throws Exception {
        // 从 classpath 加载资源
        try (InputStream is = Main.class.getClassLoader()
                .getResourceAsStream("config.properties")) {
            Properties props = new Properties();
            props.load(is);
            System.out.println("App Name: " + props.getProperty("app.name"));
        }
        System.out.println("Hello from JAR!");
    }
}
```

###### `config.properties`：

```properties
app.name=MyJavaApp
```

---

##### 🔧 三、完整使用流程

###### 步骤 1：编译 Java 源码

```bash
# 创建输出目录
mkdir -p target/classes

# 编译（包含资源复制）
javac -d target/classes src/main/java/com/example/*.java

# 手动复制资源（或用构建工具自动处理）
cp src/main/resources/* target/classes/
```

> ✅ 编译后结构：
```
target/classes/
├── com/example/Main.class
├── com/example/Utils.class
└── config.properties
```

---

###### 步骤 2：创建 MANIFEST.MF（仅可执行 JAR 需要）

创建 `manifest.txt`（注意：最后一行必须空行！）：
```txt
Main-Class: com.example.Main

```

> ⚠️ 注意：
> - `Main-Class` 后是**全限定类名**（无 `.class`）
> - 文件末尾必须有一个**空行**，否则 JVM 无法识别

---

###### 步骤 3：打包成 JAR

**场景 A：可执行 JAR（带主类）**

```bash
jar cfm my-app.jar manifest.txt -C target/classes .
```

参数说明：
- `c`：create（创建）
- `f`：指定文件名
- `m`：包含 manifest 文件
- `-C target/classes .`：切换到 `target/classes` 目录，打包当前目录所有内容

**场景 B：普通库 JAR（无主类）**

```bash
jar cf my-lib.jar -C target/classes .
```

---

###### 步骤 4：验证 JAR 内容

```bash
# 查看内容
jar tf my-app.jar

# 查看 MANIFEST.MF
jar xf my-app.jar META-INF/MANIFEST.MF
cat META-INF/MANIFEST.MF
```

输出应包含：
```
Main-Class: com.example.Main
```

---

###### 步骤 5：运行 JAR

**可执行 JAR：**

```bash
java -jar my-app.jar
```

输出：
```
App Name: MyJavaApp
Hello from JAR!
```

**库 JAR（作为依赖）：**

```bash
# 假设另一个程序依赖 my-lib.jar
java -cp "my-lib.jar:." AnotherMainClass
```

---

##### 🛠️ 四、使用构建工具自动化（推荐）

###### Maven 示例（`pom.xml`）

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>

  <!-- 打包可执行 JAR -->
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.3.0</version>
        <configuration>
          <archive>
            <manifest>
              <mainClass>com.example.Main</mainClass>
            </manifest>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

构建命令：
```bash
mvn clean package
# 输出：target/my-app-1.0.jar
```

###### Gradle 示例（`build.gradle`）

```groovy
plugins {
    id 'java'
}

version = '1.0'

jar {
    manifest {
        attributes 'Main-Class': 'com.example.Main'
    }
}
```

构建命令：
```bash
gradle build
# 输出：build/libs/my-app-1.0.jar
```

---

##### 📦 五、JAR 的典型用途总结

| 类型           | 是否可执行 | 用途                                 | 如何使用                            |
| -------------- | ---------- | ------------------------------------ | ----------------------------------- |
| **可执行 JAR** | ✅ 是       | 独立应用（如 CLI 工具、Spring Boot） | `java -jar app.jar`                 |
| **库 JAR**     | ❌ 否       | 提供 API/工具类                      | 被其他项目通过 `-cp` 或构建工具依赖 |

---

##### ⚠️ 六、常见问题排查

###### ❌ 问题1：`no main manifest attribute`

- 原因：MANIFEST.MF 中缺少 `Main-Class`
- 解决：检查 manifest 文件格式，确保有空行，类名正确

###### ❌ 问题2：`ClassNotFoundException`

- 原因：依赖的第三方库未包含
- 解决：
  - **方案1**：使用 **fat JAR / uber JAR**（把依赖打包进去）
    - Maven: `maven-shade-plugin`
    - Gradle: `shadow` 插件
  - **方案2**：运行时通过 `-cp` 手动指定依赖

###### ❌ 问题3：资源文件找不到

- 原因：资源未正确打包进 JAR
- 解决：确保资源在 `src/main/resources`（Maven）或打包时复制到 `classes`

---

##### ✅ 七、最佳实践

1. **优先使用 Maven/Gradle** 自动处理编译、资源、打包。
2. **可执行 JAR 推荐使用 fat JAR**（尤其 Spring Boot）避免依赖缺失。
3. **库 JAR 不要包含依赖**（避免冲突），由使用者管理依赖。
4. **始终测试 JAR 是否能独立运行**。

---

如果你有具体需求（如 Spring Boot 打包、包含依赖、多模块项目等），可以告诉我，我会给出针对性方案！

#### 24.jar的使用场景

JAR（Java Archive）包是 Java 生态中最基础、最核心的打包格式，其使用场景非常广泛。以下是 **JAR 包的主要使用场景分类与详解**，帮助你理解在什么情况下该用 JAR、怎么用、以及为什么用。

---

###### ✅ 一、作为**可执行应用程序**（Executable Application）

##### 🎯 场景描述：

将整个 Java 应用打包成一个 JAR 文件，用户只需运行 `java -jar app.jar` 即可启动程序。

##### 🔧 典型应用：

- 命令行工具（CLI）：如数据库迁移工具、日志分析器
- 桌面应用（配合 JavaFX/Swing）
- **Spring Boot 微服务**（内嵌 Tomcat/Jetty，无需外部容器）

##### 📦 要求：

- `MANIFEST.MF` 中必须包含：
  ```manifest
  Main-Class: com.example.Main
  ```
- 推荐使用 **Fat JAR / Uber JAR**（包含所有依赖）

##### 💡 示例：

```bash
java -jar my-spring-boot-app.jar
# 启动一个 RESTful 服务，监听 8080 端口
```

> ✅ **优势**：部署简单，一个文件搞定，适合容器化（Docker）。

---

###### ✅ 二、作为**类库（Library）** 供其他项目依赖

##### 🎯 场景描述：

将通用功能（如工具类、SDK、算法模块）打包成 JAR，供其他 Java 项目引用。

##### 🔧 典型应用：

- 工具库：Apache Commons、Guava、Lombok
- 企业内部 SDK：支付 SDK、日志封装、加密组件
- 第三方 API 客户端：如阿里云 SDK、微信 SDK

##### 📦 要求：

- 不需要 `Main-Class`
- 通常**不包含依赖**（避免版本冲突）
- 通过 Maven/Gradle 发布到仓库（如 Maven Central、私有 Nexus）

##### 💡 示例（Maven 依赖）：

```xml
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>32.1.3-jre</version>
</dependency>
```

> ✅ **优势**：模块化开发、代码复用、版本管理清晰。

---

##### ✅ 三、作为**插件或扩展模块**

###### 🎯 场景描述：

主程序在运行时动态加载 JAR 文件，实现插件化架构。

###### 🔧 典型应用：

- IDE 插件（如 IntelliJ 插件）
- 游戏模组（Mod）
- 规则引擎的动态规则包
- 自定义数据处理器（如 Flink/Spark 的 UDF）

###### 📦 实现方式：

- 使用 `URLClassLoader` 动态加载 JAR：
  ```java
  URL jarUrl = new File("plugins/my-plugin.jar").toURI().toURL();
  URLClassLoader loader = new URLClassLoader(new URL[]{jarUrl});
  Class<?> pluginClass = loader.loadClass("com.example.PluginImpl");
  ```

> ✅ **优势**：无需重启主程序即可扩展功能，支持热插拔。

---

##### ✅ 四、用于**测试或调试**

###### 🎯 场景描述：

将测试代码或调试工具打包成 JAR，方便在不同环境运行。

###### 🔧 典型应用：

- 性能压测工具
- 数据库连接测试器
- 环境诊断脚本（检查 JDK 版本、网络、证书等）

###### 💡 示例：

```bash
java -jar env-checker.jar --host=mydb.example.com --port=3306
```

> ✅ **优势**：轻量、便携、无需安装完整项目。

---

##### ✅ 五、作为**构建中间产物**

###### 🎯 场景描述：

在 CI/CD 流程中，JAR 是编译后的标准产物，用于后续测试、部署或发布。

###### 🔧 典型流程：

```
源码 → 编译 → classes → 打包为 JAR → 单元测试 → 发布到仓库 / 构建 Docker 镜像
```

###### 💡 示例（Dockerfile）：

```dockerfile
FROM openjdk:17
COPY target/my-app.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

> ✅ **优势**：标准化交付物，便于自动化。

---

##### ✅ 六、特殊场景：**多模块项目中的模块打包**

###### 🎯 场景描述：

大型项目拆分为多个子模块，每个子模块打包为独立 JAR。

###### 🔧 示例（Maven 多模块）：

```
parent/
├── core/          → core-1.0.jar（公共逻辑）
├── web/           → web-1.0.jar（依赖 core）
└── batch/         → batch-1.0.jar（依赖 core）
```

> ✅ **优势**：解耦、独立编译、按需部署。

---

###### 🚫 不适合用 JAR 的场景

| 场景                                  | 建议替代方案            |
| ------------------------------------- | ----------------------- |
| 传统 Java Web 应用（需部署到 Tomcat） | ✅ 使用 **WAR**          |
| 需要 Web 容器管理会话、安全等         | ✅ WAR 或 Jakarta EE EAR |
| 非 Java 项目（如 Python、Node.js）    | ❌ JAR 不适用            |

> 💡 但注意：**Spring Boot 应用虽然是 Web 服务，但通常仍用 JAR（内嵌容器）**，这是现代主流做法。

---

##### 📊 总结：JAR 包使用场景速查表

| 使用场景                    | 是否可执行 | 是否含依赖          | 典型用户       |
| --------------------------- | ---------- | ------------------- | -------------- |
| 独立应用（Spring Boot/CLI） | ✅ 是       | ✅ 通常含（Fat JAR） | 运维、终端用户 |
| 工具类库                    | ❌ 否       | ❌ 通常不含          | 开发者         |
| 插件/扩展                   | ⚠️ 可能     | 视情况              | 主程序动态加载 |
| 测试工具                    | ✅ 是       | ✅ 可含              | QA、DevOps     |
| 构建产物                    | -          | -                   | CI/CD 系统     |
| 多模块子项目                | ❌ 否       | ❌ 不含              | 其他模块依赖   |

---

##### 🔚 结语

JAR 是 Java 的“瑞士军刀”——  
- 小到一个工具函数，  
- 大到一个完整微服务，  
都可以用 JAR 高效打包和分发。

**现代 Java 开发中，JAR 已成为事实上的标准交付格式**，尤其在云原生和微服务架构下，其地位愈发重要。

#### 25.java包结构

Java 包结构是组织代码的核心方式，其设计遵循**命名规范、功能模块化、可维护性**三大原则，同时直接影响框架（如 Spring Boot、MyBatis）的组件扫描、类加载等核心功能。以下是 Java 包结构的详细解析：

##### 一、包的核心作用

1. **命名空间隔离**：避免类重名（如 `com.example.User` 和 `com.test.User` 是两个不同的类）。
2. **代码模块化**：按功能或业务拆分代码，便于团队协作和维护。
3. **权限控制**：`default` 访问权限的类/成员仅能在同一包内访问。
4. **框架适配**：Spring、MyBatis 等框架依赖包路径扫描组件（如 `@ComponentScan`、Mapper 接口扫描）。

##### 二、包的命名规范

###### 1. 基础命名规则

- **格式**：采用**反向域名+项目名+模块名**的层级结构，全小写字母，用 `.` 分隔。
- **示例**：
  - 个人项目：`com.example.springbootdemo`
  - 企业项目：`com.alibaba.cloud.nacos.config`
  - 开源项目：`org.springframework.boot.web`

###### 2. 禁忌规则

- 不能使用 Java 关键字（如 `java.lang`、`com.int`）。
- 避免使用特殊字符、数字开头（如 `com.example.123demo` 不合法）。
- 不建议使用默认包（无显式包名），会导致组件扫描异常。

##### 三、典型包结构设计

Java 包结构通常按**功能分层**或**业务模块**设计，以下是两种主流方案：

###### 1. 功能分层结构（适合中小型项目）

按代码的功能角色拆分，体现 MVC 分层思想，结构清晰：
```
com.example.springbootdemo  // 项目根包（启动类所在位置）
├── SpringbootDemoApplication.java  // 启动类（必须在根包下）
├── controller  // 控制器层：处理 HTTP 请求，返回响应
│   └── UserController.java
├── service     // 服务层：封装业务逻辑
│   ├── UserService.java       // 服务接口
│   └── impl                   // 服务实现类
│       └── UserServiceImpl.java
├── mapper      // 数据访问层：MyBatis Mapper 接口
│   └── UserMapper.java
├── entity      // 实体类：映射数据库表或业务模型
│   └── User.java
├── dto         // 数据传输对象：用于接口入参/出参（避免直接暴露实体类）
│   ├── UserRequestDTO.java
│   └── UserResponseDTO.java
├── config      // 配置类：框架配置、Bean 定义
│   ├── MyBatisConfig.java
│   └── WebConfig.java
├── exception   // 异常处理：自定义异常、全局异常处理器
│   ├── BusinessException.java
│   └── GlobalExceptionHandler.java
├── util        // 工具类：通用工具方法（如日期、字符串处理）
│   └── DateUtil.java
└── constant    // 常量类：定义静态常量（如状态码、配置键）
    └── ResultCode.java
```

###### 2. 业务模块结构（适合大型项目）

按业务领域拆分，每个模块包含自身的分层代码，降低模块间耦合：
```
com.example.ecommerce  // 电商项目根包
├── EcommerceApplication.java  // 启动类
├── user        // 用户模块
│   ├── controller
│   │   └── UserController.java
│   ├── service
│   ├── mapper
│   └── entity
├── order       // 订单模块
│   ├── controller
│   ├── service
│   ├── mapper
│   └── entity
├── product     // 商品模块
│   ├── controller
│   ├── service
│   ├── mapper
│   └── entity
├── common      // 公共模块：通用工具、常量、异常（供所有业务模块依赖）
│   ├── util
│   ├── constant
│   └── exception
└── config      // 全局配置
    └── WebConfig.java
```

##### 四、框架适配的包结构注意事项

###### 1. Spring Boot 适配

- 启动类必须放在**项目根包下**，确保 `@SpringBootApplication` 的 `@ComponentScan` 能扫描到所有子包的组件。
- 若需自定义扫描范围，可通过 `@ComponentScan(basePackages = "com.example.module")` 指定。

###### 2. MyBatis 适配

- Mapper 接口包需与 `application.yml` 中 `mybatis-plus.mapper-locations`（XML 文件路径）或 `@MapperScan` 注解配置一致。
- 示例：`@MapperScan("com.example.springbootdemo.mapper")` 扫描指定包下的 Mapper 接口。

###### 3. 包权限控制

- 同一包下的类可访问彼此的 `default` 成员（无访问修饰符）。
- 跨包访问需使用 `public` 修饰，或通过 `protected` 实现子类访问。

##### 五、包结构优化建议

1. **控制包的层级深度**：建议层级不超过 4-5 层，过深会导致路径冗长（如 `com.example.a.b.c.d.e` 不推荐）。
2. **模块边界清晰**：业务模块间通过接口通信，避免直接依赖其他模块的内部类。
3. **避免冗余包**：若某包下仅包含 1-2 个类，可考虑合并到其他相关包（如小型项目可省略 `dto` 包，直接使用实体类）。
4. **遵循“单一职责”**：每个包只负责一类功能（如 `util` 包仅放工具类，不混入业务逻辑）。

##### 六、常见错误包结构示例（需规避）

1. **默认包启动类**：启动类无包名，导致 `@ComponentScan` 扫描整个类路径，引发 Bean 冲突。
2. **启动类放在子包下**：如启动类在 `com.example.springbootdemo.controller`，无法扫描到 `service`、`mapper` 等子包的组件。
3. **功能混乱的包**：`util` 包中混入业务逻辑类，或 `entity` 包中包含工具方法。

##### 总结

Java 包结构的核心是**模块化、规范性和框架适配性**。中小型项目推荐使用“功能分层结构”，大型项目推荐“业务模块结构”。合理的包结构能提升代码可维护性，减少框架配置问题，同时降低团队协作成本。

#### 26.ThreadLocal 内存泄漏

我用**最简单的话 + 原理图**，把 **为什么泄漏、怎么泄漏、怎么避免** 一次性讲透，面试必问！

---

##### 一、先看核心结构图（必须看懂）

```
每个线程(Thread)
  ↓
有一个 ThreadLocalMap (线程自带的map)
  ↓
key：WeakReference<ThreadLocal>  【弱引用】
value：你存的数据               【强引用】
```

###### 关键结论：

**ThreadLocalMap 的 key 是弱引用，value 是强引用！**

---

##### 二、内存泄漏到底怎么发生的？（一步一步）

###### 1. 正常情况

```
ThreadLocal tl = new ThreadLocal();
tl.set("我是数据");
```
结构：
```
ThreadLocalMap
  ┌───────────┐    ┌───────────┐
  │  tl(弱)   │ →  │  "我是数据"│
  └───────────┘    └───────────┘
```

###### 2. 外部引用置 null

```
tl = null;  // 外部没有强引用了
```

###### 3. GC 回收 key

因为 **key 是弱引用**，GC 发现没有强引用，**直接回收 key**
变成：

```
ThreadLocalMap
  ┌───────────┐    ┌───────────┐
  │   null    │ →  │  "我是数据"│  ← value 还在！强引用！
  └───────────┘    └───────────┘
        ↑
     key没了
```

###### 4. 最终结果

**key 变成 null，但 value 还在！**
这个 value **再也访问不到，也无法被GC回收** → **内存泄漏！**

---

##### 三、一句话总结泄漏原因

###### **ThreadLocalMap 使用 弱引用key + 强引用value**

当 **ThreadLocal 外部引用消失** 后：
- key 被 GC 回收
- **value 依然强引用，无法回收**
- 变成 **无效的脏数据** → 内存泄漏

---

##### 四、什么时候泄漏最严重？

**线程池！线程池！线程池！**

因为线程池的**线程是复用的，永远不销毁**
所以 **ThreadLocalMap 永远存在**
里面的 **脏value永远不回收**
→ **内存越占越多 → OOM**

---

##### 五、怎么避免内存泄漏？（万能方案）

###### **使用完 ThreadLocal 必须手动 remove()！**

```java
// 1. 使用
tl.set("数据");

// 2. 用完必须调用！！！
tl.remove();
```

###### remove() 做了什么？

```
把当前线程的 ThreadLocalMap 中
这个 key 对应的 entry（key+value）
一起删掉！彻底清理！
```

---

##### 六、面试标准答案（背这个）

ThreadLocal 内存泄漏原因：
1. **ThreadLocalMap 的 key 是弱引用**，ThreadLocal 外部引用置 null 后，key 会被 GC 回收。
2. **value 是强引用**，不会被回收。
3. 若**线程长期存活**（线程池），value 会一直占内存，造成内存泄漏。

解决方案：
**使用完 ThreadLocal 必须调用 remove() 方法手动清理数据。**

---

##### 七、超级记忆口诀

**弱key强value，用完不删必泄漏！**
**线程池里最危险，remove 必须记心间！**

---

##### 总结

1. **泄漏根源**：弱引用key被回收，强引用value残留
2. **最大风险**：线程池（线程不销毁 → map不销毁 → value不回收）
3. **唯一根治**：**手动调用 remove()**

#### 27.什么是 Java 序列化？如何实现？Serializable 作用？

##### 什么是序列化？

把**Java 对象**转为**字节序列**，便于：

- 网络传输
- 写入文件
- 保存到磁盘

反序列化：把字节序列恢复为对象。

##### 如何实现？

让类实现 **Serializable 接口** 即可（空接口，标记作用）。

```java
public class User implements Serializable { }
```

##### Serializable 接口作用

**标记接口**：告诉 JVM 这个类**允许被序列化**。

建议加上：

```java
private static final long serialVersionUID = 1L;
```

作用：**防止反序列化时版本不一致报错**。



#### 28、Object 类中**核心常用方法（必须背）**

一共 **11 个方法**，重点是下面 **9 个高频**：

##### 1. **equals(Object obj)**

- 默认：和 `==` 一样，**比较对象地址**
- 建议重写：比较**对象内容是否相等**
- 常用类：String、Integer 都重写了

##### 2. **hashCode()**

- 返回对象的**哈希码 int 值**
- 用于 **HashMap/HashSet 快速定位**
- 规范：**equals 相同，hashCode 必须相同**

##### 3. **toString()**

- 默认返回：`类名@哈希值`
- 建议重写：输出对象**属性内容**，方便打印查看

##### 4. **getClass()**

- 返回对象的 **Class 对象**（字节码对象）
- **反射的入口方法**

##### 5. **clone()**

- **对象克隆（拷贝）**
- 浅拷贝
- 必须实现 `Cloneable` 接口，否则抛异常

##### 6. **finalize()**

- **垃圾回收前调用**（GC 清理对象时）
- 已废弃（JDK9 标记过时）
- 不推荐使用

##### 7～9. **线程相关方法（3个）**

必须在**同步代码块中**调用：
- **wait()**：让当前线程等待，释放锁
- **notify()**：随机唤醒一个等待线程
- **notifyAll()**：唤醒所有等待线程

---

##### 最精简总结（面试必背版）

**Object 九大方法一句话记忆：**

1. **equals**：比较相等（默认比地址）
2. **hashCode**：获取哈希码
3. **toString**：转字符串
4. **getClass**：获取 Class 对象（反射）
5. **clone**：对象浅拷贝
6. **finalize**：GC 前调用（已废弃）
7. **wait**：线程等待，释放锁
8. **notify**：唤醒单个线程
9. **notifyAll**：唤醒全部线程

---

##### 面试官最爱问的 3 组合（超级重点）

1. **equals + hashCode**：必须同时重写，保证 HashMap 正常
2. **wait/notify/notifyAll**：线程通信，必须配合 synchronized
3. **getClass + toString + clone**：反射、打印、拷贝基础

### 二、基本数据类型与包装类

#### **1.Java 中的基本数据类型有哪些？它们分别占用多少字节？**

Java 中的基本数据类型有：

- `byte`：1 字节
- `short`：2 字节
- `int`：4 字节
- `long`：8 字节
- `float`：4 字节
- `double`：8 字节
- `char`：2 字节
- `boolean`：1 字节（实际占用可能依 JVM 实现而异）

##### Byte表示范围

```java
public class Test2
{
    public void add(Byte b)
    {
        b = b++;
    }
    public void test()
    {
        Byte a = 127;
        Byte b = 127;
        add(++a);
        System.out.print(a + " ");
        add(b);
        System.out.print(b + "");
    }
}
// 输出：-128 127
```

`a=127`，执行 `++a` 时发生拆箱运算并进行前置自增：`127+1` 溢出为 `-128`（byte 溢出取模），再装箱回 `Byte`，因此 `a` 变为 `-128`。`add(++a)` 中的方法体 `b = b++` 对形参 `b` 先自增再把旧值赋回，等价于无效操作，且按值传递不会影响实参，所以 `a` 仍为 `-128`。
`b=127`，执行 `add(b)` 也无效，`b` 仍为 `127`。

调用 `add(b)` 时，把 `b`（值为 127）的**引用值**传递给 `add` 方法的参数 `b`（方法内的局部变量）；

`add` 方法内 `b = b++`：

- 首先，Byte 是**不可变类**（内部 `value` 是 `final` 的），`b++` 本质是：

```
// b++ 的底层逻辑（Byte不可变，自增会创建新对象）
byte temp = b.byteValue();
temp++; // 127+1=-128，但这是临时值
b = Byte.valueOf(temp); // 方法内的局部变量b指向新对象，但外部b不受影响
```

因此 `add` 方法执行完，外部 `b` 仍然是 `127`。

#### **2.基本数据类型和引用数据类型有什么区别？**

- **基本数据类型**：存储实际的值，直接保存在栈中。
- **引用数据类型**：存储的是对象的引用地址，指向堆中的对象。引用数据类型包括类、接口、数组等。

#### **3.什么是包装类？Java 中每种基本数据类型对应的包装类是什么？**

**包装类**：将基本数据类型封装为对象的类。Java 中每种基本数据类型对应的包装类如下：

- `byte` → `Byte`
- `short` → `Short`
- `int` → `Integer`
- `long` → `Long`
- `float` → `Float`
- `double` → `Double`
- `char` → `Character`
- `boolean` → `Boolean`

#### **4.包装类的自动装箱和自动拆箱是指什么？请举例说明。**

- **自动装箱**：将基本数据类型转换为对应的包装类对象，例如：
   `int x = 5;`
   `Integer obj = x;`  // 自动装箱
- **自动拆箱**：将包装类对象转换为对应的基本数据类型，例如：
   `Integer obj = 10;`
   `int x = obj;`  // 自动拆箱

#### **5.int 和 Integer 有什么区别？在使用时需要注意哪些问题？**

- `int` 是基本数据类型，存储的是整数值。
- `Integer` 是 `int` 类型的包装类，存储的是一个 `Integer` 对象的引用。
   使用时需要注意：`int` 的值直接存储，`Integer` 需要进行装箱和拆箱，且在进行比较时，应该使用 `equals()` 方法而不是 `==`（`==` 比较的是对象引用）。

#### **6.为什么 Java 要提供包装类？包装类有哪些常用的方法？**

**Java 是面向对象语言，基本类型不属于对象**

基本数据类型（`int`/`char`/`double` 等）没有对象属性、不能调用方法，包装类把它们变成**对象**，符合 Java 面向对象设计。

**集合（Collection）只能存储对象，不能存基本类型**

`List`、`Set`、`Map` 不支持 `int`、`char` 等，必须用 `Integer`、`Character` 包装类。

**提供丰富的工具方法**

包装类内置字符串互转、最大值、最小值、类型判断等功能，基本类型无法直接实现。

**支持泛型、反射、多态等高级特性**

泛型 `<T>` 只能接收引用类型，不能用基本类型；方法参数需要对象时，必须使用包装类。

##### 包装类常用方法（高频必考）

###### 1. 字符串 → 基本类型

```
parseXxx(String s)
```

例：`Integer.parseInt("123")` → `int 123`

###### 2. 基本类型 / 字符串 → 包装类对象

```
valueOf(参数)
```

例：`Integer.valueOf(10)`、`Integer.valueOf("20")`

###### 3. 包装类对象 → 基本类型

```
xxxValue()
```

例：`Integer.intValue()`、`Double.doubleValue()`

###### 4. 获取类型常量（常用）

- `MAX_VALUE`：最大值

- MIN_VALUE：最小值

  例：Integer.MAX_VALUE

###### 5. 类型判断方法（Character 独有）

- `Character.isDigit(char)`：判断是否数字
- `Character.isLetter(char)`：判断是否字母
- `Character.isUpperCase(char)`：是否大写

###### 6. 比较两个值

`compare(a, b)`：返回 int 结果（-1、0、1）

#### **7.在进行包装类对象比较时，使用 == 和 equals() 方法有什么区别？**

- `==` 比较的是引用地址，即是否是同一个对象。
- `equals()` 比较的是对象的内容，即值是否相等。
   使用包装类比较时，推荐使用 `equals()` 方法。

##### 一、核心区别（结合原理+示例）

你总结的核心结论是正确的，下面结合 Java 包装类（如 `Integer`/`Double`/`Boolean` 等）的特性，补充完整的原理和易错场景：

| 比较方式   | 比较本质                                                     | 适用场景                             | 包装类特殊注意点                               |
| ---------- | ------------------------------------------------------------ | ------------------------------------ | ---------------------------------------------- |
| `==`       | 比较**对象引用地址**（是否为同一个对象）；<br>若为基本类型，则比较**值**。 | 仅判断两个变量是否指向堆中同一个对象 | 包装类存在「常量池缓存」，小范围值会复用对象   |
| `equals()` | 比较**对象的内容/值**（需重写该方法，包装类已重写）          | 判断两个包装类对象的值是否相等       | 会先判断类型是否一致，类型不同直接返回 `false` |

##### 二、代码示例：直观理解区别

###### 1. 基础场景（无缓存）

```java
public class WrapperCompare {
    public static void main(String[] args) {
        // 场景1：new 创建的包装类对象（地址不同）
        Integer a = new Integer(100);
        Integer b = new Integer(100);
        
        System.out.println(a == b); // false（地址不同）
        System.out.println(a.equals(b)); // true（值相同）

        // 场景2：基本类型 vs 包装类（自动拆箱）
        Integer c = 100;
        int d = 100;
        System.out.println(c == d); // true（c 自动拆箱为基本类型，比较值）

        // 场景3：类型不同的包装类（equals 判类型）
        Integer e = 100;
        Long f = 100L;
        System.out.println(e.equals(f)); // false（类型不同，即使值相同）
    }
}
```

###### 2. 易错场景：包装类的常量池缓存

Java 为了优化性能，对 `Byte`/`Short`/`Integer`/`Long`（-128 ~ 127）、`Character`（0 ~ 127）的包装类对象做了**常量池缓存**——该范围内的值直接复用对象，超出范围则新建对象：
```java
public class WrapperCache {
    public static void main(String[] args) {
        // 缓存范围内（-128~127）：复用对象，地址相同
        Integer g = 100;
        Integer h = 100;
        System.out.println(g == h); // true（缓存复用）
        System.out.println(g.equals(h)); // true

        // 缓存范围外：新建对象，地址不同
        Integer i = 200;
        Integer j = 200;
        System.out.println(i == j); // false（超出缓存，新建对象）
        System.out.println(i.equals(j)); // true（值相同）
    }
}
```

##### 三、核心建议

1. **比较包装类的值**：无论是否在缓存范围内，一律使用 `equals()` 方法，避免缓存导致的误判；
2. **包装类 vs 基本类型**：`==` 会触发自动拆箱，比较的是值，此时可用 `==`，但为了统一风格，也推荐用 `equals()`；
3. **跨类型比较**：`equals()` 会先判断类型，若需跨类型比较值（如 `Integer` 和 `Long`），需先转为同一类型再比较。

---

###### 总结

1. **`==` 核心**：比较引用地址（对象）/ 值（基本类型）；包装类缓存范围内的小值会复用对象，导致 `==` 看似“比较值”，实则是地址相同。
2. **`equals()` 核心**：包装类已重写该方法，优先判断类型，再比较值；是比较包装类值的**唯一可靠方式**。
3. **最佳实践**：包装类对象比较值 → 用 `equals()`；基本类型比较 → 用 `==`；跨类型比较 → 先统一类型再用 `equals()`。

#### **8.当基本数据类型转换为包装类时，如果值为 null 会发生什么情况？**

当基本数据类型转换为包装类时，如果值为 `null`，会抛出 `NullPointerException` 异常。

#### **9.Long 类的 valueOf() 方法有什么特点？它和 new Long() 有什么区别？**

- `valueOf()` 方法：如果传入的值在缓存范围内（-128 到 127），它会返回一个缓存的对象，而不是每次都创建新对象。
- `new Long()`：每次都会创建一个新的对象，因此更消耗内存。

#### **10.包装类在集合框架中起到了什么作用？为什么集合不能直接存储基本数据类型？**

 包装类可以将基本数据类型封装为对象，而集合框架要求存储的是对象。基本数据类型不能直接作为对象存储在集合中，因此需要使用对应的包装类。

#### **11.什么是拆箱空指针异常？在什么情况下会发生？如何避免？**

 **拆箱空指针异常**：当尝试将 `null` 值拆箱为基本数据类型时，抛出 `NullPointerException` 异常。
 避免方法：确保在拆箱前，包装类对象不是 `null`。

#### **12.比较两个 Double 类型的值时，为什么不建议使用 == 方法？应该使用什么方法？**

 `==` 比较的是两个 `Double` 对象的引用是否相同，而不是值是否相等。由于浮点数可能存在精度误差，建议使用 `Double.compare()` 或 `equals()` 方法来比较值。

在比较`double`类型（基本数据类型）和`Double`类型（包装类）的值时，不建议直接使用`==`运算符，主要原因如下：

##### 1. 为什么不建议对 `double` 使用 `==`？

`double` 是浮点型数据，在计算机中存储的是近似值（受二进制表示方式限制），例如：

```java
double a = 0.1 + 0.2;
double b = 0.3;
System.out.println(a == b);  // 输出 false
```
这是因为 `0.1 + 0.2` 的实际计算结果是 `0.30000000000000004`，与 `0.3` 并不完全相等，直接用 `==` 会导致逻辑错误。

##### 2. 为什么不建议对 `Double`（对象）使用 `==`？

`Double` 是包装类，属于引用类型。`==` 对引用类型比较的是**对象的内存地址**，而非实际值。例如：
```java
Double x = 1.0;
Double y = 1.0;
System.out.println(x == y);  // 可能输出 false（地址不同）
```
即使两个 `Double` 对象的值相同，它们也可能是不同的对象（内存地址不同），导致 `==` 判断为 `false`。

##### 正确的比较方式

##### （1）比较 `double` 基本类型

应通过**判断两个值的差值是否小于一个极小的阈值（epsilon）** 来确定是否相等，例如：
```java
double a = 0.1 + 0.2;
double b = 0.3;
double epsilon = 1e-9;  // 阈值，可根据精度需求调整

if (Math.abs(a - b) < epsilon) {
    System.out.println("相等");  // 输出：相等
}
```

##### （2）比较 `Double` 包装类

- 先确保对象不为 `null`（避免空指针异常）；
- 再调用 `doubleValue()` 转为基本类型，然后用阈值法比较；
- 或直接使用 `equals()` 方法（但需注意 `equals()` 对精度敏感，且 `null` 处理）。

示例：
```java
Double x = 1.0;
Double y = 1.0;

// 方法1：转为基本类型后用阈值比较（推荐，可控制精度）
if (x != null && y != null && Math.abs(x.doubleValue() - y.doubleValue()) < 1e-9) {
    System.out.println("相等");
}

// 方法2：使用equals()（需注意精度严格匹配，且不能有null）
if (x != null && x.equals(y)) {
    System.out.println("相等");
}
```

##### 总结

- 对 `double` 基本类型：用**阈值法**（`Math.abs(a - b) < epsilon`）比较；
- 对 `Double` 包装类：先判空，再转基本类型用阈值法，或谨慎使用 `equals()`（精度敏感）。

核心原因：浮点型的二进制存储特性导致精确值比较不可靠，引用类型的 `==` 比较的是地址而非值。

##### 注意

`Double` 类的 `equals()` 方法对精度**非常敏感**，意思是它会严格比较两个对象的**实际存储值**，哪怕只有极小的差异（比如小数点后第15位不同），也会返回 `false`。

这与浮点型数据的存储特性有关——很多小数（如 `0.1`）在计算机中无法被精确表示，只能存储近似值，这会导致 `equals()` 可能出现不符合直觉的结果。

###### 举例说明

```java
Double a = 0.1 + 0.2;  // 实际存储值约为 0.30000000000000004
Double b = 0.3;        // 实际存储值约为 0.29999999999999999

System.out.println(a.equals(b));  // 输出 false
```
从数学角度看 `0.1+0.2` 应该等于 `0.3`，但由于存储的近似值不同，`equals()` 会判定它们不相等。

###### 为什么会这样？

`Double.equals()` 的底层实现逻辑是：**先判断类型是否相同，再直接比较两个对象的原始二进制值**（即严格的数值相等）。源码简化如下：

```java
public boolean equals(Object obj) {
    return (obj instanceof Double) 
        && (doubleToLongBits(this.value) == doubleToLongBits(((Double)obj).value));
}
```
其中 `doubleToLongBits()` 会直接转换浮点值的二进制表示，哪怕是极小的精度差异，转换后的值也会不同，导致 `equals()` 返回 `false`。

###### 何时使用 `equals()`？

只有在**明确需要严格比较两个浮点值的实际存储值**时，才适合用 `Double.equals()`。例如：
- 确认两个值是否完全一致（包括精度细节）；
- 作为 `HashMap` 的键时（依赖 `equals()` 判断键是否相同）。

但对于大多数业务场景（如比较计算结果、判断数值是否“逻辑相等”），`equals()` 过于严格，更推荐使用**阈值法**（比较差值是否小于极小值）。

#### **13.Byte 类型的取值范围是多少？它的包装类有什么特殊之处？**

- `byte` 类型的取值范围是 -128 到 127。
- `Byte` 类是包装类，并且实现了 `Comparable<Byte>` 接口，可以与其他 `Byte` 对象进行比较。

#### **14.自动装箱的实现原理是什么？在编译和运行阶段分别会进行哪些操作？**

- **编译时**：Java 编译器会自动将**基本数据类型**转换为对应的包装类对象。
- **运行时**：JVM 会将基本数据类型的**值**自动装箱成对象，通常通过调用 `valueOf()` 方法实现。

#### **15.包装类的缓存机制是指什么？哪些包装类实现了缓存机制？缓存的范围是多少？**

包装类的缓存机制指的是，JVM 为了优化性能，缓存了一定范围内的常用值，避免每次都创建新对象。

- `Byte`、`Short`、`Integer`、`Long` 等包装类都实现了缓存机制。
- 例如，`Integer` 类缓存的是 -128 到 127 范围内的对象。



##### 1. 什么是包装类的缓存机制？

包装类的缓存机制是 Java 为了**优化性能、节省内存**而设计的一种机制。

JVM 在启动时，会预先创建并缓存一定范围内的包装类对象。当程序通过**自动装箱**（如 `Integer i = 100;`）或调用 `valueOf()` 方法获取该范围内的数值时，JVM 会直接返回缓存中已有的对象实例，而不会每次都创建新的对象。

##### 2. 哪些包装类实现了缓存机制？

并非所有包装类都有缓存。通常只有**使用频率高、取值范围相对离散或固定**的类型才实现了缓存。

| 包装类          | 是否缓存 | 缓存范围         | 说明                                                         |
| --------------- | -------- | ---------------- | ------------------------------------------------------------ |
| **`Integer`**   | 是       | **-128 到 127**  | 默认范围，上限可通过 JVM 参数 `-Djava.lang.Integer.IntegerCache.high` 调整。 |
| **`Byte`**      | 是       | **-128 到 127**  | 范围固定（因为 byte 的取值范围本身就是 -128~127）。          |
| **`Short`**     | 是       | **-128 到 127**  | 范围固定。                                                   |
| **`Long`**      | 是       | **-128 到 127**  | 范围固定。                                                   |
| **`Character`** | 是       | **0 到 127**     | 对应 ASCII 字符范围。                                        |
| **`Boolean`**   | 是       | **true / false** | 仅缓存两个静态常量实例。                                     |
| **`Float`**     | 否       | 无               | 浮点数取值范围太广且离散，缓存无意义。                       |
| **`Double`**    | 否       | 无               | 同上。                                                       |

##### 💡 关键补充（面试加分项）

- 触发条件：缓存只在调用 `valueOf()`或进行自动装箱时生效。

  - 如果你使用 `new Integer(100)` 这种方式，JVM 会**强制创建**一个新的对象，而不会去使用缓存池中的对象。

- 比较陷阱

  ：这是面试必考的陷阱！

  - 在缓存范围内（-128~127），使用 `==` 比较两个装箱后的 Integer 变量会返回 `true`（因为引用的是同一个缓存对象）。
  - 超出缓存范围，使用 `==` 比较会返回 `false`（因为是两个不同的对象）。
  - **结论**：在比较包装类的值时，应始终使用 `.equals()` 方法，而不是 `==`。



#### **16.当把一个 int 类型的值赋给 Integer 类型的变量时，会发生自动装箱，这个过程中会创建新的对象吗？**

- 如果赋值的 `int` 在 `Integer` 的缓存范围内（-128 到 127），不会创建新的对象，而是返回缓存的对象。
- 如果值超出缓存范围，则会创建新的 `Integer` 对象。

#### **17.Short 类型的包装类和 int 类型的包装类之间如何进行转换？**

可以通过 `Short` 的 `shortValue()` 方法将 `Short` 转换为 `short` 类型，再通过 `int` 的值进行转换。反之，也可以通过 `int` 的构造方法将 `int` 转换为 `Short` 对象。

#### **18.包装类的 parseXxx() 方法和 valueOf() 方法有什么区别？**

- `parseXxx()`：将字符串解析为基本数据类型（例如：`Integer.parseInt()`）。
- `valueOf()`：将字符串转换为对应类型的包装类对象（例如：`Integer.valueOf()`）。

#### **19.在多线程环境下，使用包装类的对象作为共享变量时，需要注意什么问题？**

包装类对象是不可变的，但在多线程环境下，由于可能存在拆箱和装箱操作，仍然需要注意线程安全问题。可以使用 `synchronized` 或 `volatile` 修饰符来保证线程安全。



##### ⚠️ 核心问题：不可变 ≠ 线程安全

包装类（`Integer`、`Long`、`Boolean` 等）确实是**不可变对象**，但共享变量本身存在**引用可见性**和**原子性**问题。

```java
public class Counter {
    private Integer count = 0;  // 共享变量
    
    public void increment() {
        count++;  // 危险！不是原子操作
    }
}
```

**`count++` 实际执行：**
1. **拆箱**：`int temp = count.intValue()` （读取）
2. **计算**：`temp = temp + 1`
3. **装箱**：`count = Integer.valueOf(temp)` （写入新对象）

> 虽然 `Integer` 不可变，但**引用变量 `count` 被重新赋值**，存在竞态条件。

---

##### 🔒 解决方案对比

| 方案                  | 实现方式                         | 适用场景                       | 特点                                 |
| --------------------- | -------------------------------- | ------------------------------ | ------------------------------------ |
| **`volatile`**        | `private volatile Integer count` | 单一写、多读，且操作是原子性的 | 保证可见性，**不保证复合操作原子性** |
| **`synchronized`**    | `synchronized` 方法或代码块      | 需要原子性操作                 | 保证可见性 + 互斥性，性能较低        |
| **`Atomic` 类** ⭐推荐 | `AtomicInteger`                  | 高频计数、累加                 | 无锁（CAS），性能最优                |

---

##### ❌ 常见误区：`volatile` 不能保证 `++` 原子性

```java
public class UnsafeVolatile {
    private volatile Integer count = 0;
    
    public void increment() {
        count++;  // ❌ 仍然线程不安全！三步操作非原子
    }
}
```

**正确做法：使用 `AtomicInteger`**

```java
public class SafeCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // ✅ CAS 原子操作，无锁线程安全
    }
    
    public int get() {
        return count.get();  // 保证可见性
    }
}
```

---

##### 📊 缓存陷阱：`Integer.valueOf()` 的缓存机制

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);  // true（缓存命中）

Integer c = 128;
Integer d = 128;
System.out.println(c == d);  // false（新建对象）
```

**影响**：虽然与线程安全无直接关系，但**错误使用 `==` 判断相等**会导致逻辑错误，应始终用 `.equals()`。

---

##### ✅ 最佳实践总结

```java
// 1. 计数/累加场景 → 用 Atomic 类
private final AtomicInteger counter = new AtomicInteger(0);

// 2. 标志位开关 → volatile boolean（原子操作）
private volatile boolean running = true;

// 3. 复杂状态管理 → synchronized 或 ReentrantLock
private final Object lock = new Object();
private Integer balance = 100;

public void withdraw(int amount) {
    synchronized (lock) {
        if (balance >= amount) {
            balance -= amount;  // 复合操作需要同步
        }
    }
}
```

---

##### 一句话修正原答案

> `volatile` 只能保证**可见性**，不能保证 `++` 这类复合操作的**原子性**；对于包装类的运算场景，**`Atomic` 类是最佳选择**，`synchronized` 是兜底方案。

#### **20.为什么 Boolean 类型的包装类只有两个实例对象？它的 valueOf() 方法是如何实现的？**

`Boolean` 类的 `valueOf()` 方法返回的只有 `Boolean.TRUE` 和 `Boolean.FALSE`，这两个对象是 `Boolean` 类的单例实例。它的实现是通过 `Boolean.TRUE` 和 `Boolean.FALSE` 实现缓存机制，避免多次创建相同的对象。

这些问题深入探讨了 Java 中基本数据类型与包装类的各个方面，适合面试时评估候选人对这部分知识的理解和应用能力。

#### 21.显示转换和隐式转换

在Java中，基本数据类型的转换分为**隐式转换**（自动转换）和**显示转换**（强制转换），用于解决不同类型之间的赋值或运算问题。两者的核心区别在于是否需要开发者手动指定转换方式。

##### 一、隐式转换（自动转换）

隐式转换是指**编译器自动完成的类型转换**，无需开发者干预。它遵循“从小到大”的规则，即**将取值范围小的类型自动转换为取值范围大的类型**，以避免数据丢失。

###### 适用场景：

1. **赋值时**：当把小范围类型的值赋给大范围类型变量时。
2. **运算时**：不同类型的数据参与运算时，会自动转换为其中范围最大的类型。

###### 基本数据类型的隐式转换顺序：

```
byte → short → int → long → float → double
          char → int （char是16位无符号整数，可转换为int）
```

###### 示例：

```java
// 1. 赋值时的隐式转换
byte b = 10;
int i = b; // byte → int（自动转换，无需手动处理）
long l = i; // int → long
float f = l; // long → float（float范围比long大）
double d = f; // float → double

// 2. 运算时的隐式转换
char c = 'A'; // 'A'的ASCII值是65
int sum = c + 10; // char → int，结果为75
double result = 10 + 3.14; // int → double，结果为13.14
```

##### 二、显示转换（强制转换）

显示转换是指**开发者手动指定的类型转换**，用于将**大范围类型转换为小范围类型**。由于可能导致数据丢失或精度降低，必须显式声明转换目标类型。

###### 语法：

```java
目标类型 变量名 = (目标类型) 源数据;
```

###### 适用场景：

1. 必须将大范围类型的值赋给小范围类型变量时；
2. 需要主动截断或转换数据精度时。

###### 示例：

```java
// 1. 基本强制转换（可能丢失精度）
double d = 3.14;
int i = (int) d; // 强制转换，结果为3（小数部分被截断）

long l = 1000L;
short s = (short) l; // long → short（如果l超过short范围，结果会溢出）

// 2. 运算中的强制转换
int a = 10;
int b = 3;
double division = (double) a / b; // 将a强制转为double，结果为3.333...（而非3）
```

###### 注意点：

- **数据丢失风险**：浮点型转整数型会截断小数部分；大范围整数转小范围可能导致溢出（如 `int i = (int) 10000000000L;` 会得到错误值）。
- **不能跨“类型族”转换**：如 boolean 不能与任何其他基本类型转换（`int i = (int) true;` 是错误的）。

##### 三、核心区别总结

| 类型       | 转换方向               | 是否需要手动干预 | 安全性               | 典型场景                 |
|------------|------------------------|------------------|----------------------|--------------------------|
| 隐式转换   | 小范围 → 大范围        | 否（自动完成）   | 安全（无数据丢失）   | 赋值、不同类型数据运算   |
| 显示转换   | 大范围 → 小范围        | 是（需强制声明） | 可能不安全（有丢失） | 必须缩小类型范围时       |

使用复合运算符（如 `+=`、`-=`、`*=`、`/=`、`%=` 等）时，**会自动进行隐式转换**，这是复合运算符的一个特殊特性。

具体来说：当使用 `a op= b` 时（`op` 是算术运算符），Java 会将其自动编译为 `a = (a的类型) (a op b)`，即**自动包含对运算结果的强制转换**，无需开发者手动添加。

##### 四、复合运算

###### 1. 基本类型的复合运算（自动隐式转换）

```java
short s = 10;
s += 5; // 等价于 s = (short) (s + 5)
// 这里 s + 5 的结果是 int 类型，但复合运算符自动将结果强转为 short

int i = 10;
i *= 0.5; // 等价于 i = (int) (i * 0.5)
// i * 0.5 结果是 double 类型，复合运算符自动转为 int（结果为5）
```

如果不使用复合运算符，直接写 `s = s + 5` 会报错（`s + 5` 是 `int` 类型，无法直接赋值给 `short`），必须手动强制转换：
```java
short s = 10;
s = (short) (s + 5); // 必须手动转换，否则编译错误
```

###### 2. 复合运算符与隐式转换的关系

复合运算符的本质是**简化了“运算 + 赋值”的过程，并自动添加了强制转换**，这属于 Java 编译器的语法糖。

它的转换规则是：
- 先按照正常运算规则计算 `a op b`（此时会发生隐式转换，如 `short + int` 结果为 `int`）；
- 再将结果强制转换为 `a` 的原始类型，赋值给 `a`。

###### 注意点

- 复合运算符的自动转换**可能导致数据丢失**（与显式强制转换的风险相同）。例如：
  ```java
  byte b = 127;
  b += 2; // 等价于 b = (byte) (127 + 2)，结果为 -127（溢出）
  ```
- 仅适用于基本数据类型，引用类型的复合运算（如 `list += element`）是语法错误。

###### 总结

复合运算符**会自动进行隐式转换（本质是自动添加强制转换）**，无需开发者手动处理类型转换，这是它与普通运算符（如 `+`、`-`）的重要区别。这种特性简化了代码，但也要注意可能的精度丢失或溢出问题。

##### 总结

- 隐式转换是“安全的自动转换”，遵循“从小到大”规则；
- 显示转换是“风险的手动转换”，需开发者对数据范围负责；
- 转换的核心目的是解决不同类型之间的赋值和运算兼容性，避免不必要的错误。



#### 22.Long 和long

在Java中，`Long sum = 0;` 和 `long sum = 0;` 的核心区别在于**数据类型的类别**（引用类型 vs 基本类型），具体区别如下：

##### 1. **数据类型本质**

- **`long sum = 0;`**  
  - `long` 是Java的**基本数据类型**（8种基本类型之一），用于表示64位整数，直接存储数值本身。  
  - `sum` 变量直接持有值 `0`，存储在栈内存中。  

- **`Long sum = 0;`**  
  - `Long` 是 `long` 对应的**包装类**（引用类型），是一个类，用于将基本类型 `long` 封装为对象。  
  - `sum` 是对象引用，指向一个 `Long` 实例，该实例在堆内存中存储数值 `0`。  
  - 这里会发生**自动装箱**：`0`（基本类型 `long`）会被自动转换为 `Long` 对象（通过 `Long.valueOf(0)` 实现）。  

##### 2. **默认值不同**

- 基本类型 `long` 的默认值是 `0L`（数值0）。  
- 引用类型 `Long` 的默认值是 `null`（未指向任何对象）。  

##### 3. **使用场景差异**

- **`long`（基本类型）**：  
  - 适合用于数值计算（如求和、累加等），性能更好（无需处理对象的创建和回收）。  
  - 不能直接用于需要对象的场景（如泛型集合 `List<Long>` 中不能放 `long`，只能放 `Long`）。  

- **`Long`（包装类）**：  
  - 适合需要对象的场景，例如泛型（`List<Long>`）、反射、序列化等。  
  - 提供了一些实用方法（如 `Long.parseLong(String)` 字符串转长整数、`Long.MAX_VALUE` 获取最大值等）。  

##### 4. **注意事项**

- **自动装箱/拆箱**：  
  `Long` 和 `long` 之间可以自动转换（装箱：`long`→`Long`；拆箱：`Long`→`long`），但频繁转换可能影响性能（尤其是在循环中）。  

- **空指针风险**：  
  `Long` 作为引用类型可以为 `null`，如果对 `null` 的 `Long` 变量进行拆箱（例如 `long value = longObj;` 而 `longObj` 为 `null`），会抛出 `NullPointerException`。  

###### 示例对比

```java
public class LongVslong {
    public static void main(String[] args) {
        // 基本类型：直接存储值，默认值0
        long primitive = 0;
        System.out.println(primitive + 1); // 输出：1（直接参与计算）

        // 包装类：引用对象，自动装箱
        Long wrapper = 0; // 等价于 Long wrapper = Long.valueOf(0);
        System.out.println(wrapper + 1); // 输出：1（自动拆箱后计算）

        // 空指针示例
        Long nullWrapper = null;
        // long value = nullWrapper; // 编译通过，但运行时抛出NullPointerException
    }
}
```

##### 总结

| 特性         | `long sum = 0;`               | `Long sum = 0;`               |
|--------------|-------------------------------|-------------------------------|
| 类型         | 基本数据类型                  | 引用类型（包装类）            |
| 存储方式     | 直接存储数值（栈内存）        | 存储对象引用（堆内存存数值）  |
| 默认值       | `0L`                          | `null`                        |
| 适用场景     | 数值计算、性能敏感场景        | 泛型、对象操作、工具方法调用  |
| 空指针风险   | 无                            | 有（可能为`null`）            |

在求和等简单数值计算场景中，优先使用 `long`（基本类型），更高效且避免空指针问题；当需要对象特性时（如泛型），再使用 `Long` 包装类。



#### 23.初始化默认值

在Java中，只有**类级别的成员变量**（包括实例变量和静态变量）会被自动赋予默认初始值。其他类型的变量（局部变量、方法参数、构造器参数等）都**不会**进行默认初始化，必须显式赋值后才能使用。

具体来说：

1. **实例变量**（属于对象的成员变量）  会被自动初始化，默认值规则：
   
   - 引用类型（如`String`、自定义类）默认值为`null`
   - 基本类型：`int`为`0`，`boolean`为`false`，`double`为`0.0`等

2. **静态变量**（属于类的成员变量，用`static`修饰）  同样会被自动初始化，默认值规则与实例变量相同。
   
3. **局部变量**（方法、代码块内部定义的变量）  必须显式初始化才能使用，否则编译报错。  
   例：
   
   ```java
   public void test() {
       int num; // 局部变量未初始化
       System.out.println(num); // 编译错误
   }
   ```

4. **方法参数/构造器参数**  
   由调用方传入值，不存在默认初始化，也无需在声明时赋值。

5. **数组元素**  
   数组本身是引用类型，当数组被创建（如`new int[3]`）时，其元素会按规则自动初始化：
   
   - 基本类型数组元素：按对应基本类型默认值初始化（如`int[]`元素为`0`）
   - 引用类型数组元素：默认值为`null`

总结：只有类的成员变量（实例变量和静态变量）以及数组元素会被Java自动赋予默认值，其他类型的变量必须显式赋值后才能使用。

#### 24. a 中 float、double、int、long 的赋值详解

在 Java 中，`int`、`long` 是**整数类型**，`float`、`double` 是**浮点类型（小数）**，它们的赋值规则有明确区别，尤其是字面量的写法和类型匹配问题，新手容易踩坑。下面我会分类型讲解赋值方式，并说明关键注意事项。

##### 一、基本类型的核心属性（先明确基础）

| 类型   | 占用字节 | 取值范围                  | 默认值 | 字面量标识  |
| ------ | -------- | ------------------------- | ------ | ----------- |
| int    | 4        | -2³¹ ~ 2³¹-1（约±21亿）   | 0      | 无（默认）  |
| long   | 8        | -2⁶³ ~ 2⁶³-1              | 0L     | L/l         |
| float  | 4        | 单精度浮点（精度约6-7位） | 0.0f   | F/f         |
| double | 8        | 双精度浮点（精度约15位）  | 0.0d   | D/d（可选） |

##### 二、逐类型讲解赋值方式

###### 1. int（整型，最常用）

`int` 是 Java 默认的整数类型，直接写数字即可，无需额外标识。

**赋值示例**：
```java
// 1. 直接赋值字面量（最基础）
int a = 100;          // 正整数
int b = -50;          // 负整数
int c = 0;            // 零

// 2. 十六进制（0x 开头）
int d = 0x1A;         // 等价于十进制 26
// 3. 八进制（0 开头，注意：容易误写，不推荐）
int e = 012;          // 等价于十进制 10
// 4. 二进制（Java 7+ 支持，0b 开头）
int f = 0b1010;       // 等价于十进制 10

// 5. 变量/表达式赋值
int g = a + b;        // 100 + (-50) = 50
int h = 10 * 20;      // 200
```

**注意**：赋值的数字不能超过 `int` 的取值范围（±21亿左右），否则会编译报错，需改用 `long`。

###### 2. long（长整型）

`long` 用于存储更大范围的整数，**字面量必须加 `L` 或 `l`（推荐大写 L，避免和数字 1 混淆）**。

**赋值示例**：
```java
// 1. 基础赋值（必须加 L）
long a = 10000000000L; // 超过 int 范围，必须加 L
long b = 500L;         // 即使没超过 int 范围，加 L 更规范
long c = -1234567890L;

// 2. 不加 L 的情况（自动类型转换，仅限数值在 int 范围内）
long d = 100;          // 编译器自动把 int 100 转为 long（合法）
// long e = 2147483648; // 报错！2147483648 超过 int 范围，必须加 L

// 3. 变量/表达式赋值
long f = a + b;        // 10000000000L + 500L = 10000000500L
long g = 10L * 20;     // 200L（int 20 自动转 long）
```

###### 3. float（单精度浮点）

`float` 是单精度小数，**字面量必须加 `F` 或 `f`**（Java 默认小数是 `double` 类型，不加 F 会编译报错）。

**赋值示例**：
```java
// 1. 基础赋值（必须加 F）
float a = 3.14F;       // 正确，加 F 标识 float 类型
float b = -0.5f;       // 小写 f 也可以，推荐大写 F
float c = 100F;        // 整数也可以赋值为 float（100.0F）

// 2. 科学计数法
float d = 1.23e5F;     // 1.23 × 10⁵ = 123000.0F
float e = 4.56e-3F;    // 4.56 × 10⁻³ = 0.00456F

// 3. 错误示例（不加 F）
// float f = 3.14;      // 报错！3.14 是 double 类型，不能直接赋值给 float

// 4. 变量/表达式赋值
float g = a + b;       // 3.14F + (-0.5F) = 2.64F
float h = 10F * 2.5F;  // 25.0F
```

###### 4. double（双精度浮点）

`double` 是 Java 默认的浮点类型，字面量可加 `D/d`（可选，不加也默认是 double），精度比 float 更高。

**赋值示例**：
```java
// 1. 基础赋值（可加 D，也可不加）
double a = 3.1415926;  // 推荐：不加 D，简洁
double b = 2.71828D;   // 加 D 也合法，小写 d 同理
double c = -100.0;     // 负小数
double d = 500d;       // 整数赋值为 double（500.0）

// 2. 科学计数法
double e = 1.23456789e8;  // 1.23456789 × 10⁸ = 123456789.0
double f = 9.87654321e-4; // 9.87654321 × 10⁻⁴ = 0.000987654321

// 3. 变量/表达式赋值
double g = a + b;      // 3.1415926 + 2.71828 = 5.8598726
double h = 10 * 3.14;  // 31.4（int 10 自动转 double）
```

##### 三、常见坑点与注意事项

1. **long 忘记加 L**：  
   比如 `long num = 2147483648;` 会报错，因为 2147483648 超过 int 范围，必须写成 `2147483648L`。

2. **float 忘记加 F**：  
   `float pi = 3.14;` 报错，因为 3.14 是 double 类型，double 转 float 是「窄化转换」，必须显式加 F 或强制转换（`(float)3.14`）。

3. **浮点类型的精度问题**：  
   `double num = 0.1;` 实际存储的是近似值，不是精确的 0.1，因此涉及金额等精确计算时，不要用 float/double，改用 `BigDecimal`。

4. **自动类型转换规则**：  
   小范围类型可自动转为大范围类型（int → long → float → double），反之需要强制转换（如 `int a = (int)100L;`）。

##### 四、完整示例代码

```java
public class TypeAssignment {
    public static void main(String[] args) {
        // int 赋值
        int intVal = 1000;
        System.out.println("int 赋值结果：" + intVal); // 1000

        // long 赋值
        long longVal = 10000000000L;
        System.out.println("long 赋值结果：" + longVal); // 10000000000

        // float 赋值
        float floatVal = 3.1415F;
        System.out.println("float 赋值结果：" + floatVal); // 3.1415

        // double 赋值
        double doubleVal = 3.141592653589793;
        System.out.println("double 赋值结果：" + doubleVal); // 3.141592653589793

        // 类型转换示例
        long convertLong = intVal; // int 自动转 long
        double convertDouble = floatVal; // float 自动转 double
        System.out.println("int 转 long：" + convertLong); // 1000
        System.out.println("float 转 double：" + convertDouble); // 3.1415000557899475
    }
}
```

---

##### 总结

1. **int**：直接赋值数字，无需标识，注意取值范围（±21亿）。
2. **long**：字面量必须加 `L`（大写），即使数值在 int 范围内，加 L 更规范。
3. **float**：字面量必须加 `F`（大写），否则默认是 double 类型，编译报错。
4. **double**：默认浮点类型，字面量可加 `D`（可选），精度更高，无需额外标识即可直接赋值。



#### 25.Java中基本数据类型的自动类型转换

当传入整数字面量5时，默认为int类型。在方法重载的情况下，Java会按照"最小代价"的原则选择最合适的方法。对于int类型的参数来说，转换优先级为:int -> long -> float -> double
或者 int -> short



#### 26. 属性（字段）和变量的区别？

##### 属性（成员变量、字段 field）

- 定义在**类中、方法外**
- 有**默认值**
- 存在**堆**
- 有权限修饰符（public、private…）

##### 变量（局部变量）

- 定义在**方法或代码块中**
- **无默认值，必须初始化**
- 存在**栈**
- 不能用权限修饰符

一句话区分：

**类里的是属性，方法里的是局部变量。**

我继续给你**最标准、最精简、面试直接背**的答案，完全对应你这份牛客原题，**通俗易懂不绕弯**。

#### 27、 什么是自动拆装箱？

**自动装箱：**  
基本类型 → 自动转换成对应的**包装类对象**  
例：`int → Integer`  

```java
Integer a = 10; // 自动装箱
```

**自动拆箱：**  
包装类对象 → 自动转换成**基本类型**  
例：`Integer → int`  
```java
int b = a; // 自动拆箱
```

底层实际是编译器帮你调用：
- 装箱：`Integer.valueOf(10)`
- 拆箱：`a.intValue()`

一句话：  
**基本类型 ↔ 包装类 自动转换，就是自动拆装箱。**

---

#### 28. 为什么要转换？（为什么需要包装类）

1. **Java 是面向对象，集合（List/Map）只能存对象，不能存基本类型。**
2. 包装类可以为 **null**，基本类型不能为 null。
3. 包装类提供很多**实用方法**（如类型转换、比较、解析）。
4. 泛型**只支持引用类型**，不支持基本类型。

一句话：  
**基本类型不是对象，无法存入集合、无法为null、无法用泛型，所以需要包装类。**



#### 29. 你平时是如何使用泛型的？

泛型：**参数化类型**，把类型当作参数传递。

主要用途：
1. **集合中使用**（最常用）  
   ```java
   List<String> list = new ArrayList<>();
   ```
   限定只能存 String，**编译期检查类型，避免强制转换**。

2. **泛型类**：类的类型不确定，用泛型表示  
   
   ```java
   class User<T> { ... }
   ```
```
   
3. **泛型方法**：方法参数/返回值使用泛型  
   
   ```java
   public <T> T get(T t) { ... }
```

4. **通配符 ?**  
   - `<?>` 任意类型  
   - `<? extends T>` 上限（T及其子类）  
   - `<? super T>` 下限（T及其父类）

**核心好处：**  
**编译期类型安全、避免强转、代码通用。**





### 三、关键字与标识符

#### **1.Java 中的标识符命名规则是什么？有哪些字符可以用来命名标识符？**

- 标识符只能包含字母、数字、下划线（_）和美元符号（$）。
- 标识符不能以数字开头。
- 标识符不能是 Java 的关键字。
- Java 标识符对大小写敏感。
- 标识符的长度没有限制，但应简短且具有描述性。

#### **2.标识符可以使用 Java 中的关键字吗？为什么？**

标识符不能使用 Java 中的关键字。因为关键字是 Java 语言的保留字，用于定义语言的语法结构，不能作为变量、方法、类等的名称。

#### **3.Java 中的关键字 `final` 有什么作用？它可以修饰哪些元素？分别有什么效果？如何初始化？**

- **`final` 修饰类**：表示该类不能被继承。
- **`final` 修饰方法**：表示该方法不能被重写。
- **`final` 修饰变量**：表示该变量是常量，值不可修改。

在Java中，final变量(成员变量)必须在以下位置之一进行初始化：
\1. 声明时直接初始化
\2. 在构造方法中初始化
\3. 在初始化块中初始化



`final`变量的初始化**不一定要放到构造函数中**，但必须在**变量使用前完成初始化**，且**只能初始化一次**。具体初始化方式取决于变量的类型（成员变量、局部变量、静态变量）和声明位置。

##### 一、`final`变量的初始化时机

###### 1. 成员变量（非静态）

`final`成员变量有两种初始化方式，二选一即可：

- **声明时直接初始化**（推荐简单值）
- **在构造函数中初始化**（适合需要依赖构造参数或复杂逻辑的值）

**示例**：
```java
public class FinalDemo {
    // 方式1：声明时直接初始化
    private final int simpleValue = 100;
    
    // 方式2：在构造函数中初始化（必须所有构造函数都初始化，或通过this()调用统一初始化）
    private final int complexValue;
    
    public FinalDemo(int param) {
        // 依赖构造参数的复杂初始化
        this.complexValue = param * 2; 
    }
    
    public FinalDemo() {
        // 多个构造函数必须都保证final变量被初始化
        this(50); // 调用另一个构造函数，避免重复代码
    }
}
```

**注意**：  
- 若类有多个构造函数，必须保证所有构造函数都初始化`final`成员变量（可通过`this(...)`调用其他构造函数间接初始化），否则编译报错。
- 初始化后不能再修改（包括构造函数内也只能赋值一次）。

###### 2. 静态`final`变量（类变量）

静态`final`变量属于类级别，初始化时机有两种：
- **声明时直接初始化**
- **在静态代码块中初始化**（适合复杂逻辑）

**示例**：
```java
public class StaticFinalDemo {
    // 方式1：声明时直接初始化
    public static final String CONSTANT = "固定值";
    
    // 方式2：静态代码块中初始化
    public static final int RANDOM_VALUE;
    
    static {
        // 复杂逻辑（如随机数、读取配置等）
        RANDOM_VALUE = (int) (Math.random() * 100);
    }
}
```

**注意**：  
- 静态`final`变量的初始化与类加载同步，且仅执行一次。
- 不能在构造函数中初始化静态`final`变量（构造函数属于对象级别，晚于类加载）。

###### 3. 局部变量

`final`局部变量（方法内、代码块内）的初始化更灵活：  

- 可以声明时初始化，也可以先声明后初始化，但**必须在使用前完成赋值**，且只能赋值一次。

**示例**：

```java
public void testLocalFinal() {
    // 方式1：声明时初始化
    final int a = 10;
    
    // 方式2：先声明，后初始化（必须在使用前）
    final int b;
    if (someCondition) {
        b = 20; // 第一次赋值
    } else {
        b = 30; // 第一次赋值（所有分支必须保证赋值）
    }
    
    System.out.println(b); // 使用前已初始化，合法
}
```

**注意**：  
- 若局部变量声明为`final`但未初始化，且存在未赋值的分支，编译报错（如`if`条件下漏写赋值）。

##### 二、核心原则总结

1. **必须初始化**：所有`final`变量（无论类型）都必须在**首次使用前**完成初始化，否则编译报错。
2. **只能初始化一次**：一旦赋值，无法再次修改（编译级别的限制）。
3. **初始化方式因类型而异**：
   - 非静态成员变量：声明时或构造函数中。
   - 静态成员变量：声明时或静态代码块中。
   - 局部变量：声明时或使用前的任意位置（需保证所有分支都赋值）。

因此，`final`变量的初始化不局限于构造函数，关键是根据变量类型选择合适的时机，确保“唯一且提前”初始化。

#### **`4.static` 关键字的作用是什么？它可以修饰类、方法、变量和代码块，分别说明其特点。**

- **修饰类**：在类的定义中不能使用 `static`，因为类本身是静态的。
- **修饰方法**：表示该方法是静态方法，可以直接通过类名调用，而不需要实例化对象。
- **修饰变量**：表示该变量是类级别的，所有对象共享该变量。
- **修饰代码块**：表示该代码块是静态初始化块，在类加载时执行。

你的总结基本正确，但关于 `static` 修饰类的描述需要修正。以下是更准确的说明：

`static` 用于表示“静态”，即属于类本身而非类的实例，它可以修饰**内部类、方法、变量和代码块**，但不能修饰顶级类（独立的类）。

##### 1. 修饰内部类（静态内部类）

`static` 可以修饰内部类（称为静态内部类），特点是：

- 无需依赖外部类的实例即可创建对象；
- 只能访问外部类的静态成员（静态变量、静态方法），不能直接访问非静态成员；
- 常用于将逻辑上相关的类组织在一起，同时避免与外部类产生强耦合。

示例：
```java
public class OuterClass {
    static class StaticInnerClass {  // 静态内部类
        // 可以独立实例化
    }
}

// 使用时无需先创建OuterClass实例
OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
```

##### 2. 修饰方法（静态方法）

- 属于类本身，而非对象，可直接通过 `类名.方法名()` 调用，无需实例化；
- 内部只能访问静态变量和静态方法，不能访问非静态成员（因非静态成员依赖于对象）；
- 不能使用 `this` 或 `super` 关键字（因无具体对象关联）。

典型应用：工具类方法（如 `Math.random()`）、工厂方法等。

##### 3. 修饰变量（静态变量/类变量）

- 属于类级别的变量，所有对象共享同一份内存空间；
- 生命周期与类一致（类加载时初始化，类卸载时销毁）；
- 访问方式：`类名.变量名` 或 `对象.变量名`（推荐前者，更清晰）。

示例：统计类的实例数量：
```java
public class Counter {
    public static int count = 0;  // 静态变量，所有实例共享
    
    public Counter() {
        count++;  // 每次创建对象时自增
    }
}
```

##### 4. 修饰代码块（静态代码块）

- 类加载时自动执行，且只执行一次（优先于构造方法）；
- 用于初始化静态变量或执行类级别的预处理操作；
- 多个静态代码块按定义顺序执行。

示例：加载lua脚本等初始化操作
```java
public class StaticBlockDemo {
    static {
        System.out.println("静态代码块执行");  // 类加载时执行
    }
    
    public StaticBlockDemo() {
        System.out.println("构造方法执行");  // 对象创建时执行
    }
}
```

##### 总结

`static` 的核心是“脱离实例独立存在”，其修饰的元素属于类本身，可直接通过类名访问，常用于共享数据、工具方法、初始化预处理等场景。

#### **`5.this` 关键字的含义是什么？它有哪些常用的用法？**

​    `this` 关键字指向当前对象的引用，常用于：

   - 区分实例变量和局部变量。
   - 调用当前对象的构造方法。
   - 访问当前对象的实例方法和变量。

被static修饰的方法中不能有this和super关键字，因为static方法被调用时，该类的对象可能还没有被创建，也就无法确定调用的是哪个对象。

你的理解基本正确，`this` 关键字确实代表当前对象的引用，它的核心作用是**明确指向当前对象的实例**，避免歧义并简化代码。以下是更详细的用法说明：

##### 1. 区分实例变量与局部变量（最常用）

当方法的参数名或局部变量名与类的实例变量名相同时，`this` 可以明确指定是实例变量：
```java
public class Person {
    private String name;  // 实例变量
    
    // 构造方法中参数名与实例变量名相同
    public Person(String name) {
        this.name = name;  // this.name 指实例变量，右侧 name 指参数
    }
    
    // 普通方法中区分
    public void setName(String name) {
        this.name = name;  // 同样通过 this 区分
    }
}
```

##### 2. 调用当前类的其他构造方法（`this(...)`）

在构造方法中，可以用 `this(参数)` 调用当前类的其他构造方法，避免代码重复（必须放在构造方法的第一行）：
```java
public class Student {
    private String name;
    private int age;
    
    // 无参构造
    public Student() {
        this("默认姓名", 18);  // 调用下面的有参构造
    }
    
    // 有参构造
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

##### 3. 作为当前对象的引用传递

可以将 `this` 作为参数传递给其他方法，代表当前对象本身：
```java
public class Order {
    private String id;
    
    public void process() {
        // 将当前订单对象传递给工具类的方法
        OrderService.handleOrder(this);
    }
}

class OrderService {
    // 接收一个Order对象并处理
    public static void handleOrder(Order order) {
        // 处理逻辑
    }
}
```

##### 4. 作为方法的返回值（链式调用）

在方法中返回 `this` 可以实现链式调用（常见于Builder模式）：
```java
public class Calculator {
    private int result;
    
    public Calculator add(int num) {
        result += num;
        return this;  // 返回当前对象，支持链式调用
    }
    
    public Calculator multiply(int num) {
        result *= num;
        return this;
    }
}

// 使用时可以链式调用
Calculator calc = new Calculator();
calc.add(2).multiply(3);  // 等价于先add再multiply
```

##### 注意点

- `this` 只能在**非静态方法、构造方法**中使用（静态方法属于类，无具体对象，因此不能用 `this`）。
- `this` 不能在类的静态代码块中使用（原因同上）。
- `this` 本质是一个“隐式参数”，编译器会自动在非静态方法中添加 `this` 引用。

总结来说，`this` 的核心作用是**明确引用当前对象**，解决命名冲突、简化构造方法调用、支持对象传递和链式编程等场景。

#### **`6.super` 关键字的作用是什么？它和 `this` 关键字有什么区别？**

- `super` 关键字用于访问父类的成员（变量或方法）。
- `this` 指向当前对象，而 `super` 指向当前对象的父类。
   区别：`this` 主要用于当前类的对象，`super` 主要用于父类。

你的理解抓住了核心，但可以更全面地展开。`super` 和 `this` 都是用于访问类成员的关键字，但它们的指向和用途有明确区别：

##### **`super` 关键字的作用**

`super` 代表**当前对象的父类实例的引用**，主要用于：

1. **访问父类的实例变量**  
   当子类与父类有同名的实例变量时，用 `super` 明确指定访问父类的变量：
   
   ```java
   class Parent {
       String name = "父类名称";
   }
   
   class Child extends Parent {
       String name = "子类名称";
       
       public void printName() {
           System.out.println(super.name);  // 访问父类的name
           System.out.println(this.name);   // 访问子类的name
       }
   }
   ```

2. **调用父类的实例方法**  
   当子类重写了父类的方法，用 `super` 可以调用父类中被重写的方法：
   
   ```java
   class Parent {
       public void say() {
           System.out.println("父类的方法");
       }
   }
   
   class Child extends Parent {
       @Override
       public void say() {
           super.say();  // 调用父类的say()
           System.out.println("子类的方法");
       }
   }
   ```

3. **调用父类的构造方法**  
   用 `super(参数)` 在子类构造方法中调用父类的构造方法（必须放在子类构造方法的第一行）：
   ```java
   class Parent {
       public Parent(String msg) {
           System.out.println("父类构造：" + msg);
       }
   }
   
   class Child extends Parent {
       public Child() {
           super("Hello");  // 调用父类带参构造
       }
   }
   ```

##### **`super` 与 `this` 的核心区别**

| 维度         | `super`                          | `this`                          |
|--------------|----------------------------------|---------------------------------|
| **指向对象** | 代表**父类的实例引用**           | 代表**当前类的实例引用**        |
| **访问成员** | 访问父类的成员（变量/方法）      | 访问当前类的成员（变量/方法）   |
| **构造调用** | `super(...)` 调用父类构造方法    | `this(...)` 调用当前类其他构造  |
| **使用场景** | 子类中需要访问父类的被隐藏/重写成员 | 当前类中区分同名的局部变量与实例变量 |
| **限制**     | 不能在静态方法中使用             | 不能在静态方法中使用            |

##### **关键总结5**

- `super` 的本质是**向上追溯父类的成员**，解决子类与父类的成员命名冲突或复用父类逻辑。
- `this` 的本质是**明确当前对象的自身引用**，解决当前类中局部变量与实例变量的命名冲突。
- 两者都不能在静态方法中使用（静态方法属于类，不依赖具体实例）。

例如：在子类中，`super.xxx` 表示“父类的xxx”，`this.xxx` 表示“子类自己的xxx”，清晰区分了继承关系中的成员归属。

#### **7.Java 中的 `abstract` 关键字有什么作用？它可以修饰哪些元素？**

- **`abstract` 修饰类**：表示该类不能实例化，必须被继承。
- **`abstract` 修饰方法**：表示该方法没有实现，必须在子类中实现。

`abstract`用于声明"抽象"的类或方法，核心目的是**定义规范但不提供具体实现**，强制子类按照规范进行实现，体现面向对象的"抽象"特性。

##### 1. 修饰类（抽象类）

- **特点**：
  - 抽象类不能被直接实例化（`new AbstractClass()`会报错），必须通过子类继承并实例化子类；
  - 抽象类可以包含普通方法（有实现）和抽象方法（无实现）；
  - 抽象类可以有构造方法（供子类调用）；
  - 子类如果继承抽象类，必须实现所有抽象方法（除非子类也是抽象类）。

- **示例**：
```java
// 抽象类
abstract class Animal {
    // 普通方法（有实现）
    public void breathe() {
        System.out.println("呼吸空气");
    }
    
    // 抽象方法（无实现，仅定义规范）
    public abstract void eat();
}

// 子类必须实现抽象方法
class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }
}
```

##### 2. 修饰方法（抽象方法）

- **特点**：
  - 抽象方法没有方法体（只有声明，以`;`结尾）；
  - 抽象方法必须存在于抽象类中；
  - 子类必须重写父类的所有抽象方法（否则子类必须声明为抽象类）；
  - 抽象方法不能被`private`、`static`、`final`修饰（这些关键字与"必须被重写"矛盾）。

- **示例**：
```java
abstract class Shape {
    // 抽象方法：定义"计算面积"的规范
    public abstract double calculateArea();
}

class Circle extends Shape {
    private double radius;
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius; // 实现具体逻辑
    }
}
```

##### 注意点

- `abstract`不能修饰变量、构造方法或代码块；
- 抽象类的意义在于**封装共性、定义规范**，子类则负责实现具体细节（如"动物"是抽象概念，"狗"、"猫"是具体实现）；
- 抽象类与接口的区别：抽象类可以有普通方法和实例变量，而接口（Java 8前）只能有抽象方法和常量。

总结：`abstract`通过"抽象类+抽象方法"的组合，实现了"规范与实现分离"，是面向对象设计中实现多态的重要机制。

#### **`8.native` 关键字是用来做什么的？使用 `native` 方法有什么优缺点？**

`native` 关键字表示方法是用其他语言（通常是 C 或 C++）实现的。优点是能够调用底层操作，缺点是不可移植且调试困难。

你的理解是正确的，`native` 关键字主要用于声明由非Java语言（通常是C/C++）实现的方法，以下是更详细的说明：

##### `native` 关键字的作用

`native` 用于标记一个方法的实现不是用Java编写的，而是由其他底层语言（如C、C++）实现，并通过JNI（Java Native Interface，Java本地接口）与Java代码交互。

它的核心用途是：
- 让Java代码调用操作系统的底层功能（如硬件操作、系统调用）；
- 复用已有的C/C++库，避免重复开发；
- 优化性能敏感的代码（如数学计算、图形渲染等）。

示例：
```java
public class NativeDemo {
    // 声明native方法（无方法体）
    public native void callNativeMethod();
    
    // 加载包含native方法实现的动态链接库
    static {
        System.loadLibrary("NativeLib"); // 加载名为NativeLib的库（如.dll或.so文件）
    }
}
```

##### 使用 `native` 方法的优缺点

###### 优点：

1. **访问底层资源**：突破Java的安全限制，直接操作硬件、系统内核等底层资源（如Java IO的底层实现就大量使用native方法）。
2. **性能优化**：对于计算密集型任务（如数值计算、图形处理），C/C++实现通常比Java更快。
3. **复用现有代码**：可以直接调用成熟的C/C++库，无需用Java重写。

###### 缺点：

1. **失去跨平台性**：native方法的实现依赖具体操作系统和硬件，不同平台需重新编译，违背Java"一次编写，到处运行"的原则。
2. **安全性风险**：native方法不受Java安全管理器的限制，可能破坏JVM的内存安全，导致程序崩溃或安全漏洞。
3. **开发复杂度高**：需要掌握JNI规范和C/C++，且调试困难（Java调试器无法跟踪到native代码）。
4. **依赖外部库**：需额外管理动态链接库（如.dll、.so文件），增加部署和维护成本。

##### 典型应用场景

- Java标准库中的底层功能（如`System.currentTimeMillis()`、`Object.hashCode()`）；
- 图形界面库（如早期的AWT依赖系统原生GUI）；
- 性能敏感的组件（如游戏引擎的渲染模块）；
- 硬件交互（如驱动程序、传感器访问）。

总结：`native` 方法是Java与底层系统交互的桥梁，适合需要高性能或访问底层资源的场景，但应谨慎使用，避免破坏Java的跨平台性和安全性。

#### 示例：`String.intern()`

在Java中，`String.intern()`是一个native方法，用于将字符串添加到字符串常量池（String Pool）并返回池中的对应字符串引用，主要作用是实现字符串的复用，节省内存。

##### 工作原理

1. **字符串常量池（String Pool）**：JVM维护的一个特殊内存区域，用于存储字符串常量，避免重复创建相同内容的字符串对象。
2. **intern()的行为**：
   - 当调用`str.intern()`时，JVM会检查字符串常量池中是否存在与`str`内容相同的字符串。
   - 若存在，直接返回常量池中该字符串的引用。
   - 若不存在，将当前字符串`str`添加到常量池，并返回其引用。

##### 代码示例

```java
public class InternDemo {
    public static void main(String[] args) {
        // s1是new创建的对象，存储在堆中
        String s1 = new String("abc");
        // s2是字符串常量，直接存储在常量池
        String s2 = "abc";
        
        // 比较引用：s1指向堆对象，s2指向常量池对象，所以不相等
        System.out.println(s1 == s2); // false
        
        // 调用intern()后，s1.intern()返回常量池中的"abc"引用（即s2的引用）
        String s3 = s1.intern();
        System.out.println(s3 == s2); // true
        System.out.println(s3 == s1); // false（s1仍指向堆中对象）


        // 另一种场景：动态创建的字符串
        String s4 = new String("a") + new String("b"); // s4是堆中对象"ab"，但常量池没有"ab"
        String s5 = s4.intern(); // 将"ab"添加到常量池，返回常量池引用
        String s6 = "ab"; // 直接引用常量池中的"ab"
        
        System.out.println(s5 == s6); // true
        System.out.println(s4 == s5); // JDK 1.7+中为true（常量池存储堆对象引用）
    }
}
```

##### 关键特性

1. **节省内存**：相同内容的字符串通过`intern()`可共享常量池中的引用，避免重复创建对象。
2. **JDK版本差异**：
   - JDK 1.6及之前：常量池位于方法区（永久代），`intern()`会复制字符串到常量池。
   - JDK 1.7及之后：常量池移至堆中，`intern()`不会复制字符串，而是存储堆中对象的引用（更高效）。
3. **性能权衡**：`intern()`需要检查常量池，频繁调用可能影响性能（常量池本质是哈希表，查找有开销）。

##### 适用场景

- 需频繁使用相同字符串（如数据库表名、配置项等），通过`intern()`复用对象。
- 对内存敏感的场景（如处理大量重复字符串的日志、文本解析）。

不建议滥用：普通业务场景中，字符串常量（如`"abc"`）会自动入池，无需显式调用`intern()`。

#### **`9.synchronized` 关键字的作用是什么？它可以修饰方法和代码块，分别说明其作用。**

- **修饰方法**：表示该方法是同步的，多个线程不能同时访问该方法。
- **修饰代码块**：表示该代码块是同步的，多个线程不能同时执行该代码块。

你的理解基本正确，`synchronized` 关键字的核心作用是**实现线程同步**，防止多个线程同时访问共享资源导致的数据不一致问题。以下是更详细的说明：

在多线程环境中，当多个线程同时操作**共享资源**（如共享变量、对象）时，可能会导致数据错乱（线程不安全）。`synchronized` 通过**加锁机制**保证：同一时间只有一个线程能执行被它修饰的代码，从而实现线程间的互斥访问。

- **非静态方法**：synchronized锁定当前实例对象（this）。
- **静态方法**：synchronized锁定类的Class对象（如MyClass.class）。

##### 1. 修饰方法（同步方法）

- **作用**：将整个方法标记为同步代码，同一时间只有一个线程能进入该方法执行。
- **锁对象**：
  - 对于**非静态同步方法**，锁是当前对象（`this`）；
  - 对于**静态同步方法**，锁是当前类的字节码对象（`类名.class`）。

- **示例**：
```java
public class SynchronizedDemo {
    // 非静态同步方法（锁是当前对象this）
    public synchronized void nonStaticSyncMethod() {
        // 线程安全的操作（如修改共享变量）
    }
    
    // 静态同步方法（锁是SynchronizedDemo.class）
    public static synchronized void staticSyncMethod() {
        // 线程安全的操作（如修改静态共享变量）
    }
}
```

##### 2. 修饰代码块（同步代码块）

- **作用**：只对代码块中的内容进行同步，比同步方法更灵活（可精确控制需要同步的范围）。
- **锁对象**：可以指定任意对象作为锁（需显式传入，如 `synchronized(锁对象) { ... }`）。

- **示例**：
```java
public class SyncBlockDemo {
    private Object lock = new Object(); // 自定义锁对象
    private int count = 0; // 共享变量
    
    public void increment() {
        // 同步代码块（锁是lock对象）
        synchronized (lock) {
            count++; // 只对修改共享变量的操作同步，提高效率
        }
        // 其他无需同步的代码（可并发执行）
    }
}
```

##### 关键区别与注意事项

| 类型         | 锁对象               | 同步范围               | 灵活性       |
|--------------|----------------------|------------------------|--------------|
| 同步方法     | 非静态：this；静态：类对象 | 整个方法体             | 低（范围固定） |
| 同步代码块   | 自定义对象           | 代码块内部             | 高（可按需控制） |

- **锁的唯一性**：只有竞争**同一把锁**的线程才会互斥；不同锁对象的同步代码可以并发执行。
- **性能影响**：同步范围越大，对性能影响越大（线程等待时间长），因此推荐用同步代码块精确控制范围。
- **可重入性**：`synchronized` 是可重入锁，同一线程可以多次获取同一把锁（不会自己锁死自己）。

例如：多个线程调用同一对象的非静态同步方法时，会竞争该对象的锁，同一时间只有一个线程能执行；而调用不同对象的同步方法时，因锁对象不同，可并行执行。

总结：`synchronized` 通过控制锁的获取与释放，确保共享资源的线程安全，是Java中最基础的线程同步机制。

#### **`10.volatile` 关键字的作用是什么？它能保证线程安全吗？为什么？**

 `volatile` 关键字确保变量在多线程环境中被及时更新，避免线程缓存数据的错误。它不能完全保证线程安全，但能保证可见性，即当一个线程修改了 `volatile` 变量的值，其他线程能立即看到。

你的理解基本正确，`volatile` 是Java中用于解决多线程变量可见性问题的关键字，以下是更详细的解释：

##### `volatile` 关键字的核心作用

`volatile` 用于修饰变量，主要保证两个特性：

1. **可见性**  
   当一个线程修改了 `volatile` 变量的值，这个新值会被**立即刷新到主内存**，同时其他线程对该变量的缓存会失效，必须从主内存重新读取最新值。  
   这避免了线程因读取自己工作内存中的"旧值"而导致的数据不一致。

2. **禁止指令重排序**  
   编译器或CPU可能会对指令进行重排序优化（不改变单线程执行结果），但在多线程环境下可能导致逻辑错误。`volatile` 会禁止这种重排序，保证代码执行顺序与源码一致。

##### `volatile` 能保证线程安全吗？**不能完全保证线程安全**。

线程安全需要满足**原子性、可见性、有序性**三个条件，而 `volatile` 仅能保证**可见性和有序性**，无法保证**原子性**。

- **原子性**：指一个操作是不可分割的，要么全部执行，要么不执行。  
  `volatile` 修饰的变量如果涉及**复合操作**（如 `i++`，实际包含读取、修改、写入三步），多线程并发时仍会出现数据错乱。

  示例：
  ```java
  public class VolatileDemo {
      private volatile int count = 0;
      
      // 多线程调用此方法，结果可能小于预期值
      public void increment() {
          count++; // 非原子操作，volatile无法保证线程安全
      }
  }
  ```
  上述代码中，`count++` 不是原子操作，即使 `count` 被 `volatile` 修饰，多个线程同时执行仍可能导致计数错误。

##### 何时适合使用 `volatile`？

1. **状态标记**：用于标记线程是否需要停止、初始化是否完成等（单一操作，无复合逻辑）。  
   示例：
   
   ```java
   private volatile boolean isRunning = true;
   
   public void stop() {
       isRunning = false; // 线程1修改状态
   }
   
   public void run() {
       while (isRunning) { // 线程2能立即看到状态变化
           // 执行任务
       }
   }
   ```




2. **双重检查锁定**：在单例模式中结合 `volatile` 防止指令重排序导致的问题。

##### 总结

- `volatile` 的核心价值是**保证变量的可见性和禁止指令重排序**，但不保证原子性。
- 对于简单的状态标记（单一读/写操作），`volatile` 可以保证线程安全；
- 对于复合操作（如 `i++`、`i = j + 1` 等），`volatile` 无法保证线程安全，需配合 `synchronized` 或原子类（如 `AtomicInteger`）使用。

#### **`11.transient` 关键字有什么作用？在什么情况下需要使用 `transient` 关键字？**

 `transient` 关键字用于指示某个变量不应被序列化。常用于敏感数据（如密码）和临时数据（如缓存）不希望被持久化时。

#### **`12.instanceof` 关键字的作用是什么？它的语法格式是怎样的？使用时需要注意什么？**

 `instanceof` 用于测试一个对象是否是某个类的实例或其子类的实例。语法格式：`object instanceof ClassName`。使用时需要注意避免空指针异常。

#### **`13.import` 关键字的作用是什么？它有什么使用方式？通配符 `\*` 的使用有什么注意事项？**

 `import` 关键字用于导入其他包中的类或整个包。使用方式：

- `import package.ClassName;`
- `import package.*;`（导入包中的所有类）
   使用通配符时，不能导入子包中的类，且会增加类加载的时间。

#### **`14.package` 关键字的作用是什么？它对类的访问权限有什么影响？**

 `package` 关键字用于定义类的包，包为类提供了命名空间。包影响类的访问权限：

- 同一包中的类可以访问彼此的默认访问权限成员。
- 不同包中的类如果要访问其他类的成员，需要通过公开的访问控制符。

#### **`15.break` 和 `continue` 关键字有什么区别？它们在循环和 `switch` 语句中分别有什么作用？**

- **`break`**：跳出当前循环或 `switch` 语句。
- **`continue`**：跳过当前循环的剩余部分，直接进入下一次循环。

#### **`16.return` 关键字的作用是什么？在有返回值的方法和无返回值的方法中使用 `return` 有什么不同？**

- `return` 用于结束方法的执行并返回值。
- 在有返回值的方法中，`return` 后面必须跟随返回值，而在无返回值的方法中，`return` 不跟值，或者完全没有 `return` 语句。

#### **`17.try`、`catch`、`finally` 关键字在异常处理中分别起到什么作用？`finally` 块中的代码一定会执行吗？**

- **`try`**：用于包围可能会抛出异常的代码块。
- **`catch`**：用于捕获并处理异常。
- **`finally`**：用于执行无论是否发生异常都要执行的代码。
   `finally` 中的代码通常会执行，但如果 JVM 被强制终止（如 `System.exit()`），`finally` 中的代码可能不会执行。



```java
public static int func (){
    try{
        return 1;
    }catch (Exception e){
        return 2;
    }finally{
        return 3;
    }
}
```

**在Java中,finally块中的代码一定会执行,且会覆盖try或catch块中的返回值**。执行顺序是:
\1. 首先执行try块中的代码
\2. 如果发生异常则执行catch块
\3. 最后一定会执行finally块

需要注意的关键点：
\1. finally块中的代码一定会执行，即使try块中有return语句，此时try块中的return 的值会暂时存储起来，直到finally执行完毕。
\2. 当finally块中包含return语句时，这个return会覆盖try块中的return值
\3. finally块通常用于释放资源，一般不建议在finally块中使用return语句，因为这会导致try块中的返回值被覆盖，破坏了程序的完整性

```JAVA
public class Test {
    public static void main(String[] args) {
        System.out.println(test());
 
    }
    private static int test() {
        int temp = 1;
        try {
            System.out.println(temp);
            return ++temp;
        } catch (Exception e) {
            System.out.println(temp);
            return ++temp;
        } finally {
            ++temp;
            System.out.println(temp);
        }
    }
}
```

\1. 首先执行try块中的System.out.println(temp)，此时temp=1，所以输出1

\2. 接着执行return ++temp语句
\- ++temp使temp变为2
\- 这个返回值2会被暂存

\3. 虽然有return语句，但finally块必须执行
\- 进入finally块，执行++temp，使temp变为3
\- 输出temp值3

\4. 最终返回之前暂存的值2

所以执行顺序是：输出1，然后输出3，最后返回值2，因此完整输出为：1,3,2

#### **`18.throw` 和 `throws` 关键字有什么区别？它们在异常处理中如何配合使用？**

- **`throw`**：用于抛出异常。
- **`throws`**：用于声明方法可能抛出的异常。
   它们配合使用，`throw` 在方法内部抛出异常，`throws` 在方法声明中声明异常。

#### **`19.strictfp` 关键字的作用是什么？在什么情况下需要使用 `strictfp` 关键字？**

 `strictfp` 用于限制浮点运算的精度，以确保不同平台的浮点运算结果一致。通常在需要保证跨平台浮点精度一致时使用

##### `strictfp` 关键字的作用

`strictfp` 是 "strict floating point" 的缩写，用于**严格限制浮点运算的精度和结果**，确保同一套浮点运算代码在不同硬件平台（如CPU架构不同）和操作系统上运行时，能得到**完全一致的结果**。

##### 为什么需要 `strictfp`？

浮点运算的结果可能受硬件影响：

- 不同CPU的浮点处理单元（FPU）可能支持不同精度（如32位、64位、80位）；
- 某些平台会使用更高精度（如80位扩展精度）临时存储中间结果，导致最终结果与其他平台产生细微差异。

`strictfp` 通过强制所有浮点运算（包括中间结果）严格遵循IEEE 754标准的精度限制（32位单精度、64位双精度），消除这种平台差异。

##### 使用场景

`strictfp` 可修饰**类、接口、方法**，表示其内部的所有浮点运算（包括成员方法、嵌套类中的运算）都遵循严格的精度规范：

```java
// 修饰类：类中所有浮点运算都严格遵循规范
strictfp class MathCalculator {
    public double calculate(double a, double b) {
        return a + b * Math.sqrt(a); // 严格精度运算
    }
}

// 修饰方法：仅该方法内的浮点运算受限制
class DataProcessor {
    strictfp void process(float x, float y) {
        float result = x / y + 0.1f; // 严格精度运算
    }
}
```

##### 注意点

1. `strictfp` 不能修饰变量、构造方法或代码块；
2. 接口被 `strictfp` 修饰后，所有实现类都会继承严格浮点特性；
3. 现代JVM（如HotSpot）在多数平台上已默认保证浮点运算的一致性，因此 `strictfp` 在实际开发中较少使用；
4. 适用于**对浮点结果一致性要求极高的场景**，如科学计算、金融量化模型、跨平台的数值校验等（结果差异可能导致逻辑错误）。

##### 总结

`strictfp` 的核心价值是**强制浮点运算遵循统一精度标准**，确保跨平台结果一致。虽然日常开发中较少用到，但在需要绝对精度一致性的场景（如科学计算、金融领域）是必要的。

#### **`20.assert` 关键字的作用是什么？它的使用语法是怎样的？在什么情况下 `assert` 语句会生效？**

 `assert` 用于调试时验证程序的假设。语法：`assert expression;` 或 `assert expression : errorMessage;`
 `assert` 语句只有在 Java 启用断言时才会生效（通过 `-ea` 参数启用）。

你的总结非常准确，`assert` 关键字是Java中用于调试和验证程序假设的工具，以下是更详细的说明：

##### `assert` 关键字的作用

`assert`（断言）用于在代码中嵌入**调试阶段的验证逻辑**，检查程序运行时是否满足预期的条件（假设）。如果条件不满足，会抛出 `AssertionError` 异常，帮助开发者快速定位逻辑错误。

它的核心用途是：
- 验证方法的输入参数是否符合预期（如非空、范围合法等）；
- 检查程序执行过程中的中间状态是否正确；
- 确认不可能出现的分支（如 `switch` 的 `default` 分支）。

##### 使用语法

`assert` 有两种语法形式：

1. **简单形式**  
   ```java
   assert 布尔表达式;
   ```
   - 如果 `布尔表达式` 为 `false`，则抛出 `AssertionError`；
   - 如果为 `true`，则无任何操作。

2. **带错误信息的形式**  
   ```java
   assert 布尔表达式 : 错误信息;
   ```
   - 如果 `布尔表达式` 为 `false`，则抛出 `AssertionError` 并携带 `错误信息`（可以是任意类型的表达式，最终会转为字符串）；
   - 错误信息有助于更清晰地定位问题。

##### 示例

```java
public class AssertDemo {
    public static void calculate(int num) {
        // 断言：输入参数必须为正数（假设条件）
        assert num > 0 : "输入参数必须大于0，当前值：" + num;
        
        // 业务逻辑
        System.out.println("计算结果：" + num * 2);
    }
    
    public static void main(String[] args) {
        calculate(5);   // 正常执行（5>0）
        calculate(-3);  // 触发断言失败，抛出AssertionError
    }
}
```

##### `assert` 语句生效的条件

`assert` 语句**默认是关闭的**（不会执行），必须通过JVM参数手动启用：
- 启用断言：`java -ea 类名` 或 `java -enableassertions 类名`；
- 禁用断言（默认）：`java -da 类名` 或 `java -disableassertions 类名`；
- 对特定类或包启用：`java -ea:com.example.MyClass 类名`（仅启用com.example.MyClass的断言）。

##### 注意事项

1. **不能替代异常处理**：断言用于调试阶段验证内部逻辑，不应在生产环境依赖它处理错误（因为默认关闭）。生产环境的参数校验应使用异常（如 `IllegalArgumentException`）。
2. **性能影响**：启用断言可能对性能有轻微影响，但关闭时无任何开销。
3. **表达式副作用**：断言的布尔表达式不应包含修改程序状态的操作（如 `assert (i++) > 0;`），因为关闭断言时这些操作不会执行，可能导致逻辑不一致。

##### 总结

`assert` 是调试阶段的辅助工具，通过验证程序假设帮助开发者发现潜在问题，但其生效依赖JVM的 `-ea` 参数，且不能替代正式的异常处理机制。合理使用可以提高代码的可维护性和健壮性。

#### 21.静态代码块和普通代码块

在Java中，静态代码块（Static Block）和普通代码块（Instance Block，也叫实例代码块）都是用于初始化的代码块，但它们的执行时机、作用范围和使用场景有明显区别：

##### 1. **静态代码块（Static Block）**

- **语法**：用 `static` 关键字修饰的代码块  
  ```java
  static {
      // 初始化代码
  }
  ```

- **执行时机**：  
  类被加载到JVM时**自动执行**（仅执行一次），优先于构造方法和普通代码块。  
  多个静态代码块按**定义顺序**依次执行。

- **作用范围**：  
  只能访问类的**静态成员**（静态变量、静态方法），不能访问实例成员（非静态变量、非实例方法）。

- **主要用途**：  
  对类的静态成员进行初始化，或执行仅需在类加载时运行一次的逻辑（如加载配置文件、注册驱动等）。

##### 2. **普通代码块（Instance Block）**

- **语法**：没有 `static` 修饰的代码块  
  ```java
  {
      // 初始化代码
  }
  ```

- **执行时机**：  
  每次创建类的**实例对象**时**自动执行**（创建一次对象执行一次），执行在构造方法之前。  
  多个普通代码块按**定义顺序**依次执行。

- **作用范围**：  
  可以访问类的**所有成员**（静态成员和实例成员）。

- **主要用途**：  
  初始化实例变量，或封装多个构造方法中重复的代码逻辑（提取公共初始化步骤）。

##### 3. **对比示例**

```java
public class BlockDemo {
    // 静态变量
    static int staticVar;
    // 实例变量
    int instanceVar;

    // 静态代码块
    static {
        staticVar = 10;
        System.out.println("静态代码块执行，staticVar = " + staticVar);
        // 不能访问实例变量：System.out.println(instanceVar); // 编译报错
    }

    // 普通代码块
    {
        instanceVar = 20;
        staticVar = 30; // 可以访问静态变量
        System.out.println("普通代码块执行，instanceVar = " + instanceVar + "，staticVar = " + staticVar);
    }

    // 构造方法
    public BlockDemo() {
        System.out.println("构造方法执行");
    }

    public static void main(String[] args) {
        System.out.println("第一次创建对象：");
        new BlockDemo();

        System.out.println("\n第二次创建对象：");
        new BlockDemo();
    }
}
```

**输出结果**：
```
静态代码块执行，staticVar = 10  // 类加载时执行一次
第一次创建对象：
普通代码块执行，instanceVar = 20，staticVar = 30  // 每次创建对象前执行
构造方法执行
第二次创建对象：
普通代码块执行，instanceVar = 20，staticVar = 30  // 再次创建对象时再次执行
构造方法执行
```

##### 4. **核心区别总结**

| 特性         | 静态代码块                  | 普通代码块                  |
|--------------|-----------------------------|-----------------------------|
| 修饰符       | 必须用 `static`             | 无 `static`                 |
| 执行时机     | 类加载时执行（仅一次）      | 每次创建实例时执行（多次）  |
| 执行顺序     | 早于普通代码块和构造方法    | 晚于静态代码块，早于构造方法|
| 访问权限     | 只能访问静态成员            | 可访问静态和实例成员        |
| 主要作用     | 初始化静态成员              | 初始化实例成员或提取构造共性|

理解两者的区别有助于在合适的场景选择正确的初始化方式，尤其是在需要控制初始化时机或复用初始化逻辑时。

#### 22.枚举

在Java中，**枚举（Enumeration，简称Enum）** 是一种特殊的数据类型，用于定义一组固定的常量集合（如季节、星期、状态码等），它提供了比传统常量（`public static final`）更安全、更具可读性的实现方式。

在 Java 中，**枚举（enum）类型**可以包含以下内容：
**正确答案：枚举常量、枚举构造函数、枚举方法**

##### 一、枚举的基本定义

枚举通过 `enum` 关键字定义，格式如下：
```java
// 定义一个表示季节的枚举
enum Season {
    SPRING, SUMMER, AUTUMN, WINTER; // 枚举常量，用逗号分隔，末尾可加封号
}
```
- 枚举常量默认是 `public static final` 的，且类型为当前枚举类（如 `Season.SPRING` 的类型是 `Season`）。
- 枚举类是**最终类（final）**，不能被继承，也不能派生子类。
- 枚举类的构造方法默认是 `private` 的，只能在枚举类内部调用（用于初始化常量）。

##### 二、枚举的核心特性

1. **固定的常量集合**  
   枚举的常量是预定义的，数量固定，编译期即可确定，避免了非法值的传入（比 `int` 常量更安全）。  
   例如：用 `Season` 枚举参数时，只能传入 `SPRING`/`SUMMER` 等，不能传入其他值。

2. **本质是类**  
   枚举本质上是 `java.lang.Enum` 的子类，自动继承 `Enum` 类的方法（如 `name()`、`ordinal()` 等），同时可以：
   - 定义成员变量和方法；
   - 实现接口；
   - 拥有构造方法（必须是 `private`）。

   示例（带成员和方法的枚举）：
   ```java
   enum Weekday {
       MONDAY("周一", 1),
       TUESDAY("周二", 2),
       // ... 其他常量

       // 成员变量
       private String chineseName;
       private int index;

       // 私有构造方法（初始化常量）
       private Weekday(String chineseName, int index) {
           this.chineseName = chineseName;
           this.index = index;
       }

       // 自定义方法
       public String getChineseName() {
           return chineseName;
       }
   }

   // 使用
   public class Test {
       public static void main(String[] args) {
           System.out.println(Weekday.MONDAY.getChineseName()); // 输出：周一
       }
   }
   ```


3. **常用方法（继承自 `Enum` 或自身定义）**  
   
   | 方法 | 功能描述 |
   |------|----------|
   | `String name()` | 返回枚举常量的名称（如 `MONDAY`） |
   | `int ordinal()` | 返回枚举常量的序号（从0开始，如 `MONDAY` 是0，`TUESDAY` 是1） |
   | `static Enum valueOf(Class enumType, String name)` | 根据名称获取枚举常量（如 `Weekday.valueOf("MONDAY")`） |
| `static Enum[] values()` | 返回所有枚举常量的数组（自动生成，用于遍历） |
   
   示例（遍历枚举）：
   ```java
   for (Weekday day : Weekday.values()) {
       System.out.println(day.name() + " -> " + day.getChineseName());
   }
   ```

##### 三、枚举的优势

1. **类型安全**  
   避免了使用 `int` 或 `String` 常量时的类型混淆（如 `int` 可能传入非法值，`String` 可能拼写错误）。

2. **可读性高**  
   枚举常量名本身具有语义（如 `SPRING` 比 `1` 更易理解），代码更清晰。

3. **功能丰富**  
   支持自定义方法、实现接口，可封装与常量相关的逻辑（如示例中通过 `getChineseName()` 获取中文名称）。

4. **便于维护**  
   常量集中管理，修改或新增常量时只需修改枚举类，无需全局查找。

##### 四、常见使用场景

1. **定义固定选项**：如季节、星期、性别、订单状态（待支付、已发货、已完成）等。  
2. **替代单例模式**：枚举是实现单例的最佳方式之一（线程安全、防止反射和序列化破坏）。  
   ```java
   enum Singleton {
       INSTANCE; // 唯一实例
       public void doSomething() { ... }
   }
   // 使用：Singleton.INSTANCE.doSomething();
   ```
3. **switch语句**：枚举是 `switch` 的理想候选，避免了 `case` 后的魔法数字。  
   ```java
   Season season = Season.SPRING;
   switch (season) {
       case SPRING: System.out.println("春暖花开"); break;
       case SUMMER: System.out.println("夏日炎炎"); break;
       // ...
   }
   ```

##### 五、注意事项

- 枚举常量必须在枚举类的第一行声明，否则编译报错。
- 枚举类不能被 `abstract` 修饰，但可以包含抽象方法（此时每个枚举常量必须实现该抽象方法）。  
  示例：
  ```java
  enum Operation {
      ADD {
          public int calculate(int a, int b) { return a + b; }
      },
      SUBTRACT {
          public int calculate(int a, int b) { return a - b; }
      };

      public abstract int calculate(int a, int b); // 抽象方法
  }
  ```


总之，枚举是Java中一种高效、安全的常量定义方式，尤其适合表示固定且有限的取值集合，在实际开发中应用广泛（如状态码、配置选项、策略模式等）。

#### 23.new


在Java中，`new` 关键字主要用于**创建对象**和**实例化数组**，是实现对象初始化和内存分配的核心操作。其使用方式可分为以下几类：

##### 一、创建类的实例对象（最常用）

通过 `new 构造方法()` 语法创建类的实例，会触发以下操作：  
1. 为对象分配内存空间；  
2. 调用类的构造方法初始化对象（给成员变量赋值等）；  
3. 返回对象的引用（地址）。

###### 语法：

```java
类名 对象名 = new 构造方法(参数列表);
```

###### 示例：

```java
// 定义一个类
class Person {
    String name;
    int age;

    // 无参构造
    public Person() {}

    // 有参构造
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

// 创建对象
public class Test {
    public static void main(String[] args) {
        // 调用无参构造
        Person p1 = new Person(); 
        p1.name = "张三";

        // 调用有参构造
        Person p2 = new Person("李四", 20); 
    }
}
```

###### 注意：

- 若类中没有显式定义构造方法，编译器会自动生成**无参构造方法**；  
- 若显式定义了构造方法，编译器不再生成无参构造，需手动定义（否则无法用 `new 类名()` 创建对象）。

##### 二、实例化数组

`new` 可用于创建数组对象（包括基本类型数组和引用类型数组），分配数组内存并指定长度。

###### 1. 一维数组

语法：
```java
数据类型[] 数组名 = new 数据类型[长度]; // 仅分配空间，默认初始化
// 或
数据类型[] 数组名 = new 数据类型[]{元素1, 元素2, ...}; // 分配空间并初始化元素
```

示例：
```java
// 基本类型数组
int[] nums = new int[3]; // 长度为3的int数组，默认值为0
nums[0] = 10;

// 引用类型数组（存储对象地址）
Person[] persons = new Person[2]; // 长度为2的Person数组，默认值为null
persons[0] = new Person("张三", 18); // 为数组元素赋值（创建Person对象）

// 简化初始化（省略new，编译器自动处理）
String[] names = {"Alice", "Bob"}; 
```

###### 2. 多维数组（以二维数组为例）

Java的多维数组本质是“数组的数组”，`new` 可分别为外层数组和内层数组分配空间。

语法：
```java
数据类型[][] 数组名 = new 数据类型[外层长度][内层长度]; // 固定内层长度
// 或
数据类型[][] 数组名 = new 数据类型[外层长度][]; // 先分配外层，内层动态分配
```

示例：
```java
// 二维int数组，3行2列
int[][] matrix = new int[3][2];
matrix[0][0] = 1;

// 内层长度不固定（锯齿数组）
int[][] jagged = new int[2][];
jagged[0] = new int[3]; // 第1行长度3
jagged[1] = new int[5]; // 第2行长度5
```

##### 三、匿名对象（直接使用`new`创建的对象，无变量引用）

若对象仅需使用一次，可省略变量名，直接通过 `new` 创建“匿名对象”，常用于临时调用方法或作为参数传递。

示例：
```java
// 匿名对象调用方法
new Person("临时用户", 0).sayHello(); 

// 匿名对象作为参数
public static void printPerson(Person p) {
    System.out.println(p.name);
}
printPerson(new Person("参数对象", 20)); // 直接传入匿名对象
```

特点：使用后无引用指向，会被垃圾回收器回收，适合单次操作。

##### 四、注意事项

1. **内存分配**：`new` 会在**堆内存**中为对象/数组分配空间，变量存储的是堆内存的引用（地址）。  
2. **构造方法必须存在**：创建对象时，`new` 后面的构造方法必须在类中存在（否则编译报错）。  
3. **不能用于抽象类/接口**：抽象类和接口不能被实例化，因此 `new 抽象类()` 或 `new 接口()` 会报错（需通过子类实现）。  
   反例：`new AbstractClass(); // 错误，抽象类不能实例化`  
4. **与`Class.newInstance()`的区别**：`new` 是关键字，编译期确定类型；`Class.newInstance()` 是反射方法，运行时动态创建对象，已被 `Constructor.newInstance()` 替代。

##### 总结

`new` 关键字的核心作用是**分配内存并初始化**，主要用于：  
- 创建类的实例（调用构造方法）；  
- 初始化一维/多维数组；  
- 创建匿名对象（临时使用）。  

它是Java中实现“面向对象”的基础操作，所有对象（除字符串常量池等特殊情况）的创建都依赖 `new` 或其底层机制（如反射、序列化）。

#### 24.`try-with-resources` 和 `try-catch-finally` 

`try-with-resources` 和 `try-catch-finally` 都是 Java 中处理异常和资源释放的机制，但两者在**资源管理方式**、**代码简洁性**和**安全性**上有显著区别。

##### 核心区别对比

| 特性                | `try-with-resources`                          | `try-catch-finally`                          |
|---------------------|-----------------------------------------------|-----------------------------------------------|
| **适用场景**        | 主要用于自动释放**实现了 `AutoCloseable` 接口**的资源（如文件流、数据库连接等）。 | 可用于释放任何资源，但需手动编写释放逻辑。               |
| **资源释放方式**    | 自动释放：资源在 `try` 块结束后（无论正常退出还是异常退出）由 JVM 自动调用 `close()` 方法。 | 手动释放：需在 `finally` 块中显式调用资源的 `close()` 或释放方法。 |
| **代码简洁性**      | 代码更简洁，无需手动编写 `finally` 块和释放逻辑。          | 代码较繁琐，需额外编写 `finally` 块，且需处理释放过程中可能的异常。 |
| **异常屏蔽问题**    | 不会屏蔽原始异常（资源释放时的异常会被抑制，可通过 `getSuppressed()` 获取）。 | 可能屏蔽原始异常（若 `finally` 块中抛出异常，会覆盖 `try` 块的异常）。 |
| **资源声明位置**    | 资源必须在 `try` 后的括号中声明（`try (资源声明) { ... }`）。 | 资源可在 `try` 块外声明，也可在 `try` 块内声明。           |

##### 1. 资源释放机制

- **`try-with-resources`**：  
  要求资源必须实现 `AutoCloseable` 接口（该接口只有一个 `close()` 方法）。当 `try` 块执行完毕（无论正常结束、抛出异常还是执行 `return`），JVM 会自动调用资源的 `close()` 方法释放资源，无需手动干预。

  示例（自动释放文件流）：
  ```java
  // 资源（BufferedWriter）在try括号中声明，自动释放
  try (BufferedWriter writer = new BufferedWriter(new FileWriter("test.txt"))) {
      writer.write("hello");
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```


- **`try-catch-finally`**：  
  资源释放需手动在 `finally` 块中编写代码（如调用 `close()` 方法）。若忘记释放或释放逻辑有误，可能导致资源泄露（如文件句柄未关闭、数据库连接未释放）。

  示例（手动释放文件流）：
  ```java
  BufferedWriter writer = null;
  try {
      writer = new BufferedWriter(new FileWriter("test.txt"));
      writer.write("hello");
  } catch (IOException e) {
      e.printStackTrace();
  } finally {
      // 手动释放资源，需判断资源是否为null，避免空指针
      if (writer != null) {
          try {
              writer.close(); // close()可能抛出异常，需嵌套try-catch
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

##### 2. 异常处理差异

- **`try-with-resources` 抑制次要异常**：  
  若 `try` 块抛出异常，同时资源的 `close()` 方法也抛出异常，`close()` 的异常会被标记为“被抑制的异常”（suppressed exception），不会覆盖原始异常。可通过 `e.getSuppressed()` 获取被抑制的异常，便于排查问题。

- **`try-catch-finally` 可能屏蔽原始异常**：  
  若 `try` 块抛出异常，且 `finally` 块中也抛出异常，最终只会保留 `finally` 块的异常，原始异常被丢失，增加调试难度。

  示例（异常屏蔽问题）：
  ```java
  try {
      int i = 1 / 0; // 抛出ArithmeticException
  } finally {
      throw new RuntimeException("finally异常"); // 覆盖原始异常
  }
  // 最终只会抛出"finally异常"，原始异常被屏蔽
  ```

##### 3. 代码简洁性

- `try-with-resources` 无需编写 `finally` 块和繁琐的资源释放逻辑，代码更短且可读性更高。
- `try-catch-finally` 需手动处理资源释放，还需判断资源是否为 `null` 及处理 `close()` 可能的异常，代码冗余。

##### 4. 适用场景

- **优先使用 `try-with-resources`**：当操作实现 `AutoCloseable` 接口的资源（如 `InputStream`、`OutputStream`、`Connection`、`Statement` 等）时，这是更安全、简洁的方式。
- **使用 `try-catch-finally`**：  
  - 处理未实现 `AutoCloseable` 接口的资源（如某些自定义资源）。
  - 需要在 `finally` 块中执行其他清理逻辑（如日志记录、状态重置等）。


##### 总结
`try-with-resources` 是 Java 7 引入的改进机制，专为资源管理设计，通过自动释放资源减少人为错误，同时避免异常屏蔽问题，代码更简洁。而 `try-catch-finally` 更通用，但在资源释放场景下较繁琐且易出错。实际开发中，对文件、数据库连接等资源，应优先使用 `try-with-resources`。

#### 25.执行顺序1

 我来详细梳理 Java 中这些成员的执行顺序，这是面试高频考点：

---

##### 执行顺序总览（由父及子，先静后动）

```
父类静态代码块/变量 → 子类静态代码块/变量 → 
父类普通代码块 → 父类构造方法 → 
子类普通代码块 → 子类构造方法
```

---

##### 详细执行顺序（同一类内）

| 优先级 | 成员                  | 执行时机         | 特点                                  |
| :----: | --------------------- | ---------------- | ------------------------------------- |
| **1**  | `static` 变量显式赋值 | 类加载时         | 按代码书写顺序                        |
| **2**  | `static` 代码块       | 类加载时         | 按代码书写顺序，与静态变量同级        |
| **3**  | `final static` 常量   | 编译期或类加载时 | 基本类型/字符串在常量池，不触发类加载 |
| **4**  | 普通变量显式赋值      | 实例化时         | 构造方法之前                          |
| **5**  | 普通代码块            | 实例化时         | 构造方法之前，按书写顺序              |
| **6**  | 构造方法              | 实例化时         | 最后执行                              |

---

##### 代码验证

```java
public class ExecutionOrder {
    // 1. 静态变量
    static String staticVar = print("静态变量");
    
    // 2. 静态代码块
    static {
        print("静态代码块");
    }
    
    // 3. final static 常量（编译期确定）
    static final String FINAL_CONST = "final常量(编译期)";
    
    // 4. 普通变量
    String normalVar = print("普通变量");
    
    // 5. 普通代码块
    {
        print("普通代码块");
    }
    
    // 6. 构造方法
    public ExecutionOrder() {
        print("构造方法");
    }
    
    static String print(String msg) {
        System.out.println(msg);
        return msg;
    }
    
    public static void main(String[] args) {
        System.out.println("=== 第一次实例化 ===");
        new ExecutionOrder();
        System.out.println("=== 第二次实例化 ===");
        new ExecutionOrder();  // 静态部分不再执行
    }
}
```

**输出结果：**
```
静态变量
静态代码块
=== 第一次实例化 ===
普通变量
普通代码块
构造方法
=== 第二次实例化 ===
普通变量
普通代码块
构造方法
```

---

##### 继承场景（重点 ⚠️）

```java
class Parent {
    static { System.out.print("1.父静块 "); }
    { System.out.print("3.父普块 "); }
    public Parent() { System.out.print("5.父构造 "); }
}

class Child extends Parent {
    static { System.out.print("2.子静块 "); }
    { System.out.print("4.子普块 "); }
    public Child() { 
        // 隐含 super()
        System.out.print("6.子构造 "); 
    }
}

// new Child() 输出：
// 1.父静块 2.子静块 3.父普块 5.父构造 4.子普块 6.子构造
```

**关键规律：**
```
先静后动（先静态后实例）
先父后子（先父类后子类）
先块后构（代码块在构造方法前）
```

---

##### 特殊场景对比

###### 1. `final static` 的陷阱

```java
public class Test {
    // 编译期常量：不触发类加载
    static final int A = 1;  
    
    // 运行期常量：触发类加载（需要计算）
    static final int B = new Random().nextInt();  
    
    static {
        System.out.println("静态代码块执行");  // 访问A时不执行，访问B时执行
    }
}
```

###### 2. 静态成员相互依赖

```java
public class Test {
    static {
        // System.out.println(a);  // ❌ 非法前向引用
        System.out.println(b);     // ✅ 可以访问后面定义的（但值为0）
    }
    
    static int a = 1;
    static int b = 2;
}
```

---

##### 记忆口诀

> **"由父及子，先静后动；变量块同级，构造最后行"**

| 阶段   | 执行内容                  | 次数          |
| ------ | ------------------------- | ------------- |
| 类加载 | 父静 → 子静               | **仅1次**     |
| 实例化 | 父普 → 父构 → 子普 → 子构 | 每次new都执行 |

#### 25.执行顺序2

 这是一个非常精细的问题！让我彻底厘清这三者的区别和执行顺序：

---

##### 核心结论

```
执行优先级：编译期常量 > static变量/代码块 > 运行期常量
```

但**实际执行顺序取决于"何时触发类加载"**。

---

##### 三类常量对比

| 类型                        | 写法                              | 存储位置          | 触发类加载？   | 执行阶段       |
| --------------------------- | --------------------------------- | ----------------- | -------------- | -------------- |
| **static 变量**             | `static int a = 1`                | 堆（方法区）      | ✅ 是           | 类加载时初始化 |
| **final 变量**              | `final int b = 2`                 | 栈/堆（实例变量） | ❌ 否（随对象） | 实例化时       |
| **static final 编译期常量** | `static final int C = 3`          | 方法区常量池      | ❌ **否**       | 编译期确定     |
| **static final 运行期常量** | `static final int D = getValue()` | 堆（方法区）      | ✅ **是**       | 类加载时初始化 |

---

##### 关键区分：编译期 vs 运行期

###### 场景1：编译期常量（不触发类加载）

```java
public class ConstTest {
    // 编译期常量：字面量或直接计算
    static final int COMPILE_CONST = 10;
    static final String STR_CONST = "hello";
    static final int EXPRESSION = 10 * 20;
    
    static {
        System.out.println("【静态代码块】执行");  // 访问上面常量时，这行不会打印！
    }
}

// 主方法
public class Main {
    public static void main(String[] args) {
        System.out.println(ConstTest.COMPILE_CONST);  // 输出：10（无静态块输出）
        System.out.println(ConstTest.EXPRESSION);     // 输出：200（无静态块输出）
    }
}
```

**原理**：编译器直接替换为常量值，**不触发 `<clinit>()` 方法**。

---

###### 场景2：运行期常量（触发类加载）

```java
public class RuntimeConst {
    // 运行期常量：需要运行时计算
    static final int RUNTIME_CONST = new Random().nextInt();
    static final int METHOD_RESULT = getValue();
    static final String NEW_STR = new String("runtime");
    
    static {
        System.out.println("【静态代码块】执行");  // 访问这些常量时，会执行！
    }
    
    static int getValue() {
        System.out.println("【getValue】执行");
        return 100;
    }
}

// 主方法
public class Main {
    public static void main(String[] args) {
        System.out.println(RuntimeConst.RUNTIME_CONST);  
        // 输出顺序：
        // 【静态代码块】执行
        // 【getValue】执行  
        // 随机数
    }
}
```

---

##### 完整执行顺序（类加载时）

当**必须触发类加载**时，执行顺序如下：

```java
public class ExecutionOrder {
    // 1. static 变量（按书写顺序）
    static int staticVar = print("static变量");
    
    // 2. static 代码块（按书写顺序，与static变量同级）
    static {
        print("static代码块");
    }
    
    // 3. static final 运行期常量（按书写顺序，视为static变量）
    static final int RUNTIME_FINAL = print("static final运行期常量");
    
    // 4. static final 编译期常量（不触发<clinit>，但在类加载时放入常量池）
    static final int COMPILE_FINAL = 999;  // 实际不参与<clinit>执行
    
    static int print(String msg) {
        System.out.println(msg);
        return 1;
    }
}
```

**输出结果**：
```
static变量
static代码块
static final运行期常量
```

> 注意：`COMPILE_FINAL` 是编译期常量，**不参与 `<clinit>()` 方法的执行序列**。

---

##### 综合对比示例

```java
public class FinalTest {
    // 编译期常量：最早"生效"，但不触发类加载
    static final int A = 1;
    
    // static变量：类加载时初始化（<clinit>）
    static int B = 2;
    
    // static代码块
    static {
        System.out.println("static块: B=" + B + ", C=" + C);
        // 可以访问后面的static final运行期常量C（值为0，未初始化）
    }
    
    // static final运行期常量：类加载时初始化（<clinit>），按书写顺序
    static final int C = getC();
    
    // final实例变量：对象创建时初始化（<init>）
    final int D = 4;
    
    static int getC() {
        System.out.println("getC()执行");
        return 3;
    }
    
    public FinalTest() {
        System.out.println("构造方法: D=" + D);
    }
    
    public static void main(String[] args) {
        System.out.println("=== 访问编译期常量A ===");
        System.out.println(FinalTest.A);  // 不触发类加载
        
        System.out.println("\n=== 第一次new对象 ===");
        new FinalTest();  // 触发类加载 + 实例化
        
        System.out.println("\n=== 第二次new对象 ===");
        new FinalTest();  // 只实例化
    }
}
```

**输出解析**：
```
=== 访问编译期常量A ===
1                          // 无类加载输出！

=== 第一次new对象 ===
static块: B=2, C=0         // C还未初始化（前向引用为0）
getC()执行                 // 按顺序执行C的初始化
构造方法: D=4

=== 第二次new对象 ===
构造方法: D=4              // 静态部分不再执行
```

---

##### 记忆口诀

> **编译期常量直接换，不触类加载最快；**  
> **static/final运行期，随类加载按序排；**  
> **实例final跟对象，构造之前赋值来。**

| 访问方式          | 类加载？           | 典型应用           |
| ----------------- | ------------------ | ------------------ |
| `类名.编译期常量` | ❌ 否               | 配置开关、版本号   |
| `类名.运行期常量` | ✅ 是               | 随机种子、动态计算 |
| `对象.final变量`  | ❌ 否（但会实例化） | 对象常量属性       |

#### 26.代码执行顺序3

Java代码执行时机：类加载、编译期、实例化 完整解析

你总结的**编译期、类加载、实例化**是Java代码执行的核心阶段，但**不是全部**，而且它们的**顺序、作用、代码执行逻辑**完全不同。

我给你用最清晰、最通俗的方式讲透，不绕弯子👇

---

##### 一、先给结论：正确的执行顺序

Java 代码生命周期 **固定顺序**：
1. **编译期**（最开始，只做一次）
2. **类加载期**（运行时，第一次使用类时）
3. **实例化**（运行时，new 对象时）
4. **方法调用**（真正执行业务代码）

你提到的三个都对，但**缺少了最关键的「方法调用」**——这才是代码真正“干活”的阶段。

---

##### 二、每个阶段到底做什么？执行哪些代码？

###### 1. 编译期（Compile Time）

**时机**：写好 `.java` → 运行 `javac` 命令
**产物**：`.class` 字节码文件
**执行什么**：
- 语法检查
- 静态常量赋值（`static final`）
- 泛型擦除、注解处理
**不执行**：任何业务逻辑、不创建对象、不运行代码

一句话：**编译期只是把源码翻译成 JVM 能看懂的文件，不运行代码。**

---

###### 2. 类加载期（Class Loading）

**时机**：**程序运行时**，第一次用到这个类（new、访问静态变量/方法、反射）
**核心目的**：把 `.class` 加载到内存，生成 `Class` 对象
**执行代码**：

- 静态代码块 `static { ... }`
- 静态变量初始化 `static int a = 10;`

**特点**：
- 一个类**只加载一次**
- 早于实例化
- 属于**类级别**，和对象无关

---

###### 3. 实例化（Instantiation）

**时机**：`new 类名()` 的时候
**核心目的**：创建对象，分配内存
**执行代码**：
1. 成员变量初始化 `int b = 20;`
2. 构造代码块 `{ ... }`
3. 构造方法 `public 类名(){ ... }`

**特点**：
- 每 new 一次，就执行一次
- 属于**对象级别**

---

###### 4. 方法调用（Run Time，真正执行代码）

**时机**：`对象.方法()` 或 `类.静态方法()`
**这才是你写的业务代码真正运行的地方**
比如：
```java
public void test() {
    System.out.println("我真正执行了！");
}
```

---

##### 三、用一段代码验证执行顺序（最直观）

```java
public class TestOrder {
    // 1. 编译期赋值
    static final int COMPILE = 100;

    // 2. 类加载：静态变量
    static String staticVar = "静态变量";

    // 3. 类加载：静态代码块
    static {
        System.out.println("1. 静态代码块（类加载时执行）");
    }

    // 4. 实例化：成员变量
    String instanceVar = "成员变量";

    // 5. 实例化：构造代码块
    {
        System.out.println("2. 构造代码块（实例化时执行）");
    }

    // 6. 实例化：构造方法
    public TestOrder() {
        System.out.println("3. 构造方法（实例化时执行）");
    }

    // 真正执行的方法
    public void run() {
        System.out.println("4. 方法调用（真正执行业务）");
    }

    public static void main(String[] args) {
        System.out.println("--- 开始执行 main ---");
        TestOrder test = new TestOrder(); // 触发实例化
        test.run(); // 触发方法执行
    }
}
```

###### 输出结果（固定顺序）

```
--- 开始执行 main ---
1. 静态代码块（类加载时执行）
2. 构造代码块（实例化时执行）
3. 构造方法（实例化时执行）
4. 方法调用（真正执行业务）
```

---

##### 四、一句话区分三个阶段

| 阶段         | 时机       | 属于        | 执行代码                       |
| ------------ | ---------- | ----------- | ------------------------------ |
| **编译期**   | 运行前     | 源码→字节码 | 语法检查、静态常量             |
| **类加载**   | 第一次用类 | 类级别      | 静态代码块、静态变量           |
| **实例化**   | new 对象   | 对象级别    | 成员变量、构造代码块、构造方法 |
| **方法调用** | 调用方法   | 运行时      | 业务逻辑（真正执行）           |

---

##### 总结

1. **Java 代码执行阶段完整顺序**：
   **编译期 → 类加载 → 实例化 → 方法调用**
2. 你说的三个都正确，但**缺少「方法调用」**——这是代码真正运行的阶段。
3. 类加载只做**类相关**的初始化，实例化只做**对象相关**的初始化。
4. 只有**方法调用**才会执行你写的业务逻辑。

#### 27.代码执行顺序题

我给你**最清晰、最准确、死记硬背都能记住**的执行顺序，直接背这 2 套规则，Java 初始化顺序永远不会错！

##### 一、静态相关：类加载时执行

**顺序固定：**
**静态变量声明（从上到下） → 静态代码块（从上到下）**

一句话：**谁写在前面，谁先执行。**

---

##### 二、实例相关：new 对象时执行

**顺序固定：**
**成员变量声明（从上到下） → 构造代码块（从上到下） → 构造方法（最后）**

这是**铁律**，永远不变！

---

##### 三、终极总顺序（最完整版）

1. **静态变量**（从上到下）
2. **静态代码块**（从上到下）
3. **成员变量**（从上到下）
4. **构造代码块**（从上到下）
5. **构造方法**（最后）

---

##### 四、用代码验证（一看就懂）

```java
public class InitOrder {

    // 1. 静态变量
    static String staticVar = "静态变量";
    
    // 2. 静态代码块
    static {
        System.out.println(staticVar);
        System.out.println("静态代码块执行");
    }

    // 3. 成员变量
    String instanceVar = "成员变量";
    
    // 4. 构造代码块
    {
        System.out.println(instanceVar);
        System.out.println("构造代码块执行");
    }

    // 5. 构造方法
    public InitOrder() {
        System.out.println("构造方法执行");
    }

    public static void main(String[] args) {
        new InitOrder();
    }
}
```

###### 输出结果（固定不变）

```
静态变量
静态代码块执行
成员变量
构造代码块执行
构造方法执行
```

---

##### 五、超级好记口诀（背会就满分）

**静变 → 静块 → 成变 → 成块 → 构造**

---

##### 总结

1. **静态**：变量先于代码块，**自上而下**
2. **实例**：变量先于代码块，代码块先于构造方法
3. **静态永远先于实例执行**

需要我给你出一道**继承场景**的顺序题吗？那是面试必考难点！

#### 28.代码执行顺序题（继承）

这是**Java面试必考、最容易错**的继承初始化顺序题，包含父类+子类、静态+实例，做完彻底搞懂！

##### 一、面试题目

请写出以下代码运行后的**输出顺序**：
```java
class Father {
    // 父类静态变量
    static String fatherStaticVar = "父类静态变量";
    // 父类成员变量
    String fatherInstanceVar = "父类成员变量";

    // 父类静态代码块
    static {
        System.out.println(fatherStaticVar);
        System.out.println("父类静态代码块");
    }

    // 父类构造代码块
    {
        System.out.println(fatherInstanceVar);
        System.out.println("父类构造代码块");
    }

    // 父类构造方法
    public Father() {
        System.out.println("父类构造方法");
    }
}

class Son extends Father {
    // 子类静态变量
    static String sonStaticVar = "子类静态变量";
    // 子类成员变量
    String sonInstanceVar = "子类成员变量";

    // 子类静态代码块
    static {
        System.out.println(sonStaticVar);
        System.out.println("子类静态代码块");
    }

    // 子类构造代码块
    {
        System.out.println(sonInstanceVar);
        System.out.println("子类构造代码块");
    }

    // 子类构造方法
    public Son() {
        System.out.println("子类构造方法");
    }
}

public class TestExtendOrder {
    public static void main(String[] args) {
        new Son();
    }
}
```

---

##### 二、先自己思考30秒，再看答案！

###### 正确输出顺序（固定不变）

```
父类静态变量
父类静态代码块
子类静态变量
子类静态代码块
父类成员变量
父类构造代码块
父类构造方法
子类成员变量
子类构造代码块
子类构造方法
```

---

##### 三、继承场景**铁律规则**（背会=满分）

###### 核心口诀：

**先静态，后实例；先父类，后子类**

###### 详细执行顺序（死记版）

1. **父类静态**（变量→代码块）
2. **子类静态**（变量→代码块）
3. **父类实例**（变量→构造代码块→构造方法）
4. **子类实例**（变量→构造代码块→构造方法）

---

##### 四、关键考点总结

1. **静态只执行1次**：类加载时运行，优先于所有实例代码
2. **继承必须先父后子**：创建子类对象，必须先初始化父类
3. **实例顺序不变**：变量 → 构造代码块 → 构造方法

---

##### 终极一句话总结

**静父 → 静子 → 实父 → 实子**
（静态父类 → 静态子类 → 实例父类 → 实例子类）

需要我给你出一道**带静态方法/成员方法调用**的进阶坑题吗？难度更高，面试更常考！

#### 29、String 为什么是不可变的？

- String 底层用 **private final char[] value** 存储字符，数组被 final 修饰，地址不能改。
- String 类本身被 final 修饰，不能被继承，无法通过子类破坏不可变性。
- 没有提供修改字符数组的方法，每次修改都会**生成新 String 对象**。
- 好处：线程安全、可以缓存哈希值、安全用作常量池键。

一句话：**底层 final 字符数组 + 类 final + 无修改入口 = 不可变。**

---

#### 30、String s = new String("xyz") 会创建几个对象？

**2 个。**
1. 字符串常量池中创建 `"xyz"` 对象（如果池中没有）。
2. `new String()` 在堆中创建一个新的 String 对象。

---

#### 31、说说你对字符串常量池的理解？

- 字符串常量池是 JVM 为节省内存设计的**缓存区域**。
- JDK 1.7 及以后，常量池从方法区移到**堆**中。
- 直接双引号 `String s = "abc"` 优先从常量池取，没有就创建并缓存。
- `new String()` 一定在堆创建新对象，不会直接用池中的。
- 目的：**节省内存、减少重复对象、提升性能**。

---

#### 32、String、StringBuffer、StringBuilder 的区别？

- **String**：不可变，每次修改生成新对象，效率低，线程安全。
- **StringBuffer**：可变，线程安全（有 synchronized），效率较低。
- **StringBuilder**：可变，**非线程安全**，效率最高。

使用场景：
- 少量操作：用 **String**
- 单线程大量拼接：**StringBuilder**
- 多线程：**StringBuffer**

---

#### 33、谈谈你对 this 关键字的理解？

this 代表**当前对象本身**。
三种用法：

1. **this.成员变量**：区分成员变量和局部变量重名。
2. **this.成员方法**：调用本类其他方法（可省略）。
3. **this()**：调用本类其他构造方法，**必须放第一行**。

---

#### 34、谈谈你对 super 关键字的理解？

super 代表**当前对象的父类引用**。
三种用法：
1. **super.成员变量**：访问父类成员变量。
2. **super.成员方法**：调用父类被重写的方法。
3. **super()**：调用父类构造方法，**必须放第一行**。

---

#### 35、this 和 super 有什么区别？

- this：访问**本类**成员、调用本类构造。
- super：访问**父类**成员、调用父类构造。
- this() 和 super() **不能同时出现**在同一个构造里。
- 都不能在 static 环境中使用。

---

#### 36、谈谈你对 static 关键字的理解？

static 表示**属于类，不属于对象**，随类加载而存在。
用法：

- **静态变量**：全类共享，一份内存。
- **静态方法**：直接用类名调用，不能访问非静态成员。
- **静态代码块**：类加载时执行一次，用于初始化。
- 静态不能使用 this/super。

---

#### 37、谈谈你对 final 关键字的理解？

final 表示**最终、不可改变**。
- 修饰**类**：类不能被继承。
- 修饰**方法**：方法不能被重写。
- 修饰**变量**：变量变为常量，只能赋值一次。
- final 和 abstract **不能一起用**（矛盾）。

---

要不要我把 **1–50 题全部整理成一页可打印的背诵版**？你直接背就能应付笔试面试。

### 四、运算符与表达式

#### **1.Java 中的算术运算符有哪些？它们的优先级是怎样的？**

Java 中的算术运算符有：`+`（加）、`-`（减）、`*`（乘）、`/`（除）、`%`（取余）。
优先级顺序（从高到低）：

1. `*`, `/`, `%`
2. `+`, `-`

##### 一、算术运算符的分类与用法

Java 中的算术运算符主要用于数值类型（`byte`/`short`/`int`/`long`/`float`/`double`）的数学运算，核心分为 5 类，以下结合示例逐一说明：

| 运算符 | 名称       | 作用                  | 示例                  | 结果         | 注意事项                               |
| ------ | ---------- | --------------------- | --------------------- | ------------ | -------------------------------------- |
| `+`    | 加法       | 数值相加 / 字符串拼接 | `3 + 5` / `"a"+"b"`   | 8 / "ab"     | 若有字符串参与，直接拼接而非计算       |
| `-`    | 减法       | 数值相减 / 取负数     | `10 - 3` / `-5`       | 7 / -5       | 单目 `-`（取负）优先级高于双目 `-`     |
| `*`    | 乘法       | 数值相乘              | `4 * 6`               | 24           | 运算结果可能超出类型范围（需注意溢出） |
| `/`    | 除法       | 数值相除              | `10 / 3` / `10.0 / 3` | 3 / 3.333... | 整数相除会舍弃小数（仅保留整数部分）   |
| `%`    | 取余（模） | 求除法的余数          | `10 % 3` / `7 % 2`    | 1 / 1        | 余数符号与被除数一致（如 `-10%3=-1`）  |

**代码示例：**

```java
public class ArithmeticOperators {
    public static void main(String[] args) {
        // 基本算术运算
        int a = 10, b = 3;
        System.out.println("加法：" + (a + b));       // 13
        System.out.println("减法：" + (a - b));       // 7
        System.out.println("乘法：" + (a * b));       // 30
        System.out.println("整数除法：" + (a / b));   // 3（舍弃小数）
        System.out.println("浮点除法：" + (a / (double)b)); // 3.3333333333333335
        System.out.println("取余：" + (a % b));       // 1
        System.out.println("负数取余：" + (-a % b));  // -1（余数符号与被除数一致）

        // 字符串拼接（+的特殊用法）
        System.out.println("拼接：" + a + b); // 103（而非13）
    }
}
```

##### 二、运算符优先级与结合性

###### 1. 优先级规则（从高到低）

- 第一级：`*`（乘）、`/`（除）、`%`（取余）
- 第二级：`+`（加）、`-`（减）

**核心逻辑**：优先级高的运算符先执行；优先级相同则按「结合性」执行（算术运算符均为「从左到右」结合）。

###### 2. 示例验证优先级

```java
public class PriorityDemo {
    public static void main(String[] args) {
        int result = 10 + 5 * 2;  // 先算 5*2=10，再算 10+10=20
        System.out.println(result); // 输出 20

        int result2 = (10 + 5) * 2; // 括号优先级最高，先算 10+5=15，再算 15*2=30
        System.out.println(result2); // 输出 30

        int result3 = 10 / 2 + 3 * 4; // 先算 10/2=5 和 3*4=12，再算 5+12=17
        System.out.println(result3); // 输出 17
    }
}
```

###### 3. 补充：括号的作用

`()`（小括号）可以强制改变运算顺序，其优先级高于所有算术运算符，是编写复杂表达式时「避免优先级错误」的最佳方式。

---

##### 总结

1. **核心运算符**：`+`（加/拼接）、`-`（减/取负）、`*`（乘）、`/`（除，整数舍弃小数）、`%`（取余，余数符号同被除数）。
2. **优先级**：`*`/`/`/`%` > `+`/`-`；同级运算符从左到右执行，括号可强制改变顺序。
3. **易错点**：整数除法舍弃小数、`+` 遇到字符串会拼接、负数取余的符号规则。



#### **2.什么是赋值运算符？除了基本的 = 之外，还有哪些复合赋值运算符？**

**赋值运算符**：用于将右边的值赋给左边的变量。基本的赋值运算符是 `=`。
**复合赋值运算符**：

- `+=`：加法赋值
- `-=`：减法赋值
- `*=`：乘法赋值
- `/=`：除法赋值
- `%=`：取余赋值
- `&=`：按位与赋值
- `|=`：按位或赋值
- `^=`：按位异或赋值
- `<<=`：左移赋值
- `>>=`：算术右移赋值
- `>>>`：逻辑右移赋值

#### **3.比较运算符有哪些？它们的返回值类型是什么？使用 == 和 equals() 比较对象有什么区别？**

比较运算符有：

- `==`（等于）
- `!=`（不等于）
- `>`（大于）
- `<`（小于）
- `>=`（大于等于）
- `<=`（小于等于）

返回值类型：`boolean`（返回 `true` 或 `false`）。
 `==` 比较的是对象的引用地址，`equals()` 比较的是对象的内容。对于对象比较，应该使用 `equals()`。



##### 一、比较运算符的完整解析

你总结的比较运算符和返回值类型是正确的，下面从**用法、适用场景、注意事项**三个维度展开详细说明：

###### 1. 比较运算符分类与用法

Java 比较运算符（也叫关系运算符）用于判断两个值/对象的关系，所有比较运算符的返回值均为 **`boolean` 类型**（仅 `true`/`false`），核心分类如下：

| 运算符 | 名称     | 作用                 | 适用类型                                     | 示例                                            | 结果         |
| ------ | -------- | -------------------- | -------------------------------------------- | ----------------------------------------------- | ------------ |
| `==`   | 等于     | 判断值/地址是否相等  | 基本类型/引用类型                            | `10 == 10` / `new Integer(10)==new Integer(10)` | true / false |
| `!=`   | 不等于   | 判断值/地址是否不等  | 基本类型/引用类型                            | `10 != 5` / `"abc" != "def"`                    | true / true  |
| `>`    | 大于     | 判断左值是否大于右值 | 数值类型（byte/short/int/long/float/double） | `15 > 8` / `3.14 > 2.71`                        | true / true  |
| `<`    | 小于     | 判断左值是否小于右值 | 数值类型                                     | `7 < 9` / `1.5 < 1.2`                           | true / false |
| `>=`   | 大于等于 | 判断左值≥右值        | 数值类型                                     | `10 >= 10` / `5 >= 8`                           | true / false |
| `<=`   | 小于等于 | 判断左值≤右值        | 数值类型                                     | `6 <= 6` / `9 <= 3`                             | true / false |

###### 2. 关键注意事项

- **基本类型 vs 引用类型**：
  - 对**基本类型**（`int`/`double` 等），`==`/`!=` 直接比较**值**；
  - 对**引用类型**（`String`/`Integer`/自定义类等），`==`/`!=` 比较**对象的内存地址**（是否为同一个对象）。
- **数值比较的类型兼容**：`>`/`<`/`>=`/`<=` 仅支持数值类型，若用于非数值类型（如 `String`）会直接编译报错；
- **浮点数比较**：避免用 `==` 比较 `float`/`double`（因精度问题，如 `0.1 + 0.2 != 0.3`），建议用差值绝对值小于极小值（如 `1e-6`）判断。

###### **代码示例：比较运算符基础用法**

```java
public class CompareOperators {
    public static void main(String[] args) {
        // 基本类型比较（值比较）
        int a = 10, b = 10;
        double c = 0.1 + 0.2, d = 0.3;
        System.out.println(a == b); // true（基本类型比较值）
        System.out.println(c == d); // false（浮点数精度问题）
        System.out.println(Math.abs(c - d) < 1e-6); // true（正确的浮点数比较方式）

        // 引用类型比较（地址比较）
        String s1 = new String("Java");
        String s2 = new String("Java");
        System.out.println(s1 == s2); // false（两个不同对象，地址不同）
        System.out.println(s1 > s2); // 编译报错（String不能用>比较）

        // 数值比较
        System.out.println(18 >= 18); // true
        System.out.println(5.5 < 5.0); // false
    }
}
```

##### 二、`==` 与 `equals()` 比较对象的核心区别

`equals()` 是 `Object` 类的方法，所有类都继承该方法，二者的核心区别可总结为下表，结合示例深入理解：

| 维度     | `==`                                 | `equals()`                              |
| -------- | ------------------------------------ | --------------------------------------- |
| 比较本质 | 比较**内存地址**（是否为同一个对象） | 比较**对象内容/值**（需重写方法才生效） |
| 适用范围 | 基本类型（值）/引用类型（地址）      | 仅引用类型（基本类型无该方法）          |
| 默认实现 | 无（运算符）                         | `Object` 类默认等价于 `==`（比较地址）  |
| 重写规则 | 无法重写                             | 可重写（如 `String`/`Integer` 已重写）  |

###### 1. 未重写 `equals()` 的场景（自定义类）

`Object` 类的 `equals()` 源码本质是 `==`，若自定义类未重写该方法，`equals()` 仍比较地址：
```java
// 自定义类（未重写equals()）
class Person {
    private String name;
    public Person(String name) { this.name = name; }
}

public class EqualsDemo1 {
    public static void main(String[] args) {
        Person p1 = new Person("张三");
        Person p2 = new Person("张三");
        
        System.out.println(p1 == p2); // false（不同对象，地址不同）
        System.out.println(p1.equals(p2)); // false（未重写，等价于==）
    }
}
```

###### 2. 重写 `equals()` 的场景（String/包装类）

`String`、`Integer` 等系统类已重写 `equals()`，改为比较**内容/值**，这是实际开发中最常用的场景：
```java
public class EqualsDemo2 {
    public static void main(String[] args) {
        // String类（重写equals()，比较内容）
        String s1 = new String("Java");
        String s2 = new String("Java");
        System.out.println(s1 == s2); // false（地址不同）
        System.out.println(s1.equals(s2)); // true（内容相同）

        // 包装类（重写equals()，比较值）
        Integer i1 = new Integer(100);
        Integer i2 = new Integer(100);
        System.out.println(i1 == i2); // false（地址不同）
        System.out.println(i1.equals(i2)); // true（值相同）

        // 包装类缓存特殊场景（-128~127复用对象）
        Integer i3 = 100;
        Integer i4 = 100;
        System.out.println(i3 == i4); // true（缓存复用，地址相同）
        Integer i5 = 200;
        Integer i6 = 200;
        System.out.println(i5 == i6); // false（超出缓存，地址不同）
        System.out.println(i5.equals(i6)); // true（始终比较值）
    }
}
```

###### 3. 核心使用原则

- 比较**基本类型**：用 `==`（无 `equals()` 方法）；
- 比较**引用类型**：
  - 想判断是否为同一个对象 → 用 `==`；
  - 想判断内容/值是否相等 → 用 `equals()`（无论是否在缓存范围）；
- 自定义类比较内容：必须重写 `equals()`（建议同时重写 `hashCode()`，遵循“相等对象的hashCode必相等”规则）。

---

##### 总结

1. **比较运算符核心**：共6种（`==`/`!=`/`>`/`<`/`>=`/`<=`），返回值均为 `boolean`；`>`/`<`/`>=`/`<=` 仅支持数值类型，`==`/`!=` 支持所有类型（基本类型比 值，引用类型比 地址）。
2. **`==` vs `equals()`**：`==` 是运算符，比较地址（引用类型）/值（基本类型）；`equals()` 是方法，默认比地址，重写后可比内容（如 `String`/包装类）。
3. **最佳实践**：对象内容比较用 `equals()`，对象身份比较用 `==`；浮点数避免直接用 `==`，自定义类比较内容需重写 `equals()`。

#### equals()、hashCode() 与 == 核心关系详解

##### 一、三者的核心定义（先理清各自的本质）

| 符号/方法    | 归属         | 核心作用                       | 比较维度                                                  |
| ------------ | ------------ | ------------------------------ | --------------------------------------------------------- |
| `==`         | 运算符       | 判断两个变量的「身份」是否一致 | 基本类型：值；引用类型：内存地址                          |
| `equals()`   | `Object`方法 | 判断两个对象的「内容」是否相等 | 未重写：地址；重写后：内容/业务规则                       |
| `hashCode()` | `Object`方法 | 返回对象的哈希码（int值）      | 用于哈希表（如HashMap）的快速查找，本质是对象的「特征值」 |

##### 二、三者的核心关系（重点）

###### 1. `==` 与 `equals()` 的关系

- **默认规则（未重写 `equals()`）**：`equals()` 等价于 `==`，均比较对象地址；
- **重写后规则**：`equals()` 比较内容，`==` 仍比较地址（二者无必然关联）；
- **关键结论**：
  - 若 `a == b` 为 `true` → 则 `a.equals(b)` 必为 `true`（同一对象，内容必然相同）；
  - 若 `a.equals(b)` 为 `true` → `a == b` 不一定为 `true`（内容相同，地址可能不同）。

**示例验证**：

```java
String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1 == s2); // false（地址不同）
System.out.println(s1.equals(s2)); // true（内容相同）

String s3 = s1;
System.out.println(s1 == s3); // true（同一对象）
System.out.println(s1.equals(s3)); // true（内容相同）
```

###### 2. `equals()` 与 `hashCode()` 的关系（必须遵守的契约）

Java 官方规定：重写 `equals()` 时，**必须同步重写 `hashCode()`**，否则会破坏哈希集合（`HashMap`/`HashSet` 等）的正常工作，核心契约如下：
- 契约1：若 `a.equals(b)` 为 `true` → 则 `a.hashCode()` 必须等于 `b.hashCode()`（相等对象的哈希码必相等）；
- 契约2：若 `a.hashCode()` 等于 `b.hashCode()` → `a.equals(b)` 不一定为 `true`（哈希码相等，内容可能不同，即「哈希碰撞」）；
- 契约3：若 `a.equals(b)` 为 `false` → `a.hashCode()` 可以相等/不等（无强制要求）。

**反例（违反契约的后果）**：

```java
// 自定义类：仅重写equals()，未重写hashCode()
class Student {
    private String id;
    public Student(String id) { this.id = id; }

    // 重写equals()：按学号比较内容
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return Objects.equals(id, student.id);
    }
    // 未重写hashCode() → 继承Object的默认实现（按地址生成哈希码）
}

public class HashCodeDemo {
    public static void main(String[] args) {
        Student s1 = new Student("001");
        Student s2 = new Student("001");
        
        System.out.println(s1.equals(s2)); // true（内容相同）
        System.out.println(s1.hashCode() == s2.hashCode()); // false（地址不同，哈希码不同）

        // 放入HashSet（依赖hashCode()和equals()）
        HashSet<Student> set = new HashSet<>();
        set.add(s1);
        set.add(s2);
        System.out.println(set.size()); // 2（本应是1，违反契约导致去重失败）
    }
}
```

**正确示例（遵守契约）**：
```java
class Student {
    private String id;
    public Student(String id) { this.id = id; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return Objects.equals(id, student.id);
    }

    // 重写hashCode()：基于equals()的比较字段生成
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

public class HashCodeDemo {
    public static void main(String[] args) {
        Student s1 = new Student("001");
        Student s2 = new Student("001");
        
        System.out.println(s1.equals(s2)); // true
        System.out.println(s1.hashCode() == s2.hashCode()); // true（符合契约）

        HashSet<Student> set = new HashSet<>();
        set.add(s1);
        set.add(s2);
        System.out.println(set.size()); // 1（去重成功）
    }
}
```

###### 3. `==` 与 `hashCode()` 的关系

- 若 `a == b` 为 `true` → `a.hashCode()` 必等于 `b.hashCode()`（同一对象，哈希码必然相同）；
- 若 `a.hashCode()` 等于 `b.hashCode()` → `a == b` 不一定为 `true`（哈希码相同，地址可能不同）。

##### 三、实际开发中的使用场景

| 场景                                  | 推荐使用                             | 原因                                    |
| ------------------------------------- | ------------------------------------ | --------------------------------------- |
| 比较基本类型的值                      | `==`                                 | 基本类型无 `equals()`/`hashCode()` 方法 |
| 比较引用类型的内容（如String/包装类） | `equals()`                           | 重写后可可靠比较内容                    |
| 判断两个引用是否指向同一对象          | `==`                                 | 直接比较地址，效率最高                  |
| 哈希集合（HashMap/HashSet）存储对象   | 必须同时重写 `equals()`+`hashCode()` | 保证查找、去重等功能正常                |
| 自定义类比较业务相等性                | 重写 `equals()`+`hashCode()`         | 遵循Java契约，避免潜在问题              |

##### 四、包装类/字符串的特殊说明

`String`、`Integer` 等系统类已遵守上述契约：
- 重写 `equals()`：比较内容/值；
- 重写 `hashCode()`：基于内容/值生成哈希码；
- 示例：
  ```java
  String s1 = new String("Java");
  String s2 = new String("Java");
  System.out.println(s1.equals(s2)); // true
  System.out.println(s1.hashCode() == s2.hashCode()); // true
  ```

---

##### 总结

1. **核心逻辑**：`==` 比地址（基本类型比 值），`equals()` 比内容（需重写），`hashCode()` 是对象特征值（需与 `equals()` 同步重写）；
2. **关键契约**：`equals()` 相等 → `hashCode()` 必相等；`hashCode()` 相等 → `equals()` 不一定相等；
3. **最佳实践**：自定义类重写 `equals()` 时，务必同步重写 `hashCode()`，且二者基于相同的字段实现；比较对象内容用 `equals()`，判断对象身份用 `==`。

#### **4.逻辑运算符有哪些？短路与（&&）和逻辑与（&）、短路或（||）和逻辑或（|）有什么区别？**

逻辑运算符有：

- `&&`（短路与）：如果左边为 `false`，右边不会计算。
- `&`（逻辑与）：无论左边结果如何，右边都会计算。
- `||`（短路或）：如果左边为 `true`，右边不会计算。
- `|`（逻辑或）：无论左边结果如何，右边都会计算。

##### 关键总结：符号重载 逻辑与（&） 和 位与（&）区别

1. **符号相同，上下文决定含义**：遇到 `&` 时，先看运算对象 —— 是 `boolean` 则为逻辑与，是整数则为位与；
2. **逻辑与管 “条件判断”，位与管 “二进制位操作”**：两者无直接关联，不要混淆；
3. 类似的符号重载还有 **逻辑或（`|`，布尔运算）** 和 **位或（`|`，整数位运算）**，区别逻辑与位与完全一致。

#### **5.位运算符有哪些？它们的作用是什么？请举例说明位与、位或、位异或的运算过程。**

位运算符有：

- `&`（位与）：两个相同位置的位都为 1，结果才为 1。
- `|`（位或）：两个相同位置的位只要有一个为 1，结果就为 1。
- `^`（位异或）：两个相同位置的位不同，结果为 1，相同则为 0。

例子：
 `5 & 3` → `0101 & 0011 = 0001`（结果为 1）
 `5 | 3` → `0101 | 0011 = 0111`（结果为 7）
 `5 ^ 3` → `0101 ^ 0011 = 0110`（结果为 6）

#### **6.移位运算符有哪些？算术右移（>>）、逻辑右移（>>>）和左移（<<）有什么区别？**

移位运算符有：

- `<<`（左移）：将位向左移动，右边补 0。
- `>>`（算术右移）：将位向右移动，左边补符号位（正数补 0，负数补 1）。
- `>>>`（逻辑右移）：将位向右移动，左边补 0，不管符号位。不存在逻辑左移。



#### **7.三元运算符的语法格式是怎样的？它的返回值类型是如何确定的？使用三元运算符需要注意什么？**

语法格式：
`condition ? expression1 : expression2;`
返回值类型由 `expression1` 和 `expression2` 确定，两者类型必须兼容。
使用时需要注意，`? :` 运算符要求两个表达式的类型兼容，或者有相同的返回类型。

#### **8.运算符的优先级从高到低大致是怎样的？在编写表达式时，如何改变运算符的执行顺序？**

运算符优先级大致顺序：

1. 括号 `()`、数组下标 `[]`
2. 后缀自增 `++`、自减 `--`
3. 取反 `~`
4. 算术运算符 `* / % + -`
5. 比较运算符 `== != < <= > >=`
6. 逻辑运算符 `&& ||`

改变顺序：使用括号 `()` 可以改变运算符的执行顺序。

##### 一、完整的运算符优先级（从高到低，核心常用部分）

你总结的优先级框架是正确的，为了更全面且实用，这里补充完整的核心优先级顺序（仅列出编程中高频使用的类别），方便你记忆和使用：

| 优先级层级 | 运算符类别             | 具体运算符                                                   | 说明                 |
| ---------- | ---------------------- | ------------------------------------------------------------ | -------------------- |
| 1（最高）  | 括号/数组/方法调用     | `()`、`[]`、`.`（方法/属性调用）                             | 括号强制改变执行顺序 |
| 2          | 单目运算符（一元）     | `++`（后缀）、`--`（后缀）、`!`（非）、`~`（按位取反）、`+`（正）、`-`（负）、`++`（前缀）、`--`（前缀） | 仅操作一个操作数     |
| 3          | 算术运算符（乘除取余） | `*`、`/`、`%`                                                | 先乘除、后加减       |
| 4          | 算术运算符（加减）     | `+`、`-`                                                     |                      |
| 5          | 比较运算符（关系）     | `<`、`<=`、`>`、`>=`                                         | 比较大小             |
| 6          | 比较运算符（相等）     | `==`、`!=`                                                   | 判断是否相等         |
| 7          | 逻辑运算符（与）       | `&&`                                                         | 短路与，优先级高于或 |
| 8          | 逻辑运算符（或）       | `||`                                                         | 短路或               |
| 9（最低）  | 赋值运算符             | `=`、`+=`、`-=`、`*=`、`/=`、`%=`                            | 最后执行             |

##### 二、优先级的实际示例

通过代码直观理解优先级，避免因优先级错误导致逻辑问题：
```java
public class PriorityExample {
    public static void main(String[] args) {
        // 示例1：算术+比较+逻辑
        boolean result1 = 10 + 5 > 3 * 4 && 8 / 2 == 4;
        // 执行顺序：
        // 1. 3*4=12，8/2=4 → 表达式变为 10+5>12 && 4==4
        // 2. 10+5=15 → 表达式变为 15>12 && 4==4
        // 3. 15>12=true，4==4=true → 表达式变为 true && true
        // 4. true && true = true
        System.out.println(result1); // 输出 true

        // 示例2：单目+算术
        int a = 5;
        int result2 = ++a * 2 - 3;
        // 执行顺序：
        // 1. ++a（前缀自增）→ a=6 → 表达式变为 6*2 -3
        // 2. 6*2=12 → 表达式变为 12-3
        // 3. 12-3=9
        System.out.println(result2); // 输出 9

        // 示例3：赋值运算符（最低）
        int b = 2;
        b += 3 * 4; // 等价于 b = b + (3*4)，而非 (b+3)*4
        System.out.println(b); // 输出 14
    }
}
```

##### 三、改变运算符执行顺序的方法

唯一且最可靠的方式是使用 **小括号 `()`**，括号内的表达式会被优先执行，无论其原本的优先级高低。

**示例：强制改变执行顺序**

```java
public class ChangeOrderExample {
    public static void main(String[] args) {
        // 未加括号：按默认优先级，先乘后加 → 10+5*2=20
        int defaultOrder = 10 + 5 * 2;
        System.out.println("默认顺序：" + defaultOrder); // 20

        // 加括号：强制先算加法 → (10+5)*2=30
        int changedOrder = (10 + 5) * 2;
        System.out.println("改变顺序：" + changedOrder); // 30

        // 复杂场景：多层括号，内层优先
        int complex = ((10 - 3) * 2) + (8 / (4 - 2));
        // 执行顺序：4-2=2 → 10-3=7 → 7*2=14 → 8/2=4 → 14+4=18
        System.out.println("多层括号：" + complex); // 18
    }
}
```

**关键建议**：即使你能熟记优先级，在编写复杂表达式时，也建议主动使用括号——既可以避免自己犯错，也能让其他阅读代码的人一眼看懂执行顺序，提升代码可读性。

---

##### 总结

1. **核心优先级（高频）**：括号/数组 > 单目运算符 > `*`/`/`/`%` > `+`/`-` > 比较运算符（`<`/`>` 先于 `==`/`!=`） > `&&` > `||` > 赋值运算符。
2. **改变执行顺序**：使用小括号 `()` 是唯一方式，括号内的表达式会被优先执行，多层括号则「内层优先」。
3. **最佳实践**：复杂表达式优先用括号明确执行顺序，而非依赖记忆优先级，减少出错概率。



##### Java 运算符优先级专项测试题

###### 核心目标

本题仅聚焦**运算符优先级**的判断与计算，所有场景均围绕「不同优先级运算符的执行顺序」设计，覆盖编程中最易混淆的优先级场景。

###### 题目要求

1. **核心任务**：不编写代码，仅通过优先级规则手动计算每个表达式的最终结果；
2. **关键要求**：写出每一步的执行顺序（标注「先算什么、后算什么」，明确优先级依据）；
3. **验证环节**：最后编写完整代码验证结果（可选，重点是优先级判断过程）。

###### 专项题目（共5题，覆盖核心优先级场景）

```java
// 请手动计算以下变量的最终值，并标注执行顺序
public class PrioritySpecialTest {
    public static void main(String[] args) {
        // 题1：算术运算符优先级（*//% > +/-）
        int q1 = 8 + 4 * 2 - 10 / 5;
        
        // 题2：括号改变优先级 + 取余
        int q2 = (12 - 5) * 3 % 4;
        
        // 题3：单目运算符（自增）> 算术运算符
        int x = 5;
        int q3 = x++ * 2 + ++x;
        
        // 题4：比较运算符 > 逻辑运算符
        boolean q4 = 15 / 3 > 4 && 7 - 2 * 3 == 1;
        
        // 题5：混合优先级（括号 > 算术 > 比较 > 逻辑）
        boolean q5 = (8 + 2) / 5 < 3 || 9 % 4 * 2 >= 5;
    }
}
```

###### 详细解题指南（优先级判断+执行步骤）

###### 题1：q1 = 8 + 4 * 2 - 10 / 5

- **优先级依据**：`*`/`/` 优先级高于 `+`/`-`，同级运算符从左到右执行。
- **执行步骤**：
  1. 先算高优先级：`4 * 2 = 8`、`10 / 5 = 2`；
  2. 再算低优先级（从左到右）：`8 + 8 = 16` → `16 - 2 = 14`；
- **最终结果**：`q1 = 14`。

###### 题2：q2 = (12 - 5) * 3 % 4

- **优先级依据**：括号 `()` 优先级最高，其次是 `*`/`%`（同级从左到右）。
- **执行步骤**：
  1. 先算括号：`12 - 5 = 7`；
  2. 再算同级运算（从左到右）：`7 * 3 = 21` → `21 % 4 = 1`；
- **最终结果**：`q2 = 1`。

###### 题3：q3 = x++ * 2 + ++x（初始x=5）

- **优先级依据**：单目运算符（`++`）优先级高于算术运算符（`*`/`+`）；
  - 后缀自增 `x++`：先使用x的值，后自增；
  - 前缀自增 `++x`：先自增，后使用x的值。
- **执行步骤**：
  1. 处理 `x++ * 2`：先使用x=5计算 `5 * 2 = 10`，然后x自增为6；
  2. 处理 `++x`：x先自增为7，再参与运算；
  3. 最后算加法：`10 + 7 = 17`；
- **最终结果**：`q3 = 17`。

###### 题4：q4 = 15 / 3 > 4 && 7 - 2 * 3 == 1

- **优先级依据**：算术运算符 > 比较运算符 > 逻辑运算符（`&&`）。
- **执行步骤**：
  1. 先算算术运算：`15 / 3 = 5`、`2 * 3 = 6` → `7 - 6 = 1`；
  2. 再算比较运算：`5 > 4 = true`、`1 == 1 = true`；
  3. 最后算逻辑运算：`true && true = true`；
- **最终结果**：`q4 = true`。

###### 题5：q5 = (8 + 2) / 5 < 3 || 9 % 4 * 2 >= 5

- **优先级依据**：括号 > 算术运算符（`*`/`%` > `+`/`-`） > 比较运算符 > 逻辑运算符（`||`）。
- **执行步骤**：
  1. 先算括号：`8 + 2 = 10`；
  2. 算算术运算：
     - 左侧：`10 / 5 = 2`；
     - 右侧：`9 % 4 = 1` → `1 * 2 = 2`；
  3. 算比较运算：`2 < 3 = true`、`2 >= 5 = false`；
  4. 算逻辑运算：`true || false = true`；
- **最终结果**：`q5 = true`。

###### 验证代码（完整可运行）

将以下代码运行，可验证你的手动计算结果是否正确：
```java
public class PrioritySpecialTest {
    public static void main(String[] args) {
        // 题1
        int q1 = 8 + 4 * 2 - 10 / 5;
        // 题2
        int q2 = (12 - 5) * 3 % 4;
        // 题3
        int x = 5;
        int q3 = x++ * 2 + ++x;
        // 题4
        boolean q4 = 15 / 3 > 4 && 7 - 2 * 3 == 1;
        // 题5
        boolean q5 = (8 + 2) / 5 < 3 || 9 % 4 * 2 >= 5;
        
        // 输出结果
        System.out.println("q1 = " + q1); // 14
        System.out.println("q2 = " + q2); // 1
        System.out.println("q3 = " + q3); // 17
        System.out.println("q4 = " + q4); // true
        System.out.println("q5 = " + q5); // true
    }
}
```

###### 优先级核心总结

1. **核心优先级链**：括号 `()` > 单目运算符（`++`/`--`） > 算术运算符（`*`/`/`/`%` > `+`/`-`） > 比较运算符 > 逻辑运算符（`&&` > `||`）；
2. **同级规则**：算术/比较/逻辑运算符均为「从左到右」执行；
3. **关键技巧**：判断表达式时，先圈出「最高优先级的运算符」先计算，再逐步处理低优先级，可避免90%的优先级错误。

#### **9.在进行整数除法运算时，如果除数和被除数都是整数，结果会怎样？如何得到小数结果？**

如果除数和被除数都是整数，结果会向下取整。若需要小数结果，可以将其中一个数转换为 `float` 或 `double`，例如：
`int result = 5 / 2;` → `result = 2`
`double result = 5.0 / 2;` → `result = 2.5`

#### **10.自增（++）和自减（--）运算符有前缀和后缀两种形式，它们的区别是什么？**

- **前缀自增/自减**：`++x`，先增加/减少值，然后返回结果。
- **后缀自增/自减**：`x++`，先返回当前值，然后再增加/减少值。

#### **11.当不同数据类型的数据进行运算时，会发生自动类型转换，转换的规则是什么？**

- 小类型转换为大类型（如 `int` 转换为 `long`）。
- 基本数据类型与包装类之间会自动进行装箱和拆箱。
- 计算过程中，如果涉及浮点数和整数，整数会被转换为浮点数。

#### **12.什么是强制类型转换？在什么情况下需要进行强制类型转换？强制类型转换可能会导致什么问题？**

 **强制类型转换**：将一种类型的数据强行转换为另一种类型。
 需要进行强制类型转换的情况：

- 从大类型转换为小类型（如 `double` 转为 `int`）。
- 处理不兼容类型时（如将对象类型转换为某个子类）。
   强制转换可能会导致数据丢失（如小数转换为整数时丢失精度），或出现 `ClassCastException`。

#### **13.表达式的类型是如何确定的？如果表达式中包含多种数据类型，最终结果的类型会是什么？**

 表达式类型由操作数的类型决定：

- 如果有混合类型的操作，较大的类型会扩展到较小的类型。
- 如果涉及 `int` 和 `double`，结果会自动转换为 `double`。

#### **14.使用位运算符如何实现两个整数的交换，而不使用临时变量？**

 使用异或运算符 `^`，例如：

```java
a = a ^ b;  
b = a ^ b;  
a = a ^ b;
```

#### **15.逻辑运算符的短路特性是指什么？在什么情况下短路特性会生效？**

 **短路特性**：当第一个操作数已经决定了结果时，第二个操作数不会再计算。

- `&&`：左边为 `false` 时，右边不会计算。
- `||`：左边为 `true` 时，右边不会计算。

#### **16.比较两个浮点数是否相等时，为什么不建议直接使用 `==` 运算符？应该采用什么方法？**

 浮点数比较时，直接使用 `==` 可能由于精度问题导致不准确。应该使用 `Math.abs(a - b) < epsilon` 进行比较，其中 `epsilon` 是允许的误差范围。

#### **17.什么是表达式的副作用？自增、自减运算符在表达式中使用时会产生副作用吗？**

 **副作用**是指表达式在计算过程中修改了某些状态或变量的值。

```
自增和自减运算符（如 `++a`、`a++`）会在表达式中产生副作用，因为它们会修改操作数的值。
```

####  18..**复合赋值运算符（如 +=、-=）在运算过程中会自动进行类型转换吗？请举例说明。**

 是的，复合赋值运算符会进行类型转换。
 例如：
 `java     int a = 5;     double b = 3.2;     a += b;  // a 会被自动转换为 double 类型     `

#### **19.位异或运算符有哪些特殊的应用场景？例如在加密、数据校验等方面。**

位异或 (`^`) 在加密和数据校验中有特殊应用：

- 用于加密算法（如交换密码）。
- 用于数据校验，如校验和计算。

#### **20.在复杂的表达式中，如何通过合理使用括号来提高代码的可读性和确保运算顺序的正确性？**

使用括号明确表达式的优先级，避免依赖运算符优先级规则，确保代码易读且无歧义。明确的括号使用有助于代码维护和理解。

### 五、流程控制语句

#### **1.Java 中的条件语句有哪些？if 语句的语法格式是怎样的？if-else 语句和 if-else if-else 语句有什么区别？**

- **条件语句**：`if`、`if-else`、`if-else if-else`、`switch`。

- **if 语句的语法格式**：

  ```java
  if (condition) {
      // 执行代码
  }
  ```

- **if-else 语句**：

  ```java
  if (condition) {
      // 执行代码1
  } else {
      // 执行代码2
  }
  ```

- **if-else if-else 语句**：用于多个条件的判断。

  ```java
  if (condition1) {
      // 执行代码1
  } else if (condition2) {
      // 执行代码2
  } else {
      // 执行代码3
  }
  ```



#### 赋值运算符考题

```java
Boolean flag = false;
if (flag = true)
{
    System.out.println("true");
}
else
{
    System.out.println("false");
}
```

这道题目考察了Java中赋值语句和条件判断的语法细节。

在这段代码中:
Boolean flag = false;
if (flag = true)

关键点在于if语句中使用了赋值运算符"="而不是比较运算符"=="。这是一个合法的语法,会先将true赋值给flag,然后将flag的新值作为if条件判断。

因此执行流程是:
\1. 初始化flag为false
\2. if语句中将true赋值给flag
\3. 判断flag的新值(true)作为条件
\4. 由于条件为true,会执行if分支,打印"true"

所以C选项是正确答案。

#### **2.switch 语句的语法格式是怎样的？switch 语句中表达式的数据类型可以是什么？在 JDK7 及以后版本有什么变化？**

- **switch 语句的语法格式**：

  ```java
  switch (expression) {
      case value1:
          // 执行代码1
          break;
      case value2:
          // 执行代码2
          break;
      default:
          // 执行代码3
  }
  ```

- **表达式的数据类型**：可以是 `byte`、`short`、`int`、`char`、`String`（JDK 7 及以后）以及枚举类型。

- **JDK 7 及以后**：`switch` 语句支持 `String` 类型的表达式。

#### **3.switch 语句中 case 子句后面的 break 语句有什么作用？如果省略 break 语句会发生什么情况？**

- **break** 语句用于退出 `switch` 语句，防止执行后续的 `case`。
- **省略 break**：会导致“穿透”现象，即执行匹配到的 `case` 语句后，继续执行下一个 `case` 语句，直到遇到 `break` 或 `switch` 结束。

#### **4.switch 语句中的 default 子句有什么作用？它的位置可以随意放置吗？**

- **default**：当所有 `case` 都不匹配时，执行 `default` 中的代码。
- **位置**：`default` 子句可以放在 `switch` 语句中的任何位置，但通常放在最后。

#### **5.Java 中的循环语句有哪些？for 循环的语法格式是怎样的？初始化表达式、循环条件表达式和更新表达式分别在什么时候执行？**

- **循环语句**：`for`、`while`、`do-while`、增强 `for`（`foreach`）。

- **for 循环的语法格式**：

  ```java
  for (initialization; condition; update) {
      // 执行代码
  }
  ```

  - **初始化表达式**：在循环开始时执行一次，通常用于初始化循环变量。
  - **循环条件表达式**：每次循环前判断，如果为 `false` 则退出循环。
  - **更新表达式**：每次循环体执行后更新循环变量。

#### **6.while 循环和 do-while 循环有什么区别？它们的语法格式分别是怎样的？在什么情况下使用 while 循环，什么情况下使用 do-while 循环？**

- **while 循环**：

  ```java
  while (condition) {
      // 执行代码
  }
  ```

  - 先判断条件，符合则执行循环体。

- **do-while 循环**：

  ```java
  do {
      // 执行代码
  } while (condition);
  ```

  - 先执行循环体，后判断条件，至少执行一次循环。

- **使用场景**：如果需要循环执行至少一次操作，可以使用 `do-while`；如果在开始时不确定是否需要执行，可以使用 `while`。

#### **7.增强 for 循环（foreach 循环）的语法格式是怎样的？它适用于哪些场景？使用增强 for 循环有什么优缺点？**

- **语法格式**：

  ```java
  for (Type element : arrayOrCollection) {
      // 执行代码
  }
  ```

- **适用场景**：遍历数组或集合。

- **优点**：简洁，避免了手动使用索引或迭代器。

- **缺点**：不允许修改集合中的元素，无法获取索引。

#### **8.break 语句在循环和 switch 语句中的作用有什么不同？如何使用 break 语句跳出多层循环？**

- **在循环中的作用**：跳出当前循环，结束循环的执行。

- **在 switch 语句中的作用**：结束 `switch` 语句，跳出 `case` 执行。

- **跳出多层循环**：可以使用标签语句，`break` 后加标签指定跳出的循环。

  ```java
  outerLoop: for (int i = 0; i < 5; i++) {
      for (int j = 0; j < 5; j++) {
          if (i == 3) {
              break outerLoop;
          }
      }
  }
  ```

#### **9.continue 语句的作用是什么？它和 break 语句有什么区别？在循环中使用 continue 语句后，循环的更新表达式会执行吗？**

- **作用**：`continue` 用于跳过当前循环中的剩余部分，直接进行下一次循环。
- **区别**：`break` 跳出整个循环，而 `continue` 只跳过当前循环，继续下次循环。
- **更新表达式**：`continue` 语句会跳过本次循环的剩余部分，但仍会执行循环的更新表达式。

#### **10.什么是无限循环？如何创建一个无限循环？在什么情况下需要使用无限循环？如何终止无限循环？**

- **无限循环**：当循环条件永远为 `true` 时，循环不会退出。

- **创建无限循环**：

  ```java
  while (true) {
      // 执行代码
  }
  ```

- **使用场景**：常用于服务器监听请求等场景，需要一直运行直到某个条件满足。

- **终止方式**：使用 `break` 或其他条件来退出。

#### **11.在循环语句中，循环条件的设计非常重要，如何避免出现死循环？**

- 确保循环条件最终会变为 `false`，并且适当修改循环变量。
- 检查是否存在忘记更新循环变量的情况。

#### **12.嵌套循环是指什么？在嵌套循环中，内层循环和外层循环的执行顺序是怎样的？如何优化嵌套循环以提高程序性能？**

- **嵌套循环**：一个循环体内包含另一个循环。
- **执行顺序**：外层循环每执行一次，内层循环会完整执行一遍。
- **优化方法**：尽量减少内层循环的执行次数，或通过提前退出内层循环减少不必要的操作。

#### **13.如何在循环过程中根据特定条件动态地改变循环的执行流程？例如提前结束循环或者跳过某次循环。**

- 使用 `break` 提前结束循环。
- 使用 `continue` 跳过当前循环，继续下一次循环。

#### **14.在 switch 语句中，如果多个 case 子句需要执行相同的代码，应该如何处理？**

- 可以将多个 `case` 合并，并在它们后面执行相同的代码。

  ```java
  case 1:
  case 2:
  case 3:
      // 执行相同的代码
      break;
  ```

#### **15.使用 if-else 语句和 switch 语句分别处理多条件判断时，各有什么优缺点？在什么情况下选择使用 switch 语句更合适？**

- **if-else 优点**：灵活，可以处理复杂的条件判断。
- **switch 优点**：对于固定值的多条件判断，代码简洁、执行效率较高。
- **选择 switch**：当判断条件是常量或枚举时，使用 `switch` 更合适，性能也更好。

#### **16.循环语句中的初始化表达式可以声明多个变量吗？这些变量的数据类型必须相同吗？**

- 可以声明多个变量，且这些变量的数据类型必须相同。

- 例如：

  ```java
  for (int i = 0, j = 10; i < j; i++, j--) {
      // 执行代码
  }
  ```

#### **17.如何使用循环语句遍历一个数组？分别使用普通 for 循环和增强 for 循环进行说明。**

 \- **普通 for 循环**：
 `java       for (int i = 0; i < array.length; i++) {           System.out.println(array[i]);       }       `
 \- **增强 for 循环**：
 `java       for (int element : array) {           System.out.println(element);       }       `

在Java中，遍历数组可以使用普通`for`循环和增强`for`循环（也称为foreach循环），两种方式各有适用场景：

##### 1. 普通 for 循环遍历数组

普通`for`循环通过**索引值**访问数组元素，适用于需要知道元素位置（索引）的场景，也可以在遍历过程中修改数组元素。

**语法格式**：
```java
for (int i = 0; i < 数组名.length; i++) {
    // 数组名[i] 表示第i个元素
}
```

**示例**：
```java
public class ArrayLoop {
    public static void main(String[] args) {
        int[] numbers = {10, 20, 30, 40, 50};
        
        // 普通for循环遍历
        System.out.println("普通for循环遍历：");
        for (int i = 0; i < numbers.length; i++) {
            System.out.println("索引 " + i + " 的值：" + numbers[i]);
            
            // 可以修改数组元素（通过索引）
            numbers[i] *= 2;
        }
        
        // 遍历修改后的数组
        System.out.println("\n修改后的数组：");
        for (int i = 0; i < numbers.length; i++) {
            System.out.println(numbers[i]);
        }
    }
}
```

**输出结果**：
```
普通for循环遍历：
索引 0 的值：10
索引 1 的值：20
索引 2 的值：30
索引 3 的值：40
索引 4 的值：50

修改后的数组：
20
40
60
80
100
```

##### 2. 增强 for 循环（foreach）遍历数组

增强`for`循环直接遍历数组中的**元素值**，无需关心索引，语法更简洁，但**无法获取元素索引**，也**不适合在遍历中修改数组元素**（修改的是临时变量，不影响原数组）。

**语法格式**：
```java
for (元素类型 变量名 : 数组名) {
    // 变量名 表示当前遍历到的元素值
}
```

**示例**：
```java
public class ArrayForeach {
    public static void main(String[] args) {
        String[] fruits = {"苹果", "香蕉", "橙子", "葡萄"};
        
        // 增强for循环遍历
        System.out.println("增强for循环遍历：");
        for (String fruit : fruits) {
            System.out.println("水果：" + fruit);
            
            // 尝试修改元素（无效，仅修改临时变量）
            fruit = "西瓜";
        }
        
        // 原数组未被修改
        System.out.println("\n原数组：");
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```

**输出结果**：
```
增强for循环遍历：
水果：苹果
水果：香蕉
水果：橙子
水果：葡萄

原数组：
苹果
香蕉
橙子
葡萄
```

##### 两种循环的对比与适用场景

| 循环类型   | 优点                                  | 缺点                                  | 适用场景                     |
|------------|---------------------------------------|---------------------------------------|------------------------------|
| 普通for循环 | 可获取索引，可修改数组元素            | 语法稍繁琐，需手动控制索引范围        | 需要索引、需要修改元素时     |
| 增强for循环 | 语法简洁，无需关心索引，不易出错      | 无法获取索引，不能有效修改原数组元素  | 仅需遍历元素值，不关心位置时 |

##### 总结

- 普通`for`循环通过索引遍历，灵活且可修改数组，适合需要索引的场景；
- 增强`for`循环直接遍历元素，代码更简洁，适合单纯读取元素的场景；
- 实际开发中可根据是否需要索引或修改元素来选择合适的循环方式。

#### **18.在循环中使用 break 语句跳出循环后，循环后面的代码会执行吗？**

- 使用 `break` 跳出循环后，循环后的代码会立即执行，但不会进入循环体内。

#### **19.什么是标签语句？它的语法格式是怎样的？如何使用标签语句控制多层循环的执行流程？**

- **标签语句**用于标识循环，以便控制跳出多层循环。

- 语法格式：

  ```java
  outerLoop: for (int i = 0; i < 5; i++) {
      for (int j = 0; j < 5; j++) {
          if (i == 3) {
              break outerLoop;
          }
      }
  }
  ```

#### **20.在实际开发中，如何根据具体的业务需求选择合适的流程控制语句，以提高代码的效率和可读性？**

- 根据需求选择最简单且易于理解的控制语句。
- 对于多条件判断，优先选择 `switch`，对于复杂条件判断选择 `if-else`。
- 尽量减少嵌套层数，避免过度复杂的控制逻辑，保持代码简洁易读。

### 六、数组

#### **1.什么是数组？数组在 Java 中的特点是什么？**

- **数组**是存储同一类型元素的固定大小的集合。
- **特点**：
  - 数组长度固定，不可改变。
  - 数组中的元素类型必须相同。
  - 数组下标从 0 开始。

#### 2.Java 数组初始化

---

##### 一、先记住一句话

**Java 数组是对象，必须先开辟内存空间才能用。**
- 要么你自己写 `new`
- 要么编译器帮你隐式 `new`（语法糖）

---

##### 二、一维数组 3 种标准初始化方式

###### 1. 动态初始化（只指定长度，不赋值）

格式：
```java
类型[] 数组名 = new 类型[长度];
```

示例：
```java
int[] arr = new int[3];
```

特点：
- 必须写 `new`
- 系统自动赋默认值
  - int → 0
  - double → 0.0
  - boolean → false
  - 引用类型 → null

---

###### 2. 静态初始化（完整版）

格式：
```java
类型[] 数组名 = new 类型[]{元素1, 元素2, ...};
```

示例：
```java
int[] arr = new int[]{1, 2, 3};
```

特点：
- 必须写 `new`
- 不能写长度（长度由元素个数自动决定）

---

###### 3. 静态初始化（简化版 / 语法糖）

格式：
```java
类型[] 数组名 = {元素1, 元素2, ...};
```

示例：
```java
int[] arr = {1, 2, 3};
```

###### 重点：为什么不用 new？

因为编译器看到 `{}` 直接在**声明时赋值**，会**自动帮你补 `new int[]`**。
所以它等价于：

```java
int[] arr = new int[]{1,2,3};
```

---

##### 三、语法糖的重要限制（高频考点）

简化版 `{}` **只能在声明数组时用**，不能分开写！

✅ 合法：
```java
int[] arr = {1,2,3};
```

❌ 非法（编译报错）：
```java
int[] arr;
arr = {1,2,3};  // 错误！
```

分开赋值必须用完整版：
```java
arr = new int[]{1,2,3};
```

---

##### 四、声明 vs 初始化（一定要分清）

```java
int[] arr;       // 只声明，没开辟空间
arr = new int[3];// 初始化，开辟空间
```

只声明不初始化，直接使用会报错：
```java
int[] arr;
System.out.println(arr); // 报错：可能未初始化
```

---

##### 五、二维数组初始化

###### 1. 动态初始化

```java
int[][] arr = new int[2][3];
```

###### 2. 静态完整版

```java
int[][] arr = new int[][]{{1,2},{3,4}};
```

###### 3. 静态简化版

```java
int[][] arr = {{1,2},{3,4}};
```

###### 不规则二维数组

```java
int[][] arr = new int[2][];
arr[0] = new int[2];
arr[1] = new int[5];
```

---

##### 六、常见错误写法总结（考试必坑）

1. ❌ `int arr[3] = {1,2,3};`
   Java 不支持声明时写长度，这是 C 语言写法。

2. ❌ `int[] arr = new int[3]{1,2,3};`
   静态初始化**不能同时写长度和元素列表**。

3. ❌ `int[] arr; arr = {1,2,3};`
   简化版 `{}` 只能在声明时用。

4. ❌ `int[] arr = new int[];`
   要么给长度，要么给元素，不能都不给。

---

##### 七、一张表总结

| 方式       | 写法             | 是否要new      | 长度由谁定 | 适用场景       |
| ---------- | ---------------- | -------------- | ---------- | -------------- |
| 动态初始化 | `new int[3]`     | 是             | 手动指定   | 先创建，后赋值 |
| 静态完整版 | `new int[]{1,2}` | 是             | 元素个数   | 任意位置赋值   |
| 静态简化版 | `{1,2}`          | 否（编译器补） | 元素个数   | 声明时赋值     |

---

##### 八、回到你最开始的题目

```java
A.int arr[3] = {1, 2, 3};   ❌ C语言写法，Java错误
B.int arr[] = new int[3];   ✅ 合法（动态初始化）
C.int[] arr = new int[]{1,2,3}; ✅ 合法（静态完整版）
D.int[] arr = {1,2,3};      ✅ 合法（语法糖，编译器自动new）
```

**D 不用 new，就是因为它是静态初始化简化语法，编译器自动隐式创建数组对象。**

如果你需要，我可以再给你出 5 道数组初始化判断题，帮你彻底巩固。

#### **3.如何创建一个数组对象？创建数组时必须指定数组的长度吗？**

- 创建数组对象的语法：

  ```java
  int[] arr = new int[5];
  ```

- **指定数组的长度**：在创建数组时必须指定数组的长度，除非初始化时直接赋值。

  ```java
  int[] arr = {1, 2, 3, 4, 5};  // 自动推断数组长度
  ```

#### **4.数组的长度是如何确定的？一旦创建数组，其长度可以改变吗？**

- **数组的长度**在创建时确定，可以通过 `arr.length` 获取。
- **不可改变**：数组长度一旦确定不能再改变。如果需要变长，必须创建新的数组并复制数据。

#### **5.如何访问数组中的元素？访问数组元素时需要注意什么问题？**

- **访问元素**：

  ```java
  int value = arr[0]; // 访问第一个元素
  ```

- **注意问题**：

  - 数组下标从 0 开始。
  - 访问时必须确保下标在数组的有效范围内，否则会抛出 `ArrayIndexOutOfBoundsException`。

#### **6.什么是数组越界异常？在什么情况下会发生数组越界异常？如何避免？**

- **数组越界异常**：访问数组时使用了无效的下标（负数或超过数组最大索引）。
- **避免方法**：
  - 确保访问的下标在 `0 <= index < arr.length` 范围内。

#### **7.一维数组和二维数组有什么区别？二维数组在内存中的存储方式是怎样的？**

- **区别**：一维数组是单一数据类型的线性集合，二维数组是数组的数组（即数组的数组）。
- **内存存储方式**：二维数组在内存中是以“数组的数组”存储的，即每一行是一个数组，存储地址连续，但行与行之间的地址可能不连续。

#### **8.如何初始化一维数组？有哪几种初始化方式？**

- **初始化方式**：

  - 静态初始化：

    ```java
    int[] arr = {1, 2, 3, 4, 5};
    ```

  - 动态初始化：

    ```java
    int[] arr = new int[5]; // 初始化时指定数组长度
    ```

#### **9.如何初始化二维数组？不规则二维数组是如何创建和初始化的？**

- **二维数组初始化**：

  ```java
  int[][] arr = new int[3][4]; // 3 行 4 列的数组
  int[][] arr2 = {{1, 2, 3}, {4, 5, 6}};
  ```

- **不规则二维数组**（每一行的列数不同）：

  ```java
  int[][] arr = new int[3][];  // 指定行数
  arr[0] = new int[2];  // 第 1 行 2 列
  arr[1] = new int[3];  // 第 2 行 3 列
  arr[2] = new int[4];  // 第 3 行 4 列
  ```

**10.如何遍历一维数组？分别使用普通 for 循环和增强 for 循环进行说明。**

- **普通 for 循环**：

  ```java
  for (int i = 0; i < arr.length; i++) {
      System.out.println(arr[i]);
  }
  ```

- **增强 for 循环**：

  ```java
  for (int element : arr) {
      System.out.println(element);
  }
  ```

#### **11.如何遍历二维数组？可以使用嵌套循环吗？请举例说明。**

- **嵌套循环遍历二维数组**：

  ```java
  for (int i = 0; i < arr.length; i++) {
      for (int j = 0; j < arr[i].length; j++) {
          System.out.println(arr[i][j]);
      }
  }
  ```

#### **12.数组作为方法参数时，是值传递还是引用传递？在方法中修改数组元素的值，会影响到方法外部的数组吗？**

- **引用传递**：数组作为方法参数时，传递的是数组的引用，而不是副本。
- **影响外部数组**：如果在方法中修改数组元素的值，外部数组的值也会发生变化。

#### **13.数组作为方法返回值时，返回的是什么？是数组的副本还是数组的引用？**

- **返回数组的引用**：方法返回的是原数组的引用，不是副本。

#### **14.什么是数组的拷贝？Java 中有哪些方法可以实现数组的拷贝？它们的区别是什么？**

- **数组拷贝**：创建一个新数组，将原数组的元素复制到新数组中。
- **方法**：
  - `System.arraycopy()`：高效的数组拷贝方法。
  - `Arrays.copyOf()`：返回一个新数组，长度可以不同。
  - `clone()`：返回一个原数组的副本。
  - **区别**：`System.arraycopy()` 可指定拷贝的起始位置，`Arrays.copyOf()` 支持动态调整数组大小，`clone()` 创建整个数组的副本。

#### **15.Arrays 类是 Java 中的一个工具类，它提供了哪些常用的方法来操作数组？例如排序、查找等。**

- **常用方法**：
  - `sort()`：排序数组。
  - `binarySearch()`：在已排序数组中查找元素。
  - `equals()`：比较两个数组是否相等。
  - `fill()`：填充数组。
  - `copyOf()`：拷贝数组。
  - `toString()`：打印数组的字符串表示。

#### **16.如何对数组进行排序？使用 Arrays 类的 sort () 方法有什么特点？它的排序算法是什么？**

- **排序**：

  ```java
  Arrays.sort(arr);
  ```

- **特点**：`Arrays.sort()` 对基本数据类型的数组使用的是优化后的快速排序算法（TimSort），对对象数组则使用的是对象的 `compareTo()` 方法。

#### **17.如何在数组中查找指定的元素？使用 Arrays 类的 binarySearch () 方法需要满足什么条件？**

- **查找元素**：

  ```java
  int index = Arrays.binarySearch(arr, value);
  ```

- **条件**：数组必须是有序的（升序或降序）。

#### **18.什么是数组的扩容？在 Java 中如何实现数组的扩容？**

- **扩容**：创建一个更大的数组，并将原数组的内容复制到新的数组中。

- **实现方法**：

  ```java
  int[] newArr = Arrays.copyOf(arr, newLength);
  ```

#### **19.数组和集合有什么区别？在什么情况下使用数组，什么情况下使用集合？**

- **区别**：
  - 数组大小固定，存储同一类型的数据。
  - 集合可以动态扩容，提供更强大的操作功能（如排序、查找）。
- **使用场景**：
  - 数组适用于已知大小且只需要存储同一类型数据的情况。
  - 集合适用于大小动态变化且需要频繁插入、删除操作的情况。

#### **20.如何判断两个数组是否相等？使用 == 运算符和 Arrays 类的 equals () 方法有什么区别？**

- **判断相等**：

  - 使用 `==` 比较的是数组的引用，不是内容。
  - 使用 `Arrays.equals()` 比较的是数组的内容。

  ```java
  boolean isEqual = Arrays.equals(arr1, arr2);
  ```

#### 21.数组与字符串

在Java中，数组（Array）和字符串（String）是两种常用的数据结构，它们既有相似之处，也有显著区别，以下是详细对比：

##### **一、本质与定义**

1. **数组（Array）**  
   - 是**相同类型元素的有序集合**，可以存储基本类型（如`int[]`）或引用类型（如`String[]`）。  
   - 定义方式：`type[] name = new type[length];` 或 `type[] name = {元素1, 元素2, ...};`。  
   - 示例：`int[] nums = {1, 2, 3};` `String[] strs = {"a", "b"};`

2. **字符串（String）**  
   - 是**不可变的字符序列**，底层通过`char[]`数组实现（Java 9+改为`byte[]`以节省空间）。  
   - 定义方式：`String str = "hello";` 或 `String str = new String("hello");`。  

##### **二、核心区别**

| 特性               | 数组（Array）                              | 字符串（String）                          |
|--------------------|-------------------------------------------|-------------------------------------------|
| **可变性**         | 可变（元素可以被修改）                     | 不可变（内容一旦创建无法修改，修改会生成新对象） |
| **长度**           | 长度固定（创建时指定，不可动态改变）       | 长度可变（通过拼接等操作生成新字符串）     |
| **元素类型**       | 只能存储同一类型的元素（基本类型或引用类型） | 只能存储字符（`char`）                     |
| **常用操作**       | 通过索引访问/修改元素（`arr[i] = value`）  | 通过方法操作（如`charAt(i)`、`substring()`） |
| **内存占用**       | 直接存储元素值（基本类型）或引用（引用类型） | 存储字符数组+额外信息（如哈希值、长度）     |

##### **三、常用操作对比**

###### **1. 访问元素**

- **数组**：通过索引直接访问，效率高（`O(1)`）。  
  ```java
  int[] arr = {10, 20, 30};
  int value = arr[1]; // 访问索引1的元素，结果为20
  arr[1] = 200; // 修改元素值
  ```

- **字符串**：通过`charAt(int index)`方法访问字符，不可直接修改。  
  
  ```java
  String str = "hello";
  char c = str.charAt(1); // 访问索引1的字符，结果为'e'
  // str.charAt(1) = 'E'; // 编译错误（不可变）
  ```

###### **2. 长度获取**

- **数组**：通过`length`属性（无括号）。  
  
  ```java
  int len = arr.length;
  ```

- **字符串**：通过`length()`方法（有括号）。  
  
  ```java
  int len = str.length();
  ```

###### **3. 遍历元素**

- **数组**：  
  ```java
  for (int i = 0; i < arr.length; i++) {
      System.out.println(arr[i]);
  }
  ```

- **字符串**：  
  ```java
  for (int i = 0; i < str.length(); i++) {
      System.out.println(str.charAt(i));
  }
  ```

###### **4. 转换关系**

- **字符串 → 字符数组**：`toCharArray()`方法。  
  ```java
  String str = "hello";
  char[] chars = str.toCharArray(); // 结果：['h','e','l','l','o']
  ```

- **字符数组 → 字符串**：通过`String`构造方法。  
  
  ```java
  char[] chars = {'h','i'};
  String str = new String(chars); // 结果："hi"
  ```

- **数组 → 字符串（非字符数组）**：需手动拼接或使用`Arrays.toString()`。  
  ```java
  int[] nums = {1,2,3};
  String str = Arrays.toString(nums); // 结果："[1, 2, 3]"
  
  // 输出：[I@15db9742，[：代表是数组，I：代表 int 类型，@ 后面：对象的哈希码（内存地址）
  System.out.println(nums); 
  // 输出：[1, 2, 3]
  System.out.println(str);
  ```

###### **5. 修改操作**

- **数组**：直接通过索引修改元素（可变）。  
  ```java
  arr[0] = 100; // 直接修改
  ```

- **字符串**：不可直接修改，需通过`StringBuilder`或`StringBuffer`处理（效率更高）。  
  ```java
  String str = "hello";
  // 错误方式（创建新对象，原对象不变）：
  String newStr = str.replace('h', 'H'); // "Hello"
  
  // 高效方式（可变字符序列）：
  StringBuilder sb = new StringBuilder(str);
  sb.setCharAt(0, 'H'); // 修改索引0的字符
  String result = sb.toString(); // "Hello"
  ```

##### **四、适用场景**

- **数组**：  
  - 需要存储相同类型的多个元素，且长度固定。  
  - 频繁访问或修改元素（如数值计算、数据缓存）。  

- **字符串**：  
  - 需要存储字符序列，且操作以查询为主（如文本处理、日志输出）。  
  - 需进行拼接、截取、替换等操作（配合`StringBuilder`使用）。  

##### **五、总结**

- 数组是**可变的、长度固定的同类型元素集合**，适合高效的元素访问和修改。  
- 字符串是**不可变的字符序列**，适合存储和处理文本信息，修改时需借助`StringBuilder`。  
- 两者可通过`toCharArray()`和`new String(char[])`相互转换，结合使用能满足更多场景需求。

### 六、数组语法

#### 数组语法


在 Java 中，数组（Array）是一种**固定长度**的容器，用于存储**相同类型**的元素。它是 Java 中最基础的数据数据结构之一，直接在内存中分配连续的存储空间，支持通过索引快速访问元素。

##### 数组的核心特点

1. **固定长度**：创建时必须指定长度，且创建后无法修改（长度不变）。
2. **同类型元素**：所有元素必须是相同的数据类型（基本类型或引用类型）。
3. **连续内存**：元素在内存中连续存储，通过索引（从 `0` 开始）快速访问。
4. **默认值**：未显式初始化的元素会被赋予默认值（如 `int` 为 `0`，对象引用为 `null`）。

##### 数组的声明与初始化

数组的使用分为**声明**和**初始化**两个步骤，常见方式如下：

###### 1. 声明数组

声明时指定元素类型和数组名，不指定长度：
```java
// 声明基本类型数组（以 int 为例）
int[] arr1;
// 或（不推荐，风格不统一）
int arr2[];

// 声明引用类型数组（以 String 为例）
String[] strArr;
```

###### 2. 初始化数组

初始化时必须指定长度或直接赋值元素（长度由元素数量决定）。

###### 方式 1：静态初始化（直接指定元素）

适用于已知初始元素的场景，长度由元素个数自动确定：
```java
// 基本类型数组
int[] nums = {1, 2, 3, 4}; // 长度为 4

// 引用类型数组
String[] names = {"Alice", "Bob", "Charlie"}; // 长度为 3

// 初始化数组为最大值
int[] nums = new int[100];
Arrays.fill(nums,Integer.MAX_VALUE);
```

###### 方式 2：动态初始化（指定长度，元素用默认值）

适用于初始元素未知，仅知道长度的场景：
```java
// 基本类型数组（默认值：int 为 0，boolean 为 false 等）
int[] scores = new int[5]; // 长度为 5，元素初始为 [0, 0, 0, 0, 0]

// 引用类型数组（默认值为 null）
Object[] objects = new Object[3]; // 元素初始为 [null, null, null]
```

###### 方式 3：先声明后初始化

```java
double[] prices;
prices = new double[]{3.99, 5.99, 9.99}; // 静态初始化
// 或
prices = new double[4]; // 动态初始化
```

###### 方式4：**匿名数组初始化**

- 直接通过 `new int[]{元素1, 元素2, ...}` 创建一个匿名数组（没有显式声明变量名）。

```java
return new int[]{i, j}; // 创建了一个长度为 2 的 int 数组，元素为 i 和 j,并返回该数组
return new int[0];		// 创建一个长度为 0 的空数组并返回：
```



##### 数组的访问与修改



通过**索引**（`0` 到 `length-1`）访问或修改元素：
```java
int[] arr = {10, 20, 30};

// 访问元素（索引 0 是第一个元素）
System.out.println(arr[0]); // 输出：10
System.out.println(arr[2]); // 输出：30

// 修改元素
arr[1] = 200;
System.out.println(arr[1]); // 输出：200
```

> 注意：若索引超出范围（<0 或 ≥length），会抛出 `ArrayIndexOutOfBoundsException` 异常。

##### 数组的长度

通过 `length` 属性（不是方法）获取数组长度：
```java
int[] arr = {1, 2, 3};
System.out.println(arr.length); // 输出：3
```

##### 数组的遍历

常见方式有 `for` 循环和增强 `for` 循环（foreach）：

###### 1. 普通 for 循环（可修改元素）

```java
int[] arr = {1, 2, 3};
for (int i = 0; i < arr.length; i++) {
    System.out.print(arr[i] + " "); // 输出：1 2 3
    arr[i] *= 2; // 修改元素（变为 2,4,6）
}
```

###### 2. 增强 for 循环（仅遍历，无法修改元素）

```java
int[] arr = {2, 4, 6};
for (int num : arr) { // num 是元素的副本
    System.out.print(num + " "); // 输出：2 4 6
    num = 0; // 不会修改原数组
}
```

##### 多维数组

Java 支持多维数组（本质是“数组的数组”），最常用的是二维数组：

###### 声明与初始化

```java
// 静态初始化（每行长度可不同，称为“不规则数组”）
int[][] matrix = {
    {1, 2, 3},
    {4, 5},
    {6}
};

// 动态初始化（先指定行数，再指定每行长度）
int[][] scores = new int[3][]; // 3 行，每行长度暂未指定
scores[0] = new int[2]; // 第 0 行长度为 2
scores[1] = new int[3]; // 第 1 行长度为 3
```

###### 遍历二维数组

```java
for (int i = 0; i < matrix.length; i++) { // 遍历行
    for (int j = 0; j < matrix[i].length; j++) { // 遍历每行的元素
        System.out.print(matrix[i][j] + " ");
    }
    System.out.println();
}
// 输出：
// 1 2 3 
// 4 5 
// 6 
```

##### 数组与集合框架的对比

| 特性               | 数组（Array）                          | 集合（如 ArrayList）                  |
|--------------------|----------------------------------------|---------------------------------------|
| 长度               | 固定，创建后不可变                      | 动态可变，自动扩容                    |
| 元素类型           | 只能存储相同类型（基本类型或引用类型）    | 泛型约束，只能存储引用类型（基本类型需装箱）|
| 功能               | 仅支持基础访问、修改，无内置方法          | 提供丰富方法（`add`/`remove`/`contains` 等）|
| 内存效率           | 连续存储，内存效率高                    | 额外存储容器信息，内存开销略大        |
| 适用场景           | 长度固定、频繁访问元素的场景              | 长度不确定、需要频繁增删元素的场景    |

##### 常用数组工具类：`Arrays`

`java.util.Arrays` 提供了数组操作的静态方法，简化开发：

```java
import java.util.Arrays;

public class ArraysDemo {
    public static void main(String[] args) {
        int[] arr = {3, 1, 4, 2};
        
        Arrays.sort(arr); // 排序：[1, 2, 3, 4]
        System.out.println(Arrays.toString(arr)); // 转为字符串：[1, 2, 3, 4]
        
        int index = Arrays.binarySearch(arr, 3); // 二分查找值 3 的索引：2
        System.out.println(index);
        
        int[] copy = Arrays.copyOf(arr, 5); // 复制数组（长度 5，新增元素为默认值 0）
        System.out.println(Arrays.toString(copy)); // [1, 2, 3, 4, 0]
    }
}
```

##### 总结

- 数组是固定长度、同类型元素的连续存储容器，适合快速随机访问。
- 声明后必须初始化（指定长度或元素），通过索引操作元素。
- 多维数组本质是“数组的数组”，支持不规则长度。
- 实际开发中，若长度不确定，优先使用 `ArrayList` 等集合类；若长度固定且需高效访问，数组更合适。
#### Arrays

在 Java 中，`Arrays` 是 `java.util` 包下的一个**工具类**，提供了大量用于操作数组（`Object[]` 或基本类型数组）的静态方法，包括排序、搜索、比较、填充、复制、转列表等。

- **Arrays 是 JDK 提供的工具类，在 java.util 包下**
- 里面全是**静态方法**，专门用来**操作数组**

提供的常用方法：

- `Arrays.sort()` 排序
- `Arrays.binarySearch()` 二分查找
- `Arrays.asList()` 数组转 List
- `Arrays.toString()` 数组转字符串
- `Arrays.copyOf()` 数组拷贝

##### 1. **排序（Sorting）**

```java
int[] arr = {3, 1, 4, 1, 5};
Arrays.sort(arr); // 升序排序
// 结果: [1, 1, 3, 4, 5]

// 对对象数组自定义排序（需传入 Comparator）
String[] strs = {"banana", "apple", "cherry"};
Arrays.sort(strs, Comparator.reverseOrder());
// 结果: ["cherry", "banana", "apple"]
```

> ⚠️ 对基本类型数组使用双轴快排（性能好但**不稳定**）；对对象数组使用归并排序（**稳定**）。

------

##### 2. **二分查找（Searching）**

要求：**数组必须已排序**，否则结果未定义。

```java
int[] arr = {1, 3, 5, 7, 9};
int index = Arrays.binarySearch(arr, 5); // 返回 2
int notFound = Arrays.binarySearch(arr, 6); // 返回负值（如 -4）
```

------

##### 3. **比较（Equality & Comparison）**

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};
boolean equal = Arrays.equals(a, b); // true

// 深度比较（用于多维数组）
int[][] a2 = {{1, 2}, {3}};
int[][] b2 = {{1, 2}, {3}};
boolean deepEqual = Arrays.deepEquals(a2, b2); // true
```

------

##### 4. **填充（Filling）**

```java
int[] arr = new int[5];
Arrays.fill(arr, 7); // [7, 7, 7, 7, 7]

// 填充部分区间
Arrays.fill(arr, 1, 3, 0); // [7, 0, 0, 7, 7]
```

------

##### 5. **复制（Copying）**

```java
int[] original = {1, 2, 3, 4, 5};

// 复制整个数组
int[] copy1 = Arrays.copyOf(original, original.length);

// 复制并扩容/截断
int[] copy2 = Arrays.copyOf(original, 3); // [1, 2, 3]
int[] copy3 = Arrays.copyOf(original, 7); // [1, 2, 3, 4, 5, 0, 0]

// 复制指定范围 [from, to)
int[] range = Arrays.copyOfRange(original, 1, 4); // [2, 3, 4]
```

------

##### 6. **转为 List（asList）**

```java
String[] arr = {"a", "b", "c"};
List<String> list = Arrays.asList(arr);
```

> ⚠️ 注意：
>
> - 返回的是 **固定大小的 List**（`java.util.Arrays.ArrayList`，不是 `java.util.ArrayList`）。
> - 不支持 `add()`、`remove()` 等结构性修改，否则抛出 `UnsupportedOperationException`。
> - 修改原数组会影响 List，反之亦然（底层共享引用）。

✅ 安全转换为可变 List：

```java
List<String> mutableList = new ArrayList<>(Arrays.asList(arr));
```

------

##### 7. **toString / deepToString（用于打印）**

```java
int[] arr = {1, 2, 3};
System.out.println(Arrays.toString(arr)); // [1, 2, 3]

int[][] matrix = {{1, 2}, {3, 4}};
System.out.println(Arrays.deepToString(matrix)); // [[1, 2], [3, 4]]
```

> ✅ 比直接 `System.out.println(arr)`（输出类似 `[I@1b6d3586`）更友好。

------

##### 🔒 线程安全？

`Arrays` 类本身是**无状态的工具类**，所有方法都是静态的，**方法内部不共享可变状态**，因此**线程安全**。
但注意：**传入的数组本身是否线程安全，取决于调用者**。

------

##### 📚 总结常用方法表

| 功能         | 方法签名示例                        |
| ------------ | ----------------------------------- |
| 排序         | `Arrays.sort(arr)`                  |
| 二分查找     | `Arrays.binarySearch(arr, key)`     |
| 判断相等     | `Arrays.equals(a, b)`               |
| 深度相等     | `Arrays.deepEquals(a, b)`           |
| 填充         | `Arrays.fill(arr, value)`           |
| 复制         | `Arrays.copyOf(arr, newLength)`     |
| 范围复制     | `Arrays.copyOfRange(arr, from, to)` |
| 转 List      | `Arrays.asList(arr)`                |
| 打印一维数组 | `Arrays.toString(arr)`              |
| 打印多维数组 | `Arrays.deepToString(arr)`          |

------

`Arrays` 是 Java 中处理数组不可或缺的工具类，熟练掌握能极大提升开发效率和代码可读性。

### 七、链表


在 Java 中，链表（Linked List）是一种常见的数据结构，其元素（节点）通过指针（引用）连接，形成链式结构。与数组相比，链表的优势在于插入、删除元素时无需移动大量数据（只需修改指针指向），但随机访问效率较低（需从头遍历）。

#### Java 中链表的两种形式

1. **自定义链表（基于 `ListNode`）**  
   手动实现节点类（`ListNode`）和链表操作，适合理解底层原理或算法场景。

2. **标准库 `LinkedList`**  
   Java 集合框架提供的 `java.util.LinkedList` 类，封装了链表的常用操作，无需手动处理节点指针，开箱即用。

#### 1. 自定义链表（基于 `ListNode`）

##### 核心结构

通过 `ListNode` 类定义节点，每个节点包含：
- 数据域（存储值）  
- 指针域（引用下一个节点）

```java
// 节点类定义
class ListNode {
    int val; // 数据域（可改为其他类型）
    ListNode next; // 指针域（指向下一个节点）

    // 构造方法
    ListNode(int val) {
        this.val = val;
        this.next = null; // 默认为 null（链表尾部）
    }
}
```

##### 基本操作示例

以单链表（每个节点仅指向后一个节点）为例，实现常见操作：

```java
public class MyLinkedList {
    private ListNode head; // 头节点（链表的起点）
    private int size; // 链表长度

    // 初始化空链表
    public MyLinkedList() {
        head = null;
        size = 0;
    }

    // 尾部添加节点
    public void add(int val) {
        ListNode newNode = new ListNode(val);
        if (head == null) { // 链表为空时，新节点作为头节点
            head = newNode;
        } else { // 遍历到尾部，添加新节点
            ListNode current = head;
            while (current.next != null) {
                current = current.next;
            }
            current.next = newNode;
        }
        size++;
    }

    // 按索引删除节点
    public boolean remove(int index) {
        if (index < 0 || index >= size) return false;

        if (index == 0) { // 删除头节点
            head = head.next;
        } else { // 找到前驱节点，修改指针
            ListNode prev = head;
            for (int i = 0; i < index - 1; i++) {
                prev = prev.next;
            }
            prev.next = prev.next.next; // 跳过待删除节点
        }
        size--;
        return true;
    }

    // 遍历链表
    public void print() {
        ListNode current = head;
        while (current != null) {
            System.out.print(current.val + " -> ");
            current = current.next;
        }
        System.out.println("null");
    }

    // 测试
    public static void main(String[] args) {
        MyLinkedList list = new MyLinkedList();
        list.add(1);
        list.add(2);
        list.add(3);
        list.print(); // 输出：1 -> 2 -> 3 -> null

        list.remove(1); // 删除索引 1 的节点（值为 2）
        list.print(); // 输出：1 -> 3 -> null
    }
}
```

#### 2. 标准库 `LinkedList`

`java.util.LinkedList` 是 Java 集合框架中实现 `List` 和 `Deque` 接口的双向链表（每个节点同时指向前后节点），提供了丰富的操作方法，无需手动管理指针。

##### 特点

- 基于双向链表实现，支持高效的首尾插入/删除（`addFirst`/`addLast`）。
- 实现了 `List` 接口，支持索引访问、迭代等操作。
- 线程不安全，多线程环境需额外同步。

##### 常用方法

```java
import java.util.LinkedList;

public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();

        // 添加元素
        list.add(1); // 尾部添加
        list.addFirst(0); // 头部添加
        list.addLast(2); // 尾部添加
        System.out.println(list); // 输出：[0, 1, 2]

        // 访问元素
        System.out.println(list.get(1)); // 输出：1（按索引访问）
        System.out.println(list.getFirst()); // 输出：0（头部元素）

        // 删除元素
        list.remove(1); // 删除索引 1 的元素
        System.out.println(list); // 输出：[0, 2]

        // 遍历
        for (int num : list) {
            System.out.print(num + " "); // 输出：0 2
        }
    } 
}
```

#### 自定义链表 vs `LinkedList`

| 场景                | 自定义链表（`ListNode`）               | `LinkedList`                          |
|---------------------|----------------------------------------|---------------------------------------|
| 用途                | 算法实现、底层原理学习                  | 实际开发中存储数据，无需关注底层细节  |
| 功能                | 需手动实现增删改查                     | 内置丰富方法（`add`/`remove`/`get` 等）|
| 复杂度              | 灵活但代码量大，易出错                 | 封装完善，易用性高                    |
| 结构                | 多为单链表                              | 双向链表（支持前后遍历）              |

#### 总结

- 自定义链表（基于 `ListNode`）适合理解链表原理和算法题（如反转、环检测）。
- `LinkedList` 是 Java 标准库提供的成熟实现，适合实际开发中快速使用，尤其适合频繁在首尾操作元素的场景。



### 八、System

`System` 类的方法覆盖了 I/O 操作、数组复制、系统信息获取、时间计时、垃圾回收等核心功能，是 Java 中与系统交互的重要工具类。其中，`arraycopy`、`currentTimeMillis`、`out`、`exit` 等是开发中最常用的方法。

Java 中的 `java.lang.System` 类提供了一系列与系统交互的静态方法，主要用于访问系统资源、进行数组操作、获取系统信息、管理垃圾回收等。这些方法可以根据功能分为以下几类：

#### 一、标准输入/输出/错误流（I/O 相关）

用于处理程序的输入、输出和错误信息输出，是最基础的 I/O 操作方法。

| 方法/字段               | 说明                                                                 |
|-------------------------|----------------------------------------------------------------------|
| `System.in`             | 标准输入流（默认对应键盘输入），类型为 `InputStream`。               |
| `System.out`            | 标准输出流（默认对应控制台输出），类型为 `PrintStream`，常用 `println()` 打印信息。 |
| `System.err`            | 标准错误流（默认对应控制台输出，通常用于输出错误信息），类型为 `PrintStream`。 |
| `System.setIn(InputStream)` | 重定向标准输入流（例如将输入来源改为文件）。                         |
| `System.setOut(PrintStream)` | 重定向标准输出流（例如将输出写入文件）。                             |
| `System.setErr(PrintStream)` | 重定向标准错误流。                                                   |

##### 相关方法树状图

###### 一、Scanner 方法树状图

```
Scanner（包装 System.in）
├─ 读取内容系列
│  ├─ 字符串
│  │  ├─ next()           读取下一个以空白（空格、换行、Tab）分隔的字符串,无法读取包含空格的完整句子，遇到空格就会停止。
│  │  └─ nextLine()       读取一整行（包含空格，直到回车）,回车符本身会被读取但不会包含在返回的字符串中（会被消耗掉）。
│  │
│  ├─ 基本数据类型
│  │  ├─ nextBoolean()    读取布尔值
│  │  ├─ nextByte()       读取 byte
│  │  ├─ nextShort()      读取 short
│  │  ├─ nextInt()        读取 int（最常用）
│  │  ├─ nextLong()       读取 long
│  │  ├─ nextFloat()      读取 float
│  │  └─ nextDouble()     读取 double（常用）
│
├─ 判断是否有输入系列（防止异常）
│  ├─ hasNext()           判断扫描器是否还有下一个输入单元（Token）。
│  ├─ hasNextLine()       判断是否还有下一行文本输入。
│  ├─ hasNextBoolean()    判断下一个输入是否是 "true" 或 "false"
│  ├─ hasNextByte()       判断下一个输入是否是 byte 格式的数字
│  ├─ hasNextShort()
│  ├─ hasNextInt()
│  ├─ hasNextLong()
│  ├─ hasNextFloat()
│  └─ hasNextDouble()
│
├─ 配置类方法
│  ├─ useDelimiter(...)   设置分隔符
│  ├─ skip(...)           跳过指定内容
│  └─ close()             关闭 Scanner（同时关闭 System.in）
```

**避坑指南（重要注释）**

1. **`next()` 与 `nextLine()` 混用陷阱**：

   - 如果你先调用了 `nextInt()` 或 `next()`，输入缓冲区中可能会残留一个**回车符** (`\n`)。
   - 紧接着调用 `nextLine()` 时，它会直接读取这个残留的回车符并返回一个空字符串，导致你无法输入。
   - **解决**：在 `nextInt()` 和 `nextLine()` 之间，多加一句 `scanner.nextLine()` 来“吃掉”那个残留的回车。

2. **异常处理**：

   - 直接使用 `nextXxx()`（如 `nextInt()`）如果用户输入了错误的类型（比如在要求输入数字时输入了字母），程序会报错崩溃。

   - **最佳实践**：永远配合 `hasNextXxx()` 使用。

   - 例如：

     ```java
     if (scanner.hasNextInt()) {
         int num = scanner.nextInt(); // 安全读取
     } else {
         System.out.println("请输入数字！");
         scanner.next(); // 清除错误的输入
     }
     ```

###### 二、System.in（InputStream）方法树状图

```
System.in → InputStream
├─ 读取字节
│  ├─ read()              读取一个字节
│  ├─ read(byte[] b)      读取多个字节到数组
│  └─ read(byte[] b, int off, int len)
│
├─ 流控制
│  ├─ available()         获取可读取字节数
│  ├─ skip(n)             跳过 n 个字节
│  ├─ mark(int)           标记位置
│  ├─ reset()             回到标记
│  ├─ markSupported()     是否支持标记
│  └─ close()             关闭标准输入（慎用）
```

###### 三、System.out / System.err（PrintStream）方法树状图

两者方法完全一致，仅用途不同：
- out：正常输出
- err：错误输出（红色、无缓冲）

```
System.out / System.err → PrintStream
├─ 基础打印（最常用）
│  ├─ print(...)          打印不换行
│  └─ println(...)        打印并换行
│
├─ 格式化打印
│  ├─ printf(格式, 参数)
│  └─ format(格式, 参数)
│
├─ 追加输出
│  └─ append(...)
│
├─ 字节写入
│  ├─ write(int b)
│  └─ write(byte[] b, int off, int len)
│
├─ 缓冲与状态
│  ├─ flush()             刷新缓冲区
│  ├─ checkError()        检查是否发生IO错误
│  └─ close()             关闭标准输出（慎用）
```

###### 四、三者整体关系总图

```
标准I/O体系
├─ 输入方向
│  └─ System.in（InputStream）
│     └─ 被包装为 Scanner
│        └─ next() / nextInt() / nextLine() 等
│
└─ 输出方向
   ├─ System.out（PrintStream）→ 正常打印
   │  └─ print() / println() / printf()
   │
   └─ System.err（PrintStream）→ 错误打印
      └─ print() / println() / printf()
```

如果你需要，我可以再给你做一张 **纯文本横向树形图** 或 **可复制进笔记的 markdown 思维导图版**。

##### 1. System.in —— 标准输入流

###### 核心定义

- 静态常量：`public static final InputStream in`
- 父类：`InputStream`（Java 所有**字节输入流**的顶层抽象类）
- 默认绑定：**操作系统标准输入（键盘）**

###### 本质

`System.in` 是**字节流**，直接读取键盘输入的**原始字节**，不直接处理字符串/数字，需要包装成字符流才能方便读取文本。

###### 常用用法（必须包装，否则用起来极麻烦）

`InputStream` 本身只能读字节，实际开发中一定会用 `Scanner` 或 `BufferedReader` 包装：
```java
// 导入Scanner工具类，用于读取键盘输入
import java.util.Scanner;
import java.util.InputMismatchException;

public class Test {
    // 程序入口：main方法
    public static void main(String[] args) {
        /*
         * 1. 创建Scanner对象
         * System.in 代表标准输入流（键盘）
         * Scanner 是包装类，让我们能方便读取字符串、整数、小数等
         */
        Scanner sc = new Scanner(System.in);

        // ---------------------- 1. 读取姓名（字符串）----------------------
        System.out.print("请输入你的姓名：");
        // next()：读取到空格/回车为止的字符串
        String name = sc.next();

        // ---------------------- 2. 读取年龄（整数 + 输入校验）----------------------
        int age = 0;
        while (true) {
            System.out.print("请输入你的年龄：");
            try {
                // nextInt()：读取整数类型，若输入非数字会抛出异常
                age = sc.nextInt();
                // 年龄合法则跳出循环
                break;
            } catch (InputMismatchException e) {
                // 如果输入不是数字，清除错误标记
                sc.next();
                System.err.println("输入错误！请输入有效的数字年龄！");
            }
        }

        // ---------------------- 3. 拓展：读取性别（nextLine）----------------------
        sc.nextLine(); // 吸收掉上一步残留的回车符
        System.out.print("请输入你的性别：");
        String gender = sc.nextLine(); // 读取一整行（可以带空格）

        // ---------------------- 4. 拓展：读取身高（小数）----------------------
        System.out.print("请输入你的身高（米）：");
        double height = sc.nextDouble();

        // ---------------------- 5. 输出所有信息 ----------------------
        System.out.println("\n===== 输入信息汇总 =====");
        System.out.println("姓名：" + name);
        System.out.println("年龄：" + age + " 岁");
        System.out.println("性别：" + gender);
        System.out.printf("身高：%.2f 米\n", height); // 格式化输出，保留2位小数

        /*
         * 关闭Scanner
         * 注意：关闭后 System.in 也会被关闭，程序不能再读取输入
         */
        sc.close();
    }
}
```

###### 底层原始用法（不推荐，仅理解原理）

直接用 `InputStream` 读字节，只能读取单个字符/字节：
```java
try {
    // 读取一个键盘输入的字节
    int b = System.in.read(); 
    System.out.println("输入的字节：" + b);
} catch (Exception e) {
    e.printStackTrace();
}
```

###### 搭配 BufferedReader(推荐)

```java
// 1. 创建对象
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

// 2. 读一行字符串
String line = br.readLine();

// 3. 转 int
int x = Integer.parseInt(br.readLine());

// 4. 读一行并切分数字
String[] s = br.readLine().split(" ");

// 5. 关闭
br.close();

// 代码题，要 throws IO异常
// BufferedReader 的 readLine () 等方法会抛出 IOException，你必须处理，要么 try-catch，要么 throws 声明。
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int T = Integer.parseInt(br.readLine());
        while (T-- > 0) {
            int n = Integer.parseInt(br.readLine());
            String[] parts = br.readLine().split(" ");
            Set<Integer> set = new HashSet<>();
            for (String s : parts) {
                set.add(Integer.parseInt(s));
            }
            System.out.println(set.size());
        }
    }
}
```

###### 注意易错

```java
输入例子：
3
2
)(
4
()()
4
))((
输出例子：
1
0
3
    
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int len ;
        String s;
        while(n-- > 0){
            len = in.nextInt();
            in.nextLine();  // 注意这一点：用来“吃掉” nextInt() 留下的回车符
            s = in.nextLine();
            System.out.println(len);
        }


    }
}
```







---

##### 2. System.out —— 标准输出流

###### 核心定义

- 静态常量：`public static final PrintStream out`
- 类型：`PrintStream`（**字节打印流**，自带缓冲、自动刷新、不会抛受检异常）
- 默认绑定：**操作系统标准输出（控制台）**

###### 核心特点

1. 是 Java 最常用的输出方式，专门用于**打印普通信息、日志、结果**；
2. 自带缓冲区，输出效率高；
3. 支持打印**任意类型**（基本类型、对象、字符串、null 等）；
4. 不会抛出 `IOException`（受检异常），使用无压力。

###### 核心方法（必记）

```java
// 1. 打印后换行（最常用）
System.out.println("Hello World"); 
// 2. 打印不换行
System.out.print("Java ");
// 3. 格式化输出（类似 C 语言 printf）
System.out.printf("姓名：%s，年龄：%d\n", "张三", 20);
```

###### 打印对象原理

打印对象时，会自动调用对象的 `toString()` 方法：
```java
User user = new User("李四", 22);
System.out.println(user); // 等价于 user.toString()
```

---

##### 3. System.err —— 标准错误流

###### 核心定义

- 静态常量：`public static final PrintStream err`
- 类型：和 `out` 完全一样 → `PrintStream`
- 默认绑定：**控制台**（和 out 同一个输出窗口）

###### 核心用途

**专门输出错误信息、异常堆栈、警告信息**，用于和普通输出做**逻辑区分**。

###### 用法示例

```java
try {
    int a = 1 / 0;
} catch (Exception e) {
    // 错误信息用 err 输出
    System.err.println("程序出错：" + e.getMessage());
    // 打印异常堆栈（标准用法）
    e.printStackTrace(); // 底层默认就是用 System.err 输出
}
```

---

##### 三、System.out 和 System.err 的关键区别

虽然两者默认都输出到控制台，但**设计用途、显示效果、执行顺序**完全不同：

###### 1. 用途区别

- `System.out`：输出**正常业务信息**（结果、日志、提示）
- `System.err`：输出**错误/异常信息**

###### 2. 控制台显示区别

主流 IDEA/Eclipse 中：
- `out` 输出：**黑色/默认颜色**
- `err` 输出：**红色字体**（醒目区分错误）

###### 3. 执行顺序区别（高频面试点）

- `System.out` **带缓冲**：数据先放缓冲区，满了才输出；
- `System.err` **无缓冲/自动刷新**：立即输出。

因此**乱序输出**是正常现象：
```java
System.out.println("普通输出1");
System.err.println("错误输出"); // 可能先打印
System.out.println("普通输出2");
```

###### 4. 重定向区别

可以单独重定向 `out` 到文件，`err` 仍输出控制台，方便排查问题。

---

##### 四、高级特性：标准流重定向（重点）

Java 允许**修改这三个流的默认目标**（不局限于键盘/控制台），使用 `System.setIn()`、`System.setOut()`、`System.setErr()` 方法。

###### 示例1：输出重定向到文件

把 `System.out` 输出到文本文件，不再打印到控制台：
```java
import java.io.FileOutputStream;

public class RedirectOut {
    public static void main(String[] args) throws Exception {
        // 重定向 out 到文件
        System.setOut(new PrintStream(new FileOutputStream("out.txt")));
        
        // 内容会写入文件，控制台不显示
        System.out.println("写入文件的内容");
        System.out.println("Java 标准流重定向");
    }
}
```

###### 示例2：输入重定向（从文件读，而非键盘）

```java
import java.io.FileInputStream;

public class RedirectIn {
    public static void main(String[] args) throws Exception {
        // 从文件读取输入，代替键盘
        System.setIn(new FileInputStream("input.txt"));
        
        Scanner sc = new Scanner(System.in);
        String content = sc.nextLine(); // 读取文件内容
        System.out.println(content);
    }
}
```

---

##### 五、底层原理（极简理解）

1. 这三个流是 **JVM 启动时自动创建**的，不需要手动 new，也不需要手动关闭（关闭会导致后续无法使用）；
2. 它们都是**字节流**，而非字符流；
3. JVM 会自动对接操作系统的标准输入/输出/错误设备。

---

##### 六、使用注意事项

1. **不要手动关闭** `System.in`/`System.out`/`System.err`，关闭后整个程序无法再使用标准IO；
2. 读取输入必须包装 `Scanner/BufferedReader`，不要直接用 `InputStream.read()`；
3. 异常信息必须用 `System.err`，普通信息用 `System.out`，规范代码；
4. 生产环境不要大量用 `System.out`，会影响性能，应使用日志框架（Logback、Log4j2）。

---

##### 总结

1. **`System.in`**：`InputStream` 类型，默认键盘输入，必须包装后使用；
2. **`System.out`**：`PrintStream` 类型，默认控制台，打印普通信息，带缓冲；
3. **`System.err`**：`PrintStream` 类型，默认控制台，打印错误信息，立即输出；
4. 三者都是 JVM 内置静态流，可重定向，是 Java 最基础的 IO 工具。



#### 二、数组复制（高效操作）

提供高效的数组复制能力，底层由 native 方法实现，性能优于手动循环复制。

| 方法                     | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)` | 将源数组 `src` 中从 `srcPos` 开始的 `length` 个元素，复制到目标数组 `dest` 中从 `destPos` 开始的位置。支持同数组内的重叠复制。 |

#### 三、系统信息与属性（获取环境信息）

用于获取或设置系统相关的属性（如操作系统、Java 版本、用户目录等）。

| 方法                     | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `System.getProperty(String key)` | 获取指定键对应的系统属性值（例如 `java.version` 表示 Java 版本，`os.name` 表示操作系统名称）。 |
| `System.getProperty(String key, String def)` | 获取系统属性，若不存在则返回默认值 `def`。                            |
| `System.setProperty(String key, String value)` | 设置系统属性（部分系统属性为只读，无法修改）。                         |
| `System.getProperties()` | 返回所有系统属性的集合（`Properties` 对象）。                         |
| `System.getenv(String name)` | 获取指定环境变量的值（例如 `PATH` 环境变量）。                        |
| `System.getenv()`        | 返回所有环境变量的集合（`Map<String, String>`）。                     |

#### 四、时间与日期（时间戳相关）

用于获取系统时间戳，通常用于性能测试、计时等场景。

| 方法                     | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `System.currentTimeMillis()` | 返回当前时间与 1970-01-01 00:00:00 GMT 的时间差（毫秒数），类型为 `long`。 |
| `System.nanoTime()`      | 返回当前系统的高精度时间戳（纳秒级），通常用于测量短时间间隔（精度高于 `currentTimeMillis`）。 |

#### 五、垃圾回收与资源管理

用于手动触发垃圾回收、加载类、退出程序等。

| 方法                     | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `System.gc()`            | 建议 JVM 执行垃圾回收（仅是建议，JVM 不一定立即执行）。               |
| `System.runFinalization()` | 建议 JVM 对所有等待回收的对象执行 `finalize()` 方法（已过时，不推荐使用）。 |
| `System.exit(int status)` | 终止当前正在运行的 Java 虚拟机。`status` 为 0 表示正常退出，非 0 表示异常退出。 |
| `System.load(String filename)` | 加载指定路径的本地库（如 `.dll` 或 `.so` 文件），路径需是绝对路径。   |
| `System.loadLibrary(String libname)` | 加载指定名称的本地库（依赖系统的库搜索路径，无需写扩展名）。           |

#### 六、安全相关（权限检查）

用于检查当前程序是否具有特定权限（主要用于安全管理器）。

| 方法                     | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `System.getSecurityManager()` | 返回当前的安全管理器（`SecurityManager`），若未设置则返回 `null`（Java 9+ 中安全管理器已被弃用）。 |
| `System.setSecurityManager(SecurityManager s)` | 设置安全管理器（Java 9+ 中已弃用）。                                  |

### 九、运算符

Java 运算符是用于执行变量和值之间运算的符号。Java 提供了丰富且分类清晰的运算符体系，掌握它们对编写正确、高效的代码至关重要。

------

#### 一、Java 运算符分类（共 8 大类）

| 类别                      | 运算符                              | 说明                               |
| ------------------------- | ----------------------------------- | ---------------------------------- |
| **1. 算术运算符**         | `+`, `-`, `*`, `/`, `%`, `++`, `--` | 数学计算                           |
| **2. 赋值运算符**         | `=`, `+=`, `-=`, `*=`, `/=`, `%=`   | 赋值操作                           |
| **3. 关系运算符**         | `==`, `!=`, `>`, `<`, `>=`, `<=`    | 比较大小/相等，返回 `boolean`      |
| **4. 逻辑运算符**         | `&&`, `||`                          |                                    |
| **5. 位运算符**           | `&`, `                              | `, `^`, `~`, `<<`, `>>`, `>>>`     |
| **6. 条件（三元）运算符** | `? :`                               | 简化 if-else                       |
| **7. instanceof 运算符**  | `instanceof`                        | 判断对象类型                       |
| **8. 其他运算符**         | `.`, `()`, `[]`, `new`              | 成员访问、方法调用、数组、对象创建 |

------

#### 二、详细说明与示例

##### 1. 算术运算符

```java
int a = 10, b = 3;
System.out.println(a + b); // 13
System.out.println(a / b); // 3（整数除法）
System.out.println(a % b); // 1（取余）

// 自增/自减
int x = 5;
System.out.println(x++); // 5（先用后加）
System.out.println(++x); // 7（先加后用）
```

> ⚠️ 注意：`/` 对整数是**截断除法**；`%` 结果符号与被除数相同（`-5 % 2 = -1`）。

------

##### 2. 赋值运算符

```java
int x = 10;
x += 5;   // 等价于 x = x + 5;
x *= 2;   // x = 20
```

> ✅ 所有复合赋值运算符（`+=`, `-=` 等）都具有**隐式类型转换**特性：
>
> ```java
> byte b = 10;
> b += 5; // 合法！等价于 b = (byte)(b + 5);
> // 但 b = b + 5; 会编译错误（int 不能直接赋给 byte）
> ```

------

##### 3. 关系运算符

```java
int a = 5, b = 10;
System.out.println(a < b);  // true
System.out.println(a == b); // false
```

> 🔍 注意：
>
> - `==` 对基本类型比较**值**，对引用类型比较**地址**；
> - 字符串比较内容请用 `equals()`！

------

##### 4. 逻辑运算符

| 运算符 | 名称     | 特点                      |
| ------ | -------- | ------------------------- |
| `&&`   | 短路与   | 左为 false，右不执行      |
| `      |          | `                         |
| `&`    | 非短路与 | 两边都执行                |
| `      | `        | 非短路或                  |
| `!`    | 逻辑非   | 取反                      |
| `^`    | 异或     | 相同为 false，不同为 true |

```java
int x = 0;
if (true || ++x > 0) {
    System.out.println(x); // 输出 0（短路，++x 未执行）
}

if (true | ++x > 0) {
    System.out.println(x); // 输出 1（非短路，执行了 ++x）
}
```

> ✅ **推荐使用 `&&` 和 `||`**，避免不必要的计算或空指针异常。

------

##### 5. 位运算符（对二进制位操作）

| 运算符 | 说明           | 示例（4 位）                       |
| ------ | -------------- | ---------------------------------- |
| `&`    | 按位与         | `5 & 3` → `0101 & 0011 = 0001` → 1 |
| `      | `              | 按位或                             |
| `^`    | 按位异或       | `5 ^ 3` → `0101 ^ 0011 = 0110` → 6 |
| `~`    | 按位取反       | `~5` → `...11111010`（补码）       |
| `<<`   | 左移           | `5 << 1` → `1010` → 10（×2）       |
| `>>`   | 右移（带符号） | `-8 >> 1` → `-4`                   |
| `>>>`  | 无符号右移     | `-8 >>> 1` → 正数（高位补 0）      |

> 💡 应用场景：权限控制、加密、高效乘除（`x << 3` = `x * 8`）。

------

##### 6. 条件运算符（三元运算符）

```java
int score = 85;
String result = score >= 60 ? "及格" : "不及格";
```

> ✅ 语法：`条件 ? 表达式1 : 表达式2`
> ⚠️ 两个表达式类型需兼容。

------

##### 7. instanceof 运算符

```java
Object obj = "Hello";
if (obj instanceof String) {
    String s = (String) obj; // 安全转型
}
```

> ✅ 用于判断对象是否是指定类或其子类的实例。
> ⚠️ 若 `obj == null`，`instanceof` 返回 `false`（不会抛空指针）。

------

##### 8. 其他运算符

- `.`：访问成员（`list.size()`）
- `()`：方法调用、改变优先级（`(a + b) * c`）
- `[]`：数组访问（`arr[0]`）
- `new`：创建对象（`new ArrayList<>()`）

------

#### 三、运算符优先级（从高到低）

> **口诀**：**单目 > 算术 > 移位 > 关系 > 位 > 逻辑 > 三元 > 赋值**
>
> 算数运算符：先乘除后加减
>
> 逻辑运算符：先与后或

| 优先级 | 运算符                                | 结合性 |
| ------ | ------------------------------------- | ------ |
| 1      | `()` `[]` `.`                         | 左     |
| 2      | `++` `--` `!` `~` `+`（正） `-`（负） | 右     |
| 3      | `*` `/` `%`                           | 左     |
| 4      | `+` `-`（加减）                       | 左     |
| 5      | `<<` `>>` `>>>`                       | 左     |
| 6      | `<` `<=` `>` `>=` `instanceof`        | 无     |
| 7      | `==` `!=`                             | 左     |
| 8      | `&`                                   | 左     |
| 9      | `^`                                   | 左     |
| 10     | `                                     | `      |
| 11     | `&&`                                  | 左     |
| 12     | `||`                                  |        |
| 13     | `? :`                                 | 右     |
| 14     | `=`, `+=`, `-=` ...                   | 右     |

> ✅ **建议**：复杂表达式**加括号**，避免依赖优先级记忆！

------

#### 四、常见陷阱与最佳实践

##### ❌ 陷阱 1：浮点数比较

```java
double a = 0.1 + 0.2;
double b = 0.3;
System.out.println(a == b); // false！（精度问题）
```

✅ 正确做法：用 `Math.abs(a - b) < 1e-6`

##### ❌ 陷阱 2：字符串用 `==`

```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2); // false
```

✅ 用 `s1.equals(s2)`

##### ❌ 陷阱 3：自动类型提升

```java
byte b = 10;
b = b + 5; // 编译错误！b+5 是 int 类型
b += 5;    // 合法（复合赋值有隐式转换）
```

------

#### 五、总结

| 原则                        | 说明                   |
| --------------------------- | ---------------------- |
| **优先级复杂时加括号**      | 提高可读性和正确性     |
| **字符串比较用 `equals()`** | 避免 `==` 的引用陷阱   |
| **逻辑运算优先用 `&&`/`     |                        |
| **位运算谨慎使用**          | 除非性能关键或底层操作 |
| **浮点数避免直接 `==`**     | 用误差范围比较         |

> 🔑 **核心思想**：
> **“运算符是工具，理解其语义和副作用，才能写出健壮代码。”**

掌握 Java 运算符，是编写清晰、高效、无 bug 代码的基础！