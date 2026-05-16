

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

| 维度         | 编译时多态（方法重载）                        | 运行时多态（方法重写）                                       |
| ------------ | --------------------------------------------- | ------------------------------------------------------------ |
| **决定时机** | 编译阶段（编译器确定调用哪个方法）            | 运行阶段（JVM根据对象实际类型确定调用哪个方法）              |
| **核心依据** | 方法签名（参数个数、类型、顺序）              | 对象的实际类型（而非声明类型）                               |
| **实现方式** | 同一类中同名不同参的方法                      | 子类重写父类方法，配合向上转型                               |
| **典型场景** | `System.out.println()` 支持多种数据类型的打印 | 父类引用指向子类对象，调用重写方法（如 `Animal a = new Dog(); a.eat();`） |
| **灵活性**   | 编译时固定，无法动态改变                      | 运行时动态绑定，更灵活，支持扩展                             |

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

| 修饰符         | 同一类中 | 同一包中 | 不同包的子类 | 不同包的非子类 |
| -------------- | -------- | -------- | ------------ | -------------- |
| `public`       | ✅        | ✅        | ✅            | ✅              |
| `protected`    | ✅        | ✅        | ✅            | ❌              |
| 默认（无修饰） | ✅        | ✅        | ❌            | ❌              |
| `private`      | ✅        | ❌        | ❌            | ❌              |

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

| 类型     | 转换方向    | 语法要求 | 安全性                 | 核心用途               |
| -------- | ----------- | -------- | ---------------------- | ---------------------- |
| 向上转型 | 子类 → 父类 | 自动转换 | 绝对安全               | 实现多态、统一接口接收 |
| 向下转型 | 父类 → 子类 | 强制转换 | 需用 `instanceof` 验证 | 恢复子类特有功能       |

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

1. **虚拟机栈（栈帧局部变量）** 引用对象
2. **方法区静态变量** 引用对象
3. **本地方法栈 JNI（Native）** 引用对象
4. **运行时常量池 / 字符串常量** 引用对象

> 只要被这 4 种直接引用 → 对象**不能被 GC 回收**

---

###### 1️⃣ 虚拟机栈：局部变量引用（最常见）

 **场景**

方法内定义的变量、形参，只要方法没结束，对象就存活。

```java
public class GcDemo {
    // 方法
    public void test() {
        // obj 是【虚拟机栈局部变量】
        Object obj = new Object(); 
        
        // 只要这个方法没执行完，new Object() 就被 GC Roots 引用，回收不掉
        System.out.println("执行中...");
    }
}
```

- GC Roots：当前线程栈帧里的 `obj`
- 作用：**方法运行期间，对象常驻**

---

###### 2️⃣ 方法区：静态变量引用

**场景**

`static` 修饰，属于类，不属于对象，只要类不卸载，永远是 GC Roots。

```java
public class GcDemo {
    // 静态变量 → 方法区 GC Roots
    private static Object staticObj = new Object();
}
```

- 哪怕 new GcDemo() 对象销毁了
- **staticObj 依然存活，不会被 GC**
- 危害：静态集合滥用 → 内存泄漏

---

###### 3️⃣ 本地方法栈：JNI / Native 引用

**场景**

Native 方法、C/C++ 代码引用 Java 对象

```java
public class GcDemo {
    // native 本地方法
    public native void nativeCall(Object obj);
}
```

- 底层 C 代码持有该对象引用
- JVM 不能随便回收，防止原生代码崩溃
- 日常开发少见，底层、框架、中间件大量存在

---

###### 4️⃣ 常量引用（运行时常量池）

**场景**

字符串常量、包装类常量池

```java
// 字符串常量池引用
String s = "abc";  

// 包装类常量（-128~127）
Integer a = 100;
```

- `"abc"` 存在字符串常量池
- 常量池中的引用，属于 GC Roots
- 只要常量不被清理，对象永远可达

---

###### 补充面试常考：

哪些**不是**GC Roots？

- 成员变量（实例变量）
- 普通对象互相引用

> 例：A 引用 B、B 引用 A，但没人用 A → 双双被 GC

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