## 二、面向对象

面向对象程序设计是现代软件开发中最重要的编程范式之一,它具有三个主要优点:

**可重用性**是面向对象最显著的优势之一。通过封装、继承等机制,可以方便地重用已有的代码,避免重复开发,提高开发效率。一个设计良好的类可以在多个项目中反复使用。

**可扩展性**体现在面向对象的继承和多态特性上。系统可以通过继承机制方便地扩展新的功能,而不需要修改原有代码,这种开放封闭原则是面向对象设计的重要原则。

**易于管理和维护**得益于面向对象的封装特性。通过将数据和操作封装在对象中,降低了程序的耦合度,当需要修改某个功能时,影响范围被限制在相关的类中,更容易进行维护。

### 面向对象（OOP）

> Java是**纯面向对象语言**，一切皆对象；面向对象是一种编程思想，把现实世界事物抽象成程序中的对象，用对象和对象之间交互完成业务。
> 四大核心特性：**封装、继承、多态、抽象**。

#### 一、面向对象 vs 面向过程

##### 面向过程(POP)

关注**步骤、过程**，怎么做；按步骤写函数，一步步执行。
代表语言：C。

> 例子：做饭：买菜 → 洗菜 → 切菜 → 下锅 → 出锅，关注每一步流程。

##### 面向对象(OOP)

关注**对象、谁来做**；把事物抽象成对象，对象拥有属性（特征）和行为（方法），对象调用方法完成功能。
> 例子：做饭场景：厨师对象，拥有做饭()行为；调用厨师.做饭()。

#### 二、类与对象（OOP基础）

- **类 Class**：模板、图纸，对一类事物的抽象描述，定义**属性（成员变量）**和**行为（成员方法）**。
- **对象 Object**：类的实例，根据模板new出来的真实实体。
> 类是抽象模板；对象是真实实例；一个类可以new出多个对象。

```java
//类：模板
public class Person{
    //属性（成员变量）
    String name;
    int age;

    //行为（成员方法）
    public void say(){
        System.out.println(name + "说话");
    }
}

//对象：实例化
Person p1 = new Person();
p1.name = "张三";
p1.age = 20;
p1.say();
```

##### 构造方法

1. 方法名和类名完全相同，**没有返回值**。
2. new对象的时候自动调用，用来给成员变量初始化。
3. 如果不写，编译器生成**无参默认构造**；一旦手写有参构造，默认无参就消失。
```java
public Person(String name,int age){
    this.name = name;
    this.age = age;
}
Person p = new Person("李四",18);
```
> `this`关键字：代表当前对象；区分成员变量和局部变量。

##### 成员变量 vs 局部变量

| |成员变量|局部变量|
|---|---|---|
|位置|类中，方法外面|方法内/方法参数|
|内存|堆（对象里）|栈|
|默认值|有默认初始值|没有默认值，必须手动赋值|
|生命周期|对象创建产生，对象回收销毁|方法调用产生，方法结束销毁|

#### 三、面向对象四大特性

##### 1.封装 Encapsulation

> **隐藏对象内部的细节，对外暴露安全访问入口**。
- 使用访问修饰符控制权限：`private < protected < 默认(不写) < public`
- 成员变量私有化`private`；提供`get/set`方法对外访问。

```java
public class Person{
    //私有，外部不能直接访问
    private int age;

    //对外提供访问入口，可以增加业务校验
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        if(age>0 && age<150){ //校验逻辑
            this.age = age;
        }
    }
}
```
封装好处：
1. 保护内部数据，防止外部随意篡改；
2. 内部实现修改，对外接口不变，降低耦合；
3. 可以在set方法增加参数校验。

访问权限表：

|修饰符|本类|同包|子类|其他包|
|---|---|---|---|---|
|private|✅|❌|❌|❌|
|default(不写)|✅|✅|❌|❌|
|protected|✅|✅|✅|❌|
|public|✅|✅|✅|✅|

##### 2.继承 Inheritance

> 子类复用父类的属性和方法，实现代码复用；使用`extends`。
> Java类是**单继承**：一个类只能直接继承一个父类；接口可以多继承。

```java
//父类
public class Animal{
    public String name;
    public void eat(){
        System.out.println("吃东西");
    }
}
//子类，继承父类，拥有父类非private属性方法
public class Dog extends Animal{
    public void bark(){
        System.out.println("汪汪叫");
    }
}
```
- `super`关键字：代表父对象；调用父类构造器`super()`、父类普通方法。
- 子类构造方法第一行默认`super()`，调用父类无参构造；父类没有无参构造，子类必须手动写`super(参数)`。
- 不能继承父类`private`成员；可以通过public/protected方法访问。

继承特点：
1. is‑a关系：`Dog is Animal`狗是动物，才适合继承；
2. 目的：代码复用，扩展功能；
3. 所有类默认直接或间接继承`Object`类，Object是所有类的根父类。

##### 3.多态 Polymorphism

> **同一个行为，不同对象表现不同形态**；编译看左边，运行看右边。
实现多态三个前提：
1. 有继承 / 实现接口；
2. 方法重写`@Override`；
3. 父类引用指向子类对象。

```java
Animal a = new Dog(); //父类引用指向子类对象
a.eat(); //运行执行Dog重写后的eat
```
> 编译期看左边父类，只能调用父类中存在的方法；运行时看右边真实对象，执行子类重写逻辑。

###### 方法重写 @Override

子类重写父类方法：方法名、参数列表、返回值一致；权限不能比父类更严格。
> 重写：发生父子类之间。
> 重载：同一个类中，方法名相同，参数列表不同，和返回值无关。

###### 向上转型 & 向下转型

1. **向上转型（多态，自动）**：父类引用接收子类对象，多态常用。
```java
Animal a = new Dog();
```
2. **向下转型（强制）**：父类引用转回子类类型，需要`instanceof`做类型判断，防止`ClassCastException`。
```java
if(a instanceof Dog){
    Dog dog = (Dog)a;
    dog.bark();
}
```

多态好处：
1. 解耦，面向父类/接口编程，屏蔽子类差异；
2. 新增子类，不需要修改原有业务代码。

##### 4.抽象 Abstraction

> 抽取事物共同特征，把不变的逻辑实现，把变化的逻辑定义为抽象规范，交给子类实现。
两种形式：
1. **抽象类 abstract class**：abstract修饰类，可以有普通方法+抽象方法；不能new，单继承。
2. **接口 interface**：定义行为能力契约；支持多实现。

> 抽象就是提取共性，分离模板和可变实现。

#### 四、Object根类

所有Java类都默认继承`java.lang.Object`，是所有类的祖先。
常用方法：

1. `equals()`：对象相等比较，默认比较地址，建议重写。
2. `hashCode()`：返回对象哈希码，重写equals要重写hashCode。
3. `toString()`：对象字符串输出，默认打印对象地址，建议重写。
4. `getClass()`：获取运行时对象字节码。
5. `wait() notify() notifyAll()`：线程等待唤醒。
6. `finalize()`：对象回收前执行，已经废弃。

#### 五、this 和 super 对比

|关键字|this|super|
|---|---|---|
|含义|代表当前对象|代表父类对象|
|调用构造|`this()`调用本类其他构造，必须第一行|`super()`调用父类构造，必须第一行|
|访问成员|`this.成员`访问本类属性方法|`super.成员`访问父类属性方法|

> `this()`和`super()`不能同时出现在同一个构造方法。

#### 六、重载 vs 重写（面试高频）

##### 重载 Overload

同一个类内部；**方法名相同，参数列表不同(个数/类型/顺序)**；返回值无关。
```java
public void add(int a){}
public void add(int a,int b){}
```

##### 重写 Override

父子类之间；子类重写父类方法；**方法名、参数、返回值一致**；权限不能比父类严格。
```java
class A{
    public void test(){}
}
class B extends A{
    @Override
    public void test(){}
}
```

| |重载Overload|重写Override|
|---|---|---|
|范围|同一个类|父子类|
|方法名|必须相同|必须相同|
|参数列表|必须不同|必须相同|
|返回值|无关|必须一致(协变除外)|
|注解|不需要|@Override|

#### 七、面向对象开发原则（简单了解）

1. 封装：高内聚，低耦合；
2. 继承：is‑a关系，不要滥用继承；优先组合代替继承；
3. 多态：面向接口/父类编程，而不是面向具体实现；
4. 组合优先于继承：`has‑a`拥有关系，用成员变量持有对象，比继承更灵活。

> has‑a：A里面有B对象，使用组合；is‑a：A是B，使用继承。

#### 八、高频面试题

##### Q1：面向对象四大特性？

**封装、继承、多态、抽象**。
- 封装：隐藏内部细节，对外提供访问；
- 继承：子类复用父类代码，extends单继承；
- 多态：同一行为不同实现；编译看左边，运行看右边；
- 抽象：提取共性，抽象类、接口定义规范。

##### Q2：多态的前提？

继承/实现接口；方法重写；父类引用指向子类对象。

##### Q3：重载和重写区别？

重载同类，参数列表不同；重写父子类，方法签名一致。

##### Q4：this和super可以同时用在一个构造方法第一行吗？

不能，二者都要求放在第一行，互斥。

##### Q5：构造方法能不能重写？

不能，构造方法名和类名绑定，不能继承，因此不存在重写；可以重载。

##### Q6：Java为什么是单继承？

避免多继承带来的方法冲突、菱形继承问题；通过接口多实现弥补。

#### ✅背诵总结

1. OOP核心：类是模板，对象是实例；构造方法用于对象初始化。
2. 封装：private私有化成员，get/set对外暴露，控制访问权限。
3. 继承`extends`单继承；super访问父类；is‑a关系；所有类继承Object。
4. 多态：父类引用指向子类对象；编译看左，运行看右；向上转型自动，向下转型需要instanceof。
5. 抽象：抽象类、接口，定义规范，子类实现。
6. 重载：同类，参数不同；重写：父子类，方法签名一致。
7. this代表当前对象；super代表父对象。

### 方法签名（Method Signature）

**一句话标准答案：**
Java 中的**方法签名 = 方法名 + 参数列表（参数类型、参数个数、参数顺序）**

**注意：方法签名不包含返回值、修饰符、异常！**

---

#### 一、完整定义

方法签名是**方法的唯一标识**，Java 编译器通过它来区分不同的方法，主要用于：
1. **方法重载（Overload）**：判断两个方法是否构成重载
2. **方法调用**：确定调用哪个具体方法

##### 方法签名包含：

- 方法名称
- 参数的**类型**
- 参数的**个数**
- 参数的**顺序**

##### 方法签名**不包含**：

- 返回值类型（`void`/`int`/`String` 等）
- 权限修饰符（`public`/`private`/`protected`）
- `static`/`final` 等修饰符
- 抛出的异常

---

#### 二、代码示例

```java
// 方法签名：add(int, int)
public int add(int a, int b) {
    return a + b;
}

// 方法签名：add(double, double) → 和上面不同，构成重载
double add(double a, double b) {
    return a + b;
}

// 方法签名：add(int, int) → 和第一个完全一样！编译报错
static void add(int x, int y) throws Exception {
}
```

##### 重点：

哪怕**返回值、修饰符、参数名、异常**都不一样，
只要**方法名 + 参数类型**相同，就属于**相同的方法签名**，编译报错。

---

#### 三、面试背诵版（精简）

**方法签名 = 方法名 + 参数列表**
作用：用于区分方法，是方法重载的依据。
**不包含**：返回值、修饰符、异常。

---

#### 总结

1. 方法签名只看**方法名**和**参数**，不看返回值；
2. 是 Java 区分方法、实现方法重载的核心依据；
3. 面试常考：**返回值不属于方法签名**。

---

### **5.多态的实现机制**

* **实现机制**：多态通过方法重载和方法重写实现。
* **编译时多态**：又称方法重载，在同一个类中，根据**方法签名**（参数个数或类型）来区分不同的方法。
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



### 多态完整示例

#### ✅ 多态前提（4个条件，背诵）

1. **继承**：子类继承父类
2. **重写**：子类重写父类方法
3. **父类引用指向子类对象**：`父类 引用 = new 子类();`
4. **向上转型**

> 核心一句话：**编译看左边（引用类型），运行看右边（实际对象）**

#### 示例代码

```java
// 父类
class Animal {
    public void shout() {
        System.out.println("动物叫");
    }
}

// 子类Dog继承Animal，重写shout
class Dog extends Animal {
    @Override
    public void shout() {
        System.out.println("汪汪汪");
    }
}

// 子类Cat继承Animal，重写shout
class Cat extends Animal {
    @Override
    public void shout() {
        System.out.println("喵喵喵");
    }
}

public class TestPoly {
    public static void main(String[] args) {
        // 多态写法：父类引用，指向子类对象
        Animal a1 = new Dog();
        Animal a2 = new Cat();

        a1.shout(); // 输出：汪汪汪
        a2.shout(); // 输出：喵喵喵
    }
}
```
#### 执行逻辑

- 编译阶段：`a1` 的类型是`Animal`，编译器去Animal类找`shout()`方法
- 运行阶段：`a1`实际对象是Dog，**执行子类重写后的shout**

#### 🔔 重点考点：成员变量不参与多态！

> **方法有多态，成员变量没有多态，变量编译运行都看左边引用类型**
```java
class Animal {
    String name = "动物";
}
class Dog extends Animal {
    String name = "小狗";
}

public class Test {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a.name); // 输出【动物】！不是小狗
    }
}
```

#### 向下转型（多态配套考点）

父类引用不能直接调用子类独有的方法，需要强转（向下转型）
```java
class Dog extends Animal{
    // 子类独有方法
    public void watchHouse(){
        System.out.println("狗看门");
    }
}

public class TestPoly {
    public static void main(String[] args) {
        Animal a = new Dog();
        // a.watchHouse(); //编译报错，编译看左边Animal没有这个方法

        //向下转型
        Dog dog = (Dog) a;
        dog.watchHouse(); //狗看门
    }
}
```
> 转型风险：如果实际对象不是Dog，会抛出`ClassCastException`
> 安全写法：`instanceof` 判断
```java
if(a instanceof Dog){
    Dog dog = (Dog)a;
    dog.watchHouse();
}
```

#### ✅ 多态好处（面试回答）

1. **代码扩展性强**：新增动物类，不需要修改原有业务代码
2. **解耦**：方法参数写父类，可以接收所有子类对象
```java
// 接收所有Animal子类
public static void makeSound(Animal animal){
    animal.shout();
}
//调用
makeSound(new Dog());
makeSound(new Cat());
```

#### 📢面试口述精简版

> Java多态建立在继承、方法重写、父类引用指向子类对象。口诀：**编译看左边，运行看右边**。调用方法时，编译期检查引用类型是否存在该方法；运行时执行对象实际类型重写后的方法。成员变量不存在多态，永远看引用类型。父类引用想要调用子类特有方法，需要向下转型，转型前建议用instanceof判断避免类型转换异常。

#### 常见追问

Q：静态方法有多态吗？
A：**静态方法没有多态！静态属于类，不属于对象。静态方法调用看左边引用类型。**
```java
Animal a = new Dog();
a.testStatic(); //执行父类Animal的静态方法
```

要不要我写一个**静态方法无多态**的对比小demo？



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

### **7.什么是向上转型和向下转型**

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

---

### **8.final对多态的影响**

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





### 接口 interface 

> 
> 接口是一种**行为规范、能力契约**，定义一组方法标准，**不关心具体怎么实现**。
> JDK版本迭代变化很大：

- JDK7及以前：接口只能有**public abstract抽象方法**、`public static final`常量；
- JDK8：新增`default默认方法`、`static静态方法`；
- JDK9：新增`private私有方法`。

#### 1. 基础语法

```
//定义接口
public interface Run {
    //常量，默认 public static final
    int MAX_SPEED = 100;

    //抽象方法，默认 public abstract
    void run();

    //JDK8 default 默认方法，带方法体，实现类可以选择重写或不重写
    default void showInfo(){
        System.out.println("跑的能力");
    }

    //JDK8 static静态方法，接口名直接调用，不能被实现类继承
    static void print(){
        System.out.println("接口静态方法");
    }

    //JDK9 private私有方法，接口内部复用，外部不可见
    private void inner(){
        System.out.println("私有方法");
    }
}
```

##### 类实现接口 `implements`

一个类可以**实现多个接口**（多实现），解决Java类单继承的限制。

```
public class Dog implements Run, Swim{
    //必须实现接口中所有抽象方法
    @Override
    public void run() {
        System.out.println("狗跑");
    }

    @Override
    public void swim() {
        System.out.println("狗游泳");
    }
}
```

> 
> 规则：

1. 实现类必须重写接口全部**抽象方法**；如果不想实现全部，该类要定义为`abstract`抽象类。
2. 实现接口重写方法，访问权限**必须是public**（接口抽象方法默认public）。

#### 2. 接口里面成员规则

##### 2.1 成员变量

> 
> 接口中定义变量，**默认修饰符：`public static final`，只能是常量**。

```
interface Demo{
    int NUM = 10;
    //等价 public static final int NUM =10;
}
```

- 必须初始化赋值，不能是普通成员变量；
- 接口不能定义实例变量；
- 可以通过`接口名.常量名`访问。

##### 2.2 方法分类

1. **抽象方法(JDK7)**：`public abstract`，可以省略；实现类必须重写。

```
void test();
```

2. **default 默认方法（JDK8）**

- 有方法体；用`default`关键字；
- **实现类可以直接使用，也可以重写覆盖**；
- 目的：接口升级，新增方法，**不用强制改动所有实现类，向后兼容**。

> 
> 场景：老项目接口需要新增功能，如果全部是抽象方法，所有实现类全部要改；default方法可以兼容老代码。

3. **static静态方法（JDK8）**

```
static void hello(){}
```

- 通过**接口名.方法名()调用**；
- **不会被实现类继承，实现类对象不能调用接口static方法**。

4. **private私有方法(JDK9)**

- 只能接口内部使用；default、static方法之间复用公共逻辑；
- 外部实现类无法访问。

> 
> ❗接口**没有构造方法，不能new接口对象**。

#### 3. 接口之间的继承 `extends`

接口可以继承接口，**支持多继承**，用`extends`，逗号隔开。

```
interface A{}
interface B{}
//接口多继承
interface C extends A,B{

}
```

子接口会继承父接口全部抽象方法、常量；子接口还可以新增自己的抽象方法。

> 
> 类是单继承；接口支持多继承。

#### 4. 多实现冲突问题（面试重点）

##### 场景1：多个接口，抽象方法签名完全一样

```
interface A{
    void hello();
}
interface B{
    void hello();
}
//实现类只需要实现一次hello()即可
class Test implements A,B{
    @Override
    public void hello() {}
}
```

##### 场景2：多个接口有**相同default默认方法**（JDK8经典冲突）

> 
> 两个接口存在一模一样的default方法，编译器直接报错，**实现类必须手动重写该default方法**。

```
interface A{
    default void test(){
        System.out.println("A");
    }
}
interface B{
    default void test(){
        System.out.println("B");
    }
}
//冲突，必须重写test
class Demo implements A,B{
    @Override
    public void test() {
        //可以选择调用父接口的默认方法
        A.super.test();
        //B.super.test();
    }
}
```

##### 场景3：父类普通方法 和 接口default方法重名

> 
> **类优先原则：父类方法优先，不会冲突，不需要重写**。

```
class Parent{
    public void test(){
        System.out.println("父类");
    }
}
interface A{
    default void test(){
        System.out.println("接口default");
    }
}
//直接使用父类test，接口default被忽略
class Son extends Parent implements A{
}
```

> 
> 口诀：**类优先，接口冲突必须重写**。

#### 5. 接口 vs 抽象类 abstract class（高频面试）

| 对比项 | interface 接口 | abstract class抽象类 |
| --- | --- | --- |
| 定义 | `interface` | `abstract class` |
| 变量 | 只能`public static final`常量，不能普通实例变量 | 普通成员变量、静态变量、常量都支持 |
| 构造器 | ❌没有构造方法 | ✅有构造方法，供子类super调用 |
| 方法(JDK8+) | abstract、default、static、private | 普通实例方法、抽象方法、static |
| 继承实现 | 类`implements`**多实现**；接口可`extends`多继承 | 类`extends`只能**单继承** |
| 访问修饰符 | 方法默认public(JDK9private除外) | public protected default private都可以 |
| 设计思想 | **can‑do：具备某种能力/行为契约** | **is‑a：属于某一类事物，模板抽取公共逻辑** |
| 对象实例 | 不能new | 不能new |

> 
> 使用选择：

1. 如果定义**能力、规范**，多个无关类都可以拥有这个行为 → 用接口；
2. 如果多个子类有大量**公共字段、公共业务逻辑**，需要抽取模板 → 抽象类。

示例：

- `Runnable`接口：代表可执行的能力；
- Spring的`AbstractFactoryBean`抽象类：抽取公共模板逻辑。

#### 6. 接口使用场景

1. **定义行为规范、约束**：JDK大量接口：`Runnable、Comparable、List、Map`。
2. **解耦，面向接口编程**：业务层写接口，写实现类；依赖接口而不是具体实现，方便切换实现。

```
//面向接口，不依赖具体实现
public interface UserService{
    void addUser();
}

//实现1
public class UserServiceImpl implements UserService{}
//实现2，随时替换
public class UserServiceNewImpl implements UserService{}
```

3. 回调、函数式接口（Lambda）> 
> 只有**一个抽象方法**的接口叫**函数式接口**，注解`@FunctionalInterface`，支持Lambda表达式。

```
@FunctionalInterface
public interface Consumer<T>{
    void accept(T t);
}
```

#### 7. @FunctionalInterface 函数式接口

- 接口**只能有一个抽象方法**；default、static不计入；
- 加上注解做编译校验，多写抽象方法直接编译报错；
- 可以使用Lambda简化编码。

```
@FunctionalInterface
interface Calc{
    int calc(int a,int b);
    //default方法不受限制
    default void print(){}
}
//lambda调用
Calc c = (x,y)->x+y;
```

#### 8.高频面试题

##### Q1：接口可以有构造方法吗？

不能。接口不能实例化，没有构造器。

##### Q2：接口的变量默认什么修饰符？

`public static final`，只能常量，必须赋值。

##### Q3：JDK8接口default和static方法区别？

1. default：有方法体，实现类可以继承、重写；通过对象调用。
2. static：接口名调用；**不会被实现类继承**。

##### Q4：实现多个接口，default方法重名怎么办？

必须手动重写该方法；可以使用`接口名.super.方法()`调用某个父接口默认实现。

##### Q5：父类方法和接口default重名？

**类优先原则**，直接使用父类方法，接口default被忽略，不用处理。

##### Q6：接口和抽象类怎么选？

- 做能力定义、多实现 → 接口；
- 抽取子类公共属性、公共逻辑模板，单继承 →抽象类。

##### Q7：接口是否可以被new？

不能，但是可以多态：`Run r = new Dog();`，接口引用指向实现类对象。

#### ✅背诵总结

1. interface定义接口；JDK7只有抽象方法和常量；JDK8新增default、static；JDK9新增private。
2. 成员变量默认`public static final`常量；接口无构造器。
3. 类`implements`实现接口，支持多实现，必须实现全部抽象方法；接口之间`extends`支持多继承。
4. default方法冲突：实现类必须重写；**父类方法优先级高于接口default**（类优先原则）。
5. `@FunctionalInterface`函数式接口，仅有一个抽象方法，支持Lambda。
6. 接口代表**can‑do能力契约**；抽象类代表**is‑a模板继承**。
7. 面向接口编程，降低耦合，便于替换实现类。

---

### **12.面向对象的五大设计原则（SOLID）**

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

### **20.请举例说明如何通过面向对象的设计思想解决一个实际问题**

* **设计思路**：使用面向对象设计，定义 `Product` 类表示商品，`Order` 类表示订单，通过组合关系表示订单中包含多个商品，使用多态和继承来处理不同类型的商品（如电子商品、实体商品等）。



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

### 对象的等于、排序、toString()

> 
> 开发中自定义实体类经常需要：对象判等、集合排序、对象打印输出；对应三组API：

1. 对象相等：`==`、`equals()`、`hashCode()`
2. 对象排序：`Comparable`、`Comparator`
3. 对象打印：`toString()`

#### 一、对象相等 equals + hashCode回顾

`Object`原生`equals()`底层就是`==`，比较对象地址。自定义对象业务相等，**必须重写equals()+hashCode()**。

```
class Student{
    private Long id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return Objects.equals(id, student.id) && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    @Override
    public String toString() {
        return "Student{id=" + id + ", name='" + name + "'}";
    }
}
```

- `Objects.equals(a,b)`：自动处理null，避免空指针。
- 参与equals比较的字段，全部要放进`Objects.hash()`。> 
> 不重写equals：两个new出来业务字段完全一样的对象，`equals`返回false。

---

#### 二、toString()

##### Object原生toString()

```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

输出示例：`com.demo.Student@234ac579`

> 
> 打印**类名+十六进制hash码**，对业务调试毫无意义。

##### 重写toString()目的

打印对象时，输出对象内部字段值，方便日志、调试。

```
Student s = new Student(1L,"张三");
System.out.println(s); 
//不重写：com.demo.Student@6d06d69c
//重写后：Student{id=1, name='张三'}
```

> 
> 触发toString的场景：

1. `System.out.println(对象)`
2. `"" + 对象`字符串拼接
3. 日志打印 `log.info("{}",student)`

> 
> 开发规范：**实体类强烈建议重写toString()**，IDEA自动生成。

---

#### 三、对象排序：Comparable 和 Comparator（面试高频）

Java中对象不能直接用`>` `<`比较；提供两套排序接口：

1. `java.lang.Comparable`：**内部比较器，写在实体类内部**
2. `java.util.Comparator`：**外部比较器，不修改实体类代码**

##### 1. Comparable 内部比较器

> 
> 实体类实现`Comparable<T>`接口，重写`compareTo()`，把排序规则写在实体类内部。

```java
class Student implements Comparable<Student>{
    private Long id;
    private Integer age;
    private String name;

    // 返回int：
    // this - o 升序；o - this 降序
    @Override
    public int compareTo(Student o) {
        //按年龄升序
        return this.age - o.age;
    }
}
```

返回值规则：

- `this > o` 返回正数
- `this == o` 返回0
- `this < o` 返回负数

使用：`Collections.sort(list)`；`Arrays.sort(数组)`直接排序。

```java
List<Student> list = new ArrayList<>();
Collections.sort(list); //直接排序，使用类内部compareTo规则
```

> 
> 缺点：排序规则写死在实体类；如果要换一套排序逻辑，需要修改实体类代码。

##### 2. Comparator 外部比较器（策略模式）

> 
> 外部比较器，**不需要修改实体类**，可以写多个不同排序策略；作为参数传给sort。

```java
//按age升序
Comparator<Student> ageAsc = new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.getAge() - s2.getAge();
    }
};
//使用
Collections.sort(list, ageAsc);
```

JDK8之后可以用Lambda简化：

```java
//年龄升序
Collections.sort(list,(s1,s2)->s1.getAge()-s2.getAge());

//链式多条件排序：先按age升序，age相等按id降序
list.sort(Comparator.comparingInt(Student::getAge)
        .thenComparingLong(Student::getId,Comparator.reverseOrder()));
```

##### Comparable vs Comparator对比

|  | Comparable | Comparator |
| --- | --- | --- |
| 包 | `java.lang` | `java.util` |
| 名称 | 内部比较器 | 外部比较器 |
| 方法 | `int compareTo(T o)` | `int compare(T o1,T o2)` |
| 实体类 | 需要实现接口，修改实体代码 | 不需要修改实体类 |
| 排序规则 | 写死在实体内部 | 可以定义多个策略，灵活切换 |
| 调用 | `Collections.sort(list)` | `Collections.sort(list,比较器)` |

> 
> 使用选择：

1. 对象有**默认自然排序**，就实现`Comparable`；例如Integer、String都实现Comparable。
2. 需要临时、多套不同排序逻辑，用`Comparator`。

> 
> String排序：按Unicode码值排序；数字包装类按数值排序。

##### ⚠️注意：compareTo / compare 返回值不要直接减Integer最大值，会int溢出！

错误写法：

```java
//age是Integer，相减有可能int溢出
return s1.getAge() - s2.getAge();
```

安全写法：

```java
//Integer内置compare静态方法，防止溢出
return Integer.compare(s1.getAge(), s2.getAge());
```

##### TreeSet / TreeMap 排序规则

1. key实现`Comparable`，使用内部compareTo；
2. 构造时传入`Comparator`，优先使用外部比较器；> 
> Tree系列集合判断元素相等：**依据compareTo/compare返回0，而不是equals！**
> 重要坑：如果compare返回0，TreeSet认为两个对象相等，会去重，哪怕equals返回false。

```
//compareTo返回0 → TreeSet判定为同一个对象，拒绝存入
```

#### 四、面试高频题

##### Q1：equals 和 compareTo 的关系？

- `equals`：判断业务是否相等；
- `compareTo`：判断大小顺序；> 
> 规范：如果`a.equals(b)`为true，建议`a.compareTo(b)==0`，否则TreeSet行为会很诡异，但语法不强制。

##### Q2：TreeSet判断元素重复是用equals吗？

不是。TreeSet依靠`compareTo / compare`返回0判定重复；HashSet依靠`hashCode + equals`。

##### Q3：Comparable和Comparator区别？

1. Comparable是内部比较器，实体类实现接口，重写compareTo；
2. Comparator外部比较器，不修改实体，多套排序策略，传入sort方法；
3. Collections.sort两个重载：一个只用Comparable，一个接收Comparator。

##### Q4：为什么要重写toString()？

Object原生toString输出对象地址，没有业务含义；重写后打印对象字段，方便日志排查问题。

##### Q5：Objects.equals()和普通a.equals(b)区别？

`Objects.equals(a,b)`会做null判断，a为null不会空指针。

##### Q6：重写equals一定要重写hashCode，那重写compareTo需要重写什么？

**不需要重写hashCode、equals**，只是业务上建议保持语义一致；只影响Tree系列集合。

#### ✅背诵总结

1. **equals/hashCode**：自定义对象业务相等，两个方法一起重写；`Objects.equals()`处理null。
2. **toString()**：Object默认打印地址；重写输出对象字段，用于日志调试，实体类建议必写。
3. **Comparable内部比较器**：实体实现接口，重写`compareTo()`，提供默认排序规则。
4. **Comparator外部比较器**：不修改实体，多套排序策略，JDK8支持链式Lambda；`Integer.compare()`避免溢出。
5. TreeSet/TreeMap根据`compareTo/compare返回0`判重；HashSet根据`hashCode+equals`判重。
6. 业务约定：`a.equals(b)`为true，最好保证`compareTo(b)==0`。

> 
> 接下来可以讲：异常体系（Error、Exception、运行时异常、受检异常）。


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

### 27.构造方法调用顺序 + 成员变量初始化

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

### 代码执行顺序

#### 静态代码块和普通代码块

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

| 特性     | 静态代码块               | 普通代码块                   |
| -------- | ------------------------ | ---------------------------- |
| 修饰符   | 必须用 `static`          | 无 `static`                  |
| 执行时机 | 类加载时执行（仅一次）   | 每次创建实例时执行（多次）   |
| 执行顺序 | 早于普通代码块和构造方法 | 晚于静态代码块，早于构造方法 |
| 访问权限 | 只能访问静态成员         | 可访问静态和实例成员         |
| 主要作用 | 初始化静态成员           | 初始化实例成员或提取构造共性 |

理解两者的区别有助于在合适的场景选择正确的初始化方式，尤其是在需要控制初始化时机或复用初始化逻辑时。

#### 25.执行顺序1

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

---

##### 一、正确的执行顺序

Java 代码生命周期 **固定顺序**：

1. **编译期**（最开始，只做一次）
2. **类加载期**（运行时，第一次使用类时）
3. **实例化**（运行时，new 对象时）
4. **方法调用**（真正执行业务代码）

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

#### 27.代码执行顺序4

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

##### 三、终极总顺序

1. **静态变量**（从上到下）
2. **静态代码块**（从上到下）
3. **成员变量**（从上到下）
4. **构造代码块**（从上到下）
5. **构造方法**（最后）

---

##### 四、用代码验证

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



### Super 和 this 

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

### 30、哈希算法

**哈希算法 = 计算 hashCode 的算法**

1. Object 类的 hashCode () 方法

   所有对象默认都有 hashCode，用于定位哈希地址。

2. HashMap、HashSet、Hashtable、ConcurrentHashMap

   底层是哈希表，通过 `hashCode()`计算数组下标，通过 `equals()`判断是否同一个元素。

3. String 类重写了 hashCode ()

   方便作为 Map 的 key，保证相同字符串 hashCode 相同。

一句话总结：**只要是哈希集合（HashMap/HashSet），底层全靠哈希算法。**



### == 、equals() 、hashCode()

> 
> 面试高频三连，重点区分**基本类型和引用类型**，重写equals必须重写hashCode原则。

#### 1. `==` 运算符

`==` 是Java运算符，分两种情况：**基本数据类型**、**引用数据类型**。

##### ① 基本数据类型（byte short int long float double char boolean）

**比较的是两边的值是否相等**。

```java
int a = 10;
int b = 10;
System.out.println(a == b); // true
```

##### ② 引用数据类型（对象、数组、String等）

**比较的是对象的内存地址（栈中保存的引用地址），判断是不是同一个对象**，**不比较对象里面的内容**。

```java
User u1 = new User(1L,"张三");
User u2 = new User(1L,"张三");
System.out.println(u1 == u2); // false，两个new，堆中两块不同内存
```

> 
> 总结：
> 
> 
> - 基本类型 `==`：比较**值**
> - 引用类型 `==`：比较**对象地址**

#### 2. equals() 方法

`equals()` 是 `Object` 类中的实例方法，所有Java对象都继承这个方法。
Object中原始源码：

```java
public boolean equals(Object obj) {
    // 原生equals底层就是 ==，比较对象地址
    return (this == obj);
}
```

> 
> ⚠️ **Object默认equals和`==`效果一样，比较地址；只有类重写equals，才会比较对象内部业务属性。**

##### 例子1：String重写了equals

String重写equals，不再比较地址，**比较字符串内容**。

```java
String s1 = new String("abc");
String s2 = new String("abc");

System.out.println(s1 == s2);     // false，不同对象地址不同
System.out.println(s1.equals(s2));// true，重写equals，比较字符内容
```

> 
> String、Integer、Long、Date等JDK内置类，**全部重写equals，比较内容**。

##### 例子2：自定义类没有重写equals

```java
class User{
    Long id;
    String name;
}
User u1 = new User(1L,"张三");
User u2 = new User(1L,"张三");
System.out.println(u1.equals(u2)); 
// false，没有重写equals，调用Object原生equals，等价于==，比较地址
```

##### ✅重写equals的规范（约定）

重写equals必须遵守5条规则：

1. **自反性**：`x.equals(x)` → true
2. **对称性**：`x.equals(y)` 为true，则`y.equals(x)`也为true
3. **传递性**：x=y，y=z，则x=z
4. **一致性**：对象属性不变，多次调用equals结果不变
5. **非空性**：`x.equals(null)` → 一定返回false

> 
> IDEA可以自动生成equals+hashCode，选择业务关键字段（id等）。

#### 3. hashCode() 方法

`hashCode()`也是Object的native本地方法，返回`int`哈希码。

> 
> hash码：对象的一个整数“指纹”，主要服务于**HashMap、HashSet、HashTable**这类哈希集合。

Object默认实现：根据对象内存地址生成hash值。

```java
public native int hashCode();
```

##### hashCode核心约定（非常重要）

> 
> Java官方约定：

1. 如果两个对象`equals()`相等，那么它们的`hashCode()`**必须相等**。
2. 如果两个对象`hashCode()`相等，**equals不一定相等（哈希碰撞）**。

👉 **重写equals，就必须重写hashCode！**
如果只重写equals，不重写hashCode，会破坏约定，导致HashMap/HashSet出现异常。

##### 错误案例：只重写equals，不重写hashCode

```java
class User {
    Long id;
    String name;
    //只重写equals，没有重写hashCode
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(id, user.id) && Objects.equals(name, user.name);
    }
}
```

放入HashSet：

```java
User u1 = new User(1L,"张三");
User u2 = new User(1L,"张三");

HashSet<User> set = new HashSet<>();
set.add(u1);
System.out.println(set.contains(u2)); // false！！
```

原因：
equals判定u1、u2业务上相等，但是hashCode是Object原生实现，基于内存地址，两个对象hashCode不一样。
HashSet先比较hashCode，hash不一样直接判定不是同一个对象，不会再调用equals。

##### 正确写法：equals 和 hashCode一起重写（JDK7+ Objects工具类）

```java
class User {
    Long id;
    String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(id, user.id) && Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        // 根据参与equals判断的字段生成hash码
        return Objects.hash(id, name);
    }
}
```

> 
> `Objects.equals(a,b)`：帮我们处理null，避免空指针。

#### 4. HashMap中put流程（结合equals+hashCode，面试高频）

1. 调用key的`hashCode()`计算hash值，定位数组桶下标。
2. 如果桶位置没有元素，直接放入。
3. 如果桶已有元素：
   - hash值不一样：直接挂链表/红黑树。
   - hash值一样（哈希碰撞）：再调用`key.equals()`判断key对象是否真正相等。
     - equals相等：覆盖旧value。
     - equals不相等：追加到链表/红黑树。

> 
> 先比hashCode，hash不同直接不相等；hash相同才调用equals。

#### 5. 容易混淆的面试题

##### Q1：`==` 和 equals 的区别

1. `==`是运算符；equals是Object的方法。
2. 基本类型`==`比较值；引用类型`==`比较内存地址。
3. equals默认等价`==`比较地址；重写后比较对象业务内容。

##### Q2：为什么重写equals必须重写hashCode？

Java规范约定equals相等的对象hashCode必须相等。
如果只重写equals，不重写hashCode，两个业务相等的对象hash码不同，存入HashMap、HashSet会出现重复存储、contains判断错误。

##### Q3：两个对象hashCode相等，equals一定true吗？

不一定，存在**哈希碰撞**，不同对象算出一样hash值，此时必须用equals再做二次校验。

##### Q4：hashCode返回值一样，equals返回false会发生什么？

HashMap会发生哈希碰撞，两个key放到同一个桶，链表上共存。

##### Q5：String s1 = "abc"; String s2 = new String("abc"); ==与equals结果？

```java
String s1 = "abc";
String s2 = new String("abc");
s1 == s2;     //false，一个常量池，一个堆对象，地址不同
s1.equals(s2);//true，重写equals比较内容
```

#### 6. 注意点

1. `Objects.equals(a,b)`推荐用来做对象判等，自动处理null，不会NPE。

```java
Objects.equals(a,b);
//等价于 a==b || (a!=null && a.equals(b))
```

2. 包装类缓存：`Integer i1=100; Integer i2=100; i1==i2 true`；大于127就false，只是缓存的特例，**对象比较优先用equals**。
3. 数组没有重写equals，数组用`Arrays.equals()`比较数组内容。

#### 极简背诵总结

1. `==`：基本类型比较**值**；引用类型比较**内存地址**。
2. `equals()`来自Object，原生底层就是`==`；重写后用来比较对象业务内容。
3. `hashCode()`返回对象哈希码，服务哈希集合；**equals相等，hashCode必须相等；hash相等equals不一定相等**。
4. 重写equals，**必须同步重写hashCode**，否则HashMap/HashSet行为异常。
5. HashMap先判断hashCode，hash冲突之后再调用equals。

> 
> 下一轮可以讲：HashMap底层原理，面试重中之重。

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

