

## 二、面向对象

面向对象程序设计是现代软件开发中最重要的编程范式之一,它具有三个主要优点:

A选项正确:**可重用性**是面向对象最显著的优势之一。通过封装、继承等机制,可以方便地重用已有的代码,避免重复开发,提高开发效率。一个设计良好的类可以在多个项目中反复使用。

B选项正确:**可扩展性**体现在面向对象的继承和多态特性上。系统可以通过继承机制方便地扩展新的功能,而不需要修改原有代码,这种开放封闭原则是面向对象设计的重要原则。

C选项正确:**易于管理和维护**得益于面向对象的封装特性。通过将数据和操作封装在对象中,降低了程序的耦合度,当需要修改某个功能时,影响范围被限制在相关的类中,更容易进行维护。

D选项错误:简单易懂并不是面向对象的特点。实际上,面向对象涉及许多复杂的概念(如封装、继承、多态等),对初学者来说往往较难理解。而且在设计模式、框架等方面也具有较高的学习成本。相比面向过程的编程方式,面向对象的抽象程度更高,理解起来更具挑战性。

### **1.OOP vs POP**

* **OOP（面向对象编程）**：通过创建对象来表示和操作数据，核心思想是将数据和操作数据的函数封装到对象中。OOP 注重对象和类的设计，强调封装、继承、多态。
  * **适用场景**：适用于大型、复杂的应用系统，如游戏开发、企业级应用。
* **POP（面向过程编程）**：注重过程的执行和数据的流转，程序的结构是由函数调用和顺序控制构成。POP 强调函数的调用，数据与操作分离。
 * **适用场景**：适用于简单的脚本、算法实现、系统底层开发等。

### 什么是面向对象？

**面向对象（OOP，Object-Oriented Programming）**  
是以**对象**为核心的编程思想：  
把现实世界中的事物抽象成**类（Class）**，事物的具体实例就是**对象（Object）**。

核心思想：  
**一切皆对象，对象 = 属性 + 行为**  
关注：**谁来做**，而不是**怎么做**（区别于面向过程）。

一句话总结：  
**面向对象是将数据和操作数据的方法封装在一起，以对象为基本单位组织程序的编程思想。**

---

### 面向对象三大特征：封装、继承、多态

#### （1）封装 Encapsulation

把**属性私有化**，对外提供**公共方法（get/set）**访问。

**好处：**
- 隐藏内部实现细节  
- 提高安全性，防止外部随意修改  
- 便于维护、降低耦合

#### （2）继承 Inheritance

子类 **extends** 父类，**复用父类属性和方法**，并可扩展自己功能。

继承在本质上是特殊~一般的关系，即常说的is-a关系。子类继承父类，表明子类是一种特殊的父类，并且具有父类所不具有的一些属性或方法。

**is-a = 是一个 / 属于… 这一类**

**好处：**
- 代码复用，减少重复  
- 体现类之间的层次关系  
- 便于扩展和维护

#### （3）多态 Polymorphism

**同一个行为，不同对象表现出不同形态。**  
前提：**继承 + 方法重写 + 父类引用指向子类对象（向上转型）**

**好处：**
- 提高程序扩展性和灵活性  
- 统一接口，便于维护  
- 符合开闭原则（对扩展开放，对修改关闭）

---

##### 多态为什么要转型？

多态默认是：**父类引用指向子类对象（向上转型）**  
格式：`Father f = new Son();`

**向上转型：自动，但只能调用父类中定义的方法，不能调用子类独有方法。**

所以**需要向下转型**，目的只有一个：  
**调用子类特有的属性和方法。**

一句话总结：  
**向上转型是为了统一接口、实现多态；  
向下转型是为了使用子类独有的功能。**

---

### Java 四大权限修饰符

范围从小到大：**private → 缺省(default) → protected → public**

| 修饰符          | 本类 | 同包类 | 子类 | 任意类 |
| --------------- | ---- | ------ | ---- | ------ |
| private         | ✅    | ❌      | ❌    | ❌      |
| default（缺省） | ✅    | ✅      | ❌    | ❌      |
| protected       | ✅    | ✅      | ✅    | ❌      |
| public          | ✅    | ✅      | ✅    | ✅      |

**记忆口诀：**
- **private：自己用**  
- **default：同包用**  
- **protected：子类+同包用**  
- **public：大家都能用**

---

### **2.什么是封装？在Java中如何实现封装？封装的主要作用是什么？**

* **封装**：将对象的状态（属性）和行为（方法）封装在类中，并通过提供对外访问的接口（getter 和 setter 方法）来控制对属性的访问。外部只知道如何操作对象，而不关心对象的具体实现。
* **实现方式**：在 Java 中，使用 `private` 修饰属性，提供 `public` 的 getter 和 setter 方法来访问属性。

  ```java
  class Person {
      private String name;
      private int age;
      
      public String getName() {
          return name;
      }
      
      public void setName(String name) {
          this.name = name;
      }
  }
  ```
* **作用**：封装可以保护数据，防止外部直接修改数据；简化代码，隐藏实现细节；提高系统的安全性和可维护性。

---

### **3.继承关系中，子类能否继承父类的所有成员？为什么？**

* **不能**：子类无法继承父类的 `private` 成员。`private` 成员只能在父类内部访问。其他的 `protected`、`public` 成员可以被继承。父类的构造方法也不能被继承。
* **原因**：`private` 成员是为了实现数据封装，不希望外部直接访问，因此无法继承。

---

### **4.什么是组合？组合与继承相比有哪些优势？在设计时如何选择组合和继承？**

* **组合**：通过在一个类中包含另一个类的实例，来实现功能复用。也就是将一个对象作为另一个对象的成员变量。
* **优势**：组合更灵活，子类可以动态决定要组合哪些类；避免了继承中的“多重继承”问题和子类过于依赖父类实现的情况。
* **选择**：当类与类之间存在“有一个”关系时，使用组合；当类与类之间存在“是一个”关系时，使用继承。

---

### **5.多态的实现机制是什么？编译时多态和运行时多态有何区别？**

* **实现机制**：多态通过方法重载和方法重写实现。
* **编译时多态**：又称方法重载，在同一个类中，根据方法签名（参数个数或类型）来区分不同的方法。
* **运行时多态**：又称方法重写，通过方法的动态绑定，子类对象调用重写的方法，方法的选择在运行时决定。

##### 多态的实现机制

多态（Polymorphism）的核心是“**同一行为的不同表现形式**”，在Java中通过以下机制实现：

1. **方法重载（Overload）**  
   在同一个类中，允许存在多个同名方法，但**参数列表（参数个数、类型、顺序）必须不同**。编译器根据方法调用时的参数匹配对应的方法，属于“编译时多态”。

2. **方法重写（Override）**  
   子类继承父类后，对父类的方法进行重新实现（方法名、参数列表、返回值类型必须与父类一致）。运行时根据对象的实际类型（而非声明类型）调用对应的方法，属于“运行时多态”。

3. **继承与向上转型**  
   运行时多态的前提是：子类对象可以向上转型为父类类型（如 `Parent p = new Child();`）。通过父类引用调用方法时，JVM会动态绑定到子类的重写方法。

##### 编译时多态 vs 运行时多态

| 维度               | 编译时多态（方法重载）                          | 运行时多态（方法重写）                          |
|--------------------|-------------------------------------------------|-------------------------------------------------|
| **决定时机**       | 编译阶段（编译器确定调用哪个方法）               | 运行阶段（JVM根据对象实际类型确定调用哪个方法）   |
| **核心依据**       | 方法签名（参数个数、类型、顺序）                 | 对象的实际类型（而非声明类型）                   |
| **实现方式**       | 同一类中同名不同参的方法                         | 子类重写父类方法，配合向上转型                   |
| **典型场景**       | `System.out.println()` 支持多种数据类型的打印    | 父类引用指向子类对象，调用重写方法（如 `Animal a = new Dog(); a.eat();`） |
| **灵活性**         | 编译时固定，无法动态改变                         | 运行时动态绑定，更灵活，支持扩展                 |

##### 1. 编译时多态（方法重载）

```java
class Calculator {
    // 重载：参数个数不同
    int add(int a, int b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
    
    // 重载：参数类型不同
    double add(double a, double b) { return a + b; }
}

public class Test {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        calc.add(1, 2);        // 调用int add(int, int)
        calc.add(1, 2, 3);     // 调用int add(int, int, int)
        calc.add(1.5, 2.5);    // 调用double add(double, double)
        // 编译时已确定调用哪个方法
    }
}
```

##### 2. 运行时多态（方法重写）

```java
class Animal {
    void eat() { System.out.println("动物吃东西"); }
}

class Dog extends Animal {
    @Override
    void eat() { System.out.println("狗吃骨头"); } // 重写父类方法
}

class Cat extends Animal {
    @Override
    void eat() { System.out.println("猫吃鱼"); } // 重写父类方法
}

public class Test {
    public static void main(String[] args) {
        Animal a1 = new Dog(); // 向上转型：父类引用指向子类对象
        Animal a2 = new Cat();
        
        a1.eat(); // 运行时调用Dog的eat()，输出"狗吃骨头"
        a2.eat(); // 运行时调用Cat的eat()，输出"猫吃鱼"
        // 运行时根据对象实际类型（Dog/Cat）确定调用的方法
        Animal a3 = new Animal();
        a3.eat();  // 动物吃东西
        
    }
}
```

##### 总结

- **编译时多态**：通过方法重载实现，编译器在编译期根据参数匹配方法，是“静态绑定”。
- **运行时多态**：通过方法重写和向上转型实现，JVM在运行期根据对象实际类型绑定方法，是“动态绑定”。
- 多态的核心价值是**解耦**：代码可以针对父类编程，无需关心具体子类，新增子类时无需修改原有代码（符合“开闭原则”）。

##### 关键原理（面试必问）

**Java 方法调用遵循：编译看左边，运行看右边**

```
Animal a1 = new Dog();
左边：Animal    编译类型
右边：Dog       运行类型（真正的对象）
```

- 编译时：检查 **Animal** 有没有 `eat()`
- 运行时：执行 **Dog** 的 `eat()`

这就是 **多态**：

**调用父类方法 → 运行子类方法**

```
父类 变量 = new 子类();
变量.方法();
```

就一定是：

**调用父类方法 → 实际运行子类重写的方法**

---

### **6.抽象类和接口都可以实现多态，在实际开发中如何选择使用它们？**

* **抽象类**：适用于类之间有共同的行为，且可以提供部分默认实现时。抽象类允许定义方法的实现。
* **接口**：适用于不同类之间共享某种功能，而没有行为实现时。接口只能定义方法的签名，不提供任何实现。

##### 一、先明确抽象类与接口的核心本质差异

| 对比项         | 抽象类 abstract class                                        | 接口 interface                                               |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计目标**   | 描述“类的共性”，体现 **“is-a” 继承关系**（如“Dog is a Animal”） | 描述“功能的契约”，体现 **“has-a” 能力关系**（如“Dog has a Runable 能力”） |
| **继承方式**   | 单继承（extends）                                            | 多实现（implements）                                         |
| **本质语义**   | **is-a**（是一个）                                           | **can-do**（具备某种能力）                                   |
| **成员变量**   | 可以有普通成员变量                                           | 只能是 `public static final` 常量                            |
| **构造方法**   | 有构造方法                                                   | 没有构造方法                                                 |
| **普通方法**   | 可以有普通方法、抽象方法                                     | 默认public abstract 方法；JDK8+：可以有默认方法 (default)、静态方法 |
| **静态代码块** | 可以有                                                       | 不能有                                                       |
| **访问修饰符** | 任意（public/protected/default）                             | 只能 public（默认就是 public）                               |
| **设计目的**   | 抽取子类**通用属性 + 通用逻辑**                              | 定义一套**行为标准 / 契约**                                  |
| **核心价值**   | 复用共性实现，减少重复代码                                   | 定义功能契约，实现跨类的功能共享（多态的“松耦合”）           |

##### 二、选择逻辑：从“关系”和“需求”切入

###### 1. 优先选抽象类的场景：类间是“is-a”关系，且存在共性实现

当多个类属于**同一“家族”**（有明确的继承关系，如“动物-狗-猫”“交通工具-汽车-火车”），且存在**可复用的共性实现**时，用抽象类。

典型场景：
- **存在“部分默认实现”**：例如“动物”抽象类中，“呼吸”是所有动物的共性行为（可提供默认实现：`public void breathe() { System.out.println("用肺呼吸"); }`），而“吃”是每个动物的差异化行为（定义为抽象方法：`public abstract void eat();`）。此时抽象类可复用“呼吸”的实现，子类只需重写“吃”。
  
  ```java
  abstract class Animal {
      // 共性实现：所有动物共享的呼吸逻辑
      public void breathe() {
          System.out.println("用肺呼吸");
      }
      // 差异化抽象方法：子类需各自实现
      public abstract void eat();
  }
  class Dog extends Animal {
      @Override
      public void eat() { System.out.println("狗吃骨头"); }
  }
  class Fish extends Animal {
      // 子类可重写共性实现（如鱼用鳃呼吸）
      @Override
      public void breathe() {
          System.out.println("用鳃呼吸");
      }
      @Override
      public void eat() { System.out.println("鱼吃饲料"); }
  }
  ```
- **需要定义实例变量**：抽象类可定义非静态的实例变量（如`Animal`的`age`、`weight`），子类继承后可直接使用；而接口的变量是`public static final`常量，无法定义实例变量。

###### 2. 优先选接口的场景：类间是“has-a”关系，仅需共享功能契约

当多个类**不属于同一“家族”**（如“狗”“汽车”“机器人”），但需要共享某一种“能力”（如“可运行”“可充电”）时，用接口——接口不关心类的本质，只定义“能做什么”的契约。

典型场景：
- **跨类共享功能**：例如“可运行”（`Runnable`）接口，“狗”（`Dog`）可以实现`Runnable`（跑），“汽车”（`Car`）也可以实现`Runnable`（行驶），但“狗”和“汽车”没有继承关系，只是都具备“运行”的能力。此时接口是唯一选择（抽象类无法实现跨家族的功能共享）。
  ```java
  // 定义“可运行”的功能契约
  interface Runnable {
      void run(); // 仅定义“能跑”，不关心谁跑、怎么跑
  }
  // 狗具备“运行”能力
  class Dog implements Runnable {
      @Override
      public void run() { System.out.println("狗用腿跑"); }
  }
  // 汽车具备“运行”能力
  class Car implements Runnable {
      @Override
      public void run() { System.out.println("汽车用轮子跑"); }
  }
  ```
- **需要多实现**：Java不支持多继承，若一个类需要具备多种能力（如“可跑”+“可充电”），只能通过多实现接口实现。例如“电动汽车”（`ElectricCar`）需要“运行”（`Runnable`）和“充电”（`Chargeable`）两种能力，可同时实现两个接口：
  ```java
  interface Chargeable {
      void charge();
  }
  class ElectricCar implements Runnable, Chargeable {
      @Override
      public void run() { /* 行驶逻辑 */ }
      @Override
      public void charge() { /* 充电逻辑 */ }
  }
  ```
- **仅需定义方法签名（无默认实现）**：若功能契约不需要默认实现（所有实现类必须自己写逻辑），用接口更纯粹（如`Comparable`接口，用于对象比较，每个类的比较逻辑完全不同，无需默认实现）。

###### 3. 特殊场景：抽象类与接口结合使用

在复杂系统中，常结合两者的优势：**抽象类实现接口，提供部分默认实现，子类继承抽象类并补充差异化实现**——既复用了代码，又保留了多实现的灵活性。

例如：Java的`Collection`框架中，`List`是接口（定义“有序集合”的契约），`AbstractList`是抽象类（实现`List`接口，提供`get()`、`size()`等方法的默认实现），`ArrayList`、`LinkedList`等子类继承`AbstractList`，只需重写差异化方法（如`add()`、`remove()`），大幅减少重复代码。
```java
// 接口：定义契约
interface List<E> {
    E get(int index);
    boolean add(E e);
    int size();
}
// 抽象类：实现部分契约，提供默认实现
abstract class AbstractList<E> implements List<E> {
    // 默认实现get()（通过循环遍历）
    @Override
    public E get(int index) {
        // 共性逻辑：检查索引合法性
        if (index < 0 || index >= size()) {
            throw new IndexOutOfBoundsException();
        }
        // 差异化逻辑：由子类实现（如ArrayList用数组取，LinkedList用链表遍历）
        return getElement(index);
    }
    protected abstract E getElement(int index);
}
// 子类：仅补充差异化实现
class ArrayList<E> extends AbstractList<E> {
    private Object[] array;
    @Override
    protected E getElement(int index) {
        return (E) array[index]; // 数组直接取值
    }
    // ... 其他方法实现
}
```

##### 三、总结：一句话选择口诀

- 若类间是 **“is-a” 继承关系 + 有共性实现** → 用抽象类；  
- 若类间是 **“has-a” 能力关系 + 仅需功能契约** → 用接口；  
- 若需要 **多能力 + 部分默认实现** → 接口 + 抽象类结合。

---

### **7.什么是向上转型和向下转型？向下转型时需要注意什么问题？如何避免转型异常？**

* **向上转型**：将子类对象赋值给父类引用，通常是安全的，编译时检查。
* **向下转型**：将父类引用转回子类引用，可能会导致 `ClassCastException` 异常。
* **避免转型异常**：在进行向下转型前，使用 `instanceof` 判断对象是否是该类型。

  ```java
  if (obj instanceof SubClass) {
      SubClass sub = (SubClass) obj;
  }
  ```

##### 一、向上转型（Upcasting）

**定义**：将**子类对象**赋值给**父类类型的引用变量**（即“小类型”转“大类型”）。  
这是多态的默认行为，编译器自动允许，且**永远安全**。

###### 特点：

- 语法：`父类类型 变量名 = new 子类类型();`  
- 本质：父类引用指向子类对象，只能访问父类中声明的成员（方法/变量），但执行的是子类重写的方法（多态特性）。  
- 场景：通常用于统一接口，屏蔽子类差异（如用父类作为方法参数，接收任意子类对象）。

###### 示例：

```java
class Animal {
    void eat() { System.out.println("动物吃东西"); }
}
class Dog extends Animal {
    @Override
    void eat() { System.out.println("狗吃骨头"); }
    void bark() { System.out.println("狗叫"); } // 子类特有方法
}

// 向上转型：Dog → Animal
Animal animal = new Dog(); // 安全，编译器允许
animal.eat(); // 执行子类Dog的eat()（多态）
animal.bark(); // 编译错误（父类Animal无bark()方法）
```

##### 二、向下转型（Downcasting）

**定义**：将**父类引用**（原本指向子类对象）转回**子类类型的引用变量**（即“大类型”转“小类型”）。  
这需要手动强制转换，且**可能不安全**（若父类引用指向的不是目标子类对象，会抛异常）。

###### 特点：

- 语法：`子类类型 变量名 = (子类类型) 父类引用;`  
- 目的：为了访问子类特有的方法或变量（向上转型后无法直接访问）。  
- 风险：若父类引用指向的对象**不是目标子类的实例**，运行时会抛出 `ClassCastException`。

###### 示例（安全与不安全的向下转型）：

```java
// 场景1：父类引用确实指向Dog对象（安全）
Animal animal1 = new Dog();
Dog dog1 = (Dog) animal1; // 成功转型
dog1.bark(); // 可调用子类特有方法（输出：狗叫）

// 场景2：父类引用指向其他子类对象（不安全）
Animal animal2 = new Cat(); // Cat是Animal的另一个子类
Dog dog2 = (Dog) animal2; // 编译通过，但运行时抛ClassCastException
```

##### 三、向下转型的注意事项与异常避免

###### 核心问题：如何确保向下转型安全？

必须通过 `instanceof` 运算符**先判断父类引用指向的对象是否为目标子类的实例**，再进行转型。

###### `instanceof` 的作用：

判断一个对象是否是某个类（或接口）的实例，返回 `boolean` 值。  
语法：`对象 instanceof 类名/接口名`

###### 安全转型示例：

```java
Animal animal = new Dog();

// 先判断，再转型（避免异常）
if (animal instanceof Dog) {
    Dog dog = (Dog) animal; // 安全转型
    dog.bark();
} else if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.meow();
}
```

###### 注意点：

1. `instanceof` 会自动处理 `null`：若对象为 `null`，则返回 `false`（避免空指针异常）。  
   ```java
   Animal animal = null;
   if (animal instanceof Dog) { ... } // 结果为false，不执行转型
   ```
2. 不要滥用向下转型：频繁的向下转型通常意味着设计不够合理（应尽量通过多态的抽象方法解决，而非强制转型）。

##### 总结

- **向上转型**：子类 → 父类，自动完成，安全，是多态的基础；  
- **向下转型**：父类 → 子类，需强制转换，有风险，必须先用 `instanceof` 判断；  
- 核心原则：转型的本质是“告诉编译器对象的真实类型”，但前提是**对象的实际类型确实与目标类型匹配**，否则必然抛出 `ClassCastException`。

---

### **8.final关键字修饰类、方法和变量时，分别对多态有什么影响？**

* **修饰类**：`final` 类不能被继承，因此无法实现多态。
* **修饰方法**：`final` 方法不能被重写，阻止了运行时多态。
* **修饰变量**：`final` 变量表示常量，不能重新赋值，影响代码的可变性。

---

### **9.什么是动态绑定？Java中动态绑定的实现原理是什么？**

* **动态绑定**：在程序运行时决定调用哪个方法。
* **实现原理**：Java 通过方法的虚拟调用机制（虚拟机查找方法表）实现动态绑定，在运行时根据对象的实际类型调用方法。



##### 一、动态绑定的本质

动态绑定（又称“后期绑定”）是指：**在程序运行时，JVM根据对象的实际类型（而非声明类型），自动选择并调用对应类的方法**。

它的核心场景是**方法重写**：当父类引用指向子类对象时，调用被重写的方法会自动执行子类的实现，而非父类的实现。

示例：
```java
class Animal {
    void eat() { System.out.println("动物吃东西"); }
}
class Dog extends Animal {
    @Override
    void eat() { System.out.println("狗吃骨头"); }
}

public class Test {
    public static void main(String[] args) {
        Animal animal = new Dog(); // 父类引用指向子类对象
        animal.eat(); // 运行时调用Dog的eat()，而非Animal的
    }
}
```
上述代码中，`animal`的声明类型是`Animal`，但实际类型是`Dog`，动态绑定确保执行的是`Dog`的`eat()`。

##### 二、Java动态绑定的实现原理

JVM通过**方法表（Method Table）** 和**动态方法调用指令**实现动态绑定，具体过程分为以下几步：

###### 1. 类加载时生成方法表

当类被加载到JVM时，虚拟机会为每个类创建一张**方法表**（本质是一个数组），存储该类所有方法的引用（包括从父类继承的方法和自己重写/新增的方法）。

- 子类的方法表会继承父类的方法表；
- 若子类重写了父类的方法，子类方法表中对应的位置会被**子类方法的引用覆盖**；
- 子类新增的方法会被添加到方法表的末尾。

例如：
- `Animal`的方法表：`[eat() -> Animal.eat()]`
- `Dog`继承`Animal`后，重写`eat()`，其方法表变为：`[eat() -> Dog.eat()]`（覆盖父类方法）

###### 2. 运行时确定对象的实际类型

当程序执行到`animal.eat()`时，JVM首先确定`animal`引用指向的**实际对象类型**（即`Dog`，通过对象头中的类型指针获取）。

###### 3. 查找方法表并调用方法

JVM根据实际类型（`Dog`）找到对应的方法表，在表中查找`eat()`方法的引用（此时找到的是`Dog.eat()`），最终执行该方法。

这个过程与静态绑定（编译时确定方法）的区别：
- 静态绑定（如静态方法、私有方法、构造方法）：编译时直接确定方法地址，无需查找方法表；
- 动态绑定：编译时仅确定方法签名，运行时通过实际类型的方法表查找具体实现。

###### 4. 关键指令：`invokevirtual`

Java字节码中，调用非私有、非静态、非final的方法时，使用`invokevirtual`指令——这是触发动态绑定的核心指令。

`invokevirtual`的执行逻辑：
1. 找到操作数栈顶的对象引用，确定其实际类型；
2. 在该类型的方法表中查找匹配的方法；
3. 若找到，直接调用；若未找到，向上遍历父类方法表查找（直到`Object`类）；
4. 若始终未找到，抛出`NoSuchMethodError`。

##### 三、动态绑定的例外情况

以下情况不会触发动态绑定（采用静态绑定）：
- **静态方法**：属于类而非对象，编译时直接绑定到类；
- **私有方法**：子类无法访问，不存在重写，编译时绑定；
- **final方法**：被`final`修饰的方法不能被重写，编译时确定；
- **构造方法**：属于特定类，编译时绑定。

##### 总结

动态绑定的核心是**“运行时根据对象实际类型查找方法表并调用方法”**，其底层依赖类加载时生成的方法表和`invokevirtual`指令。这一机制使Java能够实现运行时多态，让代码更灵活、更易于扩展——通过父类引用即可调用不同子类的实现，无需关心具体类型。

---

### **10.接口中的方法默认是public abstract，JDK 8之后对接口做了哪些扩展？这些扩展对面向对象设计有什么影响？**

java 接口的修饰符默认是public abstract，接口默认隐式为 `abstract`（即使不写，编译器也会自动添加），显式声明也合法（表示接口是抽象的，需要被实现）。*`final` 不能修饰接口（`final` 类不可被继承 / 实现，与接口 “被实现” 的本质冲突）。接口的修饰符也不能使用private和protected*

* **JDK 8 扩展**：

  * **默认方法（default）**：接口可以有方法的默认实现。
  * **静态方法（static）**：接口可以定义静态方法。
  * **私有方法（private）**：接口可以定义私有方法来避免代码重复。
* **影响**：提供了更灵活的接口设计，允许在接口中添加实现而不破坏现有的实现，增加了代码的可维护性。

---

### **11.什么是依赖注入？它体现了面向对象的哪些设计原则？**

* **依赖注入**：一种设计模式，允许将类所依赖的对象通过外部配置注入，而不是在类内部创建。这减少了类之间的耦合。
* **设计原则**：

  * **单一职责原则**：每个类专注于某个责任，依赖注入可以减少类之间的相互依赖。
  * **开闭原则**：依赖注入让系统在扩展时更容易，而不需要修改现有代码。

---

### **12.面向对象的五大设计原则（SOLID）分别是什么？请简要说明每个原则的含义。**

* **S**: 单一职责原则（SRP）：每个类只做一件事。
* **O**: 开闭原则（OCP）：对扩展开放，对修改关闭。
* **L**: 里氏替换原则（LSP）：子类对象可以替换父类对象，且程序行为不变。
* **I**: 接口隔离原则（ISP）：不应该强迫客户依赖它们不使用的方法。
* **D**: 依赖倒置原则（DIP）：高层模块不依赖于低层模块，二者都依赖于抽象。

##### 1. 单一职责原则（Single Responsibility Principle, SRP）

**含义**：一个类应该只有**一个引起它变化的原因**，即只负责一项明确的职责或功能。  
**核心**：避免“大而全”的类，将不同的职责分离到不同的类中。  

**示例**：  

- 错误设计：一个 `User` 类同时负责用户信息管理（如 `setName()`）和用户数据持久化（如 `saveToDatabase()`）。  
- 正确设计：拆分为 `User` 类（仅管信息）和 `UserRepository` 类（仅管数据库操作），两者职责单一，修改其中一个不会影响另一个。  

**好处**：降低类的复杂度，提高代码可读性和可维护性，减少修改时的副作用。

##### 2. 开闭原则（Open-Closed Principle, OCP）

**含义**：软件实体（类、模块、接口等）应该**对扩展开放，对修改关闭**。即新增功能时，应通过扩展现有代码实现，而非修改已有代码。  
**核心**：依赖抽象（父类/接口）而非具体实现，通过多态扩展功能。  

**示例**：  
- 错误设计：一个 `ShapeCalculator` 类的 `calculateArea()` 方法通过 `if-else` 判断图形类型（`Circle`/`Rectangle`），新增 `Triangle` 需修改该方法。  
- 正确设计：定义 `Shape` 接口（含 `calculateArea()`），`Circle`/`Rectangle`/`Triangle` 实现接口；`ShapeCalculator` 仅调用 `Shape` 的方法，新增图形只需新增实现类，无需修改计算器。  

**好处**：减少对现有代码的修改，降低引入 bugs 的风险，适应需求变化。

##### 3. 里氏替换原则（Liskov Substitution Principle, LSP）

**含义**：子类对象可以**完全替换父类对象**（即任何使用父类的地方，都能无差别地使用其子类），且不会改变程序的正确性。  
**核心**：子类必须遵守父类的行为契约，不能破坏父类的预期功能。  

**示例**：  
- 错误设计：父类 `Bird` 有 `fly()` 方法，子类 `Penguin`（企鹅）继承后重写 `fly()` 抛出“不能飞”异常——此时用 `Penguin` 替换 `Bird` 会导致程序错误。  
- 正确设计：拆分 `Bird` 为 `FlyingBird`（含 `fly()`）和 `NonFlyingBird`，`Penguin` 继承 `NonFlyingBird`，避免违反父类契约。  

**好处**：保证继承关系的合理性，确保多态机制正确运行，增强代码的健壮性。

##### 4. 接口隔离原则（Interface Segregation Principle, ISP）

**含义**：客户端不应该被迫依赖它**不需要的接口方法**。即接口应最小化，避免设计庞大的“万能接口”。  
**核心**：将大接口拆分为多个小而专的接口，客户端仅依赖自己需要的接口。  

**示例**：  
- 错误设计：一个 `Worker` 接口包含 `work()`、`eat()`、`sleep()` 方法，让 `Robot`（机器人）实现该接口——但 `Robot` 不需要 `eat()` 和 `sleep()`，却被迫实现它们（可能空实现）。  
- 正确设计：拆分接口为 `Workable`（`work()`）、`Eatable`（`eat()`）、`Sleepable`（`sleep()`），`Human` 实现所有接口，`Robot` 仅实现 `Workable`。  

**好处**：减少接口冗余，避免客户端依赖无关方法，降低类之间的耦合度。

##### 5. 依赖倒置原则（Dependency Inversion Principle, DIP）

**含义**：高层模块不依赖低层模块，两者都应依赖**抽象**；抽象不依赖细节，细节依赖抽象。  
**核心**：通过抽象（接口/抽象类）连接高层和低层模块，而非直接依赖具体实现。  

**示例**：  
- 错误设计：高层模块 `OrderService` 直接依赖低层模块 `MySQLRepository`（具体数据库实现），若替换为 `MongoDBRepository`，需修改 `OrderService`。  
- 正确设计：定义抽象 `Repository` 接口，`MySQLRepository` 和 `MongoDBRepository` 实现它；`OrderService` 仅依赖 `Repository` 接口，替换数据库时只需切换实现类，无需修改高层。  

**好处**：降低模块间的耦合，使高层模块更稳定，便于替换低层实现（如切换数据库、框架等）。

##### 总结

SOLID 原则的核心目标是**降低耦合、提高内聚**，使代码更灵活、更易于维护和扩展：  
- 单一职责和接口隔离关注“拆分”（降低复杂度）；  
- 开闭原则和依赖倒置关注“抽象”（支持扩展）；  
- 里氏替换原则关注“继承合理性”（保证多态正确运行）。  

实际开发中，需结合业务场景灵活应用，而非机械遵守——原则是指导，而非约束。

我们用“电商订单系统”的设计过程来理解SOLID原则，通过“反例→改进”的对比，直观感受每个原则的作用：

##### 场景：设计一个简单的订单处理系统

初始需求：创建订单、计算金额、保存订单到数据库、发送邮件通知。

###### 1. 单一职责原则（SRP）

**反例**：一个`Order`类包揽所有工作：
```java
class Order {
    private List<Product> products;
    
    // 计算金额（职责1）
    public double calculateTotal() { ... }
    
    // 保存到数据库（职责2）
    public void saveToDB() { ... }
    
    // 发送邮件（职责3）
    public void sendEmail() { ... }
}
```
问题：修改数据库逻辑时可能影响金额计算，违反“单一职责”。

**改进**：按职责拆分3个类：
- `Order`：仅管理订单信息（如商品列表）
- `OrderCalculator`：专门计算金额
- `OrderRepository`：专门处理数据库操作
- `EmailService`：专门负责发送邮件

###### 2. 开闭原则（OCP）

**反例**：计算金额时硬编码支持的支付方式：
```java
class OrderCalculator {
    public double calculate(Order order, String paymentType) {
        if (paymentType.equals("ALIPAY")) {
            return order.getTotal() * 0.95; // 支付宝95折
        } else if (paymentType.equals("WECHAT")) {
            return order.getTotal() * 0.96; // 微信96折
        }
        // 新增支付方式需修改此处
        return order.getTotal();
    }
}
```
问题：新增“银联支付”时必须修改`calculate`方法，违反“对修改关闭”。

**改进**：通过接口扩展：
```java
// 抽象接口（对扩展开放）
interface Payment {
    double discount(double total);
}

// 具体实现
class Alipay implements Payment {
    public double discount(double total) { return total * 0.95; }
}
class WechatPay implements Payment {
    public double discount(double total) { return total * 0.96; }
}

// 计算器依赖抽象，无需修改（对修改关闭）
class OrderCalculator {
    public double calculate(Order order, Payment payment) {
        return payment.discount(order.getTotal());
    }
}
```
新增支付方式只需新增`Payment`实现类，无需修改计算器。

###### 3. 里氏替换原则（LSP）

**反例**：子类破坏父类行为：
```java
// 父类：正常订单
class NormalOrder {
    public double getTotal() {
        return products.stream().mapToDouble(p -> p.price).sum();
    }
}

// 子类：特价订单（错误实现）
class DiscountOrder extends NormalOrder {
    @Override
    public double getTotal() {
        if (products.isEmpty()) {
            throw new RuntimeException("空订单不能计算"); // 父类允许空订单返回0
        }
        return super.getTotal() * 0.8;
    }
}
```
问题：父类允许空订单（返回0），但子类抛出异常，此时用`DiscountOrder`替换`NormalOrder`会导致程序错误，违反LSP。

**改进**：子类必须遵守父类契约：

```java
class DiscountOrder extends NormalOrder {
    @Override
    public double getTotal() {
        // 保持与父类一致：空订单返回0
        if (products.isEmpty()) return 0; 
        return super.getTotal() * 0.8;
    }
}
```

###### 4. 接口隔离原则（ISP）

**反例**：设计一个“万能接口”：
```java
// 大而全的接口，包含所有可能的操作
interface OrderService {
    void createOrder();
    void cancelOrder();
    void refundOrder();
    void exportOrder(); // 导出订单（普通用户用不到）
}

// 普通用户服务被迫实现不需要的方法
class UserOrderService implements OrderService {
    public void createOrder() { ... }
    public void cancelOrder() { ... }
    public void refundOrder() { ... }
    public void exportOrder() { 
        throw new UnsupportedOperationException("普通用户不能导出"); // 被迫实现
    }
}
```
问题：普通用户用不到`exportOrder`，却被迫依赖该方法，违反ISP。

**改进**：拆分接口：
```java
// 基础接口（所有用户都需要）
interface BasicOrderService {
    void createOrder();
    void cancelOrder();
    void refundOrder();
}

// 扩展接口（仅管理员需要）
interface AdvancedOrderService {
    void exportOrder();
}

// 普通用户仅依赖基础接口
class UserOrderService implements BasicOrderService { ... }

// 管理员服务可依赖扩展接口
class AdminOrderService implements BasicOrderService, AdvancedOrderService { ... }
```

###### 5. 依赖倒置原则（DIP）

**反例**：高层模块依赖低层具体实现：
```java
// 低层模块：具体数据库实现
class MySQLOrderRepository {
    public void save(Order order) { ... }
}

// 高层模块：订单服务直接依赖MySQL
class OrderService {
    private MySQLOrderRepository repository = new MySQLOrderRepository();
    
    public void createOrder(Order order) {
        repository.save(order); // 直接依赖具体数据库
    }
}
```
问题：若换成`MongoDB`，必须修改`OrderService`，违反DIP。

**改进**：依赖抽象而非具体：
```java
// 抽象接口
interface OrderRepository {
    void save(Order order);
}

// 低层模块实现抽象
class MySQLOrderRepository implements OrderRepository { ... }
class MongoDBOrderRepository implements OrderRepository { ... }

// 高层模块依赖抽象
class OrderService {
    private OrderRepository repository;
    
    // 通过构造器注入抽象（而非直接new具体类）
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
    
    public void createOrder(Order order) {
        repository.save(order); // 不关心具体是MySQL还是MongoDB
    }
}
```
切换数据库时，只需传入不同的`OrderRepository`实现，高层模块无需修改。

###### 一句话记忆总结

- **SRP**：一个类只干一件事（订单类只管订单信息）  
- **OCP**：加功能只加代码，不改旧代码（加支付方式只加类）  
- **LSP**：子类能当父类用，还不捣乱（特价订单不能比普通订单多抛异常）  
- **ISP**：接口别太大，用啥给啥（普通用户别逼他实现管理员功能）  
- **DIP**：高层低层都找抽象当靠山（订单服务只认“保存接口”，不管具体数据库）

---

### **13.什么是重载和重写？它们在多态实现中分别扮演什么角色？**

* **重载（Overloading）**：同一方法名，根据参数类型或数量的不同来区分。属于编译时多态。
* **重写（Overriding）**：子类重新定义父类的方法，方法签名相同。属于运行时多态。

---

### **14.什么是抽象方法？抽象方法与普通方法相比有什么特点？为什么抽象类中可以包含非抽象方法？**

* **抽象方法**：没有方法体的方法，必须在子类中实现。
* **区别**：抽象方法没有实现，而普通方法有实现。
* **非抽象方法**：抽象类中可以有实现方法，以便提供默认的行为，子类可以选择重写。

---

### **15.如何理解“里氏替换原则”？违反里氏替换原则会导致什么问题？**

* **里氏替换原则**：子类可以替换父类对象，并且程序的行为不变。
* **违反的后果**：可能会破坏程序的正常行为，引入错误。

---

### **16.什么是接口隔离原则？它与单一职责原则有什么区别和联系？**

* **接口隔离原则**：一个接口不应该强迫实现类依赖不需要的方法。
* **区别**：单一职责原则关注类的职责，接口隔离原则关注接口的设计。
* **联系**：两者都强调高内聚低耦合的设计思想。

---

### **17.什么是多态的“开-闭原则”？如何通过多态实现代码的扩展性？**

- **开-闭原则**：对扩展开放，对修改关闭。通过多态，可以在不修改原有代码的情况下扩展系统功能。
- **实现方式**：通过继承和接口的多态性，可以通过扩展新类或接口来实现新功能，而不影响已有代码。

---

### **18.在继承关系中，构造方法的调用顺序是怎样的？为什么子类构造方法会默认调用父类的无参构造方法？**

* **调用顺序**：首先调用父类的构造方法，再调用子类的构造方法。
* **原因**：父类的构造方法初始化父类的状态，子类必须确保父类的状态初始化正确。

---

### **19.什么是匿名内部类？它在实现多态时有什么应用场景？**

* **匿名内部类**：没有类名的类，通常用于实现接口或继承类的简单场景。
* **应用场景**：当只需要实现一次性功能时（如事件监听器、回调函数等）。

---

### **20.请举例说明如何通过面向对象的设计思想解决一个实际问题（如电商系统中的商品与订单关系）。**

* **设计思路**：使用面向对象设计，定义 `Product` 类表示商品，`Order` 类表示订单，通过组合关系表示订单中包含多个商品，使用多态和继承来处理不同类型的商品（如电子商品、实体商品等）。



### 21.public、protected、private以及默认修饰符

在 Java 面向对象编程中，`public`、`protected`、`private` 和默认权限（无修饰符）是**访问控制修饰符**，用于限制类、属性和方法的可见范围，核心作用是封装（隐藏内部实现，只暴露必要接口）。

在Java中，private和protected修饰符不能直接应用于外部类（即顶级类）。具体说明如下：

1. private
   - 只能用于修饰类的成员（变量、方法、构造方法），表示类内部可见。
   - 不能修饰类本身，因为类的可见性需通过public或默认（包级私有）控制
2. protected
   - 用于修饰类的成员时，表示同一包内或子类可见。
   - 不能修饰外部类，外部类只能通过public或默认修饰符定义可见性
3. public和final
   - public可修饰类，表示类对所有包可见。
   - final可修饰类，表示该类不可被继承

**`private` 只修饰内部类，不能修饰外部类，没有内部类的情况下也不能修饰外部类**

- **作用范围**：仅能在**外部类的内部**访问该内部类，外部类之外的任何地方（包括同一包中的其他类、外部类的子类）都无法访问。

 **`protected` 只能修饰内部类，不能修饰外部类，没有内部类的情况下也不能修饰外部类**

- 作用范围：
  - 在**外部类内部**可以直接访问；
  - 在**与外部类同一包的其他类**中可以访问；
  - 在**外部类的子类**中（无论是否同包）可以访问。

它们的权限范围从大到小排序为：`public` > `protected` > 默认权限（包访问权限） > `private`。

#### 1. `public`（公共权限）

- **权限范围**：所有地方可见（同一类、同一包、不同包的类均可访问）。
- **适用场景**：需要被广泛访问的类、属性或方法（如工具类的核心方法、接口的公开方法）。

示例：
```java
public class PublicDemo {
    public String publicField = "公共字段";
    
    public void publicMethod() {
        System.out.println("这是公共方法");
    }
}

// 不同包的类也可访问
package other;
import com.example.PublicDemo;

public class Test {
    public static void main(String[] args) {
        PublicDemo demo = new PublicDemo();
        System.out.println(demo.publicField); // 合法
        demo.publicMethod(); // 合法
    }
}
```

#### 2. `protected`（受保护权限）

- **权限范围**：
  - 同一类中可见；
  - 同一包中的类可见；
  - 不同包中的**子类**可见（通过子类实例或继承访问）。
- **适用场景**：需要被子类继承或同包类访问的成员（如父类的核心属性或方法）。

示例：
```java
package com.example;
public class Parent {
    protected String protectedField = "受保护字段";
    
    protected void protectedMethod() {
        System.out.println("这是受保护方法");
    }
}

// 同一包的类可访问
package com.example;
public class SamePackageClass {
    public void test() {
        Parent parent = new Parent();
        System.out.println(parent.protectedField); // 合法
    }
}

// 不同包的子类可访问
package other;
import com.example.Parent;

public class Child extends Parent {
    public void test() {
        // 访问父类的protected成员（通过继承）
        System.out.println(protectedField); // 合法
        protectedMethod(); // 合法
        
        // 通过父类实例访问则不允许
        Parent parent = new Parent();
        // System.out.println(parent.protectedField); // 编译错误
    }
}
```

#### 3. 默认权限（包访问权限，无修饰符）

- **权限范围**：仅同一包中的类可见（同一类、同一包的其他类可访问，不同包的类不可访问，包括子类）。
- **适用场景**：仅需在同一包内协同工作的类或成员（如包内工具类、辅助类）。

示例：
```java
package com.example;
// 无修饰符，默认权限
class DefaultClass {
    String defaultField = "默认字段";
    
    void defaultMethod() {
        System.out.println("这是默认方法");
    }
}

// 同一包的类可访问
package com.example;
public class SamePackageTest {
    public void test() {
        DefaultClass obj = new DefaultClass();
        System.out.println(obj.defaultField); // 合法
    }
}

// 不同包的类不可访问
package other;
import com.example.DefaultClass; // 编译错误（DefaultClass不是public）

public class OtherPackageTest {
    public void test() {
        // DefaultClass obj = new DefaultClass(); // 编译错误
    }
}
```

#### 4. `private`（私有权限）

- **权限范围**：仅当前类内部可见（其他类，包括子类和同包类，均不可访问）。
- **适用场景**：类的内部实现细节（如私有属性、工具方法），需通过 `public`/`protected` 方法间接访问（封装的核心）。

示例：
```java
public class PrivateDemo {
    private String privateField = "私有字段";
    
    private void privateMethod() {
        System.out.println("这是私有方法");
    }
    
    // 内部可访问
    public void accessPrivate() {
        System.out.println(privateField); // 合法
        privateMethod(); // 合法
    }
}

public class OtherClass {
    public void test() {
        PrivateDemo demo = new PrivateDemo();
        // System.out.println(demo.privateField); // 编译错误
        // demo.privateMethod(); // 编译错误
    }
}
```

#### 权限范围对比表

| 修饰符       | 同一类中 | 同一包中 | 不同包的子类 | 不同包的非子类 |
|--------------|----------|----------|--------------|----------------|
| `public`     | ✅        | ✅        | ✅            | ✅              |
| `protected`  | ✅        | ✅        | ✅            | ❌              |
| 默认（无修饰）| ✅        | ✅        | ❌            | ❌              |
| `private`    | ✅        | ❌        | ❌            | ❌              |

#### 核心原则

- **封装优先**：尽可能使用更严格的权限（如 `private`），仅暴露必要的接口（`public`）。
- **继承控制**：`protected` 用于允许子类扩展父类功能，但限制非子类访问。
- **包内协同**：默认权限适合包内类之间的协作，避免外部干扰。

合理使用访问修饰符是面向对象封装特性的核心体现，能提高代码的安全性、可维护性和复用性。



### 22.内部类和外部类

在Java中，内部类（Inner Class）是定义在另一个类（称为外部类，Outer Class）内部的类。这种嵌套结构允许内部类与外部类建立紧密的关联，便于访问彼此的成员，同时也能实现代码的封装和逻辑聚合。

#### 一、内部类与外部类的基本关系

1. **语法结构**  
   内部类定义在外部类的类体中，形式如下：
   ```java
   class Outer { // 外部类
       // 外部类的成员（属性、方法等）
       class Inner { // 内部类
           // 内部类的成员
       }
   }
   ```

2. **访问权限**  
   - **内部类访问外部类**：内部类可以直接访问外部类的所有成员（包括私有`private`成员），无需特殊语法。  
     例：内部类`Inner`可直接使用外部类`Outer`的`private`属性`a`。
   - **外部类访问内部类**：外部类不能直接访问内部类的成员，必须通过内部类的实例对象访问（即使内部类成员是`public`）。  
     例：外部类需通过`new Inner().b`访问内部类的`public`属性`b`。

#### 二、内部类的分类

根据声明方式和特性，内部类可分为以下4种：

##### 1. 成员内部类（Member Inner Class）

- 定义在外部类的成员位置（与外部类的属性、方法同级），无`static`修饰。
- **特性**：
  - 依赖外部类实例：必须先创建外部类实例，才能创建成员内部类实例。  
    例：`Outer outer = new Outer(); Outer.Inner inner = outer.new Inner();`
  - 可访问外部类的所有成员（包括`private`），外部类需通过内部类实例访问内部类成员。
  - 可被`private`、`protected`、`public`或默认访问修饰符修饰（控制其他类对它的访问）。

##### 2. 静态内部类（Static Nested Class）

- 定义在外部类的成员位置，有`static`修饰（是唯一带`static`的内部类）。
- **特性**：
  - 不依赖外部类实例：可直接通过`外部类名.内部类名`创建实例，无需外部类对象。  
    例：`Outer.StaticInner inner = new Outer.StaticInner();`
  - 只能访问外部类的`static`成员（不能直接访问非静态成员）。
  - 相当于外部类的“静态成员”，可独立存在，常用于与外部类逻辑相关但无需依赖实例的场景（如工具类）。

##### 3. 局部内部类（Local Inner Class）

- 定义在外部类的方法或代码块内部（局部位置）。
- **特性**：
  - 作用域仅限于当前方法/代码块，外部无法直接访问（无访问修饰符）。
  - 可访问外部类的所有成员，以及所在方法的`final`（或 effectively final）局部变量（JDK 8+ 允许隐式final）。
  - 常用于创建仅在当前方法中使用的临时类（如回调对象）。

##### 4. 匿名内部类（Anonymous Inner Class）

- 没有类名的局部内部类，通常用于快速创建接口或抽象类的实例（一次性使用）。
- **特性**：
  - 语法紧凑：`new 父类/接口() { 实现方法 };`  
    例：`Runnable r = new Runnable() { public void run() {} };`
  - 隐式继承父类或实现接口，只能实现一个父类/接口。
  - 作用域同局部内部类，且不能有构造方法（因无类名）。

#### 三、内部类的核心作用

1. **封装性**：内部类可被声明为`private`，仅外部类可见，隐藏实现细节。  
   例：外部类的辅助逻辑可封装在私有内部类中，避免暴露给其他类。
2. **便捷访问**：内部类直接访问外部类成员，无需通过`getter`/`setter`，简化代码。
3. **逻辑聚合**：将与外部类强关联的类定义在内部，体现“部分-整体”关系（如`Map.Entry`是`Map`的内部类，代表键值对）。
4. **回调机制**：匿名内部类常用于事件监听、线程创建等场景，简化回调代码。

#### 四、注意事项

- 内部类会持有外部类的引用（成员内部类），若内部类生命周期过长，可能导致外部类实例无法被GC回收，引发内存泄漏（静态内部类无此问题）。
- 局部内部类和匿名内部类访问的局部变量必须是`final`或 effectively final（值不会被修改），确保变量在内部类中引用时状态一致。
- 静态内部类与外部类的关系更松散，仅通过类名关联，适合定义独立的工具方法或数据结构。

总之，内部类是Java中增强封装性和代码组织性的重要机制，合理使用可使代码更简洁、逻辑更清晰，但需注意其与外部类的依赖关系和生命周期问题。

#### 匿名内部类考法

```java
public class Main {
    private int num = 10;
 
    public static void main(String[] args) {
        Main main = new Main() {
            @Override
            public void printNum() {
                System.out.println(num);
            }
        };
        main.printNum();
    }
 
    public void printNum() {
        System.out.println(num);
    }
}
```

**1. 代码结构**

- 类Main有一个实例变量private int num = 10。
- 有一个public void printNum()方法，打印num。
- 在main方法中，创建了一个Main的**匿名子类**，并重写了printNum()方法。

------

**2. 问题所在**

在匿名类中重写的方法里，直接访问了num：

```java
@Override
public void printNum() {
    System.out.println(num);
}
```

这里的num是Main类的实例变量，但它是private的。
在子类（即使是匿名子类）中，**不能直接访问父类的私有成员**。

------

**3. 编译错误信息**

如果尝试编译，会得到类似这样的错误：

```
Main.java:8: 错误: num 在 Main 中是 private 访问控制
                System.out.println(num);
                                   ^
```

因为num是private，在子类中无法直接访问。

------

**4. 修正方式**

如果要在子类中访问num，可以：

- 将num改为protected或public。
- 或者通过父类的公有/受保护的方法来访问（例如调用super.printNum()或提供一个getNum()方法）。

------

**5. 结论**

这段代码在编译时就会出错，因为子类无法直接访问父类的私有字段。

**正确答案是：C. 编译错误** ✅

### 23.向上转型 & 向下转型

在Java中，**向上转型（Upcasting）** 和**向下转型（Downcasting）** 是与类的继承关系相关的类型转换机制，用于在父类和子类类型之间进行转换，核心目的是实现多态和类型适配。

**一句话精准解释（面试专用）**

1. **向上转型**：把子类对象赋值给父类引用（自动完成），体现 "is-a" 关系，比如`Animal animal = new Cat();`，一句话总结：**向上转型是子类对象自动转为父类类型，只能调用父类定义的属性 / 方法，是多态的基础**。
2. **向下转型**：把父类引用强转回子类类型（必须手动强转），一句话总结：**向下转型是父类类型强制转回子类类型，目的是调用子类特有方法，但需先通过 instanceof 判断类型避免 ClassCastException**。

**补充面试加分版（更简洁）**

- 向上转型：子类→父类，自动转，看得到父类的东西，丢子类特有能力；
- 向下转型：父类→子类，强制转，要先判断类型，找回子类特有能力。

**总结**

1. 向上转型核心：自动、父类视角、多态基础；
2. 向下转型核心：强制、需 instanceof 校验、恢复子类特有行为。

#### 一、向上转型（Upcasting）

**定义**：将子类对象赋值给父类类型的变量（或作为父类类型的返回值/参数），属于**自动转换**，无需显式声明。

##### 语法：

```java
父类类型 变量名 = new 子类类型();
```

##### 示例：

```java
class Animal { // 父类
    public void eat() {
        System.out.println("动物吃东西");
    }
}

class Dog extends Animal { // 子类
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }
    public void bark() { // 子类特有方法
        System.out.println("狗叫");
    }
}

// 向上转型：Dog对象转换为Animal类型
Animal animal = new Dog(); // 自动转换，无需强制类型转换
```

##### 特点：

1. **安全且自动**：向上转型是编译器允许的默认行为，不会出现运行时错误。
2. **“缩小”可见性**：转型后，变量只能访问父类中定义的成员（包括被子类重写的方法），**无法访问子类特有成员**。  
   例如：`animal.bark()` 会编译报错（`bark()` 是Dog特有方法）。
3. **多态的基础**：向上转型后，调用父类方法时，实际执行的是子类的重写实现（动态绑定），这是多态的核心体现。  
   例如：`animal.eat()` 会输出 `“狗吃骨头”`（执行Dog的重写方法）。

#### 二、向下转型（Downcasting）

**定义**：将父类类型的变量（实际指向子类对象）转换为子类类型，属于**强制转换**，必须显式声明。

##### 语法：

```java
子类类型 变量名 = (子类类型) 父类变量;
```

##### 示例：

```java
// 基于上面的Animal和Dog类
Animal animal = new Dog(); // 先向上转型

// 向下转型：将Animal类型转换为Dog类型（必须显式强制转换）
Dog dog = (Dog) animal; 
dog.bark(); // 此时可调用子类特有方法，输出：“狗叫”
```

##### 特点：

1. **需要显式强制转换**：必须通过 `(子类类型)` 声明，否则编译报错。
2. **“扩大”可见性**：转型后，变量可以访问子类的所有成员（包括继承自父类和自身特有的）。
3. **安全性依赖实际类型**：  
   - 只有当父类变量**实际指向的是子类对象**时，向下转型才安全，否则会抛出 `ClassCastException`（运行时异常）。  
   - 例如：`Animal animal = new Animal(); Dog dog = (Dog) animal;` 会抛异常（animal实际是Animal对象，无法转为Dog）。

##### 安全转型的前提：使用 `instanceof` 判断

为避免 `ClassCastException`，向下转型前通常用 `instanceof` 运算符判断父类变量的实际类型：
```java
if (animal instanceof Dog) { // 判断animal实际是否为Dog对象
    Dog dog = (Dog) animal; // 安全转型
    dog.bark();
} else {
    System.out.println("无法转换为Dog类型");
}
```
- `instanceof` 作用：判断左边对象是否为右边类（或其子类）的实例，返回 `boolean`。

#### 三、转型的核心目的

1. **向上转型**：  
   - 简化代码：通过父类类型统一接收不同子类对象，体现“抽象编程”思想（如用 `List` 接收 `ArrayList`、`LinkedList`）。  
   - 实现多态：让同一父类类型的变量调用方法时，根据实际子类对象表现出不同行为。

2. **向下转型**：  
   - 恢复子类特性：当需要使用**子类特有的方法或属性时（即父类没有的方法）**，通过向下转型“还原”子类类型。  
   - 场景：多态场景中，已知父类变量实际指向某子类对象，需调用该子类特有功能（如集合框架中对特定实现类的操作）。

#### 四、常见误区

- 误以为“父类对象可以直接转为子类”：**错误**。向下转型的前提是父类变量实际指向的是子类对象，纯父类对象无法转为子类（如 `new Animal()` 不能转为 `Dog`）。
- 过度使用向下转型：向下转型会破坏多态的抽象性，增加代码耦合度，应尽量通过父类接口设计逻辑，减少转型需求。

#### 总结

| 类型       | 转换方向       | 语法要求       | 安全性               | 核心用途                     |
|------------|----------------|----------------|----------------------|------------------------------|
| 向上转型   | 子类 → 父类    | 自动转换       | 绝对安全             | 实现多态、统一接口接收       |
| 向下转型   | 父类 → 子类    | 强制转换       | 需用 `instanceof` 验证 | 恢复子类特有功能             |

向上转型是多态的基础，而向下转型是多态场景中“特殊化处理”的补充，两者结合使Java的继承和多态机制更灵活。

### 24.类的加载和对象创建

在 Java 中，**类的加载**和**对象创建**是两个紧密关联关联但不同的过程**：

类加载是将类的字节码加载到内存并生成 `Class` 对象的过程（类级别的初始化），

而对象创建是基于已加载的类实例化出具体对象的过程（对象级别的初始化）。

**Java类加载的两个重要特点：**
\1. 父类优先：保证父类的静态初始化先于子类执行
\2. 静态先行：所有静态初始化都在实例创建之前完成

#### 一、类的加载（Class Loading）

类的加载是 JVM 将 `.class` 字节码文件加载到内存，并对其进行验证、准备、解析，最终生成 `java.lang.Class` 对象的过程。这一过程由**类加载器（ClassLoader）** 完成，且**每个类仅被加载一次**（确保内存中只有一个 `Class` 对象）。

##### 类加载的核心步骤：

1. **加载（Loading）**  
   - 类加载器根据类的全限定名（如 `com.example.User`）查找 `.class` 文件（可能来自本地磁盘、网络、jar 包等）。  
   - 将字节码文件读入内存，生成一个代表该类的 `Class` 对象（存储在方法区），作为后续操作的入口。

2. **验证（Verification）**  
   
- 校验字节码的合法性（如是否符合 Java 语法规范、是否存在安全隐患），防止恶意字节码危害 JVM。
  
3. **准备（Preparation）**  
   - 为类的**静态变量**分配内存（在方法区），并设置默认初始值（如 `int` 为 0，`boolean` 为 `false`，引用类型为 `null`）。  
   - 示例：`public static int num;` 在准备阶段会被初始化为 0。

4. **解析（Resolution）**  
   
- 将类中的符号引用（如类名、方法名的字符串）转换为直接引用（内存地址），确保后续调用能直接定位到目标。
  
5. **初始化（Initialization）**  
   - 执行类的**静态代码块**和**静态变量的显式赋值**（按代码顺序执行）。  
   - 触发时机：当类被主动使用时（如创建对象、调用静态方法、访问静态变量等）。  
   - 示例：  
     ```java
     public class User {
         public static int num = 10; // 显式赋值
         static {
             System.out.println("静态代码块执行"); // 静态代码块
         }
     }
     // 当首次使用 User 类时（如 new User()），初始化阶段会先执行 num=10，再执行静态代码块
     ```

#### 二、对象的创建（Object Creation）

对象创建是基于已加载的类（即 `Class` 对象），在堆内存中实例化出具体对象的过程。一个类可以创建多个对象，每个对象拥有独立的实例变量。

##### 对象创建的核心步骤：

1. **分配内存**  
   - JVM 在堆内存中为新对象分配空间（大小在类加载时已确定）。  
   - 分配方式：指针碰撞（内存连续）或空闲列表（内存碎片化），具体取决于垃圾收集器。

2. **初始化实例变量**  
   
- 为对象的**实例变量**设置默认初始值（与类加载的“准备阶段”类似，如 `int` 为 0，引用类型为 `null`）。
  
3. **执行实例初始化**  
   - 调用构造方法，执行**实例代码块**和**实例变量的显式赋值**（按代码顺序执行），最终完成对象的初始化。  
   - 示例：  
     ```java
     public class User {
         private String name; // 实例变量
         {
             name = "默认名称"; // 实例代码块
             System.out.println("实例代码块执行");
         }
         public User(String n) {
             name = n; // 构造方法赋值（覆盖实例代码块的默认值）
         }
     }
     // 创建对象时：new User("张三") 
     // 执行顺序：实例变量默认初始化（name=null）→ 实例代码块（name="默认名称"）→ 构造方法（name="张三"）
     ```

4. **对象引用返回**  
   
   - 将堆内存中对象的地址返回给栈中的引用变量（如 `User u = new User();` 中，`u` 存储对象地址）。

#### 三、类加载与对象创建的关联

1. **依赖关系**：对象创建**必须依赖类加载完成**。只有当类被成功加载并初始化后，才能通过 `new`、反射等方式创建对象。  
2. **触发时机**：类加载通常在“首次主动使用类”时触发（可能是创建对象，也可能是调用静态方法）；而对象创建是类加载后的具体实例化操作。  
3. **内存分配**：  
   - 类加载的相关数据（`Class` 对象、静态变量、方法信息）存储在**方法区**。  
   - 对象的实例变量存储在**堆内存**，对象引用存储在**栈内存**。

#### 四、典型案例

```java
public class Demo {
    public static void main(String[] args) {
        // 首次使用 User 类，触发类加载（加载→验证→准备→解析→初始化）
        User u1 = new User(); // 类加载完成后，创建第一个对象
        User u2 = new User(); // 类已加载，直接创建第二个对象
    }
}

class User {
    static {
        System.out.println("User 类初始化（静态代码块）"); // 仅执行一次（类加载时）
    }
    {
        System.out.println("User 实例初始化（实例代码块）"); // 每个对象创建时执行一次
    }
}
```

**执行结果**：  
```
User 类初始化（静态代码块）  // 类加载时执行，仅一次
User 实例初始化（实例代码块） // 创建 u1 时执行
User 实例初始化（实例代码块） // 创建 u2 时执行
```

#### 总结

- **类加载**：将类的字节码加载到内存，生成 `Class` 对象，初始化静态资源（仅一次）。  
- **对象创建**：基于已加载的类，在堆中分配内存，初始化实例资源（每个对象独立执行）。  
- 二者的核心区别：类加载是“类级别的初始化”，对象创建是“实例级别的初始化”，且对象创建依赖于类加载的完成。

### 25.对象的等于、排序和toString()

在 Java 中，**对象的等于（equality）、排序（ordering）和字符串表示（`toString`）** 是三个核心行为，分别通过重写以下三个方法来定制：

- `equals(Object obj)`：定义对象“相等”的逻辑  
- `compareTo(T o)` 或 `Comparator`：定义对象的排序规则  
- `toString()`：定义对象的字符串表示

------

#### 一、`equals()` —— 对象是否“相等”

##### ✅ 默认行为（来自 `Object`）

```java
public boolean equals(Object obj) {
    return (this == obj); // 仅比较引用是否相同
}
```

##### 🔁 何时重写？

当业务逻辑认为“内容相同即相等”时（如两个 `Person` 对象姓名和 ID 相同就算相等）。

##### 📜 重写规范（必须满足以下 5 条）

1. **自反性**：`x.equals(x)` → `true`  
2. **对称性**：`x.equals(y)` ⇔ `y.equals(x)`  
3. **传递性**：若 `x.equals(y)` 且 `y.equals(z)`，则 `x.equals(z)`  
4. **一致性**：多次调用结果不变（前提：对象未修改）  
5. **非空性**：`x.equals(null)` → `false`

##### ✅ 推荐写法（使用 `Objects.equals`）

```java
public class Person {
    private String name;
    private int id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return id == person.id && Objects.equals(name, person.name);
    }
}
```

> ⚠️ 重写 `equals()` 时，**必须同时重写 `hashCode()`**（见下文）。

------

#### 二、`hashCode()` —— 与 `equals()` 配套使用

##### 📜 规范

- 如果 `x.equals(y) == true`，则 `x.hashCode() == y.hashCode()`  
- 如果 `x.hashCode() != y.hashCode()`，则 `x.equals(y) == false`

##### ✅ 推荐写法

```java
@Override
public int hashCode() {
    return Objects.hash(name, id);
}
```

> 💡 用途：用于 `HashMap`、`HashSet` 等哈希结构，保证正确性和性能。

------

#### 三、排序 —— `Comparable` vs `Comparator`

##### 方式 1：实现 `Comparable<T>`（自然排序）

适用于有**唯一、明确的默认排序规则**的对象（如 `String`、`Integer`）。

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age); // 按年龄升序
    }
}
```

使用：

```java
List<Person> list = ...;
Collections.sort(list); // 自动按 age 排序
```

##### 方式 2：使用 `Comparator`（定制排序）

适用于**多种排序需求**或**无法修改类源码**的情况。

```java
// 按姓名排序
list.sort(Comparator.comparing(Person::getName));

// 多级排序：先按年龄，再按姓名
list.sort(Comparator.comparingInt(Person::getAge)
                   .thenComparing(Person::getName));
```

> ✅ 推荐：优先使用 `Comparator`（更灵活、函数式、避免污染类设计）。

------

#### 四、`toString()` —— 对象的字符串表示

##### ✅ 作用

- 调试、日志、打印时提供可读信息
- 被 `System.out.println()`、`String.valueOf()`、字符串拼接自动调用

##### ✅ 推荐格式

```java
@Override
public String toString() {
    return "Person{" +
           "name='" + name + '\'' +
           ", age=" + age +
           '}';
}
// 输出类似：com.example.Person@1b6d3586，这种默认格式对调试不友好，通常建议在自定义类中重写 toString()。
// 重写toString()后，输出：Person{name='Alice', age=30}
```

##### 🛠️ 工具辅助

- **Lombok**：`@ToString` 注解自动生成  
- **IDE**：自动生成 `toString()` 方法  
- **Apache Commons Lang**：`ToStringBuilder`

------

#### 🔗 三者关系总结

| 方法          | 用途         | 是否必须重写              | 配套要求                   |
| ------------- | ------------ | ------------------------- | -------------------------- |
| `equals()`    | 判断逻辑相等 | 按需                      | 必须重写 `hashCode()`      |
| `hashCode()`  | 支持哈希结构 | 重写 `equals()` 时必须    | 与 `equals()` 一致         |
| `compareTo()` | 自然排序     | 按需（实现 `Comparable`） | 与 `equals()` 一致（建议） |
| `toString()`  | 可读字符串   | 强烈建议                  | 无                         |

> ⚠️ **重要建议**：  
>
> - 如果 `a.equals(b) == true`，那么 `a.compareTo(b)` **应该返回 0**（虽然不是强制，但违反会导致 `TreeSet` 等集合行为异常）。  
> - `toString()` 不应抛异常、不应修改对象状态。

------

#### ✅ 完整示例（含 Lombok 简化版）

##### 手动实现：

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    // 构造、getter 省略

    @Override
    public boolean equals(Object o) { /* ... */ }

    @Override
    public int hashCode() { /* ... */ }

    @Override
    public int compareTo(Person p) {
        return Integer.compare(this.age, p.age);
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}
```

##### 使用 Lombok（推荐）：

```java
import lombok.*;

@Data
@ToString
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    @Override
    public int compareTo(Person p) {
        return Integer.compare(this.age, p.age);
    }
}
```

------

通过合理重写这三个方法，你的对象将具备：

- 正确的相等判断（可用于 `Set` 去重、`Map` 查找）  
- 灵活的排序能力（可用于 `List.sort()`、`TreeMap` 等）  
- 清晰的调试信息（提升开发效率）

这是 Java 面向对象编程的**基础且关键**的实践。

### 26.协变类型

在 Java 中，“**协变返回类型**（Covariant Return Type）” 是一个重要的面向对象特性，常出现在方法重写（Override）场景中。下面为你清晰、系统地解释 **什么是协变返回类型**，以及它与“协变类型”概念的关系。

------

#### 一、什么是协变返回类型（Covariant Return Type）？

> ✅ **定义**：
> 在子类重写父类方法时，**允许返回类型是父类方法返回类型的子类型**，这种特性称为 **协变返回类型**。

这是从 **Java 5 开始支持** 的特性。

##### 📌 关键点：

- 只适用于 **方法重写**（Override）
- 返回类型必须是 **原返回类型的子类或实现类**
- 提高了代码的 **类型安全性和可读性**

------

#### 二、示例说明

##### 父类：

```java
class Animal {
    // 返回 Animal 类型
    Animal create() {
        return new Animal();
    }
}
```

##### 子类（合法重写，使用协变返回）：

```java
class Dog extends Animal {
    // ✅ 合法！返回类型 Dog 是 Animal 的子类 → 协变返回
    @Override
    Dog create() {
        return new Dog();
    }
}
```

##### 使用：

```java
Animal a = new Animal();
Animal a2 = a.create(); // 返回 Animal

Dog d = new Dog();
Dog d2 = d.create();   // 返回 Dog —— 更精确，无需强转！
```

> 💡 如果没有协变返回，子类 `create()` 必须返回 `Animal`，调用方需手动强转：
>
> ```java
> Dog d2 = (Dog) d.create(); // 不安全、不优雅
> ```

------

#### 三、“协变类型” vs “协变返回类型”

| 概念                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **协变**（Covariance） | 泛型/类型系统中的概念：若 `A` 是 `B` 的子类型，则 `Container<A>` 是 `Container<B>` 的子类型（如 Java 中的 `List<? extends Number>`） |
| **协变返回类型**       | 特指 **方法重写时返回类型的子类型化**，是协变思想在方法返回值上的具体应用 |

> 🔍 简单说：
> **协变返回类型是“协变”在方法重写场景下的一个特例**。

------

#### 四、为什么叫“协变”？

- “协” = 一起变化  
- 当子类重写方法时，**返回类型“跟随”类的继承关系一起变得更具体**（从 `Animal` → `Dog`），即“协同变化”，故称“协变”。

------

#### 五、注意事项

1. **仅限返回类型**：参数列表必须完全相同（否则是重载，不是重写）。

2. 必须是子类型

   ：不能是无关类型。

   ```java
   class Cat extends Animal { }
   class Dog extends Animal {
       @Override
       Cat create() { } // ❌ 编译错误！Cat 不是 Dog 的子类，也不是 Animal 的“更具体”返回（对 Dog 来说）
   }
   ```

3. 接口也支持

   ：

   ```java
   interface Factory {
       Cloneable create();
   }
   class MyFactory implements Factory {
       @Override
       public String create() { return "hello"; } // ✅ String 实现了 Cloneable
   }
   ```

------

#### 六、实际应用场景

- Builder 模式

  ：

  ```java
  abstract class Builder<T extends Builder<T>> {
      public T setName(String name) {
          // ...
          return (T) this;
      }
  }
  
  class UserBuilder extends Builder<UserBuilder> {
      public UserBuilder setAge(int age) { return this; } // 返回具体类型，链式调用更流畅
  }
  ```

- **工厂方法模式**：子类工厂返回具体产品类型。

- 克隆方法

  ：

  ```java
  class Point {
      @Override
      protected Object clone() { ... }
  }
  
  class ColorPoint extends Point {
      @Override
      protected ColorPoint clone() { ... } // 协变返回，避免强转
  }
  ```

------

#### ✅ 总结

| 项目         | 说明                                               |
| ------------ | -------------------------------------------------- |
| **定义**     | 子类重写方法时，返回类型可以是父类返回类型的子类型 |
| **引入版本** | Java 5                                             |
| **作用**     | 提升类型安全，避免强制类型转换，增强 API 表达力    |
| **本质**     | 面向对象“里氏替换原则”的体现，也是“协变”思想的应用 |
| **关键规则** | 仅用于重写；返回类型必须是子类型；参数列表不变     |

> 💡 **记住一句话**：
> **“重写方法时，返回类型可以‘更具体’，但不能‘更宽泛’。”**

这个特性虽小，却是 Java 设计中体现类型安全与灵活性的重要细节，在面试和高质量代码中经常出现！



### 27.Java中构造方法调用顺序以及成员变量初始化

```JAVA
class Parent {
    public String name = "Parent";
    public Parent() {
        sayName();
    }
    public void sayName() {
        System.out.println("say hi in parent : " + name);
    }
}
 
class Son extends Parent {
    public String name = "Son";
    public Son() {
        sayName();
    }
    public void sayName() {
        System.out.println("say hi in Son : " + name);
    }
}
 
public class Test {
 
    public static void main(String[] args) {
        new Son();
    }
 
}
```

这道题目考察了Java中构造方法调用顺序以及成员变量初始化的细节。

执行 new Son() 时的过程如下：

\1. 先调用父类Parent的构造方法(这是Java的规则,子类构造前必须先构造父类)
\2. 此时Son类的成员变量name还未初始化,值为null
\3. 父类构造方法中调用sayName(),由于Java的动态绑定机制,调用的是Son类的sayName()方法
\4. 因此第一次输出 "say hi in son : null"
\5. 父类构造完成后,执行Son类的构造方法
\6. Son类的name被初始化为"Son"
\7. Son构造方法中调用sayName(),此时输出 "say hi in son : Son"

所以最终输出结果为:
say hi in son : null
say hi in son : Son

#### 一、先给出执行结果

运行这段代码，控制台输出如下：
```
say hi in Son : null
say hi in Son : Son
```

#### 二、逐行拆解执行流程（结合构造顺序+方法重写+变量初始化）

##### 步骤1：执行 `new Son()`，触发子类对象创建，先调用父类构造方法

Java 中子类构造方法默认第一行隐含 `super()`（调用父类无参构造），因此先执行 `Parent()` 构造方法。

##### 步骤2：父类初始化阶段（关键：子类成员变量未初始化）

1. 父类 `Parent` 的成员变量初始化：`public String name = "Parent";` → 父类 `name` 赋值为 "Parent"；
2. 执行父类构造方法 `Parent()`，调用 `sayName()` 方法；
   - 由于 `Son` 重写了 `sayName()` 方法，**运行时多态**会执行子类 `Son` 的 `sayName()` 方法（而非父类）；
   - 此时子类 `Son` 的构造方法还未执行，子类的成员变量 `name` 仅完成「默认初始化」（值为 `null`），未执行 `public String name = "Son";` 这一步；
   - 因此子类 `sayName()` 输出：`say hi in Son : null`。

##### 步骤3：子类初始化阶段

1. 父类构造执行完毕后，执行子类 `Son` 的成员变量初始化：`public String name = "Son";` → 子类 `name` 赋值为 "Son"；
2. 执行子类构造方法 `Son()`，调用 `sayName()` 方法；
   - 此时子类 `name` 已完成显式初始化，值为 "Son"；
   - 执行子类 `sayName()` 方法，输出：`say hi in Son : Son`。

#### 三、核心考点解析（面试高频）

##### 1. 运行时多态（方法重写）

- 父类构造方法中调用的 `sayName()`，并非父类自身的方法，而是子类重写后的版本；
- 原因：Java 中方法调用遵循「运行时类型」，此时创建的对象是 `Son` 类型，因此执行子类的 `sayName()`。

##### 2. 成员变量初始化顺序（核心坑点）

- 子类的成员变量 `name` 初始化分为两步：
  1. **默认初始化**：对象创建时，JVM 先给所有成员变量赋默认值（`String` 类型为 `null`）；
  2. **显式初始化**：父类构造执行完毕后，才会执行子类成员变量的显式赋值（`name = "Son"`）；
- 因此父类构造中调用子类方法时，子类成员变量尚未完成显式初始化，值为 `null`。

##### 3. 成员变量无多态（与方法对比）

- 方法有运行时多态，成员变量是「编译时静态绑定」：
  - 若代码中直接访问 `name`（如 `System.out.println(name)`），父类访问的是父类的 `name`，子类访问的是子类的 `name`；
  - 本例中是通过方法调用访问子类 `name`，因此体现的是子类变量的状态。

#### 四、避坑建议（开发规范）

1. **禁止在构造方法中调用可重写的方法**：
   - 父类构造调用子类重写的方法时，子类对象尚未完全初始化，易导致空指针/数据异常；
   - 若必须调用，需将方法声明为 `final`（禁止子类重写）或 `private`（无法被重写）。
   优化示例：
   ```java
   // 父类方法加 final，禁止子类重写
   public final void sayName() {
       System.out.println("say hi in parent : " + name);
   }
   ```
2. **构造方法仅做简单初始化**：避免在构造中执行复杂逻辑、调用外部方法，降低初始化风险；
3. **成员变量初始化优先用构造方法**：而非依赖显式赋值，确保初始化顺序可控。

#### 五、核心总结

1. 执行结果：先输出 `say hi in Son : null`，再输出 `say hi in Son : Son`；
2. 核心原因：父类构造调用子类重写的方法，此时子类成员变量未完成显式初始化，值为 `null`；
3. 关键知识点：方法有运行时多态，成员变量无多态；子类初始化需等父类构造执行完毕；
4. 开发禁忌：父类构造中禁止调用可被重写的方法，避免子类未初始化导致的异常。

### 27.Java构造方法调用顺序 + 成员变量初始化（结构化+实战版）

#### 一、核心结论（先总后分）

面试官您好！Java中**成员变量初始化**和**构造方法调用**遵循“先初始化成员，再执行构造逻辑；先父类，再子类”的核心规则，具体顺序可拆解为「类加载阶段的静态初始化」和「对象创建阶段的实例初始化」两部分，下面分场景详细说明：

#### 二、完整执行顺序（结合示例）

##### 1. 基础规则（无继承）

对象创建时，执行顺序为：
```
1. 静态成员变量（类变量）初始化 → 2. 静态代码块 → 3. 实例成员变量初始化 → 4. 实例代码块 → 5. 构造方法
```

##### 2. 继承场景（核心）

子类创建对象时，会先触发父类的初始化/构造，完整顺序：
```
父类静态成员变量 → 父类静态代码块 → 子类静态成员变量 → 子类静态代码块 → 
父类实例成员变量 → 父类实例代码块 → 父类构造方法 → 
子类实例成员变量 → 子类实例代码块 → 子类构造方法
```

##### 3. 实战代码验证（最易理解）

```java
// 父类
class Parent {
    // 1. 父类静态成员变量
    static String parentStaticVar = "父类静态变量初始化";
    // 3. 父类实例成员变量
    String parentInstanceVar = "父类实例变量初始化";

    // 2. 父类静态代码块
    static {
        System.out.println(parentStaticVar);
        System.out.println("父类静态代码块执行");
    }

    // 4. 父类实例代码块
    {
        System.out.println(parentInstanceVar);
        System.out.println("父类实例代码块执行");
    }

    // 5. 父类构造方法
    public Parent() {
        System.out.println("父类构造方法执行");
    }
}

// 子类
class Child extends Parent {
    // 6. 子类静态成员变量
    static String childStaticVar = "子类静态变量初始化";
    // 8. 子类实例成员变量
    String childInstanceVar = "子类实例变量初始化";

    // 7. 子类静态代码块
    static {
        System.out.println(childStaticVar);
        System.out.println("子类静态代码块执行");
    }

    // 9. 子类实例代码块
    {
        System.out.println(childInstanceVar);
        System.out.println("子类实例代码块执行");
    }

    // 10. 子类构造方法
    public Child() {
        System.out.println("子类构造方法执行");
    }
}

// 测试类
public class InitOrderDemo {
    public static void main(String[] args) {
        System.out.println("===== 创建第一个子类对象 =====");
        Child child1 = new Child();
        
        System.out.println("\n===== 创建第二个子类对象 =====");
        Child child2 = new Child();
    }
}
```

##### 4. 运行结果（关键：静态内容仅执行一次）

```
===== 创建第一个子类对象 =====
父类静态变量初始化
父类静态代码块执行
子类静态变量初始化
子类静态代码块执行
父类实例变量初始化
父类实例代码块执行
父类构造方法执行
子类实例变量初始化
子类实例代码块执行
子类构造方法执行

===== 创建第二个子类对象 =====
父类实例变量初始化
父类实例代码块执行
父类构造方法执行
子类实例变量初始化
子类实例代码块执行
子类构造方法执行
```

#### 三、关键细节解析

##### 1. 静态内容（静态变量+静态代码块）

- **执行时机**：类加载时（第一次使用类时，如创建对象、调用静态方法）执行，**仅执行一次**；
- **执行顺序**：按代码编写顺序初始化（先声明的静态变量/代码块先执行）；
- **作用**：初始化类级别的全局变量，与对象无关。

##### 2. 实例内容（实例变量+实例代码块+构造方法）

- **执行时机**：每次创建对象时执行（创建几次就执行几次）；
- **实例变量 vs 实例代码块**：按编写顺序执行（如先写实例变量，再写实例代码块，则变量先初始化）；
- **构造方法**：是实例初始化的最后一步，核心用于初始化对象的业务逻辑，可通过 `this()` 调用本类其他构造方法，或 `super()` 调用父类构造方法（`super()` 必须是构造方法第一行）。

##### 3. 继承的核心规则

- 子类构造方法默认第一行隐含 `super()`（调用父类无参构造），若父类无无参构造，需显式调用 `super(参数)`；
- 父类的实例初始化（变量+代码块+构造）完成后，才会执行子类的实例初始化；
- 静态内容仅类加载时执行一次，因此子类多次创建对象时，父/子类静态内容不会重复执行。

##### 4. 特殊场景：成员变量的初始化方式优先级

成员变量有多种初始化方式，执行优先级为：
```
默认初始化（JVM赋默认值，如int=0、Object=null）→ 显式初始化（如 String name = "张三"）→ 实例代码块初始化 → 构造方法赋值
```
示例：
```java
class Test {
    // 1. 默认初始化：name=null，age=0
    String name;
    int age;

    // 2. 显式初始化：name="默认名称"
    String name = "默认名称";

    // 3. 实例代码块：age=18
    {
        age = 18;
    }

    // 4. 构造方法：name="构造方法赋值"
    public Test() {
        name = "构造方法赋值";
    }
}

// 测试：最终 name="构造方法赋值"，age=18
```

#### 四、面试高频问题补充

##### 1. 为什么先执行父类构造，再执行子类构造？

子类继承父类的成员变量/方法，必须先让父类完成初始化，子类才能正常使用父类的资源（如父类的成员变量），否则会出现空指针/未初始化的问题。

##### 2. 静态代码块和构造方法的区别？

| 维度     | 静态代码块                          | 构造方法                       |
| -------- | ----------------------------------- | ------------------------------ |
| 执行时机 | 类加载时执行，仅一次                | 创建对象时执行，每次创建都执行 |
| 访问范围 | 仅能访问静态成员（类变量/静态方法） | 可访问静态+实例成员            |
| 核心作用 | 初始化类级别的资源                  | 初始化对象级别的资源           |

##### 3. 实例代码块的作用？

- 提取多个构造方法的公共初始化逻辑（避免代码重复）；
- 优先级高于构造方法，可在构造方法前完成实例变量的初始化。

#### 五、核心总结

1. **核心顺序**：静态内容（父→子，仅一次）→ 实例内容（父：变量→代码块→构造 → 子：变量→代码块→构造）；
2. **继承关键**：子类构造默认调用父类无参构造，父类初始化完成后子类才初始化；
3. **变量初始化**：默认值→显式初始化→实例代码块→构造方法（优先级从低到高）；
4. **静态特性**：类加载时执行，仅一次，与对象创建无关。





### 28.抽象类试题

#### Java 修饰符混用问题解析（错误选项判断）

这道题考察 Java 中核心修饰符（`abstract`、`final`、`static`、`private`）的组合规则，需要逐一分析每个选项的正确性，最终找出**错误的说法**。

##### 一、选项逐一拆解

###### 选项 A：abstract 不能与 final 并列修饰同一个类

✅ **说法正确**（不是错误选项）
- 核心逻辑：
  - `abstract` 修饰的类是「抽象类」，目的是**被继承**（子类必须实现抽象方法）；
  - `final` 修饰的类是「最终类」，目的是**禁止被继承**；
  - 二者语义完全冲突，因此不能并列修饰同一个类，编译器会直接报错。

###### 选项 B：abstract 类中可以有 private 的成员

✅ **说法正确**（不是错误选项）
- 核心逻辑：
  - 抽象类的核心限制是「不能被实例化」，但对成员的访问修饰符无特殊限制；
  - 抽象类中可以包含 `private` 成员（变量/方法）—— 只是 `private` 方法无法被子类重写（但抽象类本身允许有私有成员）。
- 示例（合法代码）：
  ```java
  abstract class AbstractTest {
      private int num = 10; // 私有成员变量（合法）
      private void test() {} // 私有成员方法（合法）
      public abstract void abstractMethod(); // 抽象方法
  }
  ```

###### 选项 C：abstract 方法必须在 abstract 类中

❌ **说法错误（但有细节补充）**
- 核心逻辑（基础规则）：
  - 普通类（非 abstract）不能包含 abstract 方法（因为普通类可实例化，抽象方法无实现，实例化后调用会出错）；
  - 因此**绝大多数场景下**，抽象方法必须定义在抽象类中。
- 关键补充（易忽略点）：
  - 接口中的所有方法（默认 `public abstract`）本质是抽象方法，但接口**不是 abstract 类**（接口用 `interface` 定义，类用 `class` 定义）；
  - 题目中说「abstract 方法必须在 abstract 类中」，未考虑接口场景，因此该说法**错误**。

###### 选项 D：static 方法中能处理非 static 的成员变量

❌ **说法错误（核心错误选项）**
- 核心逻辑：
  - `static` 方法属于「类」，在类加载时就存在，不依赖任何对象；
  - 非 static 成员变量属于「对象」，必须创建对象后才存在；
  - static 方法执行时，可能还没有任何对象被创建，因此**无法直接访问非 static 成员变量**（编译器报错：`Cannot make a static reference to the non-static field`）。
- 反例（编译报错）：
  ```java
  class Test {
      private int num = 10; // 非static成员变量
      
      public static void staticMethod() {
          System.out.println(num); // 编译报错！static方法无法访问非static变量
      }
  }
  ```
- 补充：static 方法若要访问非 static 成员变量，必须**先创建对象**，通过对象引用访问（如 `new Test().num`），但这不属于「直接处理」，题目中「能处理」默认指「直接处理」，因此说法错误。

##### 二、最终结论

**错误的选项是 C、D**（若题目为多选，核心错误是 D，C 属于「不严谨的错误」）。

> 补充：部分教材/面试题中，若将「接口」排除在「类」的范畴外，可能认为 C 正确，但从严格语法角度，接口不是 abstract 类，因此 C 错误。

---

##### 总结

1. **核心规则1**：`abstract` 与 `final` 修饰类时语义冲突，无法混用（A 正确）。
2. **核心规则2**：抽象类可包含私有成员（B 正确）。
3. **核心规则3**：抽象方法不仅可在抽象类中，还可在接口中（接口非 abstract 类），因此 C 错误。
4. **核心规则4**：static 方法不能直接访问非 static 成员变量（D 错误）。

#### 选项中哪一行代码可以添加 到题目中而不产生编译错误？

```java
public abstract class MyClass {
     public int constInt = 5;
     //add code here
     public void method() {
     }
}
```



这道题目考察了Java抽象类的语法规则和成员定义。

A选项"public abstract void method(int a);"是正确的,因为:
\1. 在抽象类中可以定义抽象方法
\2. 抽象方法的语法格式正确:使用abstract修饰符,没有方法体,以分号结束
\3. 参数列表和返回值类型的定义也符合Java语法规则

分析其他错误选项:

B选项"constInt = constInt + 5;"错误:
\- 这是一个语句而不是成员定义
\- **类体中只能包含成员定义,不能直接写执行语句**

C选项"public int method();"错误:
\- 这个方法声明与已有的method()方法冲突
\- Java不支持仅通过返回值类型来区分重载方法

D选项"public abstract void anotherMethod() {}"错误:
\- **抽象方法不能有方法体**
\- 这里错误地为抽象方法添加了空的方法体{}
\- **正确的抽象方法声明应该以分号结束而不是大括号**

总的来说,在抽象类中添加新的抽象方法声明(A选项)是完全合法的,而其他选项都违反了Java的语法规则。



### Super 和 this 面试核心考点

#### 一、核心定义（必背）

| 关键字 | 核心含义                                                     |
| ------ | ------------------------------------------------------------ |
| this   | 代表**当前对象的引用**（调用者本身），可用于访问当前对象的属性、方法、构造器 |
| super  | 代表**父类对象的引用**，可用于访问父类的属性、方法、构造器（突破封装） |

#### 二、核心使用场景（高频考点）

##### 1. 访问属性（解决命名冲突）

**面试题**：当子类和父类属性重名/方法参数与成员变量重名时，如何区分？

```java
class Parent {
    String name = "父类";
}

class Child extends Parent {
    String name = "子类";

    void showName(String name) {
        System.out.println(name); // 方法参数
        System.out.println(this.name); // 当前对象的成员变量（子类name）
        System.out.println(super.name); // 父类的成员变量（父类name）
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        new Child().showName("参数"); 
        // 输出：参数 → 子类 → 父类
    }
}
```
**面试回答**：
- `this.属性`：访问当前对象的成员变量（优先子类）；
- `super.属性`：强制访问父类的成员变量（突破子类重写）。

##### 2. 调用方法（覆盖/重写场景）

**面试题**：子类重写父类方法后，如何调用父类的原方法？
```java
class Parent {
    void say() {
        System.out.println("父类的say方法");
    }
}

class Child extends Parent {
    @Override
    void say() {
        super.say(); // 调用父类被重写的方法
        System.out.println("子类的say方法");
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        new Child().say();
        // 输出：父类的say方法 → 子类的say方法
    }
}
```
**核心考点**：
- `this.方法()`：调用当前对象的方法（优先子类重写后的版本）；
- `super.方法()`：强制调用父类的方法（绕过子类重写）；
- 注意：`super` 不能调用父类的 `static` 方法（`super` 针对对象，`static` 属于类）。

##### 3. 调用构造器（重中之重）

**面试高频题**：`this()` 和 `super()` 的使用规则？
###### 核心规则（必背，答错直接扣分）

1. **位置要求**：必须是构造器的**第一行语句**（两者互斥，只能选其一）；
2. **this()**：调用当前类的其他构造器（重载构造器）；
3. **super()**：调用父类的构造器（若省略，JVM 自动调用父类无参构造器）；
4. **链式调用**：`this()` 内部可间接调用 `super()`，但最终必须追溯到父类构造器。

###### 代码示例（面试常考错题）

```java
class Parent {
    // 父类有参构造器（无无参构造器）
    public Parent(String name) {
        System.out.println("父类有参构造器：" + name);
    }
}

class Child extends Parent {
    // 子类构造器1
    public Child() {
        super("默认名称"); // 必须手动调用父类有参构造器（否则编译报错）
        System.out.println("子类无参构造器");
    }

    // 子类构造器2（调用自身其他构造器）
    public Child(String name) {
        this(); // 调用当前类的无参构造器（第一行）
        System.out.println("子类有参构造器：" + name);
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
        new Child("测试");
        // 输出：
        // 父类有参构造器：默认名称
        // 子类无参构造器
        // 子类有参构造器：测试
    }
}
```
**面试易错点**：
- 若父类只有有参构造器，子类构造器必须显式调用 `super(参数)`，否则编译失败（JVM 无法自动调用父类无参构造器）；
- `this()` 和 `super()` 不能同时出现在构造器第一行（互斥）。

#### 三、核心区别（面试必问对比题）

| 对比维度     | this                                      | super                                     |
| ------------ | ----------------------------------------- | ----------------------------------------- |
| 本质         | 当前对象的引用                            | 父类对象的引用                            |
| 访问范围     | 访问当前类的属性/方法/构造器              | 访问父类的属性/方法/构造器                |
| 构造器调用   | 调用本类其他构造器（`this()`）            | 调用父类构造器（`super()`）               |
| 位置要求     | 构造器中必须在第一行；方法中无限制        | 构造器中必须在第一行；方法中无限制        |
| 存在范围     | 所有非静态方法/构造器（静态方法中不能用） | 所有非静态方法/构造器（静态方法中不能用） |
| 访问静态成员 | 可通过 `this.静态属性`（不推荐）          | 不可通过 `super.静态属性`（编译报错）     |

#### 四、高频面试题 & 标准答案

##### 题1：静态方法中能否使用 this/super？为什么？

**标准答案**：不能。
- `this` 和 `super` 都代表“对象的引用”，而静态方法属于**类**，不属于具体对象（静态方法加载时，对象可能还未创建）；
- 编译阶段就会报错，提示“不能从静态上下文中引用非静态变量 this/super”。

##### 题2：子类构造器中省略 super() 会发生什么？

**标准答案**：
- JVM 会自动在子类构造器的第一行添加 `super()`，调用父类的无参构造器；
- 若父类没有无参构造器（只有有参构造器），则编译失败，必须手动调用 `super(参数)`。

##### 题3：this 和 super 是对象吗？

**标准答案**：不是。
- `this` 和 `super` 是**关键字**，不是对象，也不是变量（没有内存地址）；
- 它们只是编译器的语法糖，用于告诉 JVM 访问当前对象/父类对象的成员。

##### 题4：子类能否通过 super 访问父类的 private 成员？

**标准答案**：不能。
- `super` 只能突破“重写/重名”的屏蔽，但不能突破 `private` 封装；
- 父类的 `private` 成员只能通过父类的 `public/protected` 方法间接访问。

#### 五、总结（核心记忆点）

1. `this` 指向“自己”，解决本类成员和参数的命名冲突，可调用本类构造器；
2. `super` 指向“父类”，解决子类重写后访问父类成员的问题，可调用父类构造器；
3. 构造器中 `this()`/`super()` 必须在第一行，且互斥；
4. 静态方法中不能用 `this/super`，核心原因是“静态属于类，无对象引用”；
5. `super` 无法访问父类 private 成员，不能突破封装。

### 30、Java 中哪里用到了哈希算法？

**哈希算法 = 计算 hashCode 的算法**

Java 里**最核心、面试必答**三处：

1. Object 类的 hashCode () 方法

   所有对象默认都有 hashCode，用于定位哈希地址。

2. HashMap、HashSet、Hashtable、ConcurrentHashMap

   底层是哈希表，通过 `hashCode()`计算数组下标，通过 `equals()`判断是否同一个元素。

3. String 类重写了 hashCode ()

   方便作为 Map 的 key，保证相同字符串 hashCode 相同。

一句话总结：**只要是哈希集合（HashMap/HashSet），底层全靠哈希算法。**

### 31、==、equals、HashCode

这**4 个问题是 Java 面试**最经典、最容易扣分、面试官连环追问**的一套题**，我给你用**最通俗、最逻辑、最容易背**的方式讲清楚，**完全面试标准答案**。

#### 一、Java 中 == 和 equals 的区别

##### 1. == 的作用

- **比较基本数据类型**：比较的是**值是否相等**。
- **比较引用数据类型**：比较的是**内存地址是否相同**（是否同一个对象）。

##### 2. equals 的作用

- 底层默认和 **== 一样**，也是比较地址。
- 但 **String、Integer、Date 等类都重写了 equals**，变成**比较内容是否相等**。

##### 一句话总结（面试必背）

**== 对于引用类型比较地址，基本类型比较值；
equals 默认比较地址，重写后比较内容。**

---

#### 二、hashCode() 方法的作用

**hashCode = 哈希码 = 对象的“指纹”**

作用：
1. **快速定位对象在哈希表（HashMap/HashSet）中的存储位置（数组下标）**。
2. 提高查找效率，不用一个个 equals 比较。

一句话：
**hashCode 用于快速找位置，equals 用于精确判断是否相同。**

---

#### 三、hashCode 和 equals 的区别

- **hashCode()：** 算一个**整数（哈希值）**，用于**快速定位**。
- **equals()：** 逐个比较内容，**精确判断是否相等**。

关系规则（Java 规范）：
1. **两个对象相同 → equals 为 true → hashCode 必须相同。**
2. **hashCode 相同，equals 不一定相同（哈希冲突）。**
3. **hashCode 不同，equals 一定为 false。**

---

#### 四、为什么重写 equals 必须重写 hashCode？（**最重要！面试必考**）

##### 核心原因：

**为了保证在 HashMap、HashSet 中正常工作，不出现BUG！**

##### 场景举例：

1. 你自定义一个类，**只重写 equals，不重写 hashCode**。
2. 两个对象内容一样 → equals 为 true。
3. 但它们的 **hashCode 不同**（继承Object的随机地址）。
4. 存入 HashSet：
   - 根据 hashCode 放到**不同位置**
   - HashSet 认为是**两个不同对象**
   - 导致**重复存入**，破坏集合规范

#### 结论（面试标准答案）

如果**只重写 equals 不重写 hashCode**：
**会导致 HashMap/HashSet 无法正确判断两个对象是否相等，出现重复对象，破坏集合唯一性。**

所以：
**equals 相同，hashCode 必须相同！
因此重写 equals 必须同时重写 hashCode。**

#### 五、**String 到底是怎么重写 equals() 的**

我直接给你讲**源码级、面试标准答案**，一步一步讲清楚：**String 到底是怎么重写 equals() 的**，非常清晰、好记。

##### 一、先记住一句话

**Object 类的 equals 本来是比较地址；String 重写了 equals，变成比较【字符串内容】。**

##### 二、String 类 equals 源码（JDK 原版）

```java
public boolean equals(Object anObject) {
    // 1. 先比较地址：如果是同一个对象，直接返回 true
    if (this == anObject) {
        return true;
    }
    // 2. 判断传入对象是不是 String 类型
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        
        // 拿到底层字符数组 value
        char[] v1 = this.value;
        char[] v2 = anotherString.value;
        
        // 3. 先比较长度是否相同
        int n = v1.length;
        if (n == v2.length) {
            // 4. 长度相同，再逐个字符比较
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            // 所有字符都一样 → return true
            return true;
        }
    }
    // 类型不同 / 长度不同 / 字符不同 → false
    return false;
}
```

##### 三、面试口述版（最标准、最简洁）

String 的 equals 重写步骤一共 **4 步**：

1. **先用 == 判断地址是否相同**，相同直接返回 true（优化）。
2. **判断传入对象是否是 String 类型**，不是直接返回 false。
3. **比较两个字符串的长度（字符数组长度）**，长度不同直接返回 false。
4. **逐个字符比较**，所有字符都相同返回 true，否则 false。

**最终效果：只要内容一样，equals 就返回 true，不管是不是同一个对象。**

首先会判断要比较的两个字符串它们的引用是否相等。如果引用相等的话，直接返回 true ，不相等的话继续下面的判断，

然后再判断被比较的对象是否是 String 的实例，如果不是的话直接返回 false，

如果是的话，再比较两个字符串的长度是否相等，如果长度不想等的话也就没有比较的必要了；

长度如果相同，会比较字符串中的每个 字符 是否相等，一旦有一个字符不相等，就会直接返回 false。

---

##### 四、超级好记总结（面试必背）

**String.equals 规则：**
> **先比地址 → 再比类型 → 再比长度 → 最后逐个比字符。**
> **只要内容一样，就返回 true。**

---

##### 五、面试官最爱追问的配套题（我一起给你）

###### 问：`==` 和 `String.equals` 区别？

- `==`：**比较地址**
- `equals`：**比较内容**

###### 问：String 为什么要重写 equals？

因为我们希望**字符串内容相同，就认为相等**，而不是比较内存地址。

###### 问：String 重写 equals 时，有没有重写 hashCode？

**重写了！**
而且 hashCode 也是**根据字符内容计算**的，保证：
**equals 相同 → hashCode 一定相同**

---

### 32.Java 创建对象的 6 种方式 + new 对象什么时候被GC回收

我给你**面试最标准、最完整、最精简**答案：

 Java 创建对象的 6 种方式 + new 对象什么时候被GC回收

#### 一、Java 创建对象的 6 种方式（必考）

1. **new 关键字（最常用）**
2. **反射：Class.newInstance()（已废弃）**
3. **反射：Constructor.newInstance()**
4. **clone() 克隆**
5. **反序列化（Deserialization）**
6. **Unsafe.allocateInstance()（底层，不推荐）**

##### 精简背诵版（面试直接说）

**new、反射、clone、反序列化、Unsafe、Constructor**

---

#### 二、最关键：**new 出来的对象什么时候被回收？（GC 时机）**

##### 1. 对象满足什么条件会被判定为**垃圾**？

**当对象没有任何 GC Roots 引用时 → 判定为可回收对象。**

###### GC Roots 只有这 4 种（固定）

1. 虚拟机栈中**局部变量**引用的对象
2. 静态变量引用的对象
3. 本地方法栈（JNI）引用的对象
4. 常量引用的对象

---

##### 2. 对象什么时候**真正被回收**？

###### 步骤：

1. 对象失去**所有 GC Roots 引用**（成为垃圾）
2. GC 开始扫描**标记阶段**：标记为垃圾
3. **第一次标记**：判断是否有 `finalize()`
4. **第二次标记**：真正判定可回收
5. **GC 执行清除阶段**：对象内存被释放

###### 结论（面试标准答案）：

- **对象不再被任何 GC Roots 引用时，成为可回收对象。**
- **但不是立即回收，而是在下一次 GC（Minor GC / Full GC）时被回收。**

---

#### 三、超级精简一句话（**背这个满分**）

1. **创建对象 6 种方式：new、反射、clone、反序列化、Unsafe、Constructor**
2. **对象无任何 GC Roots 引用 → 成为垃圾**
3. **垃圾不会立即回收，在下次 GC 时被回收**

---

#### 四、面试官最爱追问的 3 个问题（必看）

##### ① 什么是 GC Roots？

**可以理解为：活着的、必须保留的引用起点。**
栈变量、静态变量、常量、本地方法引用。

##### ② 一个对象会自救吗？

可以。
在 `finalize()` 中**重新把自己赋值给 GC Roots**，可以**自救一次**，第二次就不行了。

##### ③ new 的对象在堆哪里分配？

- 优先 **Eden区**
- 大对象直接进 **老年代**

## 三、集合框架

### 1. **Java集合框架的整体结构是怎样的？主要分为哪两大接口体系？**

- **整体结构**：Java集合框架包含了各种类型的集合类和接口。集合框架提供了统一的接口、类以及算法，用于存储和操作数据。
- **两大接口体系**：
  - **Collection接口体系**：包括 `List`、`Set`、`Queue` 等接口。
  - **Map接口体系**：包括 `Map` 和其常用实现类，如 `HashMap`、`TreeMap`、`LinkedHashMap`。

#### Java集合框架的整体结构

Java集合框架（Java Collections Framework）是用于存储和操作一组对象的统一架构，主要由**接口**（定义规范）、**实现类**（具体实现）和**工具类**（如`Collections`）组成，整体结构遵循“接口-实现类”的分层设计，方便开发者灵活选择和替换具体实现。

框架的核心目标是：
- 提供统一的操作接口（如添加、删除、遍历）；
- 封装不同的数据结构（如数组、链表、哈希表、树）；
- 支持高效的元素访问和修改。

Map接口和Collection接口是所有集合框架的父接口：
Collection接口的子接口包括：Set接口和List接口
Map接口的实现类主要有：HashMap、TreeMap、Hashtable、**ConcurrentHashMap**以及**Properties**等
Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等
List接口的实现类主要有：ArrayList、LinkedList、Stack以及Vector等

##### 两大核心接口体系

集合框架的顶层接口分为两大分支，分别对应“单个元素集合”和“键值对集合”：

##### 1. **Collection接口体系**

用于存储**单个元素的集合**（一组对象），是所有单列集合的根接口，定义了通用操作（如`add()`、`remove()`、`size()`等）。其主要子接口及特点：

| 子接口       | 核心特点                                  | 典型实现类                  |
|--------------|-------------------------------------------|-----------------------------|
| `List`       | 有序（元素存入顺序与取出顺序一致）、可重复 | `ArrayList`（数组实现）、`LinkedList`（链表实现）、`Vector`（线程安全，古老类） |
| `Set`        | 无序（存入顺序与取出顺序可能不一致）、不可重复 | `HashSet`（哈希表，无序）、`LinkedHashSet`（哈希表+链表，有序）、`TreeSet`（红黑树，可排序） |
| `Queue`      | 队列（先进先出FIFO），支持队列操作        | `LinkedList`（双端队列）、`PriorityQueue`（优先队列，按优先级出队） |
| `Deque`      | 双端队列（两端均可进出）                  | `ArrayDeque`（数组实现）、`LinkedList` |

##### 2. **Map接口体系**

用于存储**键值对（key-value）映射**的集合，每个键（key）唯一，值（value）可重复。其主要实现类及特点：

| 实现类           | 核心特点                                  | 底层数据结构                |
|------------------|-------------------------------------------|-----------------------------|
| `HashMap`        | 无序，key允许为`null`（仅一个）           | 数组+链表/红黑树（JDK 8+）  |
| `LinkedHashMap`  | 有序（按插入顺序或访问顺序）              | `HashMap` + 双向链表        |
| `TreeMap`        | 可排序（按key自然排序或自定义排序）        | 红黑树                      |
| `Hashtable`      | 线程安全（古老类，不推荐），key不允许为`null` | 数组+链表                   |
| `ConcurrentHashMap` | 线程安全（高效，推荐）                    | 分段锁/ CAS机制（JDK 8+）   |

##### 框架的继承关系简化图

```
Collection（根接口）
├─ List（有序可重复）
│  ├─ ArrayList
│  ├─ LinkedList（同时实现Deque）
│  └─ Vector
│     └─ Stack（栈，已过时，推荐用Deque的实现类）
├─ Set（无序不可重复）
│  ├─ HashSet (底层是根据 HashMap 实现的)
│  │  └─ LinkedHashSet（保证插入顺序）
│  └─ TreeSet（自然排序/定制排序）
└─ Queue（队列，先进先出）
   └─ Deque（双端队列，可操作首尾）
      ├─ LinkedList（Deque的常用实现，非线程安全）
      ├─ ArrayDeque（基于数组的Deque实现，效率高于LinkedList，非线程安全）
      └─ ConcurrentLinkedDeque（线程安全的Deque实现，适用于并发场景）
      
Map（根接口，独立体系，键值对存储）
├─ HashMap（非线程安全，无序，JDK1.8后基于数组+链表/红黑树）
│  └─ LinkedHashMap（保证插入顺序/访问顺序）
├─ TreeMap（自然排序/定制排序，基于红黑树）
├─ Hashtable（线程安全，已过时，推荐用ConcurrentHashMap）
└─ ConcurrentHashMap（线程安全，高效并发，JDK1.8后基于CAS+synchronized）
```

##### 核心工具类

- `Collections`：提供操作集合的静态方法（如排序`sort()`、查找`binarySearch()`、同步化`synchonizedList()`等）。
- `Arrays`：提供操作数组的静态方法（如数组转集合`asList()`、排序`sort()`等）。

##### 总结

- Java集合框架通过**Collection**（单列）和**Map**（键值对）两大体系，覆盖了几乎所有常用的数据结构；
- 接口定义规范，实现类提供具体功能，开发者可根据需求（如是否有序、是否线程安全、查询效率等）选择合适的实现；
- 掌握两大体系的区别（单列vs键值对）和主要实现类的特点，是灵活使用集合框架的基础。

#### Java 集合框架中各实现类的方法

可分为两类：**继承自接口的通用方法**和**实现类特有的方法**。以下是核心实现类的主要方法整理（基于你提供的体系图）：

##### 一、List 体系实现类

###### 1. ArrayList

- **继承自 List 接口的核心方法**：
  - `add(E e)`：添加元素到末尾
  - `add(int index, E e)`：指定位置插入元素
  - `get(int index)`：获取指定索引元素（随机访问，O(1) 效率）
  - `set(int index, E e)`：替换指定索引元素
  - `remove(int index)`：删除指定索引元素（后续元素需移位，O(n) 效率）
  - `size()`：获取元素数量
  - `contains(Object o)`：判断是否包含元素
  - `subList(int fromIndex, int toIndex)`：获取子列表（视图，修改会影响原列表）

- **特有方法**：
  - `ensureCapacity(int minCapacity)`：手动扩容，避免多次自动扩容的性能消耗
  - `trimToSize()`：将容量调整为实际元素数量，节省内存

###### 2. LinkedList

- **继承自 List 接口的方法**（与 ArrayList 一致，但性能不同）：
  - `get(int index)`：获取元素（需从头/尾遍历，O(n) 效率，不适合随机访问）
  - `add(int index, E e)`：指定位置插入（链表操作，O(1) 定位后插入，但定位需 O(n)）

- **实现 Deque 接口的队列/栈方法**（特有核心）：
  - 队列操作（FIFO）：
    - `addFirst(E e)` / `offerFirst(E e)`：头部添加元素
    - `addLast(E e)` / `offerLast(E e)`：尾部添加元素
    - `getFirst()` / `peekFirst()`：获取头部元素（不删除）
    - `getLast()` / `peekLast()`：获取尾部元素（不删除）
    - `removeFirst()` / `pollFirst()`：删除并返回头部元素
    - `removeLast()` / `pollLast()`：删除并返回尾部元素
  - 栈操作（LIFO，替代过时的 Stack）：
    - `push(E e)`：等价于 `addFirst(E e)`（压栈）
    - `pop(E e)`：等价于 `removeFirst()`（出栈）

###### 3. Vector（线程安全，已基本被 ArrayList 替代）

- 方法与 ArrayList 基本一致，但**所有方法都被 `synchronized` 修饰**（线程安全）：
  - `addElement(E obj)`：等价于 `add(E e)`
  - `elementAt(int index)`：等价于 `get(int index)`
  - `removeElementAt(int index)`：等价于 `remove(int index)`

- 特有方法：
  - `capacity()`：返回当前容量（ArrayList 需通过反射获取）
  - `copyInto(Object[] anArray)`：将元素复制到指定数组

add = 强行加

offer = 试着提供一下

get：**正经获取**

peek：**轻轻看一眼，不拿走**

**remove**：移除、删掉

语气强硬：**必须删掉，删不到就报错**

**poll**：抽取、取出

语气温和：**有就拿，没有就算了**

**失败抛异常：add、remove、element**

**失败不抛异常：offer、poll、peek**



##### 二、Set 体系实现类

###### 1. HashSet

- **继承自 Set 接口的核心方法**：
  - `add(E e)`：添加元素（依赖 `hashCode()` 和 `equals()` 去重），添加成功返回true，否则返回false。
  - `remove(Object o)`：删除元素
  - `contains(Object o)`：判断是否包含元素（哈希查找，O(1) 效率）
  - `size()`：获取元素数量
  - `isEmpty()`：判断是否为空
- `retainAll(Collection<?> c)` 用于**保留集合中与另一个集合共有的元素**（即求两个集合的交集），同时删除当前集合中不存在于另一个集合中的元素。如果当前集合因调用该方法而发生了改变（即有元素被删除），则返回 `true`；否则返回 `false`。
  
- 无特有方法，完全依赖 Set 接口和 HashMap 底层实现（HashSet 本质是 HashMap 的 Key 集合）。

###### 2. LinkedHashSet

- 方法与 HashSet 完全一致（继承自 HashSet），但**通过双向链表维护插入顺序**，因此：
  - `iterator()` 迭代时按插入顺序返回元素
  - 无额外特有方法，所有功能与 HashSet 一致，仅多了顺序保证

###### 3. TreeSet

- **继承自 Set 接口的方法**（与 HashSet 一致，但添加了排序特性）：
  - `add(E e)`：添加元素（需元素可比较，或通过构造器传入 `Comparator`）
  - `contains(Object o)`：判断包含（基于排序树查找，O(log n) 效率）

- **特有排序相关方法**：
  - `first()`：返回最小元素
  - `last()`：返回最大元素
  - `lower(E e)`：返回小于 e 的最大元素
  - `higher(E e)`：返回大于 e 的最小元素
  - `subSet(E fromElement, E toElement)`：返回子集合（从 from 到 to，左闭右开）
  - `headSet(E toElement)`：返回小于 to 的所有元素
  - `tailSet(E fromElement)`：返回大于等于 from 的所有元素

##### 三、Queue 体系实现类

###### 1. PriorityQueue（优先队列，元素按优先级排序）

- **队列核心方法**：
  - `add(E e)` / `offer(E e)`：添加元素（按优先级排序，默认自然排序）
  - `peek()`：获取队首元素（不删除，优先级最高的元素）
  - `poll()`：删除并返回队首元素
  - `remove()`：删除队首元素（与 poll() 类似，但为空时抛异常）

- 特有方法：
  - `comparator()`：返回用于排序的比较器（无则返回 null，即自然排序）

Queue体系的实现类围绕“队列操作”核心，不同实现类在方法上遵循Queue接口规范，同时根据自身特性（如双端、优先级、线程安全）扩展了额外能力。以下按实现类分类，梳理核心方法及使用场景：

###### 一、通用Queue接口核心方法（所有实现类均支持）

Queue接口定义了“先进先出（FIFO）”的基础操作，分为两类：**抛出异常**和**返回特殊值**（更推荐，避免异常处理），适用于所有实现类。

| 操作类型 | 抛出异常的方法 | 返回特殊值的方法 | 说明 |
|----------|----------------|------------------|------|
| 添加元素 | `add(E e)`     | `offer(E e)`     | 向队列尾部添加元素；若队列满，`add`抛`IllegalStateException`，`offer`返回`false` |
| 删除元素 | `remove()`     | `poll()`         | 移除并返回队列头部元素；若队列空，`remove`抛`NoSuchElementException`，`poll`返回`null` |
| 获取元素 | `element()`    | `peek()`         | 返回（不删除）队列头部元素；若队列空，`element`抛`NoSuchElementException`，`peek`返回`null` |

##### 二、各实现类的方法与特性

###### 1. 基础双端实现：LinkedList（实现Queue+Deque+List）

LinkedList同时具备队列、双端队列、列表的能力，方法最灵活，核心补充**双端操作方法**（来自Deque接口）。

| 方法分类       | 核心方法                                                                 | 说明                                                                 |
|----------------|--------------------------------------------------------------------------|----------------------------------------------------------------------|
| 通用Queue方法  | `offer(e)`、`poll()`、`peek()`                                           | 基础尾部添加、头部删除/获取，非阻塞                                  |
| 双端操作方法   | 头部操作：`addFirst(e)`/`offerFirst(e)`、`removeFirst()`/`pollFirst()`、`getFirst()`/`peekFirst()` | 直接操作队列头部（如`offerFirst`在头部插入，实现“栈”的`push`效果）    |
|                | 尾部操作：`addLast(e)`/`offerLast(e)`、`removeLast()`/`pollLast()`、`getLast()`/`peekLast()` | 直接操作队列尾部（与通用Queue的`offer`功能一致）                      |
| List额外方法   | `add(int index, e)`、`get(int index)`、`remove(int index)`               | 支持按索引操作（因底层是链表，索引操作效率低，不推荐频繁使用）        |

**示例**：用LinkedList实现栈（先进后出）

```java
LinkedList<String> stack = new LinkedList<>();
stack.push("a"); // 等价于 addFirst("a")
stack.push("b");
System.out.println(stack.pop()); // 等价于 removeFirst()，输出 "b"
```

###### 2. 高效双端实现：ArrayDeque（实现Deque）

ArrayDeque是Deque的数组实现，双端操作效率比LinkedList高，方法与LinkedList的“双端操作”完全一致，但**不支持List的索引方法**，且**禁止存储null**。

| 核心方法（与LinkedList双端方法一致） | 说明                                                                 |
|--------------------------------------|----------------------------------------------------------------------|
| `offerFirst(e)`/`offerLast(e)`        | 头部/尾部添加元素，非阻塞，返回`true`（因无界，不会返回`false`）      |
| `pollFirst()`/`pollLast()`            | 头部/尾部删除并返回元素，空队列返回`null`                             |
| `peekFirst()`/`peekLast()`            | 头部/尾部获取元素，空队列返回`null`                                   |
| `push(e)`/`pop()`                     | 等价于`addFirst(e)`/`removeFirst()`，用于模拟栈操作（推荐替代过时的Stack） |

**注意**：ArrayDeque无`add(int index, e)`这类List方法，仅专注双端队列操作，效率更高。

###### 3. 优先级队列：PriorityQueue（实现Queue）

PriorityQueue打破FIFO，按“优先级”取元素，核心方法与通用Queue一致，但**元素必须支持排序**（实现`Comparable`或传入`Comparator`）。

| 核心方法       | 说明                                                                 |
|----------------|----------------------------------------------------------------------|
| 构造方法       | `PriorityQueue()`（默认自然排序，小顶堆）、`PriorityQueue(Comparator<? super E> c)`（自定义排序） | 初始化时指定排序规则，如`new PriorityQueue<>(Collections.reverseOrder())`实现大顶堆 |
| 通用Queue方法  | `offer(e)`、`poll()`、`peek()`                                       | `poll()`/`peek()`返回的是“优先级最高”的元素（如小顶堆返回最小元素）  |
| 额外方法       | `size()`、`contains(Object o)`、`clear()`                             | 常规的大小查询、包含判断、清空操作，无特殊扩展方法                   |

**示例**：用PriorityQueue实现大顶堆（取最大元素）
```java
// 自定义比较器，实现大顶堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
// 这种 变量的类型声明 方式更好，面向接口编程，而非面向实现，便于后续类型变更，只修改右边就可以了
Queue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(3);
maxHeap.offer(1);
maxHeap.offer(5);
System.out.println(maxHeap.poll()); // 输出 "5"（优先级最高）
```

###### 4. 线程安全队列：ConcurrentLinkedQueue（实现Queue）

ConcurrentLinkedQueue是无锁线程安全队列，基于CAS实现并发安全，方法与通用Queue一致，**无额外扩展方法**，但保证多线程下的操作安全性。

| 核心方法       | 说明                                                                 |
|----------------|----------------------------------------------------------------------|
| `offer(e)`      | 尾部添加元素，线程安全，无界队列（永远返回`true`，不会满）            |
| `poll()`        | 头部删除并返回元素，线程安全，空队列返回`null`（非阻塞）              |
| `peek()`        | 头部获取元素，线程安全，空队列返回`null`                             |
| `isEmpty()`     | 判断队列是否为空，线程安全（注意：判断后若立即操作，仍可能有并发问题） |

**示例**：多线程向ConcurrentLinkedQueue添加元素
```java
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
// 线程1添加元素
new Thread(() -> queue.offer("thread1")).start();
// 线程2添加元素
new Thread(() -> queue.offer("thread2")).start();
System.out.println(queue.poll()); // 线程安全获取，可能返回"thread1"或"thread2"
```

###### 5. 阻塞队列：LinkedBlockingQueue（实现BlockingQueue）

LinkedBlockingQueue是阻塞式线程安全队列，实现了`BlockingQueue`接口，在通用Queue方法基础上，补充了**阻塞式方法**（队列满时添加、空时获取会阻塞线程）。

| 方法分类       | 核心方法                                                                 | 说明                                                                 |
|----------------|--------------------------------------------------------------------------|----------------------------------------------------------------------|
| 阻塞添加       | `put(E e)`                                                               | 尾部添加元素；若队列满，阻塞线程直到有空间                             |
| 阻塞获取       | `take()`                                                                 | 头部删除并返回元素；若队列空，阻塞线程直到有元素                       |
| 超时阻塞       | `offer(E e, long timeout, TimeUnit unit)`、`poll(long timeout, TimeUnit unit)` | 添加/获取元素时，若满/空则阻塞指定时间，超时后返回`false`/`null`       |
| 通用方法       | `offer(e)`、`poll()`、`peek()`                                           | 非阻塞，满时`offer`返回`false`，空时`poll`返回`null`                  |

**示例**：用LinkedBlockingQueue实现生产者-消费者模型
```java
LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10); // 容量10
// 生产者：添加元素，满时阻塞
new Thread(() -> {
    try {
        queue.put(1); // 若队列满，阻塞
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();
// 消费者：获取元素，空时阻塞
new Thread(() -> {
    try {
        queue.take(); // 若队列空，阻塞
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();
```

###### 三、关键方法对比与选择

| 实现类                | 核心方法特点                          | 线程安全 | 阻塞性 | 适用场景                          |
|-----------------------|---------------------------------------|----------|--------|-----------------------------------|
| LinkedList            | 支持双端+List索引方法                  | 否       | 非阻塞 | 单线程灵活操作（队列/栈/列表）    |
| ArrayDeque            | 双端操作高效，无List方法，禁null      | 否       | 非阻塞 | 单线程栈/双端队列（替代Stack）    |
| PriorityQueue         | 按优先级取元素，需排序                | 否       | 非阻塞 | 单线程优先级任务处理（如TopK）    |
| ConcurrentLinkedQueue | 无锁安全，非阻塞，无界                | 是       | 非阻塞 | 高并发无界任务存储（如日志收集）  |
| LinkedBlockingQueue   | 支持阻塞方法，可选有界                | 是       | 阻塞   | 并发场景需控制容量（如线程池队列）|






##### 四、Map 体系实现类

###### 1. HashMap

- **继承自 Map 接口的核心方法**：
  - `put(K key, V value)`：添加键值对（Key 重复则覆盖 Value）
  - `get(Object key)`：根据 Key 获取 Value（哈希查找，O(1) 效率）
  - `remove(Object key)`：删除键值对
  - `containsKey(Object key)`：判断是否包含指定 Key
  - `containsValue(Object value)`：判断是否包含指定 Value（需遍历，O(n) 效率）
  - `keySet()`：返回所有 Key 的 Set 视图
  - `values()`：返回所有 Value 的 Collection 视图
  - `entrySet()`：返回所有键值对（`Map.Entry`）的 Set 视图
  - **`merge`：合并两个键值对，键重复时的 value 计算**（如累加、取最大值 / 最小值等）。它的核心作用是：根据指定的键，要么将新值放入 Map（键不存在时），要么通过自定义逻辑合并旧值和新值（键存在时）。
- `getOrDefault(Object key, V defaultValue)` 是 Java 8 为 `Map` 接口新增的方法，用于**安全地获取 Map 中指定 Key 对应的 Value**，当 Key 不存在时返回一个默认值，避免了手动判断 `null` 的繁琐逻辑。
  
- 特有方法：
  - `putIfAbsent(K key, V value)`：Key 不存在时才添加（Java 8+）
  - `computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)`：Key 不存在时通过函数计算 Value 并添加（Java 8+）
  - `resize()`：扩容（内部方法，自动触发）

###### 2. LinkedHashMap

- 方法与 HashMap 完全一致，但**通过双向链表维护顺序**（插入顺序或访问顺序）：
  - 默认按插入顺序迭代（`entrySet()`、`keySet()` 等）
  - 构造器可指定 `accessOrder = true`，按访问顺序排序（`get` 或 `put` 会移动元素到链表尾部）

- 特有方法（保护方法，可重写实现 LRU 缓存）：
  - `removeEldestEntry(Map.Entry<K, V> eldest)`：当添加新元素时，判断是否删除最旧元素（默认返回 false，即不删除）

###### 3. TreeMap

- **继承自 Map 接口的方法**（与 HashMap 一致，但添加了排序特性）：
  - `put(K key, V value)`：添加键值对（Key 需可比较，或通过构造器传入 `Comparator`）
  - `get(Object key)`：根据 Key 获取 Value（基于红黑树查找，O(log n) 效率）

- **特有排序相关方法**：
  - `firstKey()` / `lastKey()`：返回最小/最大 Key
  - `lowerKey(K key)` / `higherKey(K key)`：返回小于/大于指定 Key 的最大/最小 Key
  - `subMap(K fromKey, K toKey)`：返回子 Map（Key 从 from 到 to，左闭右开）
  - `headMap(K toKey)` / `tailMap(K fromKey)`：返回 Key 小于 to / 大于等于 from 的子 Map

###### 4. Hashtable（线程安全，已基本被 ConcurrentHashMap 替代）

- 方法与 HashMap 类似，但**所有方法都被 `synchronized` 修饰**（线程安全）：
  - `put(K key, V value)`：Key/Value 不能为 null（与 HashMap 最大区别）
  - `get(Object key)`：获取 Value
  - `elements()`：返回所有 Value 的枚举（Enumeration）

###### 5. ConcurrentHashMap（线程安全，高效并发）

- 方法与 HashMap 类似，但支持**并发操作**（无需整体加锁，性能优于 Hashtable）：
  - `put(K key, V value)`：线程安全的添加操作
  - `get(Object key)`：无锁获取 Value（高效）
  - `size()`：返回元素数量（并发环境下可能为近似值）
  - `forEach(BiConsumer<? super K, ? super V> action)`：并发遍历

- 特有并发方法：
  - `putIfAbsent(K key, V value)`：原子操作，Key 不存在时添加
  - `remove(Object key, Object value)`：仅当 Key 关联的 Value 匹配时才删除（原子操作）
  - `replace(K key, V oldValue, V newValue)`：仅当 Key 关联的 Value 为 oldValue 时才替换（原子操作）

##### 总结

- **通用方法**：各实现类均实现了顶层接口（List/Set/Map）的抽象方法，功能一致但性能因底层结构（数组/链表/哈希表/红黑树）不同而有差异。
- **特有方法**：多与实现类的特性相关（如 LinkedList 的队列操作、TreeSet/TreeMap 的排序方法、ConcurrentHashMap 的并发原子操作）。
- 实际开发中，优先使用接口定义变量（如 `List<String> list = new ArrayList<>()`），仅在需要特有方法时才使用具体类声明。

以下是Java集合框架中核心接口及实现类的**常用方法汇总**，按接口体系分类说明，涵盖List、Set、Queue/Deque、Map的核心操作：

#### 一、Collection 根接口（所有集合的基础）

定义了集合的通用操作，List、Set、Queue均继承此接口。

| 方法                 | 说明                                                                 |
|----------------------|----------------------------------------------------------------------|
| `int size()`         | 返回集合中元素的数量                                                 |
| `boolean isEmpty()`  | 判断集合是否为空                                                     |
| `boolean contains(Object o)` | 判断集合是否包含指定元素（依赖`equals()`方法）                       |
| `Iterator<E> iterator()` | 返回迭代器，用于遍历集合元素                                         |
| `Object[] toArray()` | 将集合转换为数组                                                     |
| `boolean add(E e)`   | 添加元素到集合（List返回true，Set若元素存在则返回false）             |
| `boolean remove(Object o)` | 从集合中移除指定元素（成功返回true）                                 |
| `boolean containsAll(Collection<?> c)` | 判断集合是否包含另一个集合的所有元素                                 |
| `boolean addAll(Collection<? extends E> c)` | 添加另一个集合的所有元素到当前集合                                   |
| `boolean removeAll(Collection<?> c)` | 移除当前集合中与另一个集合共有的元素                                 |
| `boolean retainAll(Collection<?> c)` | 保留当前集合中与另一个集合共有的元素（交集）                         |
| `void clear()`       | 清空集合所有元素                                                     |

#### 二、List 接口（有序可重复）

继承Collection，增加了“索引操作”能力。

| 通用方法             | 说明                                                                 |
|----------------------|----------------------------------------------------------------------|
| `E get(int index)`   | 获取指定索引的元素                                                   |
| `E set(int index, E element)` | 替换指定索引的元素，返回被替换的旧元素                               |
| `void add(int index, E element)` | 在指定索引插入元素，后续元素后移                                     |
| `E remove(int index)` | 移除指定索引的元素，返回被移除的元素                                 |
| `int indexOf(Object o)` | 返回指定元素第一次出现的索引（不存在返回-1）                         |
| `int lastIndexOf(Object o)` | 返回指定元素最后一次出现的索引（不存在返回-1）                       |
| `ListIterator<E> listIterator()` | 返回列表迭代器，支持双向遍历和修改                                   |

##### 1. ArrayList（数组实现）

除List接口方法外，无特有方法，核心是**随机访问高效**（`get/set` 时间复杂度 `O(1)`）。

##### 2. LinkedList（链表实现，同时实现Deque）

额外支持Deque的双端操作（见下文Deque方法），List接口方法中**插入/删除高效**（`add/remove` 时间复杂度 `O(1)`，需定位索引时 `O(n)`）。

##### 3. Vector（线程安全的ArrayList，已过时）

方法与ArrayList一致，但**所有方法加了`synchronized`锁**（线程安全但效率低），推荐用`Collections.synchronizedList(new ArrayList<>())`替代。

#### 三、Set 接口（无序不可重复）

继承Collection，无特有方法，核心是**元素唯一**（依赖`equals()`和`hashCode()`）。

##### 1. HashSet（哈希表实现）

- 基于HashMap实现，元素存储在HashMap的key中。
- 无序（迭代顺序与插入顺序无关），查询/添加/删除效率高（`O(1)`）。

##### 2. LinkedHashSet（哈希表+链表）

- 继承HashSet，内部维护双向链表记录插入顺序。
- 迭代顺序与插入顺序一致，性能略低于HashSet（链表维护开销）。

##### 3. TreeSet（红黑树实现）

- 基于TreeMap实现，元素按自然排序或自定义比较器排序。
- 支持排序相关方法：
  - `E first()`：返回第一个（最小）元素
  - `E last()`：返回最后一个（最大）元素
  - `E lower(E e)`：返回小于e的最大元素
  - `E higher(E e)`：返回大于e的最小元素

#### 四、Queue 接口（队列，FIFO）

继承Collection，增加队列特有操作（分“抛异常”和“返回特殊值”两类）。

| 操作类型 | 抛异常方法       | 返回特殊值方法   | 说明                                                                 |
|----------|------------------|------------------|----------------------------------------------------------------------|
| 添加     | `add(E e)`       | `offer(E e)`     | 向队尾添加元素；队列满时，`add`抛异常，`offer`返回false             |
| 删除     | `remove()`       | `poll()`         | 移除并返回队首元素；队列空时，`remove`抛异常，`poll`返回null         |
| 获取     | `element()`      | `peek()`         | 返回（不删除）队首元素；队列空时，`element`抛异常，`peek`返回null   |

#### Deque 接口（双端队列，继承Queue）

支持首尾双向操作，方法前缀`First`（头部）、`Last`（尾部）。

| 头部操作（抛异常） | 头部操作（返回特殊值） | 尾部操作（抛异常） | 尾部操作（返回特殊值） | 说明                                                                 |
|---------------------|------------------------|---------------------|------------------------|----------------------------------------------------------------------|
| `addFirst(E e)`     | `offerFirst(E e)`      | `addLast(E e)`      | `offerLast(E e)`       | 向头部/尾部添加元素                                                 |
| `removeFirst()`     | `pollFirst()`          | `removeLast()`      | `pollLast()`           | 移除并返回头部/尾部元素                                             |
| `getFirst()`        | `peekFirst()`          | `getLast()`         | `peekLast()`           | 返回（不删除）头部/尾部元素                                         |
| `push(E e)`         | （等价于`addFirst(e)`） | `pop()`             | （等价于`removeFirst()`） | 模拟栈操作（先进后出）                                               |

##### Deque 实现类特有说明

- **ArrayDeque**：无特有方法，双端操作效率高于LinkedList，**禁止存储null**。
- **LinkedList**：同时实现List和Deque，可混用索引操作和双端操作。
- **ConcurrentLinkedDeque**：线程安全，基于CAS实现，方法与Deque一致，无阻塞。



#### **⚠️ Stack使用建议**

尽管 `java.util.Stack` 类仍然存在，但它在现代 Java 开发中**不被推荐使用**。官方文档建议优先使用 `Deque` 接口及其实现类（如 `ArrayDeque`）来实现栈的功能，因为 `Deque` 提供了更完整、更一致的 LIFO 操作集合，且性能通常更优4。

#### 五、Map 接口（键值对存储，独立体系）

不继承Collection，核心是“键唯一，值可重复”。

| 通用方法                 | 说明                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `V put(K key, V value)`  | 添加键值对，若key已存在则覆盖旧值，返回旧值（无则返回null）           |
| `V get(Object key)`      | 根据key获取value（key不存在返回null）                                |
| `V remove(Object key)`   | 根据key移除键值对，返回被移除的value（无则返回null）                 |
| `boolean containsKey(Object key)` | 判断是否包含指定key                                                  |
| `boolean containsValue(Object value)` | 判断是否包含指定value                                                |
| `int size()`             | 返回键值对数量                                                       |
| `boolean isEmpty()`      | 判断是否为空                                                         |
| `void clear()`           | 清空所有键值对                                                       |
| `Set<K> keySet()`        | 返回所有key的Set集合（可遍历key）                                    |
| `Collection<V> values()` | 返回所有value的Collection集合（可遍历value）                          |
| `Set<Map.Entry<K,V>> entrySet()` | 返回键值对Entry的Set集合（可遍历键值对）                             |

这几个是 Java 8 引入的函数式方法，专门用来简化 HashMap 的操作，特别是处理“存在则更新，不存在则插入”这种逻辑时，代码会非常

##### **1.** `computeIfAbsent(K key, Function mappingFunction)`

**含义**：**“如果没有，就算一下放进去”**

- 如果指定的键 **不存在** 或者 对应的值为 `null`，则计算给定的函数值，并将其放入 Map 中。
- 如果键 **已存在**，则 **不做任何操作**，直接返回现有的值。

##### **2.** `computeIfPresent(K key, BiFunction remappingFunction)`

**含义**：**“如果有，就算一下更新它”**

- 只有当指定的键 **存在** 且 值不为 `null` 时，才执行计算函数来更新值。
- 如果键 **不存在**，则什么都不做，返回 `null`。

**典型场景**：计数器累加。**2.** `computeIfPresent(K key, BiFunction remappingFunction)`

**含义**：**“如果有，就算一下更新它”**

- 只有当指定的键 **存在** 且 值不为 `null` 时，才执行计算函数来更新值。
- 如果键 **不存在**，则什么都不做，返回 `null`。

**典型场景**：计数器累加。



##### **3.** `compute(K key, BiFunction remappingFunction)`

**含义**：**“不管有没有，都重新算一下”**

- 无论键是否存在，都会执行计算函数。
- 你可以根据旧值（`oldValue`）来决定新值。如果旧值不存在，`oldValue` 为 `null`。
- **注意**：如果计算结果为 `null`，则会将该键从 Map 中 **删除**。

**典型场景**：复杂的更新逻辑，或者初始化并更新。

##### 4.`map.getOrDefault(key, defaultValue)`

**获取键对应的值，如果键不存在，则返回一个默认值。**

##### 5.putIfAbsent(K key, V value)

```java
V putIfAbsent(K key, V value)
```

- **逻辑**：如果 `key` 不存在（或对应 value 为 `null`），则添加 `key-value` 并返回 `null`；如果 `key` 已存在，则**不修改原有值**，直接返回已存在的 `value`。
- **完美匹配你的需求**：返回值可判断“是否是新添加的元素”，结合 `keySet().indexOf()`（或自定义索引逻辑）就能实现“存在返回索引、不存在添加并返回新索引”。





##### 1. HashMap（哈希表实现，非线程安全）

- JDK1.8后底层为“数组+链表+红黑树”，查询/添加/删除效率高（`O(1)`）。
- 无序（迭代顺序与插入顺序无关），允许key为null（仅一个），value为null。

##### 2. LinkedHashMap（继承HashMap）

- 内部维护双向链表记录键值对的**插入顺序**或**访问顺序**（通过构造函数指定）。
- 特有方法：`boolean removeEldestEntry(Map.Entry<K,V> eldest)`，可用于实现LRU缓存（移除最久未访问元素）。

##### 3. TreeMap（红黑树实现）

- 键按自然排序或自定义比较器排序，支持排序相关操作：
  - `K firstKey()` / `K lastKey()`：返回最小/最大key
  - `SortedMap<K,V> subMap(K fromKey, K toKey)`：返回key在[fromKey, toKey)范围内的子Map
- 不允许key为null（会抛异常）。

##### 4. Hashtable（线程安全，已过时）

- 方法与HashMap类似，但**所有方法加了`synchronized`锁**（线程安全但效率低）。
- 不允许key或value为null，推荐用`ConcurrentHashMap`替代。

##### 5. ConcurrentHashMap（线程安全，高效并发）

- JDK1.8后基于“CAS+synchronized”实现，支持高并发（仅锁定链表/红黑树的首节点）。
- 方法与HashMap类似，额外支持原子操作：
  - `V putIfAbsent(K key, V value)`：若key不存在则添加，返回null；否则返回现有value
  - `boolean remove(Object key, Object value)`：仅当key对应value匹配时才移除，返回是否成功

#### 总结

- **List**：核心是“有序+索引”，`get/set` 用ArrayList，`add/remove` 频繁用LinkedList。
- **Set**：核心是“去重”，无序用HashSet，需顺序用LinkedHashSet，需排序用TreeSet。
- **Queue/Deque**：核心是“队列操作”，单线程双端操作首选ArrayDeque，并发用ConcurrentLinkedDeque。
- **Map**：核心是“键值对”，非并发用HashMap/LinkedHashMap/TreeMap，并发用ConcurrentHashMap。

掌握这些方法是熟练使用Java集合的基础，不同实现类的选择需结合“线程安全”“排序需求”“操作效率”等场景判断。


------

### 2. **Collection接口和Map接口有什么本质区别？它们各自的核心实现类有哪些？**

- **本质区别**：
  - `Collection` 用于存储单一对象的集合，代表对象的集合。
  - `Map` 用于存储键值对（key-value），每个元素包含一个键和一个对应的值。
- **核心实现类**：
  - **Collection接口**：`ArrayList`、`LinkedList`、`HashSet`、`TreeSet`、`PriorityQueue`。
  - **Map接口**：`HashMap`、`TreeMap`、`LinkedHashMap`、`Hashtable`、`ConcurrentHashMap`。

------

### 3. **ArrayList和LinkedList的底层数据结构是什么？它们在增删改查操作上有什么性能差异？**

- **底层数据结构**：
  - `ArrayList`：基于动态数组。
  - `LinkedList`：基于双向链表。
- **性能差异**：
  - `ArrayList`：对于随机访问操作（`get()`）速度较快，时间复杂度为 O(1)。但是对于插入、删除操作（尤其是中间位置的操作），需要移动大量元素，时间复杂度为 O(n)。
  - `LinkedList`：对于插入、删除操作较为高效（O(1)），但访问元素时需要遍历链表，时间复杂度为 O(n)。

------

### 4. **HashMap和HashTable的区别是什么？为什么HashTable被认为是线程不安全的替代方案？**

- **区别**：
  - **HashMap**：允许 `null` 键和 `null` 值，线程不安全。
  - **Hashtable**：不允许 `null` 键或 `null` 值，线程安全（但性能较差，因其大部分操作都被同步化）。
- **线程安全问题**：`Hashtable` 的同步机制会导致性能瓶颈，且并发访问时可能导致问题，因此不推荐使用。

##### HashMap与Hashtable的核心区别

| 维度                | HashMap                                  | Hashtable                              |
|---------------------|------------------------------------------|----------------------------------------|
| **线程安全性**       | 线程不安全（无同步机制）                  | 线程安全（方法被`synchronized`修饰）    |
| **null值支持**       | 允许1个`null`键和多个`null`值            | 不允许`null`键或`null`值（会抛`NullPointerException`） |
| **底层数据结构**     | JDK 8前：数组+链表；JDK 8后：数组+链表/红黑树（链表长度>8时转红黑树） | 始终是数组+链表（无红黑树优化）         |
| **继承关系**         | 继承`AbstractMap`类                      | 继承古老的`Dictionary`类                |
| **迭代器类型**       | 快速失败迭代器（`fail-fast`）             | 安全失败迭代器（`fail-safe`，但不保证） |
| **性能**             | 无同步开销，并发环境下效率高              | 方法同步导致性能低（多线程竞争同一把锁） |
| **初始容量与扩容**   | 初始容量16，扩容为原容量×2               | 初始容量11，扩容为原容量×2 + 1          |
| **使用场景**         | 单线程环境或通过`Collections.synchronizedMap()`包装后用于多线程 | 不推荐使用，多线程场景推荐`ConcurrentHashMap` |

##### 为什么Hashtable被视为“线程不安全的替代方案”？

这里的表述更准确的是：**Hashtable虽然线程安全，但因性能问题和设计缺陷，已被更高效的线程安全实现（如ConcurrentHashMap）替代**，具体原因：

1. **同步效率极低**  
   Hashtable的线程安全是通过对**整个方法加锁**（`synchronized`修饰方法）实现的，意味着任何线程操作Hashtable时，都会独占整个对象的锁。多线程并发访问时，所有操作都会串行执行（即使操作不同的键），导致严重的性能瓶颈。

   例如：线程A执行`put("a", 1)`时，线程B执行`put("b", 2)`必须等待A释放锁，即使两个操作完全不冲突。


2. **迭代器的“安全失败”不可靠**  
   Hashtable的迭代器被称为“安全失败”（`fail-safe`），但这是通过**复制底层数组**实现的——迭代时遍历的是数组副本，因此不会抛出`ConcurrentModificationException`。但这会导致：
   - 额外的内存开销（复制数组）；
   - 迭代器无法反映实时修改（遍历的是旧数据）。


3. **功能设计落后**  
   - 不支持`null`键值（限制了使用场景）；
   - 底层无红黑树优化（链表过长时查询效率低，时间复杂度O(n)）；
   - 继承自古老的`Dictionary`类，而非`AbstractMap`，与集合框架的主流设计脱节。


4. **更好的替代方案存在**  
   JDK 5引入的`ConcurrentHashMap`解决了Hashtable的所有缺陷：
   - 采用**分段锁**（JDK 7）或**CAS + synchronized**（JDK 8+）实现高效并发，允许多线程同时操作不同段/桶；
   - 支持`null`值（键仍不允许`null`）；
   - 底层有红黑树优化，查询效率高；
   - 迭代器支持“弱一致性”（能反映部分实时修改，且无额外复制开销）。

##### 总结

- HashMap适用于**单线程环境**，性能优异但线程不安全；
- Hashtable虽线程安全，但因**同步效率低、设计落后**，已被淘汰；
- 多线程场景下，**优先使用ConcurrentHashMap**（高效并发），而非Hashtable或通过`Collections.synchronizedMap()`包装的HashMap。

简单说：Hashtable的“线程安全”是以牺牲性能和灵活性为代价的，而ConcurrentHashMap在保证线程安全的同时，实现了更高的并发效率，因此成为了更好的替代方案。



------

### 5. **HashSet的底层实现原理是什么？它如何保证元素的唯一性？**

- **底层实现原理**：`HashSet` 底层是基于 `HashMap`，存储元素时将元素作为 `Map` 的键，值使用一个固定的对象。
- **保证唯一性**：通过 `HashMap` 的 `key` 保证元素的唯一性。插入元素时，如果 `hashCode()` 值相同且 `equals()` 方法返回 `true`，则认为元素已存在，不会添加。

#### HashSet 的底层实现原理

HashSet 本质是**HashMap 的“包装类”**，其内部维护了一个 HashMap 实例，并利用 HashMap 的 key 来存储元素，value 则使用一个固定的空对象（`PRESENT`，一个静态的 `Object` 实例）。

简化的源码逻辑（核心部分）：
```java
public class HashSet<E> {
    // 底层的HashMap
    private transient HashMap<E, Object> map;
    // 固定的value，所有元素共享
    private static final Object PRESENT = new Object();

    // 构造器：初始化HashMap
    public HashSet() {
        map = new HashMap<>();
    }

    // 添加元素：本质是向HashMap中put(key, PRESENT)
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }

    // 其他方法（如remove、contains等）均委托给HashMap实现
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
}
```

可见，HashSet 的所有操作（添加、删除、查询）都是通过调用底层 HashMap 的对应方法实现的，自身仅做了简单封装。

#### HashSet 如何保证元素的唯一性？

元素的唯一性完全依赖于**底层 HashMap 对 key 的唯一性保证**，具体通过以下两步校验：

1. **第一步：计算哈希值（hashCode()）**  
   当添加元素 `e` 时，HashSet 会调用 `e.hashCode()` 计算哈希值，根据哈希值确定该元素在 HashMap 数组中的存储位置（桶索引）。  
   - 若两个元素的 `hashCode()` 不同，直接认为是不同元素，存入不同位置；  
   - 若 `hashCode()` 相同，进入第二步校验。

2. **第二步：判断内容是否相等（equals()）**  
   当哈希值相同时，HashMap 会调用元素的 `equals()` 方法，比较两个元素的实际内容：  
   - 若 `equals()` 返回 `true`，则认为是同一元素，不添加（HashMap 的 `put` 方法会覆盖旧值，但因 HashSet 的 value 固定，实际表现为“不添加”）；  
   - 若 `equals()` 返回 `false`，则认为是不同元素（哈希冲突），会以链表或红黑树的形式存入同一桶中。

#### 关键注意点

1. **重写 hashCode() 和 equals() 的规范**  
   为了保证 HashSet 能正确判断元素唯一性，存储的元素必须**同时重写 `hashCode()` 和 `equals()`**，且需满足：  
   - 若 `a.equals(b) == true`，则 `a.hashCode() == b.hashCode()`（必须保证）；  
- 若 `a.hashCode() == b.hashCode()`，`a.equals(b)` 可以为 `false`（哈希冲突允许存在）。  
  

若只重写其中一个，会导致唯一性判断出错（例如，两个内容相同的对象因哈希值不同被视为不同元素，或哈希值相同但内容不同的对象被视为同一元素）。

2. **无序性的原因**  
   HashSet 的“无序”是因为元素的存储位置由哈希值决定，与插入顺序无关（除非使用 `LinkedHashSet`，它通过链表维护了插入顺序）。

3. **null 元素的处理**  
   HashSet 允许存储一个 `null` 元素（因为 HashMap 允许 `null` 作为 key，且仅允许一个）。

#### 总结

- HashSet 底层通过 HashMap 实现，元素存储在 HashMap 的 key 位置，value 固定为一个空对象；  
- 元素唯一性由 **hashCode() 哈希值校验 + equals() 内容校验** 共同保证；  
- 使用时必须为存储的元素正确重写 `hashCode()` 和 `equals()` 方法，否则会导致逻辑错误。



#### 示例： HashSet的实现

使用 `HashSet` 可以存储不重复的元素，它基于 `HashMap` 实现，具有无序性（存储顺序与插入顺序无关）和高效的查找、添加、删除操作。以下是 `HashSet` 的详细使用方法：

##### 一、基本使用步骤

###### 1. 导入包

```java
import java.util.HashSet;
import java.util.Set;
```

###### 2. 创建 `HashSet` 对象

```java
// 创建默认初始容量（16）的HashSet
Set<String> set = new HashSet<>();

// 创建指定初始容量的HashSet（如初始容量为100）
Set<Integer> numSet = new HashSet<>(100);
```

###### 3. 核心方法使用

| 方法                | 功能描述                                  | 示例                                  |
|---------------------|-------------------------------------------|---------------------------------------|
| `add(E e)`          | 添加元素（成功返回`true`，重复元素返回`false`） | `set.add("apple");`                   |
| `remove(Object o)`  | 删除指定元素（存在则删除并返回`true`）    | `set.remove("apple");`                |
| `contains(Object o)`| 判断是否包含指定元素（返回`boolean`）     | `boolean hasApple = set.contains("apple");` |
| `size()`            | 返回元素个数                              | `int size = set.size();`              |
| `isEmpty()`         | 判断是否为空（返回`boolean`）             | `boolean empty = set.isEmpty();`      |
| `clear()`           | 清空所有元素                              | `set.clear();`                        |

##### 二、遍历 `HashSet` 的方式

###### 1. 增强 for 循环（推荐）

```java
Set<String> fruits = new HashSet<>();
fruits.add("apple");
fruits.add("banana");
fruits.add("orange");

for (String fruit : fruits) {
    System.out.println(fruit); // 输出顺序不确定（无序性）
}
```

###### 2. 迭代器（`Iterator`）

```java
Iterator<String> iterator = fruits.iterator();
while (iterator.hasNext()) {
    String fruit = iterator.next();
    System.out.println(fruit);
    // 遍历中删除元素（必须用迭代器的remove()）
    if (fruit.equals("banana")) {
        iterator.remove();
    }
}
```

##### 三、存储自定义对象的注意事项

`HashSet` 判断元素唯一性依赖 **`hashCode()` 和 `equals()` 方法**，存储自定义对象时必须同时重写这两个方法，否则会导致重复元素无法去重。

###### 示例：存储 `User` 对象

```java
class User {
    private String name;
    private int age;

    // 构造器
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 重写hashCode()：保证相同对象的哈希值相同
    @Override
    public int hashCode() {
        // 结合name和age计算哈希值（可使用Objects工具类）
        return java.util.Objects.hash(name, age);
    }

    // 重写equals()：定义对象相等的规则
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true; // 同一对象
        if (obj == null || getClass() != obj.getClass()) return false; // 类型不同
        User user = (User) obj;
        // name和age都相同则认为是同一对象
        return age == user.age && java.util.Objects.equals(name, user.name);
    }

    // 重写toString()方便打印
    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + "}";
    }
}

// 使用HashSet存储User
public class Main {
    public static void main(String[] args) {
        Set<User> users = new HashSet<>();
        users.add(new User("Alice", 20));
        users.add(new User("Bob", 22));
        users.add(new User("Alice", 20)); // 重复元素，不会被添加

        System.out.println(users.size()); // 输出：2（去重成功）
        for (User u : users) {
            System.out.println(u);
        }
    }
}
```

##### 四、`HashSet` 的特性总结

1. **无序性**：元素存储顺序与插入顺序无关（若需有序，可使用 `LinkedHashSet`）。
2. **唯一性**：依赖 `hashCode()` 和 `equals()` 去重，必须为自定义对象重写这两个方法。
3. **高效性**：添加、删除、查找操作的时间复杂度平均为 O(1)。
4. **允许 null 元素**：但只能存储一个 `null`（因为 `null` 的 `hashCode()` 固定为 0）。
5. **线程不安全**：多线程环境下需使用 `Collections.synchronizedSet(new HashSet<>())` 包装。

##### 五、常见场景

- 存储不重复的元素（如去重）；
- 快速判断元素是否存在（如黑名单校验）；
- 实现集合运算（如交集、并集，可配合 `retainAll()`、`addAll()` 等方法）。

通过上述方法，可以灵活使用 `HashSet` 处理各种去重和快速查找场景。

------

### 6. **TreeSet和TreeMap是如何实现排序功能的？自定义对象如何在这两个集合中实现排序？**

- **实现排序功能**：`TreeSet` 和 `TreeMap` 都是基于红黑树实现的，它们会自动根据元素的自然顺序（通过 `Comparable`）或者通过提供的比较器 `Comparator` 进行排序。
- **自定义对象排序**：自定义对象需要实现 `Comparable` 接口，重写 `compareTo()` 方法，或使用 `Comparator` 接口定义排序规则。


你的总结准确概括了TreeSet和TreeMap的排序机制，以下是更详细的实现原理和自定义对象排序的具体方式：

##### 一、TreeSet和TreeMap的排序实现原理

TreeSet和TreeMap底层都基于**红黑树（一种自平衡的二叉搜索树）** 实现，排序的核心是通过**比较机制**确定元素在树中的位置，保证集合中的元素始终处于有序状态。

###### 排序的核心逻辑：

1. **二叉搜索树的特性**：对于树中的每个节点，其左子树的所有元素都小于该节点，右子树的所有元素都大于该节点（通过比较器或自然排序判断）。
2. **插入元素时的排序**：
   - 新元素插入时，通过比较机制与树中节点依次比较，确定其在左子树或右子树的位置；
   - 红黑树的自平衡机制会自动调整树结构，避免出现极端不平衡的情况（保证查询、插入、删除的时间复杂度为O(log n)）。

###### TreeSet与TreeMap的关系：

和HashSet类似，**TreeSet底层依赖TreeMap实现**：
- TreeSet将元素存储在TreeMap的`key`位置，`value`使用一个固定的空对象；
- 排序逻辑完全复用TreeMap的key排序机制。

##### 二、自定义对象的排序方式

TreeSet和TreeMap对元素（或key）的排序依赖两种比较方式，自定义对象需通过以下方式指定排序规则：

###### 方式1：实现`Comparable`接口（自然排序）

自定义类实现`Comparable`接口，重写`compareTo()`方法，定义“自然排序”规则。

**示例**：定义`User`类，按`id`升序排序

```java
// 实现Comparable接口，指定比较类型为User
class User implements Comparable<User> {
    private int id;
    private String name;

    // 重写compareTo()：返回负数（当前对象小）、0（相等）、正数（当前对象大）
    @Override
    public int compareTo(User other) {
        // 按id升序排序（this.id - other.id）
        return this.id - other.id;
        // 若需降序，可改为 other.id - this.id
    }

    // 构造器、getter等省略
}

// 使用TreeSet（依赖自然排序）
TreeSet<User> set = new TreeSet<>();
set.add(new User(2, "B"));
set.add(new User(1, "A"));
// 遍历结果：按id升序 [User(1, "A"), User(2, "B")]
```

###### 方式2：使用`Comparator`比较器（定制排序）

当无法修改自定义类（如第三方类），或需要临时改变排序规则时，可通过`Comparator`接口定义排序逻辑。

**示例**：对`User`类按`name`长度排序（不修改User类）
```java
// 定义Comparator比较器（按name长度升序）
Comparator<User> nameLengthComparator = (u1, u2) -> {
    return u1.getName().length() - u2.getName().length();
};

// 创建TreeSet时传入比较器
TreeSet<User> set = new TreeSet<>(nameLengthComparator);
set.add(new User(1, "Alice"));  // name长度5
set.add(new User(2, "Bob"));    // name长度3
// 遍历结果：按name长度升序 [User(2, "Bob"), User(1, "Alice")]
```

##### 三、关键注意事项

1. **两种排序方式的优先级**：
   - 若同时提供`Comparator`和`Comparable`，**`Comparator`优先级更高**（定制排序覆盖自然排序）。

2. **排序与元素唯一性的关系**：
   - TreeSet和TreeMap判断元素（或key）是否唯一，**完全依赖比较结果**：
     - 若`compareTo()`或`compare()`返回`0`，则认为是相同元素（即使`equals()`返回`false`），会被视为重复元素而不被添加。
   - 建议：`compareTo()`的结果应与`equals()`保持一致（即`compareTo() == 0`时，`equals()`也应返回`true`），避免逻辑混乱。

3. **null值处理**：
   - TreeSet的元素和TreeMap的key**不允许为null**（会抛出`NullPointerException`），因为比较时无法调用`null`的`compareTo()`方法；
   - （HashSet和HashMap允许null是因为它们依赖hashCode()，而null的hashCode()被定义为0）。

##### 总结

- TreeSet和TreeMap通过**红黑树**实现排序，依赖比较机制确定元素位置；
- 自定义对象排序有两种方式：**实现`Comparable`（自然排序）** 或**提供`Comparator`（定制排序）**；
- 排序的核心是`compareTo()`或`compare()`方法的实现，其返回值决定了元素的顺序和唯一性。

简单说：Tree系列集合的排序是“比较驱动”的，自定义对象必须明确告诉集合“如何比较大小”，才能正确排序和去重。

------

### 7. **什么是迭代器（Iterator）？使用迭代器遍历集合时需要注意哪些问题？**

- **迭代器**：`Iterator` 是用于遍历集合元素的工具，提供了 `hasNext()`、`next()`、`remove()` 方法。
- **注意问题**：在使用迭代器时，不能在迭代过程中修改集合结构（例如通过 `add()` 或 `remove()` 方法），否则会抛出 `ConcurrentModificationException`。



##### 一、迭代器（Iterator）的本质与作用

迭代器（`java.util.Iterator`）是Java集合框架中用于**统一遍历集合元素的接口**，它封装了集合的底层数据结构（如数组、链表、红黑树），为所有集合（如`List`、`Set`）提供了一致的遍历方式，开发者无需关心集合的具体实现细节。

###### 1. 核心设计目的

- **解耦遍历逻辑与集合实现**：无论集合底层是数组还是链表，都能通过`Iterator`的`hasNext()`和`next()`方法遍历，无需为不同集合编写不同遍历代码。
- **支持安全的元素删除**：提供`remove()`方法，可在遍历过程中安全删除元素（避免直接操作集合导致的索引混乱）。

###### 2. 核心方法

| 方法名         | 作用                                                                 |
|----------------|----------------------------------------------------------------------|
| `boolean hasNext()` | 判断集合中是否还有未遍历的元素，有则返回`true`，无则返回`false`。    |
| `E next()`      | 返回下一个未遍历的元素；若已无元素，会抛出`NoSuchElementException`。 |
| `void remove()` | 删除`next()`方法最后返回的元素；若未调用`next()`就调用`remove()`，或连续调用`remove()`，会抛出`IllegalStateException`。 |

###### 3. 基本使用示例（遍历`HashSet`）

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("B");
set.add("C");

// 1. 获取集合的迭代器
Iterator<String> iterator = set.iterator();

// 2. 遍历集合
while (iterator.hasNext()) {
    String element = iterator.next(); // 获取下一个元素
    System.out.println(element);
    
    // 3. 安全删除元素（通过迭代器的remove()）
    if (element.equals("B")) {
        iterator.remove(); 
    }
}

System.out.println(set); // 输出：[A, C]
```

##### 二、使用迭代器遍历集合的核心注意事项

###### 1. 严禁“并发修改”：避免`ConcurrentModificationException`

这是最常见的问题——**在迭代器遍历期间，若直接通过集合的方法（如`add()`、`remove()`、`clear()`）修改集合结构，会触发`ConcurrentModificationException`**。

##### 原因：
迭代器内部维护了一个“修改计数器”（`modCount`），记录集合被修改的次数。迭代器初始化时会记录当前的`modCount`，每次调用`next()`时会检查集合的`modCount`是否与初始化时一致：
- 若一致：正常遍历；
- 若不一致：说明集合被“意外修改”，直接抛出异常（防止遍历过程中集合结构变化导致的遍历混乱，如数组越界、链表断裂）。

##### 错误示例（触发异常）：
```java
Iterator<String> iterator = set.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    if (element.equals("B")) {
        set.remove(element); // 直接调用集合的remove()，触发ConcurrentModificationException
    }
}
```

##### 正确做法：
- 若需删除元素：通过迭代器自身的`remove()`方法（如上文示例）；
- 若需添加元素：迭代器不支持`add()`，可改用`ListIterator`（仅`List`接口支持，提供`add()`方法）。

###### 2. 避免“重复调用`remove()`”或“未调用`next()`就调用`remove()`”

迭代器的`remove()`方法有严格限制：**只能删除`next()`最后返回的元素，且每次`next()`后只能调用一次`remove()`**。

##### 错误示例：
```java
Iterator<String> iterator = set.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    if (element.equals("B")) {
        iterator.remove(); 
        iterator.remove(); // 连续调用，抛出IllegalStateException
    }
}
```

##### 正确逻辑：
确保`remove()`与`next()`一一对应，每次`remove()`前必须有`next()`调用。

###### 3. 迭代器遍历是“单向的”，不支持反向遍历

`Iterator`是单向迭代器，只能从集合头部遍历到尾部，无法像`List`的索引那样“回头”访问已遍历的元素。若需反向遍历，可使用：
- `List`接口的`ListIterator`（支持`hasPrevious()`和`previous()`方法）；
- 对集合进行反向排序后再遍历（如`TreeSet`的`descendingIterator()`）。

###### 4. 集合结构变化后，迭代器会失效

若集合在迭代过程中被其他线程修改（如多线程环境下），或当前线程通过其他方式修改（如外层代码调用`clear()`），已创建的迭代器会立即失效，后续调用`hasNext()`或`next()`会抛出异常。

##### 多线程场景的解决方案：
- 使用线程安全的集合（如`CopyOnWriteArrayList`），其迭代器是“弱一致性”的，不会抛出`ConcurrentModificationException`（但可能遍历到旧数据）；
- 在遍历期间对集合加锁（如`synchronized`），禁止其他线程修改。

###### 5. 避免“空指针异常”：检查`hasNext()`后再调用`next()`

若未调用`hasNext()`直接调用`next()`，或`hasNext()`返回`false`后仍调用`next()`，会抛出`NoSuchElementException`。

##### 正确遍历流程：
```java
// 必须先通过hasNext()判断，再调用next()
while (iterator.hasNext()) {
    String element = iterator.next(); // 安全调用
}
```

##### 三、迭代器的扩展：`ListIterator`与“增强for循环”

- **`ListIterator`**：仅`List`接口支持，是`Iterator`的子类，额外提供`add()`（添加元素）、`set()`（修改元素）、反向遍历等功能，适合对`List`进行复杂操作。
- **增强for循环（foreach）**：本质是`Iterator`的语法糖，编译后会自动转换为`Iterator`遍历。若在foreach中修改集合，同样会抛出`ConcurrentModificationException`。

```java
// foreach遍历（本质是Iterator）
for (String element : set) {
    if (element.equals("B")) {
        set.remove(element); // 同样触发ConcurrentModificationException
    }
}
```


##### 总结
- 迭代器是集合的“统一遍历工具”，核心价值是解耦遍历逻辑与集合实现；
- 使用时最关键的是**避免并发修改**：修改集合必须通过迭代器的`remove()`（或`ListIterator`的`add()`），而非集合自身的方法；
- 其他注意事项（如`remove()`的调用限制、单向遍历、空指针预防）需严格遵循，避免抛出运行时异常。
### Queue的遍历

Queue（队列）在 Java 中是一个 **Collection** 集合，它的遍历方式与普通集合类似，但由于队列具有 **FIFO（先进先出）** 的特性，遍历时通常更关注从队首开始的顺序。

Queue 最常用的实现类是 **`LinkedList`**（双向链表）和 **`ArrayDeque`**（双端队列）。

以下是 Queue 的几种常见遍历方式，按**开发优先级**排序：

#### 1. 迭代器遍历 (Iterator) —— **最推荐**

这是最安全、最标准的遍历方式。它不会改变队列内部的元素状态，只是读取。

```java
import java.util.LinkedList;
import java.util.Iterator;
import java.util.Queue;

public class QueueTraverse {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();
        queue.add("A");
        queue.add("B");
        queue.add("C");

        // 1. 获取迭代器
        Iterator<String> iterator = queue.iterator();
        
        // 2. 遍历
        while (iterator.hasNext()) {
            String element = iterator.next();
            System.out.println(element); 
            // 输出顺序：A -> B -> C
        }
        
        // 注意：遍历后，队列依然保留所有元素
        System.out.println(queue); // [A, B, C]
    }
}
```

#### 2. 增强 For 循环 (For-Each) —— **最简洁**

底层原理也是使用迭代器，代码写法更优雅，适合简单的遍历场景。

```java
Queue<Integer> queue = new LinkedList<>();
queue.add(1);
queue.add(2);
queue.add(3);

// 直接遍历集合
for (Integer num : queue) {
    System.out.println(num);
}
```

#### 3. 轮询取出 (poll()) —— **消耗式遍历**

这种方式会**逐个取出并删除**队列中的元素。遍历结束后，原队列会变空（`queue.isEmpty()` 变为 `true`）。
**适合：** 你需要处理完队列中的所有任务并清空队列时使用。

```java
Queue<String> queue = new LinkedList<>();
queue.add("Apple");
queue.add("Banana");
queue.add("Orange");

System.out.println("处理前队列大小: " + queue.size()); // 3

// 循环取出队首元素，直到队列为空
while (!queue.isEmpty()) {
    String item = queue.poll(); // 取出并移除
    System.out.println("处理: " + item);
}

System.out.println("处理后队列大小: " + queue.size()); // 0
```

#### 4. 索引遍历 (for i) —— **不推荐**

`Queue` 接口本身**不保证通过索引访问元素**（它不是 List）。
*   **例外**：如果你确定你的实现类是 `LinkedList`，可以强制转型为 `List` 来使用索引，但这违背了 Queue 的设计初衷，且性能较差（O(n)）。

#### 5. Java 8 Stream 流遍历

利用 Stream API 进行流式处理，适合复杂的过滤、映射等操作。

```java
Queue<Double> queue = new LinkedList<>();
queue.add(1.5);
queue.add(2.5);
queue.add(3.5);

// 遍历并打印
queue.stream().forEach(System.out::println);

// 过滤遍历 (只打印大于2的元素)
queue.stream()
     .filter(d -> d > 2.0)
     .forEach(System.out::println);
```

---

#### ⚠️ 重要避坑指南

1. **`peek()` vs `poll()`**
    *   **`peek()`**：只看队首，**不删除**，队列为空时返回 `null`。
    *   **`poll()`**：看队首，**并删除**，队列为空时返回 `null`。
    *   *做遍历循环时，如果不小心用错，容易导致死循环或误删数据。*

2. **关于 `ConcurrentLinkedQueue`**
    *   如果是在多线程环境下使用 `ConcurrentLinkedQueue`，其迭代器是 **弱一致性** 的。遍历过程中如果其他线程修改了队列，遍历结果可能不会反映最新的更改，但不会抛出异常。

#### 总结

*   **只读、不改变队列：** 使用 **增强 For 循环** 或 **迭代器**。
*   **处理完即清空：** 使用 **`while (!queue.isEmpty()) { queue.poll(); }`**。

需要我针对 `Deque`（双端队列）的遍历再做补充说明吗？它和 Queue 的遍历逻辑有一些细微差别。

### HashMap的遍历

在 Java 中遍历 HashMap 有多种常用写法，不同写法适配不同场景（比如是否需要键、值、键值对，是否需要遍历中删除元素等），下面按**常用程度+场景适配**整理完整写法，附代码示例和核心说明：

#### 一、最常用的遍历方式（推荐）

##### 1. 遍历所有键值对（EntrySet）—— 需同时用 Key 和 Value

这是性能最优的写法（仅遍历一次），也是日常开发中最常用的方式。
```java
import java.util.HashMap;
import java.util.Map;

public class HashMapTraverse {
    public static void main(String[] args) {
        // 初始化测试HashMap
        Map<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Java", 8);
        hashMap.put("Python", 3);
        hashMap.put("C++", 5);

        // 方式1：EntrySet 增强for循环（推荐，简洁高效）
        for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
            String key = entry.getKey(); // 获取键
            Integer value = entry.getValue(); // 获取值
            System.out.println("Key: " + key + ", Value: " + value);
        }

        // 方式1的Lambda简化版（Java 8+）
        hashMap.forEach((key, value) -> {
            System.out.println("Key: " + key + ", Value: " + value);
        });
    }
}
```

##### 2. 仅遍历所有键（KeySet）—— 只需 Key 或通过 Key 查 Value

如果只需要键，或需要通过键手动获取值（注意：通过 `get(key)` 会多一次哈希查询，性能略低）：
```java
// 方式2：KeySet 增强for循环
for (String key : hashMap.keySet()) {
    Integer value = hashMap.get(key); // 按需获取值
    System.out.println("Key: " + key + ", Value: " + value);
}
```

##### 3. 仅遍历所有值（Values）—— 只需 Value 无需 Key

```java
// 方式3：Values 增强for循环
for (Integer value : hashMap.values()) {
    System.out.println("Value: " + value);
}
```

#### 二、需要遍历中删除元素的写法（迭代器）

如果需要在遍历过程中删除元素，**必须使用 Iterator**（否则会抛出 `ConcurrentModificationException` 异常）：
```java
import java.util.Iterator;
import java.util.Map;

// 方式4：Iterator 遍历（支持遍历中删除元素）
Iterator<Map.Entry<String, Integer>> iterator = hashMap.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<String, Integer> entry = iterator.next();
    String key = entry.getKey();
    Integer value = entry.getValue();
    
    // 示例：删除Value为5的元素
    if (value == 5) {
        iterator.remove(); // 安全删除，不会触发并发修改异常
        continue;
    }
    System.out.println("Key: " + key + ", Value: " + value);
}
// 验证删除结果
System.out.println("删除后HashMap: " + hashMap);
```

#### 三、不推荐的写法（了解即可）

- **普通for循环+索引**：HashMap 是无序的，没有索引概念，无法直接通过索引遍历；
- **forEach+remove**：Lambda 遍历中调用 `hashMap.remove()` 会抛出 `ConcurrentModificationException`，必须用迭代器删除。

#### 四、各写法对比与选择建议

| 写法              | 优点                 | 缺点                           | 适用场景                   |
| ----------------- | -------------------- | ------------------------------ | -------------------------- |
| EntrySet 增强for  | 简洁、性能最优       | 无法遍历中删除（除非用迭代器） | 只需遍历，无需删除元素     |
| EntrySet Iterator | 支持遍历中删除       | 代码稍繁琐                     | 遍历过程中需要删除元素     |
| KeySet 增强for    | 仅需键时简洁         | 性能略低（多一次get查询）      | 只需键，或少量场景查值     |
| Values 增强for    | 仅需值时极简         | 无法获取键                     | 只关心值，不关心键         |
| Lambda forEach    | 代码最简洁（Java8+） | 遍历中删除需用迭代器           | Java8+，无需删除元素的场景 |

#### 总结

1. 日常遍历（需Key+Value）：优先用 `EntrySet` 增强for循环或 Lambda `forEach`；
2. 遍历中删除元素：必须用 `Iterator` 遍历 `EntrySet`；
3. 仅需Key/Value：分别用 `KeySet`/`Values` 增强for循环；
4. 避免用 `KeySet+get(key)` 遍历（性能低），除非仅需键的场景。

### HashMap遍历中添加元素

在 `EntrySet` 增强 for 循环中**直接向 HashMap 添加元素会抛出 `ConcurrentModificationException`（并发修改异常）**，这是 HashMap 快速失败（fail-fast）机制导致的，下面结合原理、代码示例和正确写法详细说明：

#### 一、为什么添加元素会报错？

HashMap 的增强 for 循环本质上是迭代器（Iterator）的语法糖，迭代器在遍历过程中会维护一个「修改次数计数器」（modCount）：
- 遍历开始时，迭代器记录当前的 modCount；
- 遍历过程中如果通过 HashMap 的 `put()`/`remove()` 等方法修改集合（添加/删除元素），会改变 modCount；
- 迭代器每次迭代时会检查 modCount 是否和初始值一致，不一致则直接抛出异常，防止遍历过程中集合结构变化导致的遍历异常。

#### 二、错误示例（添加元素触发异常）

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapAddInFor {
    public static void main(String[] args) {
        Map<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Java", 8);
        hashMap.put("Python", 3);

        // 错误：增强for循环中添加元素，触发ConcurrentModificationException
        try {
            for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
                String key = entry.getKey();
                // 示例：添加新元素
                hashMap.put("C++", 5); 
                System.out.println("Key: " + key + ", Value: " + entry.getValue());
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("报错：" + e.getMessage());
        }
    }
}
```
运行结果：直接抛出 `ConcurrentModificationException`，无法正常遍历。

#### 三、正确写法（支持遍历中添加元素）

如果需要在遍历过程中**添加元素**，有两种可靠方式：

##### 1. 方式1：先遍历收集需添加的元素，遍历结束后统一添加（推荐，简单安全）

这是最常用的方案，避免遍历和修改同时进行：
```java
import java.util.HashMap;
import java.util.Map;

public class HashMapAddSafe {
    public static void main(String[] args) {
        Map<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Java", 8);
        hashMap.put("Python", 3);

        // 步骤1：遍历HashMap，收集需要添加的元素（或直接记录添加逻辑）
        Map<String, Integer> toAddMap = new HashMap<>();
        for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
            System.out.println("遍历元素：Key=" + entry.getKey() + ", Value=" + entry.getValue());
            // 模拟需要添加的元素
            if ("Java".equals(entry.getKey())) {
                toAddMap.put("C++", 5);
                toAddMap.put("Go", 4);
            }
        }

        // 步骤2：遍历结束后，统一添加元素
        hashMap.putAll(toAddMap);

        // 验证结果
        System.out.println("添加后HashMap：" + hashMap);
    }
}
```
输出结果：
```
遍历元素：Key=Java, Value=8
遍历元素：Key=Python, Value=3
添加后HashMap：{Java=8, Python=3, C++=5, Go=4}
```

##### 2. 方式2：使用迭代器（仅支持删除，不支持添加）

⚠️ 注意：HashMap 的迭代器（Iterator）仅提供 `remove()` 方法（支持遍历中删除），**没有 `add()`/`put()` 方法**，因此无法通过迭代器添加元素——这也是为什么方式1是添加元素的最优解。

#### 四、补充说明

1. **快速失败机制的例外**：如果通过迭代器的 `remove()` 方法删除元素，不会触发异常（因为迭代器会同步更新 modCount）；但 `put()`/`putAll()` 等添加操作，无论通过哪种方式在遍历中执行，都会触发异常。
2. **Java 8+ 的 `forEach` 方法**：Lambda 形式的 `hashMap.forEach(...)` 本质也是迭代器遍历，同样不能在其中直接 `put()` 添加元素，否则也会抛出异常。

#### 总结

1. EntrySet 增强 for 循环中**不能直接添加元素**，会触发 `ConcurrentModificationException`；
2. 遍历中需添加元素时，优先采用「先遍历收集、遍历结束后统一添加」的方式；
3. 迭代器仅支持遍历中删除元素，不支持添加，添加操作需在遍历完成后执行。



------

### 8. **ConcurrentHashMap在JDK 7和JDK 8中的实现有什么区别？**

- **JDK 7**：`ConcurrentHashMap` 将整个桶划分为多个段（Segment），每个段有自己的锁，锁粒度较粗。
- **JDK 8**：将每个桶变成了 `Node`，并通过 `synchronized` 和 CAS（Compare-And-Swap）来保证线程安全，锁粒度更小，性能更高。
- **保证线程安全**：通过分段锁和 CAS 操作，保证多个线程能够并发访问不同的段，同时不会发生冲突。

---

#### **ConcurrentHashMap 在 JDK 7 vs JDK 8 的区别**

| 特性             | **JDK 7**                                                | **JDK 8**                                    |
| ---------------- | -------------------------------------------------------- | -------------------------------------------- |
| **数据结构**     | 数组 + 链表，**分段数组（Segment[]）**                   | 数组 + 链表 + **红黑树**（链表长度≥8时转换） |
| **锁机制**       | **分段锁（Segment）**，每个 Segment 继承 `ReentrantLock` | **CAS + synchronized**，锁单个桶的头节点     |
| **锁粒度**       | 较粗（锁一段，含多个桶）                                 | 更细（只锁一个桶）                           |
| **并发度**       | 默认 16 个 Segment（固定）                               | 理论上无上限（桶级别）                       |
| **哈希冲突处理** | 链表                                                     | 链表 → 红黑树（优化查询效率）                |
| **size 计算**    | 分段统计后累加，多次尝试                                 | **CounterCell[] + baseCount**，CAS 更新      |

---

#### 线程安全保证机制（分版本说明）

1. **JDK 7**：

   核心是**分段锁（Segment）** —— 将整个哈希表拆分为多个独立的 Segment（每个 Segment 是一个小的 HashMap），每个 Segment 自身持有一把 ReentrantLock 锁。多线程访问时，只有操作同一个 Segment 的线程会竞争锁，操作不同 Segment 的线程可完全并行，通过 "锁分段" 降低锁竞争，保证线程安全。

2. **JDK 8**：

   抛弃分段锁，采用 "**CAS + synchronized + 自旋**" 的组合方案：

   - 对桶的首节点加 synchronized 锁（仅锁定当前桶，而非整个段），锁粒度大幅降低；
   - 新增 / 修改节点时，先用 CAS 尝试无锁更新（如 CAS 设置桶的首节点），失败后再降级为 synchronized 加锁；
   - 扩容时采用自旋 + CAS 保证多线程协同扩容，避免阻塞；
   - 同时保留 volatile 修饰节点的 value 和 next 指针，保证可见性。

#### **如何保证线程安全？**

##### **JDK 7：分段锁（Segment Locking）**

```java
// 结构：Segment[] → HashEntry[] → 链表
static final class Segment<K,V> extends ReentrantLock { ... }
```
- 将 Map 分成 16 个 Segment，每个 Segment 独立加锁
- 不同 Segment 的读写可**并行**，同一 Segment 需**串行**

##### **JDK 8：CAS + synchronized**

```java
// 核心操作：putVal 方法
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 1. 计算 hash，定位桶
    // 2. 桶为空：CAS 直接插入（无锁）
    // 3. 桶非空：synchronized 锁定头节点，再遍历/插入
}
```

| 场景       | 实现方式                                           |
| ---------- | -------------------------------------------------- |
| **桶为空** | **CAS** 无锁插入，失败自旋                         |
| **桶非空** | **synchronized** 锁定头节点，保证链表/树操作原子性 |
| **扩容**   | 多线程协作迁移数据（`transfer` 方法）              |
| **计数**   | `CounterCell[]` 分散热点，CAS 更新                 |

---

#### **一句话总结**

> **JDK 7 用"分段锁"把 Map 切成 16 份，JDK 8 用"CAS + 细粒度 synchronized"锁单个桶，并引入红黑树解决哈希冲突，并发性能更高。**

---

需要我展开 **扩容机制** 或 **size 计算的优化细节** 吗？这也是高频追问点。

### **ConcurrentHashMap**

**ConcurrentHashMap** 是 Java 中 **高并发场景下线程安全的哈希表**，核心是通过 **细粒度锁 + CAS + 无锁读** 兼顾安全与性能。下面以 **JDK 1.8（主流）** 为主，完整详解。

---

#### 一、为什么要用 ConcurrentHashMap？

- **HashMap**：线程不安全（并发扩容死循环、数据覆盖）
- **Hashtable / Collections.synchronizedMap**：全表锁，并发极差（同一时刻只能一个线程操作）
- **ConcurrentHashMap**：**局部锁 + 无锁读 + 多线程扩容**，并发能力极强

---

#### 二、JDK 1.8 底层结构

**Node 数组 + 链表 + 红黑树**（同 JDK8 HashMap，但线程安全）


- **Node**：普通节点，`val` 和 `next` 用 **volatile** 修饰（保证可见性、禁止重排）
- **TreeNode**：链表长度 ≥8 且数组长度≥64 时转为红黑树（查询 O(logn)）
- **ForwardingNode**：扩容标记节点（标记桶正在迁移）

---

#### 三、JDK 1.8 线程安全三剑客

##### 1. volatile（无锁读）

- 读操作 **全程不加锁**
- 靠 `volatile` 保证：
  - **可见性**：修改立即刷回主存，其他线程读到最新值
  - **禁止指令重排**：防止读到半初始化对象

##### 2. CAS（Compare-and-Swap）无锁更新

- 无锁原子操作：**比较内存值 == 预期值 → 才更新**
- 用于：
  - 数组初始化
  - 空桶插入新节点
  - 扩容标记、size 计数
- 优点：**无锁冲突时性能极高**

##### 3. synchronized 桶级锁

- CAS 失败时才加锁：**只锁当前桶的头节点**
- 锁粒度极小：不同桶完全并行
- JVM 对 `synchronized` 深度优化（偏向锁/轻量级锁），比 ReentrantLock 更快

---

#### 四、核心方法流程（put / get / 扩容）

##### 1. put 流程（最核心）

1. key/value 不允许为 null（HashMap 允许）
2. 二次哈希（spread）→ 减少碰撞
3. 自旋 + CAS 尝试**无锁插入空桶**
4. 若桶非空：
   - 遇到 **ForwardingNode**（正在扩容）→ **当前线程协助扩容**
   - 否则：**synchronized (头节点 f)** 加锁
     - 链表：遍历找到 key 覆盖 / 尾插
     - 红黑树：树节点插入
5. 链表 ≥8 且数组≥64 → **树化**
6. 检查是否需要扩容

##### 2. get 流程（全程无锁）

1. 哈希 → 定位桶
2. 遍历链表/红黑树
3. 用 `volatile` 保证读到最新值
4. **不锁、不阻塞、极快**

##### 3. 多线程协助扩容（JDK1.8 亮点）

- 单线程扩容会阻塞 → JDK8 支持**多线程协同迁移**
- 每个线程分到一段桶，**并发搬家**
- 用 **ForwardingNode** 标记迁移完成
- 迁移期间读写照常：
  - 读：能正常读
  - 写：会协助扩容 → 提升并发吞吐量

---

#### 五、JDK 1.7 vs JDK 1.8 对比（面试必问）

| 维度       | JDK 1.7                         | JDK 1.8                          |
| ---------- | ------------------------------- | -------------------------------- |
| **结构**   | Segment[] + HashEntry[] + 链表  | Node[] + 链表 + 红黑树           |
| **锁**     | Segment 分段锁（ReentrantLock） | **CAS + synchronized（桶头锁）** |
| **锁粒度** | 段（默认16）                    | **单个桶（更细）**               |
| **并发度** | 固定（分段数）                  | 动态（桶数）                     |
| **扩容**   | 单Segment独立扩容               | **多线程协助扩容**               |
| **查询**   | 链表O(n)                        | 链表O(n) / 红黑树O(logn)         |



---

#### 六、关键特性

1. **读完全无锁，写局部锁** → 高并发读场景性能接近 HashMap
2. **key/value 不允许 null**（避免歧义：null 是不存在还是值为 null）
3. **弱一致性**：
   - `size()`/`isEmpty()` 是**近似值**（并发修改不锁全表）
   - 迭代器是**弱一致**：不抛 ConcurrentModificationException
4. **原子操作**：
   - `putIfAbsent(k,v)`：不存在才 put
   - `computeIfAbsent`/`compute`/`merge`：复合原子操作

---

#### 七、与 HashMap / Hashtable 对比

| 特性      | HashMap          | Hashtable             | ConcurrentHashMap       |
| --------- | ---------------- | --------------------- | ----------------------- |
| 线程安全  | ❌                | ✅（全表synchronized） | ✅（CAS+桶synchronized） |
| 锁粒度    | -                | 整张表                | 单个桶                  |
| 并发      | 不安全           | 串行                  | 高并发                  |
| null键/值 | ✅                | ❌                     | ❌                       |
| 结构      | 数组+链表+红黑树 | 数组+链表             | 数组+链表+红黑树        |
| 扩容      | 单线程           | 单线程                | **多线程协助**          |

---

#### 八、一句话总结（面试必背）

**JDK1.7 分段锁；JDK1.8 CAS + synchronized 桶级锁 + 红黑树 + 多线程扩容，读无锁、写锁头、并发强、性能高。**

---

#### 桶

**桶 = ConcurrentHashMap / HashMap 底层数组的一个位置**
也就是 `Node[] table` 里的**一个下标元素**。

---

##### 1. 最直观理解

底层是个数组：
```java
Node<K,V>[] table = new Node[16];
```

- `table[0]` → 0 号桶
- `table[1]` → 1 号桶
- `table[2]` → 2 号桶
- ……
- `table[15]` → 15 号桶

**每一个格子，就是一个“桶”。**

---

##### 2. 桶里放什么？

每个桶里放的是：
- 要么：`null`（空桶）
- 要么：一个**链表头节点**
- 要么：一棵**红黑树根节点**

不同 key 算出 hash，落到同一个桶里，就形成链表/红黑树。

---

##### 3. 为什么叫“桶”？

形象比喻：
- 整个数组是一**排桶**
- 每个 key 根据 hash 被丢进**对应的桶里**
- 一个桶可以装多个节点（拉链法）

所以大家习惯叫：
**hash 桶 / 桶位 / 桶节点**

---

##### 4. 回到 ConcurrentHashMap 里的“桶级锁”

意思就是：
**只锁这一个数组位置，不锁整张表。**

- 线程 A 在锁 `table[3]`
- 线程 B 可以同时操作 `table[5]`
- 互不干扰 → 并发极高

这就是 **桶级锁** 的意义。

---

##### 超级精简总结

- **桶 = 数组的一个下标位置**
- **桶级锁 = 只锁这一个位置**
- 桶里装：链表 or 红黑树



------

### 9. **什么是fail-fast机制？它和fail-safe机制有什么区别？哪些集合实现了这些机制？**

- **fail-fast机制**：指的是在遍历集合时，如果集合在遍历过程中被修改，迭代器会抛出 `ConcurrentModificationException`。
- **fail-safe机制**：在遍历集合时，如果集合被修改，迭代器仍然能够正常工作，通常会工作在集合的一个副本上。
- **实现**：
  - `fail-fast`：`ArrayList`、`HashMap`、`HashSet`。
  - `fail-safe`：`CopyOnWriteArrayList`、`CopyOnWriteArraySet`。

------

### 10. **ArrayList的扩容机制是怎样的？初始容量和负载因子对其性能有什么影响？**

- **扩容机制**：当 `ArrayList` 的容量不足时，容量会翻倍（默认情况下）。
- **初始容量**：设置初始容量可以避免多次扩容，减少性能开销。
- **负载因子**：负载因子影响扩容时机，通常默认是 0.75。负载因子较大时，会减少扩容的次数，但会增加查询时间；负载因子较小时，扩容次数增多，但查询性能较好。

##### 一、ArrayList的底层结构与扩容触发条件

ArrayList底层基于**动态数组**实现（数组长度固定，ArrayList通过“复制旧数组→创建新数组”实现动态扩容）。

###### 扩容触发时机：

当调用`add()`方法添加元素时，若当前元素数量（`size`）等于数组容量（`elementData.length`），则触发扩容。

核心判断逻辑（简化源码）：
```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1); // 检查是否需要扩容
    elementData[size++] = e;
    return true;
}

// 确保容量至少为minCapacity（当前所需最小容量）
private void ensureCapacityInternal(int minCapacity) {
    if (minCapacity > elementData.length) {
        grow(minCapacity); // 触发扩容
    }
}
```

##### 二、扩容的具体过程（grow()方法）

扩容的核心是`grow()`方法，步骤如下：

1. **计算新容量**：
   - 旧容量（`oldCapacity`）= 当前数组长度；
   - 新容量默认扩容为 **`oldCapacity + (oldCapacity >> 1)`**（即旧容量的1.5倍，通过右移1位实现除以2的高效计算）；
   - 若新容量仍小于所需最小容量（`minCapacity`），则直接将新容量设为`minCapacity`（如初始添加大量元素时）；
   - 若新容量超过ArrayList的最大限制（`Integer.MAX_VALUE - 8`），则取`Integer.MAX_VALUE`。

2. **复制元素到新数组**：
   - 通过`Arrays.copyOf()`创建新数组（长度为新容量）；
   - 将旧数组的元素复制到新数组，旧数组被垃圾回收。

简化源码：
```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 1.5倍扩容
    if (newCapacity < minCapacity) {
        newCapacity = minCapacity;
    }
    if (newCapacity > MAX_ARRAY_SIZE) {
        newCapacity = hugeCapacity(minCapacity);
    }
    // 复制旧数组到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

##### 三、初始容量对性能的影响

ArrayList的**初始容量**是创建时指定的数组长度（默认初始容量为10，若通过`new ArrayList(initialCapacity)`指定，则初始容量为`initialCapacity`）。

###### 影响：

- **初始容量过小**：添加元素时会频繁触发扩容（每次扩容都需要复制数组，时间复杂度为O(n)），导致性能损耗；
- **初始容量过大**：会浪费内存空间（数组中未使用的位置占用内存）。

###### 最佳实践：

若已知大致元素数量，创建时指定初始容量（如`new ArrayList(1000)`），可减少扩容次数，平衡时间和空间效率。

##### 四、常见误区：ArrayList没有“负载因子”

需要注意：**负载因子（load factor）是哈希表（如HashMap）的概念**，用于判断何时触发扩容（`元素数量 > 容量 × 负载因子`）。

而ArrayList的扩容仅与“元素数量是否等于当前容量”有关，**与负载因子无关**——只要`size == 容量`，就会立即扩容，不存在“负载因子”的计算逻辑。

##### 总结

- **扩容机制**：当元素数量达到当前容量时，默认扩容为原容量的1.5倍（通过复制数组实现）；
- **初始容量**：合理设置初始容量可减少扩容次数，避免频繁复制数组的性能开销；
- **负载因子**：与ArrayList无关，是哈希表（如HashMap）的属性。

###### 简单说：ArrayList的扩容是“满了就扩1.5倍”，初始容量设置得越合理，性能越好。

### 11. **HashMap的底层数据结构在JDK 8中有什么变化？为什么要做这样的调整？**

- **变化**：JDK 8 中，`HashMap` 在链表长度超过 8 时会转换成红黑树，以提高查询效率。
- **原因**：当链表长度过长时，性能会退化成 O(n)，转换为红黑树可以使查询操作的时间复杂度降低为 O(log n)。

#### 面试级精准解答（结构清晰 + 底层逻辑 + 面试加分点）

#### 一、JDK 8 HashMap 底层数据结构核心变化

不只是 “链表转红黑树”，完整变化包含 2 个核心点：

1. **基础结构升级**：从 JDK 7 的 “数组 + 单向链表”，变为 JDK 8 的 “数组 + 单向链表 + 红黑树”；
2. **辅助优化**：链表节点（Entry）重命名为 Node，且链表从**单向**改为**双向**（为红黑树转换做铺垫）；
3. **触发条件**：当单个桶（数组位置）的链表长度 ≥ 8，且数组容量 ≥ 64 时，链表转为红黑树；若后续树节点数量 ≤ 6，会回退为链表（避免红黑树维护成本高于收益）。

#### 二、为什么做这样的调整？（底层原因 + 面试加分）

1. 核心痛点解决：

   JDK 7 中 HashMap 存在 “哈希冲突严重时，链表过长导致查询性能退化” 的问题 —— 链表查询的时间复杂度是 O (n)，当链表长度达到 8 时，查询效率大幅下降；而红黑树是平衡二叉树，查询 / 插入 / 删除的时间复杂度稳定在 O (log n)，能显著提升高冲突场景下的性能。

2. 阈值设计的合理性：

   链表转红黑树的阈值设为 8，是基于泊松分布的统计结论：正常哈希分布下，链表长度达到 8 的概率仅为 0.00000006（亿分之六），既避免了正常场景下频繁转树的开销，又能覆盖极端哈希冲突的场景；回退阈值设为 6（而非 8），是为了避免 “阈值边界反复切换”（比如长度在 8 左右波动时，频繁转树 / 转链表）。

3. 数组容量≥64 的前置条件：

   避免数组容量过小时（比如刚初始化），因哈希分布不均导致局部链表过长就转树 —— 此时优先扩容数组（让元素更均匀分布），比维护红黑树成本更低。

#### 面试一句话总结（核心记忆）

JDK 8 把 HashMap 的 “数组 + 链表” 升级为 “数组 + 链表 + 红黑树”，当链表长度≥8 且数组容量≥64 时转红黑树，核心是解决链表过长导致的查询性能从 O (n) 退化的问题，通过红黑树将查询复杂度优化到 O (log n)。

#### 总结

1. 核心变化：新增红黑树结构，链表长度≥8 且数组容量≥64 时转树，节点改为双向链表；
2. 调整原因：解决链表过长的查询性能问题，利用红黑树的 O (log n) 复杂度提升极端场景性能，同时通过阈值设计平衡红黑树的维护成本。

### HashMap

---

#### 一、一句话核心

**HashMap = 数组 + 链表 + 红黑树**
- 基于 **哈希表** 实现
- **线程不安全**
- 增删改查接近 O(1)，性能极高
- JDK 1.8 优化：链表过长转**红黑树**

---

#### 二、底层结构（JDK 1.8）

```
Node<K,V>[] table → 数组（桶）
   ↓
每个位置是链表或红黑树
```

- **Node 数组**：默认长度 **16**，必须是 **2的n次方**
- **链表**：哈希冲突时，同一桶里的节点串起来
- **红黑树**：当链表长度 ≥8 **且数组长度≥64** 时转为树，查询从 O(n) → O(logn)

---

#### 三、最重要流程：put 方法

1. **key 做 hash**
   ```java
   hash = key.hashCode() ^ (key.hashCode() >>> 16)
   ```
   目的：让高位也参与运算，**减少冲突**。

2. **计算下标（桶位置）**
   ```java
   index = hash & (table.length - 1)
   ```
   等价取模，但更快。

3. 分情况：
   - 桶为空 → **直接放**
   - 桶不为空
     - key 已存在 → **覆盖 value**
     - 是链表 → 尾插，长度≥8 树化
     - 是红黑树 → 树节点插入

4. 大小超过阈值 → **扩容**

---

#### 四、扩容机制（高频考点）

##### 1. 扩容条件

- 元素个数 > **容量 × 负载因子（0.75）**
- 默认：16 × 0.75 = **12** 就扩容

##### 2. 扩容规则

- **新容量 = 旧容量 × 2**（永远是2的幂）
- 重新计算所有下标
- JDK 1.8 优化：
  节点在新数组中 **要么在原位置，要么原位置+旧容量**
  不用重新全部 hash

##### 3. 为什么容量必须是 2 的n次方？

- `hash & (len-1)` 计算下标极快
- 分布均匀，减少冲突
- 扩容时方便重新定位

---

#### 五、哈希冲突怎么解决？

**拉链法（链地址法）**
- 同一个桶里用链表串起来
- JDK 1.7 头插
- JDK 1.8 **尾插**（避免扩容死循环）

冲突严重 → 转红黑树。

---

#### 六、JDK 1.7 vs JDK 1.8 区别

| 对比      | JDK 1.7   | JDK 1.8          |
| --------- | --------- | ---------------- |
| 结构      | 数组+链表 | 数组+链表+红黑树 |
| 插入      | 头插法    | 尾插法           |
| 哈希冲突  | 链表O(n)  | 树化O(logn)      |
| 扩容      | 易死循环  | 修复死循环       |
| hash 计算 | 简单      | 更散列           |

---

#### 七、为什么 HashMap 线程不安全？

1. **多线程 put 会覆盖数据**
2. **JDK1.7 扩容会产生环形链表，导致死循环**
3. 不是原子操作，没有锁

高并发下必须用 **ConcurrentHashMap**。

---

#### 八、高频面试题（必背）

##### 1. HashMap 的 key 为什么常用 String、Integer？

因为它们**不可变、重写了 equals() 和 hashCode()**，保证哈希稳定。

##### 2. 为什么重写 equals 必须重写 hashCode？

- equals 相等 → hashCode **必须相等**
- 否则会存到不同桶，找不到对象

##### 3. 负载因子为什么是 0.75？

空间和时间的平衡：
- 太大：冲突严重
- 太小：浪费空间

##### 4. 为什么树化阈值是 8？

泊松分布统计，冲突到 8 的概率极低，
树化是**极端冲突兜底方案**。

---

#### 九、一句话终极总结

**HashMap = 数组+链表+红黑树，通过哈希定位，冲突拉链，过长树化，自动扩容，线程不安全，性能极高。**

------

### 12. **什么是哈希冲突？HashMap中是如何解决哈希冲突的？**

- **哈希冲突**：当多个键的 `hashCode` 值相同或相近时，发生哈希冲突，即多个元素被映射到相同的桶中。
- **解决方法**：
  - `HashMap` 使用链表法（JDK 7）或红黑树（JDK 8）来解决哈希冲突，将所有哈希冲突的元素放入一个链表或树结构中。

------

### 13. **LinkedHashMap如何实现元素的有序性？它和HashMap在性能上有什么差异？**

- **实现有序性**：`LinkedHashMap` 使用双向链表维护元素的插入顺序或访问顺序。
- **性能差异**：`LinkedHashMap` 在插入和删除操作上略慢于 `HashMap`，因为它需要维护链表结构。

------

### 14. **Vector和ArrayList有什么区别？在多线程环境下应该如何选择使用它们？**

- **区别**：
  - `Vector` 是线程安全的，但性能较差，因为它的方法都使用了同步机制。
  - `ArrayList` 不是线程安全的，性能较好。
- **多线程环境下选择**：如果需要线程安全，可以使用 `Vector` 或 `CopyOnWriteArrayList`，但性能上会有所妥协；如果线程安全不是问题，可以选择 `ArrayList`。

------

### 15. **什么是集合的视图（View）？keySet()、entrySet()和values()方法返回的视图有什么特点？**

- **集合视图**：视图是集合的只读映射，可以通过视图来操作集合的内容。
- **视图特点**：
  - `keySet()`：返回集合中所有键的视图。
  - `entrySet()`：返回所有键值对的视图。
  - `values()`：返回集合中所有值的视图。

------

### 16. **Collections工具类提供了哪些常用的方法？如何使用它实现集合的排序、查找和同步控制？**

- **常用方法**：
  - `sort()`：对集合进行排序。
  - `binarySearch()`：在已排序的集合中查找元素。

- `synchronizedList()`、`synchronizedMap()`：为集合提供同步控制。

  - **示例**：

    ```java
    List<Integer> list = Arrays.asList(3, 1, 2);
    Collections.sort(list);
    ```





##### 一、Collections工具类的常用方法分类

Collections是Java集合框架的“工具类”，所有方法均为`static`，主要功能包括：集合排序、查找、修改、同步控制、不可变集合创建等。

##### 二、核心方法及使用示例

###### 1. 排序相关方法

- **`sort(List<T> list)`**：对List按**自然顺序**排序（要求元素实现`Comparable`接口）。
- **`sort(List<T> list, Comparator<? super T> c)`**：按**自定义比较器**排序。

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 2));

// 自然排序（从小到大）
Collections.sort(numbers);
System.out.println(numbers); // [1, 2, 3, 4]

// 自定义比较器（从大到小）
Collections.sort(numbers, (a, b) -> b - a);
System.out.println(numbers); // [4, 3, 2, 1]
```

###### 2. 查找相关方法

- **`binarySearch(List<? extends Comparable<? super T>> list, T key)`**：在**已排序的List**中二分查找元素，返回索引（未找到返回负数）。
- **`max(Collection<? extends T> coll)`** / **`min(Collection<? extends T> coll)`**：查找集合中的最大/小元素（依赖自然排序或比较器）。

```java
List<String> words = new ArrayList<>(Arrays.asList("apple", "banana", "cherry"));
Collections.sort(words); // 二分查找前必须先排序

// 二分查找
int index = Collections.binarySearch(words, "banana");
System.out.println(index); // 1（找到，索引为1）

// 查找最大/小元素
String maxWord = Collections.max(words);
String minWord = Collections.min(words);
System.out.println(maxWord); // cherry（按字母序最大）
System.out.println(minWord); // apple（按字母序最小）
```

###### 3. 同步控制方法（线程安全包装）

为非线程安全的集合（如ArrayList、HashMap）提供**线程安全的包装类**，通过`synchronized`关键字实现同步。

- **`synchronizedList(List<T> list)`**：返回线程安全的List。
- **`synchronizedSet(Set<T> s)`**：返回线程安全的Set。
- **`synchronizedMap(Map<K,V> m)`**：返回线程安全的Map。

```java
// 创建线程安全的List
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
syncList.add("a");
syncList.add("b");
// 多线程环境下操作syncList，无需手动加锁

// 线程安全的Map
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
```

**注意**：这些方法返回的同步集合性能较低（全局锁），高并发场景推荐使用`java.util.concurrent`包下的集合（如`CopyOnWriteArrayList`、`ConcurrentHashMap`）。

###### 4. 其他常用方法

- **`reverse(List<?> list)`**：反转集合元素顺序。
- **`shuffle(List<?> list)`**：随机打乱集合元素（如洗牌）。
- **`fill(List<? super T> list, T obj)`**：用指定元素填充集合（覆盖所有元素）。
- **`copy(List<? super T> dest, List<? extends T> src)`**：复制源集合到目标集合（要求目标集合容量≥源集合）。

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));

// 反转
Collections.reverse(list);
System.out.println(list); // [3, 2, 1]

// 打乱
Collections.shuffle(list);
System.out.println(list); // 随机顺序（如 [2, 3, 1]）

// 填充
Collections.fill(list, 0);
System.out.println(list); // [0, 0, 0]
```

###### 5. 不可变集合创建

返回**不可修改的集合**（任何修改操作会抛出`UnsupportedOperationException`）：

- **`unmodifiableList(List<? extends T> list)`**
- **`unmodifiableSet(Set<? extends T> s)`**
- **`unmodifiableMap(Map<? extends K,? extends V> m)`**

```java
List<String> original = new ArrayList<>();
original.add("a");
// 创建不可变视图
List<String> unmodifiable = Collections.unmodifiableList(original);
unmodifiable.add("b"); // 抛出UnsupportedOperationException
```

**用途**：保护集合不被意外修改（如返回方法内部集合时，避免外部篡改）。

##### 三、总结

Collections工具类的核心价值是**“封装集合的通用操作”**，常用场景包括：
- 排序：`sort()`（自然排序/比较器排序）；
- 查找：`binarySearch()`、`max()`、`min()`；
- 线程安全：`synchronizedXxx()`（简单场景使用）；
- 其他操作：反转、打乱、填充、创建不可变集合等。

使用时需注意：二分查找前必须先排序，同步集合性能有限，不可变集合禁止修改操作。这些方法大幅减少了重复代码，是操作集合的“利器”。
### 17. **什么是弱引用（WeakReference）？WeakHashMap与普通HashMap有什么区别？**

- **弱引用**：弱引用指的是一种不阻止对象被垃圾回收的引用类型。垃圾回收时，如果只有弱引用指向某对象，那么该对象会被回收。
- **区别**：`WeakHashMap` 中的键是弱引用，当键不再被其他对象引用时，垃圾回收器会自动回收键及其对应的值。而 `HashMap` 中的键值不会被垃圾回收。

------

### 18. **如何将一个集合转换为线程安全的集合？不同的转换方式有什么优缺点？**

- **转换方式**：
  - 使用 `Collections.synchronizedXXX()` 方法将集合包装为线程安全的集合。
  - 使用 `CopyOnWriteXXX` 类（如 `CopyOnWriteArrayList`）来实现线程安全。
- **优缺点**：
  - `Collections.synchronizedXXX()`：适合较少修改的集合，性能较差。
  - `CopyOnWriteXXX`：适合频繁读取而少量修改的集合，性能较好，但内存消耗较大。

------

### 19. **集合框架中的空集合（empty set）和不可变集合（unmodifiable set）有什么区别？如何创建它们？**

- **区别**：

  - **空集合**：没有元素的集合，通常通过 `Collections.emptyList()` 或 `Collections.emptySet()` 来创建。
  - **不可变集合**：集合创建后不能修改，通过 `Collections.unmodifiableXXX()` 方法创建。

- **示例**：

  ```java
  Set<String> emptySet = Collections.emptySet();
  Set<String> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>());
  ```

------

### 20. **在实际开发中，如何根据业务场景选择合适的集合实现类？请举例说明选择依据。**

- **选择依据**：
  - **查询性能**：如果需要频繁查询，选择 `HashSet` 或 `HashMap`。
  - **元素有序性**：如果需要按顺序处理元素，选择 `TreeSet` 或 `TreeMap`。
  - **线程安全**：如果涉及并发访问，可以使用 `CopyOnWriteArrayList`、`ConcurrentHashMap` 或 `Collections.synchronizedXXX()`。
  - **内存消耗与性能**：对于需要快速读取且修改较少的场景，`ArrayList` 或 `LinkedHashMap` 性能较好。

### 21. sort方法实现的底层原理


Java 中 `Collections.sort()` 方法（以及 `Arrays.sort()`，两者底层逻辑一致）的底层实现原理会根据排序对象的类型和版本有所不同，核心是**根据元素类型选择高效的排序算法**，以平衡时间复杂度和实际性能。

#### 一、核心原则：“不同场景，不同算法”

Java 对排序算法的选择遵循“因地制宜”的原则，主要依据：
- 元素类型（基本类型 / 对象类型）；
- 数据量大小；
- 是否实现了 `Comparable` 接口或使用 `Comparator`。

JDK 7 及之后的版本中，核心排序算法是**双轴快速排序（Dual-Pivot Quicksort）**、**归并排序（Merge Sort）** 和**插入排序（Insertion Sort）** 的组合，通过“分段策略”实现高效排序。

#### 二、具体实现逻辑（以 JDK 8 为例）

##### 1. 对基本类型数组（如 `int[]`、`long[]`）

采用 **双轴快速排序（Dual-Pivot Quicksort）**，这是对传统快速排序的优化：
- **传统快排**：选择一个轴点（pivot），将数组分为“小于轴点”和“大于轴点”两部分；
- **双轴快排**：选择两个轴点（pivot1 < pivot2），将数组分为三部分：小于 pivot1、介于 pivot1 和 pivot2 之间、大于 pivot2，减少递归深度，提高缓存利用率。

**适用场景**：基本类型（`int`、`long`、`char` 等），因为它们无需考虑稳定性（排序后相等元素的相对位置不变）。

**时间复杂度**：平均 O(n log n)，最坏 O(n²)（但通过随机化轴点选择，实际很少出现最坏情况）。

##### 2. 对对象数组（如 `Integer[]`、自定义对象数组）

采用 **TimSort 算法**（归并排序的优化版本，由 Python 创始人 Tim Peters 设计）：
- **核心思想**：利用数据中可能存在的“已排序片段（run）”，通过归并这些片段实现高效排序，减少比较次数。
- **步骤**：
  1. 扫描数组，找出所有已排序的片段（升序或降序，降序会转为升序）；
  2. 按一定规则合并这些片段（类似归并排序的合并步骤），最终得到完整的有序数组。

**适用场景**：对象数组，因为 TimSort 是**稳定排序**（相等元素的相对位置不变），而对象排序通常需要保持稳定性（如先按年龄排序，再按姓名排序时，同年龄的人姓名排序应保留原相对位置）。

**时间复杂度**：O(n log n)，且在实际数据（尤其是有部分已排序的数据）上表现优于传统归并排序。

##### 3. 小数据量优化：插入排序

无论是基本类型还是对象类型，当排序的子数组长度小于**阈值**（JDK 中默认是 47 或 60，不同版本略有差异）时，会改用**插入排序**：
- 插入排序在小数据量时效率更高（无需递归或复杂的分区操作，常数时间小）；
- 当子数组长度小于阈值时，双轴快排或 TimSort 会切换为插入排序。

#### 三、`Collections.sort()` 与 `Arrays.sort()` 的关系

`Collections.sort(List)` 底层依赖 `Arrays.sort()` 实现：

1. 将 List 转换为数组（`list.toArray()`）；
2. 调用 `Arrays.sort()` 对数组排序；
3. 将排序后的数组元素复制回 List（通过迭代器修改）。

简化源码逻辑：
```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    Object[] a = list.toArray();
    Arrays.sort(a); // 核心排序逻辑
    ListIterator<T> i = list.listIterator();
    for (Object e : a) {
        i.next();
        i.set((T) e); // 复制回List
    }
}
```

#### 四、总结

Java 中 `sort` 方法的底层实现是“**算法组合 + 场景适配**”的典范：
- 基本类型用**双轴快速排序**（追求效率，无需稳定）；
- 对象类型用**TimSort**（稳定排序，适应实际数据特性）；
- 小数据量用**插入排序**（优化常数时间）。

这种设计既保证了理论上的高效性（O(n log n)），又在实际场景中（尤其是有部分有序的数据）表现优异，是 Java 集合排序的核心支撑。



### 22.HashSet和LinkedHashSet的区别

`HashSet` 和 `LinkedHashSet` 都是 Java 中 `Set` 接口的实现类，均保证元素唯一性，但在**底层结构**和**元素顺序**上有显著区别：

#### 1. 底层数据结构

- **HashSet**：基于 **哈希表（HashMap）** 实现，仅存储元素的哈希值用于快速查找，没有额外的顺序维护结构。
- **LinkedHashSet**：继承自 `HashSet`，底层在哈希表基础上额外维护了一个 **双向链表**，用于记录元素的**插入顺序**。

#### 2. 元素顺序

- **HashSet**：**不保证任何顺序**。元素的存储位置由哈希值计算决定，迭代时的顺序可能与插入顺序无关，甚至在扩容后发生变化。
- **LinkedHashSet**：**保证插入顺序**。通过双向链表记录元素插入的先后顺序，迭代时会严格按照元素的插入顺序输出。

#### 3. 性能

- **HashSet**：由于无需维护链表结构，**插入、删除、查询的性能略高于** `LinkedHashSet`（差异在大数据量下才明显）。
- **LinkedHashSet**：因为要额外维护双向链表，会消耗更多内存，且插入/删除时需要同时操作哈希表和链表，性能略低。

#### 4. 适用场景

- **HashSet**：适合只需要去重、不关心元素顺序的场景（如存储唯一标识集合）。
- **LinkedHashSet**：适合需要去重且**保留插入顺序**的场景（如实现“最近访问记录”“去重且保持顺序的列表”）。

#### 示例对比

```java
Set<String> hashSet = new HashSet<>();
Set<String> linkedHashSet = new LinkedHashSet<>();

hashSet.add("b");
hashSet.add("a");
hashSet.add("c");

linkedHashSet.add("b");
linkedHashSet.add("a");
linkedHashSet.add("c");

System.out.println(hashSet);      // 输出可能为 [a, b, c] 或其他无序组合
System.out.println(linkedHashSet); // 必定输出 [b, a, c]（严格按插入顺序）
```


总结：`LinkedHashSet` 可以看作是 `HashSet` 的“有序版本”，通过牺牲少量性能和内存换取了插入顺序的保证。



### 23.HashMap和LinkedHashMap的区别

`HashMap` 和 `LinkedHashMap` 都是 Java 中 `Map` 接口的实现类，用于存储键值对（Key-Value），且 Key 具有唯一性，但二者在**底层结构**、**元素顺序**和**性能特性**上有显著区别：

#### 1. 底层数据结构

- **HashMap**：基于 **哈希表（数组 + 链表/红黑树）** 实现，仅通过 Key 的哈希值计算存储位置，无额外结构维护顺序。
- **LinkedHashMap**：继承自 `HashMap`，在哈希表基础上额外维护了一个 **双向链表**，用于记录键值对的**插入顺序**或**访问顺序**。

#### 2. 元素顺序

- **HashMap**：**不保证任何顺序**。键值对的存储位置由 Key 的哈希值决定，迭代时的顺序可能与插入顺序无关，且在扩容后可能发生变化。
- **LinkedHashMap**：**可以保证顺序**，具体分为两种：
  - **插入顺序（默认）**：迭代时按键值对的插入先后顺序输出（先插入的先遍历）。
  - **访问顺序**：通过构造方法 `LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)` 设置 `accessOrder = true` 后，每次访问（`get` 或 `put` 已存在的 Key）会将该键值对移到链表尾部，迭代时按“最近最少访问”到“最近访问”的顺序输出（可用于实现 LRU 缓存）。

#### 3. 性能与内存

- **HashMap**：由于无需维护链表结构，**插入、删除、查询的性能略高于** `LinkedHashMap`（大数据量下差异更明显），内存占用更少。
- **LinkedHashMap**：因额外维护双向链表，插入和删除时需要同时操作哈希表和链表，性能略低，且内存消耗更大。

#### 4. 适用场景

- **HashMap**：适合只需要键值对存储、不关心顺序的场景（如普通缓存、数据映射），是最常用的 Map 实现类。
- **LinkedHashMap**：适合需要**保留插入顺序**（如日志记录、有序配置项）或需要**实现 LRU 缓存**（通过访问顺序）的场景。

##### 示例对比

```java
// HashMap 不保证顺序
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("b", 2);
hashMap.put("a", 1);
hashMap.put("c", 3);
System.out.println(hashMap); // 输出可能为 {a=1, b=2, c=3} 或其他无序组合

// LinkedHashMap 默认按插入顺序
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("b", 2);
linkedHashMap.put("a", 1);
linkedHashMap.put("c", 3);
System.out.println(linkedHashMap); // 必定输出 {b=2, a=1, c=3}（插入顺序）

// LinkedHashMap 按访问顺序（LRU 特性）
Map<String, Integer> accessOrderedMap = new LinkedHashMap<>(16, 0.75f, true);
accessOrderedMap.put("b", 2);
accessOrderedMap.put("a", 1);
accessOrderedMap.get("b"); // 访问 "b"，会移到链表尾部
System.out.println(accessOrderedMap); // 输出 {a=1, b=2}（最近访问的 "b" 在最后）
```


总结：`LinkedHashMap` 是 `HashMap` 的增强版，通过牺牲少量性能和内存，换取了对键值对顺序的控制能力，适用于需要有序性的场景。

### 24.`Set<Integer> set = new LinkedHashSet<>(list); 和 LinkedHashSet<Integer> set = new LinkedHashSet<>(list)`; 两者的区别


这两行代码的**核心功能完全一致**（都是创建一个包含 `list` 元素的 `LinkedHashSet`，实现去重并保留插入顺序），但在 **变量类型声明** 和 **后续可调用的方法范围** 上存在关键区别，本质是 Java 中“父类引用指向子类对象”的多态特性体现。

#### 1. 变量类型与方法访问权限

两者的核心差异在于 **左侧变量的声明类型**（`Set<Integer>`  vs  `LinkedHashSet<Integer>`），这决定了后续能通过 `set` 变量调用哪些方法：

| 对比维度                | `Set<Integer> set = new LinkedHashSet<>(list)` | `LinkedHashSet<Integer> set = new LinkedHashSet<>(list)` |
|-------------------------|------------------------------------------------|-----------------------------------------------------------|
| **变量声明类型**        | 父接口 `Set`（`LinkedHashSet` 实现了 `Set`）   | 具体实现类 `LinkedHashSet`                                |
| **可调用的方法**        | 仅能调用 `Set` 接口定义的方法（如 `add()`、`remove()`、`contains()`、`size()` 等） | 可调用 `LinkedHashSet` 的所有方法：<br>1. 继承自 `Set` 的方法；<br>2. `LinkedHashSet` 独有的方法（如 `spliterator()` 增强实现、`removeEldestEntry()` 等）；<br>3. 继承自 `HashSet` 的方法（如 `clone()`）。 |
| **是否暴露实现细节**    | 隐藏了具体实现（只暴露接口），符合“面向接口编程”原则 | 暴露了具体实现类，耦合了 `LinkedHashSet` 的实现细节       |

#### 2. 代码灵活性与扩展性

- **`Set<Integer> set = ...`（父接口声明）**：  
  更符合“面向接口编程”的设计思想——后续若需将 `LinkedHashSet` 替换为其他 `Set` 实现类（如 `HashSet`、`TreeSet`），只需修改右侧的创建逻辑，左侧变量声明和所有调用 `set` 的代码无需改动，灵活性更高。  
  示例：  
  ```java
  // 后续若需改用 TreeSet（按自然排序），仅需修改右侧
  Set<Integer> set = new TreeSet<>(list); 
  // 左侧变量类型不变，调用 set.add()、set.contains() 等代码无需修改
  ```

- **`LinkedHashSet<Integer> set = ...`（具体类声明）**：  
  后续若要替换为其他 `Set` 实现类（如 `TreeSet`），左侧变量类型必须同步修改（否则会出现“类型不兼容”编译错误），扩展性较差。  
  示例：  
  ```java
  // 若想改用 TreeSet，左侧必须改为 TreeSet<Integer>，否则编译报错
  TreeSet<Integer> set = new TreeSet<>(list); 
  ```

#### 3. 示例：方法调用差异

假设需要调用 `LinkedHashSet` 继承自 `HashSet` 的 `clone()` 方法（该方法在 `Set` 接口中未定义）：
```java
// 1. 父接口声明：编译报错（Set 接口没有 clone() 方法）
Set<Integer> set1 = new LinkedHashSet<>(list);
Set<Integer> clonedSet1 = set1.clone(); // 错误：Cannot resolve method 'clone()'

// 2. 具体类声明：正常编译（LinkedHashSet 继承了 clone() 方法）
LinkedHashSet<Integer> set2 = new LinkedHashSet<>(list);
LinkedHashSet<Integer> clonedSet2 = set2.clone(); // 正确：可调用 clone()
```

#### 总结：如何选择？

| 场景                     | 推荐写法                                  | 原因                                                                 |
|--------------------------|-------------------------------------------|----------------------------------------------------------------------|
| 仅需使用 `Set` 接口的通用功能（去重、判断包含等），不依赖具体实现 | `Set<Integer> set = new LinkedHashSet<>(list)` | 面向接口编程，降低耦合，便于后续替换实现类（如 `TreeSet`）。         |
| 需要调用 `LinkedHashSet` 独有的方法（如 `clone()`、`removeEldestEntry()`） | `LinkedHashSet<Integer> set = new LinkedHashSet<>(list)` | 必须通过具体类声明才能访问非 `Set` 接口定义的方法。                 |

日常开发中，**优先推荐第一种写法**（父接口声明），除非明确需要使用 `LinkedHashSet` 的特有方法——这是保证代码灵活性和可维护性的最佳实践。

#### 四种区别


你列出的四个写法中，后两种（`LinkedHashSet<Integer> set = new Set<>(list);` 和 `Set<Integer> set = new Set<>(list);`）在 Java 中是**编译错误**的，因为 `Set` 是接口（interface），不能直接通过 `new` 关键字实例化（接口没有构造方法）。

下面先明确**合法写法**的区别，再解释错误写法的问题：

##### 一、两种合法写法的区别（前两种）

```java
// 写法1：接口声明，实现类实例化
Set<Integer> set1 = new LinkedHashSet<>(list);

// 写法2：实现类声明，实现类实例化
LinkedHashSet<Integer> set2 = new LinkedHashSet<>(list);
```

两者的核心区别在于**变量声明类型**，具体差异如下：

| 维度                | 写法1（`Set<Integer> set1`）                          | 写法2（`LinkedHashSet<Integer> set2`）                |
|---------------------|------------------------------------------------------|------------------------------------------------------|
| **变量类型**        | 接口 `Set`（`LinkedHashSet` 的父接口）                | 具体实现类 `LinkedHashSet`                           |
| **可调用的方法**    | 只能调用 `Set` 接口中定义的方法（如 `add()`、`contains()`） | 可调用 `LinkedHashSet` 的所有方法（包括继承自 `Set` 和自身特有的方法，如 `clone()`） |
| **灵活性**          | 高：未来可无缝替换为其他 `Set` 实现类（如 `TreeSet`）  | 低：与 `LinkedHashSet` 强耦合，替换实现类需修改声明类型 |
| **设计原则**        | 符合“面向接口编程”，隐藏实现细节                      | 暴露实现细节，耦合度高                                |

##### 二、两种错误写法的问题（后两种）

```java
// 写法3：错误
LinkedHashSet<Integer> set3 = new Set<>(list);

// 写法4：错误
Set<Integer> set4 = new Set<>(list);
```

这两种写法都会导致**编译错误**，原因是：  
`Set` 是 Java 中的**接口**（`interface`），而接口的特性是：  

1. 没有构造方法，不能通过 `new Set(...)` 实例化；  
2. 必须通过其实现类（如 `HashSet`、`LinkedHashSet`、`TreeSet`）才能创建对象。  

因此，`new Set<>(list)` 本身就是语法错误，无论左侧声明为哪种类型，都会提示“`Set` 是抽象的，无法实例化”。

##### 总结

- 只有前两种写法是合法的，区别在于变量声明类型（接口 vs 实现类），影响代码的灵活性和可调用的方法范围。  
- 后两种写法因试图实例化接口 `Set` 而导致编译错误，在 Java 中永远无法运行。  

日常开发中，**推荐第一种写法**（`Set<Integer> set = new LinkedHashSet<>(list)`），这是“面向接口编程”的最佳实践，能降低代码耦合度，便于后续扩展。



#### 菱形语法


在Java中，这两种写法的**功能是完全相同的**，都是创建一个存储`Integer`类型元素的`HashSet`集合。它们的区别主要体现在**语法版本和代码简洁性**上：

1. **`Set<Integer> seen = new HashSet<>();`**  
   这是Java 7及以上版本引入的**菱形语法（Diamond Syntax）**。  
   编译器会根据左边声明的泛型类型（`Set<Integer>`）自动推断出右边`HashSet`的泛型类型（`Integer`），因此可以省略右边的泛型参数（`Integer`），仅保留`<>`。  
   这种写法更简洁，减少了代码冗余。

2. **`Set<Integer> seen = new HashSet<Integer>();`**  
   这是Java 7之前的标准写法，需要在创建`HashSet`时显式指定泛型类型（`Integer`）。  
   虽然在Java 7及以上版本中仍然可以使用，但显得不够简洁。

**总结**：  
两种写法在功能上没有区别，推荐使用第一种（菱形语法），因为它更简洁且符合现代Java的编码习惯。菱形语法的引入就是为了简化泛型代码的编写，避免重复声明泛型类型。



### 25.comparator

`Comparator`（比较器）是 Java 中的一个函数式接口（`@FunctionalInterface`），位于 `java.util` 包下，主要用于**定义两个对象之间的比较规则**，实现对象的自定义排序。

它与 `Comparable` 的核心区别是：`Comparable` 是对象“自身携带”的排序逻辑（如 `String` 自带的字典序排序），而 `Comparator` 是“外部定义”的排序规则，更灵活。

#### 核心方法

`Comparator` 接口的核心抽象方法是：
```java
int compare(T o1, T o2);
```
- 返回值规则：
  - 若 `o1` 应排在 `o2` 前面，返回**负数**（如 `-1`）；
  - 若 `o1` 与 `o2` 排序位置相同，返回**0**；
  - 若 `o1` 应排在 `o2` 后面，返回**正数**（如 `1`）。

#### 基本使用场景

当需要对**自定义对象**或**改变默认排序规则**时，`Comparator` 非常有用。

##### 示例1：自定义对象排序

假设有 `Student` 类，需按年龄排序：
```java
class Student {
    private String name;
    private int age;
    
    // 构造器、getter
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public int getAge() { return age; }
    public String getName() { return name; }
    @Override
    public String toString() { return name + "(" + age + ")"; }
}
```

使用 `Comparator` 定义排序规则：
```java
import java.util.*;

public class ComparatorDemo {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
            new Student("张三", 20),
            new Student("李四", 18),
            new Student("王五", 22)
        );
        
        // 1. 匿名内部类写法（传统）
        Comparator<Student> ageComparator = new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                // 按年龄升序：s1.age - s2.age
                return s1.getAge() - s2.getAge();
            }
        };
        
        // 2. Lambda 表达式写法（简洁）
        Comparator<Student> ageComparatorLambda = (s1, s2) -> s1.getAge() - s2.getAge();
        
        // 排序
        students.sort(ageComparatorLambda);
        System.out.println(students); 
        // 输出：[李四(18), 张三(20), 王五(22)]
    }
}
```

##### 示例2：改变默认排序规则

`String` 默认按字典序排序，若需按长度排序：
```java
List<String> words = Arrays.asList("apple", "banana", "cat", "dog");

// 按字符串长度升序排序
words.sort((s1, s2) -> s1.length() - s2.length());
System.out.println(words); // 输出：[cat, dog, apple, banana]
```

#### 常用默认方法与工具方法

Java 8 为 `Comparator` 增加了许多默认方法和静态方法，简化复杂排序逻辑：

1. **链式排序（多条件排序）**  
   使用 `thenComparing` 实现“主条件排序后，按次条件排序”：
   ```java
   // 先按年龄升序，年龄相同则按姓名字典序
   Comparator<Student> comparator = Comparator
       .comparingInt(Student::getAge)  // 主条件：年龄
       .thenComparing(Student::getName); // 次条件：姓名
   ```

2. **逆序排序**  
   使用 `reversed()` 实现降序：
   ```java
   // 按年龄降序
   students.sort(Comparator.comparingInt(Student::getAge).reversed());
   ```

3. **静态工具方法**  
   - `comparingInt`：按 `int` 类型属性排序（如 `Comparator.comparingInt(Student::getAge)`）
   - `comparing`：按任意类型属性排序（如 `Comparator.comparing(Student::getName)`）
   - `naturalOrder`：按自然顺序排序（等价于 `Comparable` 的排序）
   - `nullsFirst` / `nullsLast`：处理 `null` 元素（`null` 排在最前/最后）

#### 与 `Comparable` 的区别

| 维度         | `Comparator`（比较器）                | `Comparable`（可比较的）              |
|--------------|---------------------------------------|---------------------------------------|
| 定义位置     | 外部定义（不修改原类）                | 类内部定义（需修改原类实现接口）      |
| 核心方法     | `compare(T o1, T o2)`                 | `compareTo(T o)`                      |
| 灵活性       | 高：可动态切换不同排序规则            | 低：排序规则固定，与类绑定            |
| 适用场景     | 临时排序、多规则排序、无法修改的类    | 类的默认排序逻辑                      |

#### 总结

`Comparator` 是实现自定义排序的核心工具，尤其适合：

- 对没有实现 `Comparable` 的类进行排序；
- 需要多种排序规则（如正序/倒序、多字段排序）；
- 避免修改原始类（遵循“开闭原则”）。

结合 Lambda 表达式和 Java 8 的默认方法，`Comparator` 可以写出简洁且灵活的排序逻辑。

### 25.Comparable补充

`Comparable` 是 Java 中一个非常重要的**泛型接口**，用于定义**对象的自然排序规则**。实现 `Comparable` 接口的类可以被自动排序（如 `Arrays.sort()`、`Collections.sort()`、`TreeSet`、`TreeMap` 等）。

---

#### 一、`Comparable` 接口定义

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

- **`compareTo(T o)` 方法**：
  - 返回 **负整数**：当前对象 **小于** `o`
  - 返回 **0**：当前对象 **等于** `o`
  - 返回 **正整数**：当前对象 **大于** `o`

> ⚠️ 必须满足**自反性、对称性、传递性**，否则排序行为未定义！

---

#### 二、为什么需要 `Comparable`？

Java 的排序方法（如 `Arrays.sort()`）需要知道“如何比较两个对象”。  
- 对于 `int`、`String` 等内置类型，Java 已经实现了 `Comparable`；
- 对于**自定义类**（如 `Student`、`Person`），必须**手动实现 `Comparable`** 才能排序。

---

#### 三、使用示例

##### ✅ 示例1：对 `String` 排序（Java 已实现）

```java
String[] arr = {"banana", "apple", "cherry"};
Arrays.sort(arr); // 按字典序升序
// 结果: ["apple", "banana", "cherry"]
```
> 因为 `String implements Comparable<String>`

---

##### ✅ 示例2：自定义类实现 `Comparable`

```java
public class Student implements Comparable<Student> {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 按年龄升序排序
    @Override
    public int compareTo(Student other) {
        return Integer.compare(this.age, other.age);
        // 或：return this.age - other.age; （但可能溢出，不推荐）
    }

    @Override
    public String toString() {
        return name + "(" + age + ")";
    }
}
```

**使用排序：**
```java
List<Student> students = Arrays.asList(
    new Student("Alice", 20),
    new Student("Bob", 18),
    new Student("Charlie", 22)
);

Collections.sort(students);
System.out.println(students);
// 输出: [Bob(18), Alice(20), Charlie(22)]
```

---

#### 四、`compareTo` 实现技巧

##### 1. **基本类型比较**

```java
// 推荐使用工具方法（避免溢出）
return Integer.compare(a, b);
return Double.compare(x, y);
```

##### 2. **多字段排序（先按年龄，再按姓名）**

```java
@Override
public int compareTo(Student other) {
    int ageCompare = Integer.compare(this.age, other.age);
    if (ageCompare != 0) {
        return ageCompare;
    }
    return this.name.compareTo(other.name); // String 已实现 Comparable
}
```

##### 3. **降序排序**

```java
// 方法1：反转比较结果
return -Integer.compare(this.age, other.age);

// 方法2：交换参数
return Integer.compare(other.age, this.age);
```

---

#### 五、`Comparable` vs `Comparator`

| 特性         | `Comparable`               | `Comparator`                    |
| ------------ | -------------------------- | ------------------------------- |
| **定义位置** | 在类内部实现（“自然排序”） | 外部类或 Lambda（“定制排序”）   |
| **修改权限** | 需要修改类源码             | 无需修改类（适合第三方类）      |
| **数量限制** | 只能有一种自然排序         | 可定义多种排序规则              |
| **典型用法** | `Arrays.sort(list)`        | `Arrays.sort(list, comparator)` |

##### ✅ 何时用哪个？

- 如果对象有**明确的自然顺序**（如数字、日期、字典序），用 `Comparable`；
- 如果需要**多种排序方式**，或不能修改类，用 `Comparator`。

---

#### 六、注意事项 ⚠️

##### 1. **一致性要求**

- `compareTo()` 的结果应与 `equals()` 一致（虽然不是强制，但强烈建议）：
  ```java
  // 如果 a.compareTo(b) == 0，最好 a.equals(b) 也为 true
  ```
  否则在 `TreeSet` 等集合中可能表现异常。

##### 2. **避免整数溢出**

```java
// ❌ 危险！可能溢出
return this.age - other.age;

// ✅ 安全
return Integer.compare(this.age, other.age);
```

##### 3. **处理 null 值**

`Comparable` 默认不支持 `null`，排序时会抛 `NullPointerException`。  
如需支持，可使用 `Comparator.nullsFirst()` 或 `nullsLast()`。

---

#### 七、常见类已实现 `Comparable`

| 类                         | 排序规则          |
| -------------------------- | ----------------- |
| `String`                   | 字典序（Unicode） |
| `Integer`, `Double`        | 数值大小          |
| `Date`, `LocalDateTime`    | 时间先后          |
| `BigInteger`, `BigDecimal` | 数值大小          |

---

#### 八、总结

> ✅ **`Comparable` = 定义对象的“默认排序规则”**  
> ✅ 实现 `compareTo()` 方法，返回负/0/正整数  
> ✅ 适用于有自然顺序的类（如学生按学号、商品按价格）  
> ✅ 与 `Comparator` 互补：一个“内建”，一个“外挂”

```java
// 一句话记住：
// 实现 Comparable，你的类就能被 Collections.sort() 自动排序！
```

如果你有具体类需要实现排序，我可以帮你写 `compareTo` 方法 😊

### 26.Map和Map.Entry的区别

`Map.Entry` 和 `Map` 是 Java 集合框架中密切相关但角色完全不同的两个接口，二者的核心区别在于**职责和功能**：

#### 1. 本质与职责

- **`Map`**：是**键值对集合的顶层接口**，代表一个“键值对容器”，用于存储和管理一系列键值对（Key-Value），并提供了对整体集合的操作（如添加、删除、查询键值对）。  
  例如：`HashMap`、`TreeMap` 都是 `Map` 的实现类，用于存储多组键值对。

- **`Map.Entry`**：是 `Map` 内部定义的**静态嵌套接口**，代表 `Map` 中的**单个键值对元素**（即“一个键对应一个值”的单元），用于描述和操作这一组具体的键值对。  
  例如：`Map` 中存储的每个 `<K, V>` 对，都是 `Map.Entry<K, V>` 类型的实例。

#### 2. 核心功能与方法

##### `Map` 接口的核心方法（操作整体集合）：

- `put(K key, V value)`：向集合中添加键值对  
- `get(Object key)`：根据键获取对应的值  
- `remove(Object key)`：根据键删除键值对  
- `keySet()`：获取所有键的集合（`Set<K>`）  
- `values()`：获取所有值的集合（`Collection<V>`）  
- `entrySet()`：获取所有键值对的集合（`Set<Map.Entry<K, V>>`）  
- `size()`：获取键值对的数量  

##### `Map.Entry` 接口的核心方法（操作单个键值对）：

- `getKey()`：获取当前键值对中的“键”（Key）  
- `getValue()`：获取当前键值对中的“值”（Value）  
- `setValue(V value)`：修改当前键值对中的“值”（返回旧值）  
- `Map.Entry.comparingByKey()` 是 Java 8 为 `Map.Entry` 接口新增的静态方法，用于**创建一个根据键（Key）对 `Map.Entry` 元素进行排序的比较器（`Comparator`）**，简化 `Map` 中键值对按 Key 排序的逻辑。用于按 Key 的自然顺序对 `Map.Entry` 元素排序（默认升序）。

#### 3. 关系与依赖

- `Map.Entry` 是 `Map` 的“内部组件”：一个 `Map` 包含多个 `Map.Entry` 实例（每个对应一组键值对）。  
- 通过 `Map.entrySet()` 可以获取所有 `Map.Entry` 的集合，从而遍历或操作单个键值对：  
  ```java
  Map<String, Integer> map = new HashMap<>();
  map.put("a", 1);
  map.put("b", 2);
  
  // 遍历所有键值对（通过 Map.Entry）
  for (Map.Entry<String, Integer> entry : map.entrySet()) {
      System.out.println("键：" + entry.getKey() + "，值：" + entry.getValue());
  }
  ```

#### 4. 通俗类比

可以把 `Map` 比作一个“字典”，而 `Map.Entry` 比作字典中的“每一页词条”：
- `Map` 负责管理整个字典（添加词条、删除词条、查找词条所在页码）；  
- `Map.Entry` 负责描述单个词条的具体内容（词条的“键”是单词，“值”是释义）。

#### 总结

| 对比维度       | `Map`                          | `Map.Entry`                      |
|----------------|-------------------------------|----------------------------------|
| 角色           | 键值对的容器（整体集合）       | 单个键值对元素（集合中的元素）   |
| 定义位置       | 顶层接口（`java.util.Map`）    | `Map` 的内部静态接口             |
| 核心功能       | 管理键值对集合（增删改查整体） | 操作单个键值对（获取/修改键和值） |
| 数量关系       | 一个 `Map` 包含多个 `Entry`    | 每个 `Entry` 属于一个 `Map`      |

简单说：`Map` 是“容器”，`Map.Entry` 是容器中的“元素”，二者是“整体”与“个体”的关系。

### 27.Collection和Collections的区别

- **Collection**：**根接口**，List/Set/Queue 的父接口。
- **Collections**：**工具类**，提供 `sort()`、`binarySearch()`、`synchronizedList()` 等静态方法。

一句话：
**Collection 是接口；Collections 是工具类。**

#### 1. 本质与定义

- **`Collection`**  
  是 Java 集合框架中的**顶层接口**（`interface`），位于 `java.util` 包下，定义了所有单列集合（存储单个元素的集合）的通用操作规范。  
  它是 `List`、`Set`、`Queue` 等接口的父接口，规定了集合的基本行为（如添加、删除、遍历元素等）。

- **`Collections`**  
  是 Java 提供的**工具类**（`class`），同样位于 `java.util` 包下，包含了一系列静态方法，用于操作或返回集合（如排序、查找、同步化等）。  
  它是集合的“辅助工具”，本身不存储元素。

#### 2. 核心功能与用法

##### `Collection` 接口（定义集合规范）

作为接口，`Collection` 主要定义了所有单列集合的通用方法，具体实现由其子类/实现类完成：
```java
// 核心方法示例（由 List、Set 等实现）
public interface Collection<E> {
    boolean add(E e);          // 添加元素
    boolean remove(Object o);  // 删除元素
    int size();                // 获取元素数量
    boolean isEmpty();         // 判断是否为空
    Iterator<E> iterator();    // 获取迭代器
    // 其他方法...
}
```

**使用场景**：作为变量类型声明，统一操作不同的集合实现类（面向接口编程）：
```java
// 用 Collection 接口接收不同实现类（多态）
Collection<String> list = new ArrayList<>();
Collection<String> set = new HashSet<>();
```

##### `Collections` 工具类（提供集合操作方法）

`Collections` 是工具类，提供静态方法操作集合，例如：
- 排序：`sort(List<T> list)`
- 查找：`binarySearch(List<? extends Comparable<? super T>> list, T key)`
- 同步化：`synchronizedList(List<T> list)`（将集合转为线程安全版本）
- 填充：`fill(List<? super T> list, T obj)`
- 获取最大/小值：`max(Collection<? extends T> coll)` / `min(...)`

**使用场景**：直接调用静态方法处理集合：
```java
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 2));

// 排序（Collections 工具类的静态方法）
Collections.sort(list);
System.out.println(list); // [1, 2, 3]

// 获取最大值
Integer max = Collections.max(list); // 3
```

#### 3. 关键区别对比

| 维度         | `Collection`                | `Collections`                  |
|--------------|-----------------------------|--------------------------------|
| 类型         | 接口（`interface`）         | 工具类（`class`，被 `final` 修饰，不能继承） |
| 核心作用     | 定义单列集合的通用规范      | 提供操作集合的静态工具方法      |
| 方法类型     | 抽象方法（由实现类实现）    | 静态方法（直接通过类名调用）    |
| 关系         | 是 `List`、`Set` 等的父接口 | 依赖 `Collection` 及其实现类，提供操作它们的工具 |
| 示例用法     | `Collection<String> c = new ArrayList<>()` | `Collections.sort(list)`       |

#### 总结

- `Collection` 是**接口**，是所有单列集合的“总规范”，定义了集合的基本行为；
- `Collections` 是**工具类**，是集合的“辅助工具”，提供了排序、查找等实用操作。

简单记忆：`Collection` 带单数词尾“-ion”，表示“一个集合接口”；`Collections` 带复数词尾“-ions”，表示“操作集合的工具类”。

### 28.CopyWrite(写时复制)

在Java中，`CopyOnWrite`（写时复制）是一种种种并发编程中的优化策略，核心思想是：**当对容器进行修改操作（如添加、删除元素）时，不直接修改原容器，而是创建一个新的副本，修改操作在副本上进行，完成后再将引用指向新副本**。

这种机制**主要用于解决并发场景下的线程安全问题**，常见的实现类有：
- `CopyOnWriteArrayList`（对应`ArrayList`的线程安全版本）
- `CopyOnWriteArraySet`（对应`Set`的线程安全版本）

#### 工作原理

1. **读操作**：直接读取原容器，无需加锁，性能高效
2. **写操作**：
   - 创建原容器的副本
   - 在副本上执行修改操作
   - 将容器引用指向新副本
3. **线程安全保障**：通过不可变的原容器保证读操作的安全性，写操作通过锁保证互斥

#### 代码示例（CopyOnWriteArrayList）

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteDemo {
    public static void main(String[] args) {
        // 创建CopyOnWriteArrayList实例
        List<String> list = new CopyOnWriteArrayList<>();
        
        // 添加元素（写操作，会触发复制）
        list.add("a");
        list.add("b");
        
        // 读取元素（读操作，直接访问原数组）
        System.out.println(list.get(0)); // 输出 "a"
        
        // 遍历元素（迭代器基于快照，不会抛出ConcurrentModificationException）
        for (String s : list) {
            System.out.println(s);
            // 遍历中修改元素不会影响当前迭代
            list.add("c");
        }
        
        System.out.println(list.size()); // 输出 3
    }
}
```

#### 优缺点分析

##### 优点：

1. **读操作高效**：无需加锁，适合读多写少的场景
2. **线程安全**：避免了`ConcurrentModificationException`（并发修改异常）
3. **迭代安全**：迭代器基于原容器的快照，不会受后续修改影响

##### 缺点：

1. **内存占用**：写操作时会复制整个容器，内存消耗翻倍
2. **数据一致性**：读操作可能读取到旧数据（最终一致性，非实时一致性）
3. **写操作性能差**：复制容器的开销大，不适合频繁修改的场景

#### 适用场景

- 读操作远多于写操作的场景（如缓存、配置列表）
- 对数据实时一致性要求不高的场景
- 需要避免迭代时出现并发修改异常的场景

#### 不适合场景：

- 写操作频繁的场景
- 内存敏感的场景
- 需要强一致性的场景





### 29.`Set<String> set = new HashSet<>(); `语法

`Set<String> set = new HashSet<>();` 是 Java 中使用**泛型（Generic）** 和**接口多态**的典型语法，用于创建一个存储 `String` 类型元素的 `HashSet` 集合对象。以下是详细解析：

#### 语法拆解

1. **`Set<String>`**  
   - `Set`：是 Java 集合框架中的**接口**（`java.util.Set`），定义了无序、不可重复元素的集合规范（如 `add`、`contains`、`size` 等方法）。  
   - `<String>`：是**泛型参数**，指定该集合只能存储 `String` 类型的元素（编译期类型检查，避免存入其他类型元素导致的运行时错误）。  


2. **`set`**  
   - 是自定义的**引用变量名**，用于指向创建的集合对象，遵循 Java 变量命名规范（小写字母开头，驼峰式）。  


3. **`new HashSet<>()`**  
   - `new`：关键字，用于创建对象，在堆内存中分配空间。  
   - `HashSet`：是 `Set` 接口的**实现类**（`java.util.HashSet`），基于哈希表实现，特点是无序、查询效率高。  
   - `<>()`：是**菱形语法（Diamond Syntax）**（JDK 7+ 引入），简化泛型声明。由于左侧已指定 `Set<String>`，右侧的泛型参数可省略（编译器会自动推断为 `<String>`），等价于 `new HashSet<String>()`。  

#### 核心思想

1. **接口编程（多态）**  
   左侧使用接口 `Set` 作为变量类型，右侧使用实现类 `HashSet` 创建对象，符合“面向接口编程”思想。这样做的好处是：  
   - 降低代码耦合度：若后续需要更换集合实现（如改为 `TreeSet`，有序集合），只需修改右侧 `new` 后的实现类（`new TreeSet<>()`），左侧变量声明无需改动。  
   - 统一操作规范：通过 `Set` 接口提供的方法操作集合，无需关心具体实现细节。  


2. **泛型安全**  
   泛型 `<String>` 限制了集合只能存储 `String` 类型元素：  
   - 编译期检查：若尝试添加非 `String` 类型元素（如 `set.add(123)`），编译器会直接报错，避免运行时的 `ClassCastException`。  
   - 避免强制转型：从集合中获取元素时，无需手动转型（如 `String s = set.iterator().next()` 直接使用，无需 `(String)` 强制转换）。  

#### 等价写法与演变

- JDK 7 之前（无菱形语法）：需显式指定两侧泛型  
  `Set<String> set = new HashSet<String>();`  

- 不使用泛型（不推荐，属于原始类型）：  
  `Set set = new HashSet();`  
  缺点：可存入任意类型元素，取出时需手动转型，存在类型安全风险。  

#### 后续使用示例

创建集合后，可通过 `Set` 接口方法操作元素：  
```java
Set<String> set = new HashSet<>();
set.add("apple");    // 添加元素
set.add("banana");
set.contains("apple"); // 判断是否包含元素（返回 true）
for (String s : set) { // 遍历元素（无序）
    System.out.println(s); 
}
```


总结：该语法通过“接口声明 + 实现类实例化 + 泛型限制”，兼顾了多态灵活性、类型安全性和代码简洁性，是 Java 集合框架的标准使用方式。

### 30.Vector/Hashtable/Stack

B选项Vector:它是线程安全的动态数组实现,其所有方法都被synchronized修饰,在多线程环境下可以安全使用。

C选项Hashtable:也是线程安全的,其所有方法都使用synchronized进行同步,可以在多线程环境下安全地进行键值对操作。

D选项Stack:继承自Vector类,因此具有Vector的线程安全特性,其push和pop等操作都是同步的。

A选项ArrayList错误原因:
ArrayList是非线程安全的集合类,它的方法没有进行同步处理。在多线程环境下同时操作ArrayList可能会导致数据不一致。如果需要在多线程环境下使用ArrayList,可以通过Collections.synchronizedList()方法将其包装成线程安全的列表。

注意:虽然BCD这些线程安全的集合类可以在多线程环境下使用,但由于其实现采用synchronized关键字进行同步,在高并发场景下可能会影响性能。在实际开发中,可以考虑使用java.util.concurrent包下的并发集合类(如ConcurrentHashMap等)来获得更好的性能。、

### 31.ArrayList、LinkedList和Vector的区别？

- ArrayList基于数组，查询较快，末尾插入O(1)，中间i处插入O(n-i)，需要移动元素，可通过序号快速获取对象，线程不安全，有预留的内存空间
- LinkedList基于双向链表，末尾插入O(1)，中间i处插入O(n-i)，但不需要移动元素，不可通过序号快速获取对象，线程不安全，没有预留的内存空间，但每个节点都有两个指针占用了内存
- Vector和ArrayList实现上基本相同，区别在于Vector是线程安全的，其各种增删改查方法加了synchronized修饰

### 32.HashMap、HashSet和HashTable的区别

- HashMap线程不安全，效率高一点，可以存储null的key和value，null的key只能有一个，null的value可以有多个。默认初始容量为16，每次扩充变为原来2倍。创建时如果给定了初始容量，则扩充为2的幂次方大小。底层数据结构为数组+链表，插入元素后如果链表长度大于阈值（默认为8），先判断数组长度是否小于64，如果小于，则扩充数组，反之将链表转化为红黑树，以减少搜索时间。
- HashTable线程安全，效率低一点，其内部方法基本都经过synchronized修饰，不可以有null的key和value。默认初始容量为11，每次扩容变为原来的2n+1。创建时给定了初始容量，会直接用给定的大小。底层数据结构为数组+链表。它基本被淘汰了，要保证线程安全可以用ConcurrentHashMap。
- HashSet是基于HashMap实现的，只是value都指向了一个虚拟对象，只用到了key

### 33.HashSet、LinkedHashSet和TreeSet的区别

- LinkedHashSet是HashSet子类，在HashSet的基础上能过够按照添加的顺序遍历
- TreeSet底层使用红黑树，插入时按照默认/自定义的key排序规则指定插入位置



### 35. List、Set、Map 区别？

- **List**：有序、可重复、有索引、可通过下标访问。
- **Set**：无序、不可重复、无索引。
- **Map**：存储**key-value**，key 不可重复，value 可重复。

### 36. HashMap 和 Hashtable 区别？

1. **线程安全**：HashMap 不安全；Hashtable 安全（方法加 synchronized）
2. **效率**：HashMap 快；Hashtable 慢
3. **null**：HashMap **key/value 都可以 null**；Hashtable 都不能 null
4. **父类**：HashMap 继承 AbstractMap；Hashtable 继承 Dictionary
5. **扩容**：HashMap 扩容 2倍；Hashtable 扩容 2倍+1
6. **哈希算法**：HashMap 优化更好

### 37. 使用 HashMap 还是 TreeMap？

- **HashMap**：查询快、无序、用于**普通快速存取**。
- **TreeMap**：**有序**（按 key 自然排序或自定义排序）。

需要**有序**用 TreeMap，否则一律 HashMap。

### 38. HashMap 实现原理（JDK1.8）

- 底层结构：**数组 + 链表 + 红黑树**
1. 根据 key 计算 **hashCode** 得到数组下标
2. 无元素直接放入
3. 有元素则 **equals 比较 key**
    - 相同：覆盖 value
    - 不同：挂链表
4. 链表长度 **>8 且数组≥64 → 转为红黑树**
5. 树节点 ≤6 → 退化为链表
6. 负载因子 **0.75**，达到容量×0.75 自动扩容为原来 2 倍

### 39. HashSet 实现原理

**HashSet 底层就是 HashMap！**
- 把元素当作 **map 的 key**，value 存一个固定的 Object 常量。
- 利用 **HashMap key 不可重复** 保证元素唯一。
- 判断重复：**hashCode() + equals()**

### 40. ArrayList 和 LinkedList 区别？

- **ArrayList**：**动态数组**
    - 查询快（随机访问）
    - 增删慢（要移动元素）
- **LinkedList**：**双向链表**
    - 查询慢
    - 增删快
    - 可作队列、栈

### 41. 数组 ↔ List 转换

- 数组 → List：
  ```java
  List<String> list = Arrays.asList(array);
  ```
- List → 数组：
  ```java
  String[] array = list.toArray(new String[0]);
  ```

注意：`Arrays.asList()` 返回固定大小列表，**不能增删**。

### 42. ArrayList 和 Vector 区别？

- **线程安全**：ArrayList 不安全；Vector 安全（synchronized）
- **效率**：ArrayList 快；Vector 慢
- **扩容**：ArrayList 1.5倍；Vector 2倍
- 现在不用 Vector，用 **Collections.synchronizedList** 或 **CopyOnWriteArrayList**

### 43. Array 和 ArrayList 区别？

- **Array**：固定长度、可存基本类型、性能高
- **ArrayList**：动态扩容、只能存对象、有丰富方法

### 44. Queue 中 poll() 和 remove() 区别？

- **remove()**：删除并返回队首，**队列为空抛异常**
- **poll()**：删除并返回队首，**队列为空返回 null，不抛异常**

### 45. 哪些集合线程安全？

- Vector、Hashtable
- **ConcurrentHashMap**
- **CopyOnWriteArrayList**
- **CopyOnWriteSet**
- Collections 工具类包装的同步集合

### 46. Iterator 是什么？

- **迭代器**，用于**遍历 Collection** 的通用接口。
- 提供统一遍历方式，不暴露底层结构。

### 47. Iterator 使用 & 特点

使用：
```java
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String s = it.next();
}
```
特点：
- 统一遍历
- 遍历中**可安全删除**（`it.remove()`）
- 只能**单向向前**

### 48. Iterator 和 ListIterator 区别？

- **Iterator**：可遍历所有 Collection，单向，只能删除
- **ListIterator**：**只能遍历 List**，**可向前可向后**，可添加、修改、获取索引

### 49. 如何确保集合不能被修改？

1. **`Collections.unmodifiableList(list)`**
    包装后**增删改抛 UnsupportedOperationException**
2. 使用 **List.of() / Set.of()**（JDK9+）
    创建不可变集合

我给你**最清晰、面试标准答案、一句话不啰嗦**版本，**背这个直接满分**。

### 50.ArrayList 和 数组（Array）的区别

#### 1. 长度

- **数组（Array）：固定长度**，一旦创建不能改变。
- **ArrayList：动态扩容**，长度自动增长，不用关心大小。

#### 2. 存储内容

- **数组：可以存基本类型 + 引用类型**  
  `int[]、double[]、User[]`
- **ArrayList：只能存** **引用类型**  
  基本类型必须用包装类：`Integer、Double`

#### 3. 功能方法

- **数组：功能极少**，只有 `length` 属性，没有增删改查方法。
- **ArrayList：集合类，方法非常多**  
  `add、remove、get、set、contains、clear` 等。

#### 4. 性能

- **数组性能更高**，开销小。
- **ArrayList 稍低**（有扩容、拷贝消耗）。

#### 5. 类型检查

- 数组**运行时检查类型**
- ArrayList **编译时泛型检查**（更安全）

---

#### 🔥 面试 **最精简必背版（背这一段就够）**

1. **数组固定长度；ArrayList 动态扩容**
2. **数组支持基本类型；ArrayList 只支持引用类型**
3. **数组无方法；ArrayList 方法丰富**
4. **数组性能更高；ArrayList 使用更方便**

---

#### 一句话终极总结

**数组是固定大小、高性能、功能简单；  
ArrayList 是动态长度、易用、功能强大，但只能存对象。**

需要我顺便帮你区分 **Array vs Arrays vs ArrayList**（三个最容易混的）吗？



### 51.ArrayList 扩容机制

我讲得**极度精简、逻辑完整、面试官最爱听**，一步都不漏。

#### 一、底层结构

ArrayList 底层是 **Object 数组**：
```java
transient Object[] elementData;
```

#### 二、默认初始化

1. **new ArrayList()**
   - 初始创建：**elementData = {} 空数组**
   - **容量 0**

2. **new ArrayList(initialCapacity)**
   - 直接创建指定大小的数组

#### 三、什么时候开始扩容？

**第一次 add() 时：**
- 容量变为 **默认容量 10**

#### 四、扩容规则（核心）

当 **元素个数 size == 数组长度 length** 时，触发扩容：
1. 新容量 = **旧容量 × 1.5**（`oldCapacity + oldCapacity >> 1`）
2. 使用 `Arrays.copyOf` **复制原数组到新数组**
3. elementData 指向新数组

示例：
10 → 15 → 22 → 33 → 49…



#### 五、完整流程（面试口述满分）

1. ArrayList 底层是 **Object[]**
2. 无参构造**初始为空数组，容量0**
3. **第一次 add 扩容为 10**
4. 满了就 **1.5 倍扩容**
5. 底层通过 **Arrays.copyOf 数组拷贝** 实现

---

#### 🔥 超精简背诵版（背这一段就满分）

1. **底层 Object 数组**
2. **默认初始容量 0，第一次 add 变为 10**
3. **满了扩容为原来的 1.5 倍**
4. **通过 Arrays.copyOf 复制数组**

---

#### 高频面试坑点（必看）

- **JDK1.7 初始是 10，JDK1.8 懒加载初始 0 → 第一次add才10**
- 扩容是**效率瓶颈**，大批量插入建议**指定初始容量**

如果你愿意，我可以把 **ArrayList vs LinkedList vs Vector 三者对比+扩容机制** 合成一张超级表，一眼记住。

### 52.HashMap 实现原理 + put流程 + 扩容机制

我给你讲**JDK1.8 HashMap 最完整、最标准、面试满分答案**，包含：
**实现原理 + put流程 + 扩容机制**，面试官所有连环问全覆盖，**纯干货、不废话、直接背**。

#### 一、HashMap 实现原理（JDK1.8）

1. **底层结构：数组 + 链表 + 红黑树**
2. **数组**是主体，叫**哈希桶（table）**
3. **链表**用于解决**哈希冲突**
4. **链表长度 > 8 且数组长度 ≥64 → 转为红黑树**
5. **红黑树节点 ≤6 → 退化为链表**
6. 默认**负载因子 0.75**
7. **阈值 threshold = 容量 × 负载因子**，达到即扩容

#### 二、HashMap put() 方法流程（**面试必考，背这9步**）

1. 若**table 数组为空或长度为0**，先**resize() 扩容（初始化）**。
2. 根据 key 的 **hashCode() 高版本扰动计算 hash**。
3. 通过 `(n-1) & hash` 计算数组**下标 index**。
4. 若该位置**为空**，直接**新建节点放入**。
5. 若该位置**已有节点**：
   - 先比较 **hash 和 key（== 或 equals）**
   - 若 key 相同，**覆盖旧 value**
6. 若该位置是**红黑树节点**，则**树插入**。
7. 若是**链表节点**，遍历到尾部插入：
   - 插入后**链表长度 > 8**，转为红黑树。
8. 插入成功后，**size++**。
9. 若 **size > 阈值**，执行 **resize() 扩容**。

#### 三、HashMap 扩容机制（resize）

1. 扩容时机：**元素个数 size > 阈值 threshold**
2. 新容量 = **旧容量 × 2**（必须是2的幂）
3. 新阈值 = **新容量 × 0.75**
4. 创建**新数组**，**重新计算所有元素下标**
5. 链表拆分规则（**JDK1.8优化，非常重要**）：
   - 根据 `hash & oldCap` 的值分为两条链表
   - 一条在**原下标**
   - 一条在**原下标 + 旧容量**
   - **保持原有顺序，不会倒置**

#### 四、超级精简总结（面试直接口述满分）

1. **HashMap 底层：数组+链表+红黑树**
2. **put：算hash → 找下标 → 冲突挂链表 → >8转红黑树 → 满了扩容**
3. **默认容量16，负载因子0.75**
4. **扩容为原来2倍，重新hash，链表拆成两条，位置不变或+旧容量**
5. **JDK1.8 链表尾插，不反转；1.7 头插，会反转、死链**

---

#### 五、面试官必追问的 5 个问题（我直接给答案）

1. **为什么容量必须是 2 的幂？**
   为了让 `(n-1) & hash` 代替取模，**计算更快、分布均匀**。

2. **为什么负载因子是 0.75？**
   空间与时间的平衡，**冲突少、利用率高**。

3. **JDK1.7 和 1.8 区别？**
   1.7：数组+链表、头插、扩容反转、会死链
   1.8：数组+链表+红黑树、尾插、不反转、安全

4. **为什么要重写 equals 必须重写 hashCode？**
   保证**相同内容的对象 hash 相同**，否则 HashMap 无法判断重复，会重复存储。

5. **HashMap 线程安全问题？**
   多线程 put 可能**数据覆盖、死循环、数据丢失**，多线程用 **ConcurrentHashMap**。

---

如果你愿意，我可以把 **HashMap 整套知识点画成一张逻辑图（put+扩容+红黑树+hash计算）**，**一张图记住所有原理**。要吗？

### 53.HashMap为什么是线程不安全

1. HashMap为什么是线程不安全的？
   - HashMap的**操作不是原子的**，多个线程操作一个HashMap对象时，可能会导致数据不一致,并发修改异常。
2. 如何实现线程安全？
   - 使用**Collections.synchronizedMap()** 方法，返回一个线程安全的Map对象。
   - 使用**ConcurrentHashMap**类，它是线程安全的。
   - 使用**锁机制**，在HashMap的操作中加显式锁

### 54.hashmap 调用get方法一定安全吗？

#### **HashMap 的 get() 方法 —— 多线程下并不绝对安全！**

很多人错误以为：**只有 put 不安全，get 是读操作，一定安全**
→ **完全错误！！！**

##### 结论先说：

**多线程环境下，即使只调用 get()，也可能出现线程安全问题！**

---

#### 一、get() 为什么不安全？（底层真实原因）

HashMap 的 **get 方法内部没有加锁**，并且会访问：
1. **table 数组（引用）**
2. **链表/红黑树节点**
3. **扩容时的结构变化**

#### 多线程场景下，`get()` 会出现 3 种安全问题：

##### 1. **扩容期间 get() —— 可能出现死循环（JDK1.7）**

JDK1.7 扩容是**头插法**，多线程会让链表成环。
此时调用 get() → **进入死循环，CPU 100%**

##### 2. **扩容期间 get() —— 数据丢失、获取 null**

扩容时：
- 旧 table → 新 table
- 节点正在迁移
**get() 可能访问到旧数组，也可能访问新数组，结果不稳定**

##### 3. **get() 访问到未完全迁移完的节点 —— 返回错误值**

扩容是**分步迁移**，不是瞬间完成。
线程 A 扩容，线程 B get →
**可能读到 null、读到旧数据、读到中间状态**

##### 4. **内存可见性问题（最根本）**

`table`、`size`、`Node` 内部的 `next` 都**没有 volatile**
→ 线程修改后，**其他线程不能保证立即看到最新值**
→ **get() 可能读到旧数组、旧节点、旧next**

> **get() 虽然不修改数据，但它读取共享数据，而共享数据没有保证可见性与有序性。**

---

#### 二、最简单一句话（面试必背）

**HashMap 的 get 方法不是线程安全的！
因为没有锁、没有 volatile，扩容时结构会发生变化，
可能导致：读到 null、数据错误、JDK1.7 下出现死循环。**

---

#### 三、面试官最想听的标准答案（直接背）

1. HashMap **get() 不加锁**，且底层数组、节点都**没有 volatile 修饰**，不保证内存可见性。
2. 多线程**并发扩容**时，链表结构可能改变，**get() 可能获取错误数据或 null**。
3. JDK1.7 下扩容采用**头插法**，会造成**环形链表**，调用 get() 会**死循环**。
4. 所以：**即使只读 get()，HashMap 也不是线程安全的。**

---

#### 四、延伸（必问）

##### 那什么 Map 是安全的？

1. **Hashtable**（方法全 synchronized，性能差）
2. **Collections.synchronizedMap**（包装锁）
3. **ConcurrentHashMap**（分段锁/ CAS + synchronized，**工业界标准**）

### ConcurrentHashMap 为什么安全？

#### JDK 1.7 **分段锁** vs JDK 1.8 **CAS + synchronized** 完整原理

#### 一、ConcurrentHashMap 为什么安全？

因为它**不是锁整个 Map**，而是**细粒度锁**：
- **1.7：分段锁（Segment），锁一段**
- **1.8：CAS + synchronized，只锁一个哈希桶（头节点）**

**并发能力远远高于 Hashtable（锁整个map）**

---

#### 二、JDK 1.7 底层：**Segment 分段锁**

##### 结构

**ConcurrentHashMap = 数组（Segment） + 数组（HashEntry） + 链表**

- 外层是 **Segment 数组（默认16个）**
- 每个 Segment 继承 **ReentrantLock**
- 每个 Segment 内部是一个小的 HashMap

##### 加锁规则

- **put 时锁住整个 Segment**
- 不同 Segment 之间**完全并发，互不阻塞**
- 并发度 = Segment 数量（默认16）

##### 优点

比 Hashtable 强 **16 倍并发**

##### 缺点

1. **锁粒度还是偏大（锁一段，不是一个节点）**
2. **查询 size() 要统计所有 Segment，效率低**
3. 结构复杂，链表长了效率下降

---

#### 三、JDK 1.8 彻底重构：**CAS + synchronized + 数组+链表+红黑树**

##### 结构

**和 HashMap 结构一样：数组 + 链表 + 红黑树**
**取消 Segment，直接使用 Node 数组**

##### 安全机制（最核心）

###### 1. **put 流程（安全的根本）**

1. 计算 hash，定位桶位置
2. 如果**桶为空 → 使用 CAS 无锁插入**
3. 如果**桶不为空 → 只锁住当前桶的头节点（synchronized）**
4. 链表或红黑树正常插入
5. 超过8转红黑树

###### 关键点：

**synchronized 只锁** **当前哈希桶头节点**
**不影响其他桶** → 并发极高！

##### 2. get 方法**完全无锁**

- get 不加锁
- 因为 `table`、`Node` 中的 `val`、`next` 都加了 **volatile**
→ **保证内存可见性，读到最新数据**

这就是 CHM 读性能极高的原因。

---

#### 四、1.7 vs 1.8 最清晰对比（面试必背）

##### 1.7

- 结构：**Segment + HashEntry + 链表**
- 锁：**ReentrantLock 分段锁**
- 锁粒度：**Segment（一段）**
- 并发度：16
- 缺点：粒度大、size() 慢、结构复杂

##### 1.8

- 结构：**Node + 链表 + 红黑树**
- 锁：**CAS + synchronized**
- 锁粒度：**每个哈希桶头节点（极小）**
- 并发度：极高（理论等于数组长度）
- 优点：**并发更高、查询更快、内存更少**

---

#### 五、一句话终极总结（面试直接说，满分）

##### ConcurrentHashMap 为什么安全？

- **1.7 使用 Segment 分段锁，锁一段数据，减少锁冲突。**
- **1.8 取消分段，采用 CAS 无锁插入 + synchronized 锁桶头节点，锁粒度极小，同时 Node 使用 volatile 保证读可见性，因此高并发安全且高效。**

---

#### 六、面试官最爱追问的 3 个问题（我直接给答案）

##### 1. 为什么1.8用 synchronized 不用 ReentrantLock？

- synchronized 已经 JVM 级别大幅优化（锁升级）
- 代码更简单、内存更少、G1 下性能更好

##### 2. 为什么读不加锁？

- `val` 和 `next` 都是 **volatile**
- 保证**可见性、禁止重排序**
- 读无锁，性能极高

##### 3. Hashtable、HashMap、ConcurrentHashMap 区别？

- Hashtable：锁整个map，线程安全，慢
- HashMap：无锁，不安全，快
- CHM：细粒度锁，安全，极高并发

### **已经用了 synchronized，为什么还要用 CAS？**

**核心一句话：
synchronized 是“加锁”，一定会有线程阻塞；
CAS 是“无锁乐观尝试”，不加锁、不阻塞、更快。**

JDK1.8 ConcurrentHashMap 之所以**超快**，就是因为：

#### **大部分场景根本不需要加锁，直接 CAS 就能搞定！**

---

#### 一、先明确：CHM 1.8 的 put 流程

```
1. 计算 hash 定位桶
2. 如果桶是空的 → **CAS 插入节点（无锁！）**
3. 如果桶不为空 → **synchronized 锁住头节点，再插入**
```

##### 重点：

**只有桶不为空，才会加锁；
桶为空，直接 CAS，无锁、无阻塞！**

---

#### 二、为什么有了 synchronized 还要 CAS？（3个最核心理由）

##### 1. **CAS 不加锁，无阻塞、无上下文切换，极快**

- synchronized 即使是轻量级锁，也需要：
  锁记录、CAS 尝试、自旋、膨胀…
- **CAS 是一条 CPU 原子指令（cmpxchg），级别比锁更低！**

> **空桶插入是最常见的场景（HashMap 中大量桶都是空的）
> 这种场景用 CAS，完全无锁，性能碾压 synchronized**

---

##### 2. **synchronized 再优化，终究是锁，有开销**

- 加锁 = 线程需要排队
- 竞争 = 阻塞 = 上下文切换 = 消耗CPU

**而 CAS 是乐观策略：
没人改，我就直接改；
有人改，我重试一次即可。
全程不挂起线程！**

---

##### 3. **降低锁竞争概率（最重要的设计思想）**

ConcurrentHashMap 1.8 的设计目标：

###### **能无锁就无锁，实在不行再加锁最小粒度。**

- 空桶：**CAS 无锁**
- 冲突：**synchronized 只锁一个头节点**

这就是它**高并发、高吞吐**的根本原因。

---

##### 三、最直白的比喻（秒懂）

- **synchronized = 红绿灯，所有车必须停下看信号**
- **CAS = 路口没人，直接开过去；有人再等**

**大部分时间路上没车，当然直接开过去最快！
没必要每次都停下来等红灯。**

---

##### 四、面试标准答案（背这段，满分）

**ConcurrentHashMap 中 CAS 和 synchronized 配合使用：
1. 桶为空时，使用 CAS 无锁插入，避免加锁开销，提高并发；
2. 桶不为空时，才使用 synchronized 锁住头节点，保证安全；
3. CAS 是 CPU 原子指令，不加锁、不阻塞，比 synchronized 更轻量；
4. 设计思想：能无锁则无锁，只在冲突时加锁，从而达到极高并发性能。**

---

#### 五、终极一句话（最精髓）

##### **CAS 用于无锁快速写入，synchronized 用于冲突时保证安全。

CAS 提升速度，synchronized 保证安全，二者缺一不可。**


### CAS 的 ABA 问题 + 为什么 AtomicStampedReference 能解决（全程大白话+底层原理）

#### 一、什么是 CAS？

**Compare And Swap（比较并交换）**
- 比较：内存值 **== 预期值**
- 相等：就**替换成新值**
- 原子性：**CPU 原语指令（cmpxchg），不可分割**

#### 二、CAS 最大缺陷：**ABA 问题**

##### 一句话解释：

**一个值原来是 A，变成了 B，又变回了 A。
CAS 检查时发现值还是 A，就以为“没人改过”，但其实**已经被修改过！**

##### 经典过程：

1. 线程1 读取值 = **A**（准备 CAS）
2. 线程2 把 A → **B**
3. 线程2 把 B → **A**
4. 线程1 执行 CAS：发现还是 A，**以为没动过，替换成功**
5. **问题：线程1 不知道中间被改过！**

##### 危害：

大部分场景没事，但**有风险场景会出 BUG**：
- 栈（LinkedList、ConcurrentLinkedList）
- 环形链表
- 对象引用变化
- 分布式锁、序号生成

---

#### 三、怎么解决 ABA？

**加版本号（时间戳/标记）！**

##### 核心思想：

**不只比较值，还要比较版本号。
只要值被改过，版本号一定变！**

即使：
A → B → A
**值一样，但版本号已经变了，CAS 就会失败！**

---

#### 四、Java 解决方案：**AtomicStampedReference**

它内部保存两个东西：
1. **引用 reference（当前值）**
2. **stamp（版本号/戳记）**

##### CAS 时比较：

- **引用 == 预期引用**
- **版本号 == 预期版本号**
**两个都相等，才允许更新！**

##### 方法：

```java
// 比较并交换
boolean compareAndSet(
    V expectedReference,   // 预期值
    V newReference,        // 新值
    int expectedStamp,     // 预期版本
    int newStamp           // 新版本
)
```

**每次修改，版本号必须变！**
这样 ABA 就**彻底不可能**。

---

#### 五、最直白比喻（秒懂）

- **普通 CAS：只看钱数量，不看流水。**
  100 → 200 → 100，你以为没动过。
- **AtomicStampedReference：看数量+版本号。**
  只要动过，**版本必变**，骗不过去。

---

#### 六、面试**满分标准答案（背这段就无敌）**

1. **ABA 问题**：CAS 只比较值，值从 A→B→A 后，CAS 误认为未修改，导致逻辑错误。
2. **根本原因**：CAS **只比较值，不比较修改次数**。
3. **解决方案**：**加入版本号（时间戳）**，每次修改版本号自增。
4. **Java 实现**：**AtomicStampedReference**，同时维护**对象引用 + 版本号**，CAS 时两者都比较，彻底解决 ABA。

---

#### 七、终极一句话（最精髓）

**ABA = 值没变但过程变了；
AtomicStampedReference = 用版本号记录所有变化，让 CAS 不再被骗。**

---

如果你愿意，我可以把：
**CAS → ABA → AtomicStampedReference → AtomicMarkableReference**
整理成**一张超级清晰图**，面试前看一遍就稳。要吗？

### 56.为什么ArrayList不是线程安全的，具体来说是哪里不安全？

我给你讲**最本质、最具体、面试能直接说出口**的答案：
**ArrayList 为什么不安全？到底哪里不安全？（不是空话，是真实代码级原因）**

#### 一句话结论

**ArrayList 所有方法都没有加锁（synchronized/Lock），多线程同时修改时，会出现：数据覆盖、size 不正确、数组越界、数据丢失。**

---

#### 一、具体不安全的3个位置（源码级，必考）

我们看 `add(E e)` 源码（JDK1.8）：
```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 1. 判断是否扩容
    elementData[size++] = e;           // 2. 赋值 + 下标自增
    return true;
}
```

**这里有两步，都不是原子操作！**

##### 1. 最经典：**size++ 导致数据覆盖、数据丢失**

`size++` 看上去一行代码，**实际分3步**：
1. 读取 size
2. size + 1
3. 写回 size

多线程场景：
- 线程A读到 size=5
- 线程B也读到 size=5
- 两个线程都往 **下标5** 放数据
→ **后写的覆盖先写的，丢一个元素！**

##### 2. **数组越界异常 ArrayIndexOutOfBoundsException**

假设当前容量 **size=9，数组长度=10（满了）**

- 线程A判断：`size+1=10` → **不需要扩容**
- 线程A挂起
- 线程B也判断：`size+1=10` → **不需要扩容**
- 线程B正常添加 → size=10
- 线程A恢复，直接执行：
  ```
  elementData[10] = e
  ```
数组最大下标是9，访问10 → **直接数组越界！**

##### 3. **扩容时数据丢失、结构污染**

多线程同时触发扩容：
- 各自创建新数组
- 各自拷贝
- 最终只有一个线程的新数组被赋值
→ **另一个线程添加的数据直接丢失**

---

#### 二、总结：ArrayList 不安全的**3个具体表现**

1. **数据覆盖/丢失**（size++ 并发问题）
2. **ArrayIndexOutOfBoundsException 数组越界**
3. **扩容导致数据丢失、结构被破坏**

---

#### 三、为什么 Vector 是安全的？

因为 Vector 所有方法都加了 **synchronized**：
```java
public synchronized boolean add(E e) {
    ...
}
```

---

#### 四、面试**最标准满分回答（背这段）**

ArrayList 线程不安全的根本原因：
1. **add 方法没有任何锁机制，不是原子操作**；
2. `size++` 分为读、改、写三步，多线程会造成**数据覆盖、丢失**；
3. 并发判断扩容时，会出现**数组越界异常**；
4. 多线程同时扩容会导致**数据丢失、结构破坏**。

---

#### 最简记忆版（超级好背）

**无锁 → size++ 并发覆盖 → 并发扩容导致越界 → 数据丢失、异常。**

### 杜绝 List 空指针。

---

#### 一、问题根源

```java
List<String> list = null;
// 直接遍历 → NPE 空指针异常
for (String s : list) {

}
list.stream().forEach(); // 同样 NPE
```

---

#### 二、方案1：普通 if 判断（最常用、易懂）

```java
if (list != null && !list.isEmpty()) {
    // 安全遍历
    for (String item : list) {
        
    }
}
```

---

#### 三、方案2：Optional 优雅写法（推荐，高级写法）

##### 1. 遍历

```java
Optional.ofNullable(list)
        .orElseGet(List::of) // null 给空集合
        .forEach(item -> {
            // 业务逻辑
        });
```

##### 2. 流式操作安全处理

```java
Optional.ofNullable(list)
        .orElse(List.of())
        .stream()
        .filter(Objects::nonNull)
        .forEach(xxx);
```

---

#### 四、方案3：工具类（项目通用）

##### Spring / 常用工具

```java
// org.springframework.util.CollectionUtils
if (!CollectionUtils.isEmpty(list)) {
    // 安全操作
}
```
`CollectionUtils.isEmpty()` 底层：
```java
public static boolean isEmpty(Collection<?> collection) {
    return collection == null || collection.isEmpty();
}
```

---

#### 五、终极最佳实践（日常开发直接用）

1. **接收参数/查询返回**，先工具类判空
```java
if (!CollectionUtils.isEmpty(list)) {
    // 业务
}
```

2. **不想写 if，优雅容错**
```java
List<Data> safeList = Optional.ofNullable(list).orElse(Collections.emptyList());
// 后续随便遍历、stream，不会NPE
safeList.forEach(xxx);
```

---

#### 六、面试一句话背诵

> List 集合操作前要防止空指针，
> 可以使用 **`CollectionUtils.isEmpty()`** 统一判空；
> 也可以通过 **Optional.ofNullable() 配合空集合兜底**，
> 把 null 转为空集合再遍历，避免循环和流式操作出现 NPE。

需要我给你整理 **Map、String、对象 全套防NPE 标准写法** 吗？统一一套规范。