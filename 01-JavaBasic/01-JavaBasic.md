### 标准通用响应

日常开发里，**后端统一返回 Response（响应体）** 分**通用基础字段**、**扩展业务字段**，分主流格式给你整理，附示例。

#### 一、标准通用响应（前后端最常用）

##### 1. 基础四件套（必选）

| 字段名    | 类型        | 说明                                   |
| --------- | ----------- | -------------------------------------- |
| `code`    | 数字/字符串 | 状态码（核心）                         |
| `message` | 字符串      | 提示信息                               |
| `data`    | 任意类型    | 业务数据（成功返回主体，失败可为null） |
| `success` | 布尔        | 简化判断：true=成功，false=失败        |

###### 示例 JSON

```json
// 成功响应
{
  "success": true,
  "code": 200,
  "message": "操作成功",
  "data": {
    "id": 1001,
    "name": "测试数据"
  }
}

// 失败响应
{
  "success": false,
  "code": 500,
  "message": "服务器内部错误",
  "data": null
}
```

---

##### 2. 分页场景 额外字段（列表接口必加）

分页接口在基础上增加：

- `total`：总数据条数
- `pages`：总页数
- `current`：当前页码
- `size`：每页条数
- `records`：当前页数据列表

**分页示例**

```json
{
  "success": true,
  "code": 200,
  "message": "查询成功",
  "data": {
    "current": 1,
    "size": 10,
    "total": 56,
    "pages": 6,
    "records": [
      {"id":1,"name":"A"},
      {"id":2,"name":"B"}
    ]
  }
}
```

---

#### 二、常见扩展字段（根据项目按需加）

1. **timestamp / ts**
   时间戳（服务器响应时间，防缓存、日志排查）
2. `traceId`
   链路追踪ID（微服务/日志排查问题必备）
3. `error` / `errors`
   详细错误信息、字段校验错误（表单校验专用）
4. `token` / `tokenHead`
   登录/刷新接口返回令牌
5. `warn`
   警告信息（非报错，只是提醒）

---

#### 三、主流状态码规范（配套 code 使用）

##### HTTP 风格码（最通用）

- `200`：请求成功
- `400`：参数错误
- `401`：未登录/Token失效
- `403`：权限不足
- `404`：接口不存在
- `500`：服务异常

##### 自定义业务码（大型项目）

如 `10001=账号不存在`、`10002=密码错误` 等。

---

#### 四、Java 实体类模板（SpringBoot 直接用）

```java
@Data
public class Result<T> {
    private Boolean success;
    private Integer code;
    private String message;
    private T data;

    // 成功静态方法
    public static <T> Result<T> success(T data) {
        Result<T> r = new Result<>();
        r.setSuccess(true);
        r.setCode(200);
        r.setMessage("成功");
        r.setData(data);
        return r;
    }

    // 失败静态方法
    public static <T> Result<T> fail(Integer code, String message) {
        Result<T> r = new Result<>();
        r.setSuccess(false);
        r.setCode(code);
        r.setMessage(message);
        r.setData(null);
        return r;
    }
}
```

---

##### 总结

- **简单接口**：`success + code + message + data` 足够
- **分页接口**：多加 `current/size/total/records`
- **微服务/线上项目**：额外加 `traceId、timestamp`
- **表单校验**：增加 `errors` 字段



### 响应格式

---

#### 1. 为什么会用 `result` 代替 `data`？

这两个字段**作用一模一样**：

- 都是放**接口返回的业务数据**
- 只是命名风格不同

##### 最常见两种响应格式

###### ① 用 data（国内最主流）

```json
{
  "success": true,
  "code": 200,
  "message": "查询成功",
  "data": {
    "id": 1,
    "name": "张三"
  }
}
```

###### ② 用 result（国外/老项目/简洁风格）

```json
{
  "success": true,
  "code": 200,
  "message": "查询成功",
  "result": {
    "id": 1,
    "name": "张三"
  }
}
```

---

#### 2. 两者区别（一句话）

**data = 数据**
**result = 结果**

**功能完全一样，只是名字不同。**

---

#### 3. 什么时候用 result 更合适？

这些场景大家更喜欢用 `result`：

- 接口返回**单一结果**（如登录结果、计算结果）
- 前端习惯用 `res.data.result`
- 老项目、PHP / Python / Node 项目
- 追求简洁、语义化

---

#### 4. 统一响应实体类（支持 result 版）

如果你想项目里用 `result` 替代 `data`，实体类这样写：

```java
@Data
public class ApiResponse<T> {
    private Boolean success;
    private Integer code;
    private String message;
    private T result; // 这里用 result 代替 data

    // 成功
    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> r = new ApiResponse<>();
        r.setSuccess(true);
        r.setCode(200);
        r.setMessage("操作成功");
        r.setResult(data);
        return r;
    }

    // 失败
    public static <T> ApiResponse<T> fail(int code, String msg) {
        ApiResponse<T> r = new ApiResponse<>();
        r.setSuccess(false);
        r.setCode(code);
        r.setMessage(msg);
        r.setResult(null);
        return r;
    }
}
```

---

#### 5. 行业小总结

- **国内公司 90% 用 `data`**
- **国外/开源项目常用 `result`**
- **功能完全一样，不影响逻辑**

---

#### 最终结论

你完全可以放心：
**用 data 或 result 都可以，项目统一就行，没有对错。**
你看到别人用 `result` 替换 `data`，就是这个原因。



### 静态代理 + 动态代理

#### 一、代理模式是什么？

**代理 = 代替真实对象做事，在不修改原代码的前提下，增强功能。**

- 真实对象：只做核心业务
- 代理对象：做**预处理、过滤、日志、事务、增强**

作用：**AOP 底层核心、解耦、功能增强**

---

#### 二、静态代理（Static Proxy）

##### 1. 定义

- **代理类在编译期就已经生成 .class 文件**
- 一个**真实类**对应一个**代理类**

##### 2. 结构

1. **公共接口（Subject）**：定义方法
2. **真实类（RealSubject）**：实现接口，核心逻辑
3. **代理类（ProxySubject）**：实现接口，持有真实对象，调用前后增强

##### 3. 代码示例

###### （1）接口

```java
public interface UserService {
    void save();
}
```

###### （2）真实类

```java
public class UserServiceImpl implements UserService {
    @Override
    public void save() {
        System.out.println("保存用户");
    }
}
```

###### （3）代理类

```java
public class UserServiceProxy implements UserService {

    // 持有真实对象
    private UserService target;

    public UserServiceProxy(UserService target) {
        this.target = target;
    }

    @Override
    public void save() {
        System.out.println("开启事务"); // 增强
        target.save(); // 调用真实方法
        System.out.println("提交事务"); // 增强
    }
}
```

###### （4）测试

```java
public class Test {
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();
        UserService proxy = new UserServiceProxy(target);
        proxy.save();
    }
}
```

##### 4. 优点

- 简单、直观、编译期生成

##### 5. **缺点（重点！）**

- **一个接口只能对应一个代理类，接口增多代理类爆炸**
- **维护困难，不易扩展**

---

#### 三、动态代理（重点！面试必考）

**运行时动态生成代理类，不需要手动写代理类。**

Java 有**两种动态代理**：

1. **JDK 动态代理（基于接口）**
2. **CGLIB 动态代理（基于继承）**

---

#### 四、JDK 动态代理（基于接口）

##### 1. 原理

- **基于接口实现**
- 利用 `java.lang.reflect.Proxy` + `InvocationHandler`
- **在内存中动态生成代理类**

##### 2. 核心 API

- `Proxy.newProxyInstance(类加载器, 接口数组, 处理器)` → 生成代理对象
- `InvocationHandler.invoke()` → 所有方法都会走这里（增强）

##### 3. 代码（必须背）

```java
public class JdkProxyFactory implements InvocationHandler {

    private Object target;

    public JdkProxyFactory(Object target) {
        this.target = target;
    }

    // 获取代理对象
    public Object getProxy() {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                this
        );
    }

    // 所有方法执行都走这里
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("JDK代理：开启事务");
        Object result = method.invoke(target, args); // 执行真实方法
        System.out.println("JDK代理：提交事务");
        return result;
    }
}
```

##### 4. 特点

- **必须有接口**
- 无侵入、代理类动态生成
- Spring AOP **默认优先使用 JDK 代理**

---

#### 五、CGLIB 动态代理（基于继承）

##### 1. 原理

- **基于继承**：动态生成**目标类的子类**作为代理
- 不需要接口
- 底层用 **ASM 字节码框架**

##### 2. 核心 API

- `Enhancer`
- `MethodInterceptor`

##### 3. 代码

```java
public class CglibProxyFactory implements MethodInterceptor {

    private Object target;

    public CglibProxyFactory(Object target) {
        this.target = target;
    }

    public Object getProxy() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass()); // 设置父类
        enhancer.setCallback(this);
        return enhancer.create(); // 创建子类代理
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("CGLIB：开启事务");
        Object result = method.invoke(target, args);
        System.out.println("CGLIB：提交事务");
        return result;
    }
}
```

##### 4. 特点

- **不需要接口**
- final 类**不能代理**（不能被继承）
- Spring AOP：**类没有接口时使用 CGLIB**

---

#### 六、静态代理 vs JDK动态代理 vs CGLIB 对比（面试必考）

##### 1. 静态代理

- 编译期生成代理类
- 一个接口一个代理
- 简单，**扩展性极差**

##### 2. JDK 动态代理

- **基于接口**
- 运行时生成代理
- 速度快
- **必须有接口**

##### 3. CGLIB 动态代理

- **基于继承（子类）**
- 无需接口
- 不能代理 final 类
- 生成代理稍慢，**执行快**

---

#### 七、Spring AOP 用哪种代理？（超级高频面试题）

- **目标类有接口 → 默认 JDK 动态代理**
- **目标类没有接口 → CGLIB**
- 可以强制 `proxy-target-class="true"` 全部用 CGLIB

---

#### 八、终极精简背诵版（面试直接说）

1. **静态代理：编译期生成，一对一，代码冗余。**
2. **JDK动态代理：基于接口，运行时生成，必须有接口。**
3. **CGLIB动态代理：基于继承，无需接口，不能代理final类。**
4. **Spring AOP：有接口用JDK，无接口用CGLIB。**
5. **代理模式 = AOP底层实现。**

---

如果你愿意，我可以给你画一张**静态代理 / JDK / CGLIB 三者原理结构图**，**一眼彻底看懂底层区别**，要吗？

### 19. **什么是类的初始化和实例化？两者有什么区别和联系？**

- **初始化**：类加载时，JVM为类分配内存并执行静态初始化块。
- **实例化**：通过 `new` 关键字创建类的对象。
- **区别**：初始化是类加载的一部分，实例化是创建对象的过程。

#### 一、先明确核心概念：类初始化 vs 实例化

在Java中，“类”是“对象”的模板，而“类初始化”和“实例化”是两个不同阶段的操作，对应“模板准备”和“用模板造对象”的过程：

| 维度         | 类的初始化（Class Initialization）                     | 类的实例化（Object Instantiation）                   |
| ------------ | ------------------------------------------------------ | ---------------------------------------------------- |
| **核心目标** | 为“类本身”分配资源、执行静态代码，让类具备被使用的条件 | 为“单个对象”分配内存、执行实例代码，创建类的具体实例 |
| **操作对象** | 类（`Class`对象，JVM层面的元数据）                     | 类的实例（普通对象，基于类模板创建的具体个体）       |
| **触发前提** | 类首次被主动使用（如调用静态方法、访问静态变量等）     | 类已完成初始化（否则会先触发类初始化）               |

#### 二、类的初始化：JVM如何“激活”一个类？

类的初始化是**JVM将类的字节码（.class文件）加载到内存后，为类的静态成员分配内存、执行静态代码的过程**，最终生成JVM可识别的`Class`对象（每个类在JVM中只有一个`Class`对象）。

##### 1. 类初始化的触发时机（“主动使用”场景）

根据Java虚拟机规范，只有当类被“主动使用”时，才会触发初始化。常见场景包括：

- 调用类的**静态方法**（如`Math.max(1,2)`）；
- 访问或修改类的**静态变量**（如`System.out.println(Student.schoolName)`）；
- 用`new`创建类的实例（实例化的前提是类已初始化）；
- 初始化子类时（会先触发父类的初始化）；
- 用反射调用类（如`Class.forName("com.example.Student")`）。

*注意：被动使用不会触发初始化，例如访问类的静态常量（`static final`修饰，编译时已确定值）、通过子类访问父类的静态变量（仅触发父类初始化，不触发子类）。*

##### 2. 类初始化的执行流程（固定顺序）

类初始化严格按以下顺序执行，且**只执行一次**（JVM保证线程安全）：

1. **加载父类**：若类有父类，先触发父类的初始化（递归执行，直到`Object`类）；
2. **分配静态变量内存**：为类的所有`static`成员变量分配内存，并设置默认值（如`int`默认0，`String`默认`null`）；
3. **执行静态代码块**：按代码编写顺序执行`static {}`中的代码（静态代码块用于初始化静态变量，如`static { schoolName = "北京大学"; }`）。

**示例：类初始化过程**

```java
class Parent {
    // 父类静态变量
    public static String parentStaticVar = "父类静态变量";
    
    // 父类静态代码块
    static {
        System.out.println("父类静态代码块执行：" + parentStaticVar);
    }
}

class Child extends Parent {
    // 子类静态变量
    public static String childStaticVar = "子类静态变量";
    
    // 子类静态代码块
    static {
        System.out.println("子类静态代码块执行：" + childStaticVar);
    }
}

public class InitDemo {
    public static void main(String[] args) {
        // 访问子类静态变量，触发子类初始化（先触发父类初始化）
        System.out.println(Child.childStaticVar);
    }
}
```

**执行结果（体现初始化顺序）**：

```
父类静态代码块执行：父类静态变量
子类静态代码块执行：子类静态变量
子类静态变量
```

#### 三、类的实例化：如何“创建一个对象”？

类的实例化是**在类已初始化的基础上，通过`new`关键字（或反射、克隆等方式）创建具体对象的过程**，每个实例都有独立的实例成员（非`static`成员）。

##### 1. 实例化的触发时机

- 显式使用`new`关键字（最常见，如`Student s = new Student()`）；
- 反射（如`Class.forName("com.example.Student").newInstance()`）；
- 克隆（调用对象的`clone()`方法，需实现`Cloneable`接口）；
- 反序列化（从字节流恢复对象，需实现`Serializable`接口）。

##### 2. 实例化的执行流程（固定顺序）

每次实例化都会执行以下步骤，**每个实例独立执行一次**：

1. **分配实例内存**：为对象的非`static`成员变量分配内存（存储在堆中），并设置默认值（如`int`默认0，`String`默认`null`）；
2. **执行父类构造器**：先调用父类的构造器（递归执行，直到`Object`的`toString()`），初始化父类的实例成员；
3. **执行实例初始化代码**：按代码编写顺序执行“实例代码块”（`{}`包裹的非静态代码）和“构造器代码”，最终完成实例成员的赋值；
4. **返回对象引用**：将堆中对象的地址赋值给栈中的引用变量（如`Student s`中的`s`）。

**示例：实例化过程**

```java
class Person {
    // 父类实例变量
    private String name;
    
    // 父类实例代码块（非静态代码块）
    {
        System.out.println("父类实例代码块执行");
        this.name = "默认姓名";
    }
    
    // 父类构造器
    public Person() {
        System.out.println("父类无参构造器执行：name=" + this.name);
    }
}

class Student extends Person {
    // 子类实例变量
    private int age;
    
    // 子类实例代码块
    {
        System.out.println("子类实例代码块执行");
        this.age = 18;
    }
    
    // 子类构造器
    public Student() {
        // 隐含调用super()（父类无参构造器）
        System.out.println("子类无参构造器执行：age=" + this.age);
    }
}

public class InstantiateDemo {
    public static void main(String[] args) {
        // new关键字触发实例化（先确保Student类已初始化）
        Student student = new Student();
    }
}
```

**执行结果（体现实例化顺序）**：

```
父类实例代码块执行
父类无参构造器执行：name=默认姓名
子类实例代码块执行
子类无参构造器执行：age=18
```

#### 四、类初始化与实例化的核心区别与联系

##### 1. 核心区别（5个关键维度）

| 对比维度     | 类的初始化                                   | 类的实例化                                         |
| ------------ | -------------------------------------------- | -------------------------------------------------- |
| **执行次数** | 每个类在JVM生命周期内**仅执行一次**          | 可执行多次（创建多少个对象，就执行多少次）         |
| **操作内容** | 处理静态成员（`static`变量、`static`代码块） | 处理实例成员（非`static`变量、实例代码块、构造器） |
| **内存分配** | 静态成员存储在“方法区”（元空间）             | 实例成员存储在“堆区”，引用存储在“栈区”             |
| **触发依赖** | 不依赖实例（可独立执行，如访问静态变量）     | 必须依赖类初始化（类未初始化则先触发初始化）       |
| **最终产物** | 生成JVM层面的`Class`对象（类元数据）         | 生成普通对象（`Class`的实例）                      |

##### 2. 核心联系（依赖关系）

类初始化是实例化的**前提和基础**，两者存在严格的先后顺序：

1. 当触发实例化（如`new Student()`）时，JVM会先检查该类是否已初始化；
2. 若未初始化，则先执行类的初始化（加载父类→分配静态变量→执行静态代码块）；
3. 类初始化完成后，才会执行实例化流程（分配实例内存→执行父类构造器→执行实例代码块和子类构造器）。

简单说：**没有“初始化的类”，就无法创建“实例化的对象”**。

#### 五、典型误区澄清

1. **误区1**：“`new`关键字触发的是类初始化”  
   错。`new`的直接目的是实例化，但会先检查类是否初始化——若未初始化，会“顺带”触发类初始化，而非`new`直接触发初始化。

2. **误区2**：“静态代码块和实例代码块都会在类加载时执行”  
   错。静态代码块在**类初始化**时执行（仅一次），实例代码块在**实例化**时执行（每个对象一次），类加载时（仅加载字节码到内存，未执行初始化）不会执行任何代码块。

3. **误区3**：“父类的构造器会在类初始化时执行”  
   错。父类的**静态代码块**在类初始化时执行，父类的**构造器**在子类实例化时执行（通过`super()`调用），属于实例化阶段的操作。


##### 总结

- **类初始化**：为“类模板”做准备，处理静态资源，仅一次，是实例化的前提；  
- **实例化**：用“类模板”造对象，处理实例资源，可多次，依赖类初始化；  
- 两者的本质是“模板激活”与“对象创建”的关系，理解其执行顺序和内容，是掌握Java内存模型和对象生命周期的关键。

------

###

### 23.方法签名

在 Java 中，**方法签名（Method Signature）** 是用于唯一标识一个方法的关键信息，由**方法名称**和**参数列表**（参数类型、顺序）组成，与返回值类型、访问修饰符、异常声明等无关。

方法签名的核心作用是：在编译阶段区分不同的方法（尤其是重载方法）。

#### 一、方法签名的组成

方法签名 = **方法名称** + **参数列表**（参数类型 + 参数顺序）

#### 示例：

```java
public class MethodSignatureDemo {
    // 方法1：签名为 add(int, int)
    public int add(int a, int b) {
        return a + b;
    }
    
    // 方法2：签名为 add(double, double)（与方法1重载）
    public double add(double a, double b) {
        return a + b;
    }
    
    // 方法3：签名为 add(int, double)（与前两者重载）
    public double add(int a, double b) {
        return a + b;
    }
}
```

- 上述 3 个 `add` 方法因**参数类型/顺序不同**，签名不同，构成重载。

#### 二、不影响方法签名的因素

以下要素**不属于方法签名**，不能用于区分方法：

1. **返回值类型**：  

   ```java
   // 编译错误：仅返回值不同，签名相同（都是 add(int, int)）
   public int add(int a, int b) { ... }
   public double add(int a, int b) { ... } // 与上冲突
   ```

2. **参数名称**：  

   ```java
   // 编译错误：仅参数名称不同，签名相同（都是 add(int, int)）
   public int add(int x, int y) { ... }
   public int add(int a, int b) { ... } // 与上冲突
   ```

3. **访问修饰符**（`public`/`private` 等）、`static`/`final` 等修饰符：  

   ```java
   // 编译错误：修饰符不同不改变签名（都是 add(int, int)）
   public int add(int a, int b) { ... }
   private int add(int a, int b) { ... } // 与上冲突
   ```

4. **异常声明**：  

   ```java
   // 编译错误：异常不同不改变签名（都是 divide(int, int)）
   public int divide(int a, int b) { ... }
   public int divide(int a, int b) throws Exception { ... } // 与上冲突
   ```

#### 三、方法签名的实际应用

1. **方法重载（Overload）**：  
   重载的本质是“同一类中存在多个**签名不同**的同名方法”，这也是方法签名最核心的应用场景。

2. **反射调用**：  
   反射中通过 `Class.getMethod(String name, Class<?>... parameterTypes)` 查找方法时，`name` 是方法名，`parameterTypes` 就是参数类型列表，二者共同构成方法签名。

   示例：

   ```java
   // 通过反射获取 add(int, int) 方法
   Class<?> clazz = MethodSignatureDemo.class;
   Method method = clazz.getMethod("add", int.class, int.class);
   ```

3. **编译器区分方法**：  
   编译时，编译器通过方法签名确定调用哪个方法，避免歧义。

#### 四、特殊情况：泛型方法的签名

泛型方法的签名会包含**类型参数的擦除结果**（泛型擦除后）：

```java
public class GenericDemo {
    // 签名为 print(List)（泛型擦除后，List<String> → List）
    public void print(List<String> list) { ... }
    
    // 编译错误：泛型擦除后签名相同（都是 print(List)）
    public void print(List<Integer> list) { ... } // 与上冲突
}
```

#### 总结

- 方法签名由**方法名 + 参数类型列表（含顺序）** 组成，是方法的唯一标识。
- 返回值、参数名、修饰符等不影响方法签名。
- 核心用途：区分重载方法、反射查找方法、编译器确定调用目标。

记住：**“同名不同参”构成重载，而“不同参”的判断依据就是方法签名**。



### **什么是Java的命令行参数？如何在程序中获取和使用命令行参数？**

- **命令行参数**：程序启动时可以传入的参数，存储在 `String[] args` 中。

- **使用**：

  ```java
  public static void main(String[] args) {
      for (String arg : args) {
          System.out.println(arg);
      }
  }
  ```

#### 一、什么是Java的命令行参数？

命令行参数是**程序启动时通过命令行传递给程序的额外信息**，本质是一组字符串。它允许用户在不修改代码的情况下，为程序提供不同的输入，使程序更灵活（例如指定配置文件路径、运行模式等）。

在Java中，命令行参数会被自动封装到`main`方法的`String[] args`数组中，程序通过读取该数组即可获取参数。

#### 二、如何在程序中获取和使用命令行参数？

##### 1. 基础语法：`main`方法的`args`参数

Java程序的入口是`main`方法，其标准定义为：

```java
public static void main(String[] args) {
    // args即为命令行参数数组
}
```

- `args`是一个`String`类型的数组，每个元素对应一个命令行参数；
- 参数之间用**空格**分隔；
- 若参数本身包含空格，需用**双引号**包裹（例如`"hello world"`会被视为一个参数）。

##### 2. 示例：获取并使用命令行参数

下面是一个简单程序，演示如何读取并处理命令行参数：

```java
public class CommandLineArgsDemo {
    public static void main(String[] args) {
        // 1. 打印参数总数
        System.out.println("参数个数：" + args.length);
        
        // 2. 遍历并打印所有参数
        for (int i = 0; i < args.length; i++) {
            System.out.println("参数" + (i + 1) + "：" + args[i]);
        }
        
        // 3. 实际场景：根据参数执行不同逻辑
        if (args.length > 0) {
            switch (args[0]) {
                case "-version":
                    System.out.println("程序版本：v1.0.0");
                    break;
                case "-help":
                    System.out.println("使用帮助：java CommandLineArgsDemo [参数]");
                    break;
                default:
                    System.out.println("未知参数：" + args[0]);
            }
        } else {
            System.out.println("请输入命令行参数");
        }
    }
}

```





##### 3. 运行程序并传递参数

编译并运行上述程序时，可在命令行中传入参数，格式如下：

```bash
# 编译Java文件
javac CommandLineArgsDemo.java

# 运行程序并传递参数（示例1：无参数）
java CommandLineArgsDemo

# 运行程序并传递参数（示例2：带多个参数）
java CommandLineArgsDemo -version "hello world" 123
```

**运行结果（示例2）**：

```
参数个数：3
参数1：-version
参数2：hello world
参数3：123
程序版本：v1.0.0
```

#### 三、命令行参数的常见应用场景

1. **指定程序运行模式**：如`-dev`（开发模式）、`-prod`（生产模式）；
2. **传递配置信息**：如文件路径（`java Program /data/config.txt`）、端口号（`java Server 8080`）；
3. **参数化执行逻辑**：如批量处理时指定处理范围（`java BatchProcessor 100 200`）；
4. **工具类程序交互**：如命令行工具的参数（`-help`查看帮助、`-output`指定输出文件）。

#### 四、注意事项

1. **参数顺序**：`args`数组严格按照命令行输入的顺序存储，程序需按约定顺序解析；
2. **类型转换**：`args`中的元素都是字符串，若需要数字、布尔等类型，需手动转换（如`int port = Integer.parseInt(args[0])`）；
3. **参数校验**：实际开发中需判断参数是否存在、格式是否正确，避免`ArrayIndexOutOfBoundsException`或转换异常；
4. **特殊字符处理**：包含空格的参数必须用双引号包裹，否则会被拆分为多个参数。

#### 总结

命令行参数通过`main`方法的`String[] args`数组传递，是Java程序与外部交互的重要方式。它的核心价值在于**允许程序在启动时动态接收输入**，无需硬编码配置，使程序更灵活、通用。实际开发中，需结合参数校验和类型转换，确保程序的健壮性。

### Java 可变参数（Varargs）

Java **可变参数**（Variable Arguments，简称 Varargs）是 JDK 5 引入的特性，允许**方法接收任意数量（0个、1个、多个）的同类型参数**，不用手动定义数组传参，简化代码编写。

#### 一、基础语法

```java
// 格式：参数类型... 参数名
方法返回值 方法名(参数类型... 可变参数名) {
    // 方法体
}
```

- 一个方法**最多只能有一个可变参数**
- 可变参数**必须放在方法参数列表的最后一位**

#### 二、简单示例

##### 1. 求任意个整数的和

```java
public class VarargsDemo {
    // 定义可变参数方法：接收任意个int值
    public static int add(int... nums) {
        int sum = 0;
        // 可变参数本质是数组，可以直接遍历
        for (int num : nums) {
            sum += num;
        }
        return sum;
    }

    public static void main(String[] args) {
        // 调用：传0个、1个、多个参数都可以
        System.out.println(add());        // 0个参数 → 输出 0
        System.out.println(add(10));      // 1个参数 → 输出 10
        System.out.println(add(1, 2, 3)); // 3个参数 → 输出 6
    }
}
```

##### 2. 搭配普通参数（必须放最后）

```java
// 正确：可变参数在最后
public static void printInfo(String name, int... scores) {
    System.out.println("姓名：" + name);
    System.out.println("分数：" + Arrays.toString(scores));
}

// 错误！可变参数不能放前面
// public static void printInfo(int... scores, String name) {}

// 调用
printInfo("张三");          // 姓名：张三，分数：[]
printInfo("李四", 90, 85); // 姓名：李四，分数：[90, 85]
```

#### 三、核心本质

**可变参数本质就是一个数组**：

- 编译器会自动把传入的多个参数封装成数组
- 你也可以**直接传数组**给可变参数

```java
int[] arr = {1,2,3};
System.out.println(add(arr)); // 直接传数组，输出 6
```

#### 四、使用规则（必记）

1. **一个方法只能有一个可变参数**
2. **可变参数必须是最后一个参数**
3. 调用时可以传：0个、1个、多个参数、数组
4. 方法内部，把可变参数当**数组**使用（遍历、获取长度等）

#### 五、重载优先级

当方法重载时，调用优先级：
**固定参数方法 > 可变参数方法**

```java
public static void test(int a) {
    System.out.println("固定参数");
}

public static void test(int... a) {
    System.out.println("可变参数");
}

test(10); // 优先调用固定参数方法 → 输出 固定参数
```

#### 六、常见使用场景

Java 很多工具类都用了可变参数：

- `Arrays.asList(T... elements)`：快速创建集合
- `String.format(String format, Object... args)`：字符串格式化

```java
List<String> list = Arrays.asList("Java", "Python", "C++");
String str = String.format("姓名：%s，年龄：%d", "小明", 20);
```

---

#### 总结

1. 可变参数用 `类型... 参数名` 定义，接收**任意数量同类型参数**
2. 本质是**数组**，方法内可直接遍历/操作
3. 只能有一个，且必须放在参数列表**最后**
4. 简化多参数/不定参数方法的编写，不用手动创建数组

### 21.classpath

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

### 25.java包结构

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

### PO、VO、BO、DTO、DAO、POJO

#### **一、6种对象的职责边界**

**对象设计的本质是关注点分离**——每个对象只做一件事，且做好它！

##### **1.1 PO**

它的含义是Persistent Object，即持久化对象。

- **职责**：与数据库表严格1:1映射，**仅承载数据存储结构**

- **特征**：

- - 属性与表字段完全对应
  - 无业务逻辑方法（仅有getter/setter）

- **代码示例**：

```
public class UserPO {  
  private Long id;      // 对应表主键  
  private String name;  // 对应name字段
}  
```

##### **1.2 DAO**

它的含义是Data Access Object，即数据访问对象。

- **职责**：**封装所有数据库操作**（CRUD），隔离业务与存储细节

- **特征**：

- - 接口方法对应SQL操作
  - 返回PO或PO集合

- **代码示例**：

```
public interface UserDao {  
    // 根据ID查询PO  
    UserPO findById(Long id);  
    
    // 分页查询  
    List<UserPO> findPage(@Param("offset") int offset, @Param("limit") int limit);  
}  
```

**底层原理**：DAO模式 = 接口 + 实现类 + PO

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ibJZVicC7nz5gIg2BDY56XC8PPKo6PVgk3wgiaPOdNEIicgqQq8qjLuibKvyPnK0WYL2fgicSEeOibp9ofUb8rv49a2iag/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=wxpic#imgIndex=1)

##### **1.3 BO**

它的含义是Business Object，即业务对象。

- **职责**：**封装核心业务逻辑**，聚合多个PO完成复杂操作

- **特征**：

- - 包含业务状态机、校验规则
  - 可持有多个PO引用

- **代码示例**：订单退款BO

```
public class OrderBO {  
   // 主订单数据  
   private OrderPO orderPO;  
   // 子订单项  
   private List<OrderItemPO> items; 
   
   // 业务方法：执行退款  
   public RefundResult refund(String reason) {     
   if (!"PAID".equals(orderPO.getStatus())) {  
      throw new IllegalStateException("未支付订单不可退款"); 
   }  // 计算退款金额、调用支付网关等  }  
}  
```

##### **1.4 DTO**

它的含义是Data Transfer Object，即数据传输对象。

- **职责**：**跨层/跨服务数据传输**，屏蔽敏感字段

- **特征**：

- - 属性集是PO的子集（如排除`password`字段）
  - 支持序列化（实现`Serializable`）

- **代码示例**：用户信息DTO

```
public class UserDTO implements Serializable {  
    private Long id;  
    private String name;  
}  
```

##### **1.5 VO**

它的含义是View Object，即视图对象。

- **职责**：**适配前端展示**，包含渲染逻辑

- **特征**：

- - 属性可包含格式化数据（如日期转`yyyy-MM-dd`）
  - 聚合多表数据（如订单VO包含用户名字）

- **代码示例**：

```
public class OrderVO {  
  private String orderNo;  
  private String createTime; // 格式化后的日期   private String userName;   // 关联用户表字段    
  
  //状态码转文字描述  
  public String getStatusText() {  
     return OrderStatus.of(this.status).getDesc();  
  }  
}  
```

##### **1.6 POJO**

它的含义是Plain Old Java Object，即普通Java对象。

- **职责**：**基础数据容器**，可扮演PO/DTO/VO角色

- **特征**：

- - 只有属性+getter/setter
  - 无框架依赖（如不继承Spring类）

- **典型实现**：Lombok简化代码

```
// 自动生成getter/setter  
@Data 
public class UserPOJO {  
  private Long id;  
  private String name;  
}  
```

#### **二、主流的对象流转模型**

##### **场景1**

传统三层架构（DAO → DTO → VO）。

**适用系统**：后台管理系统、工具类应用**核心流程**：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ibJZVicC7nz5gIg2BDY56XC8PPKo6PVgk3MblpCTHeEroCsKXibBG5f4UqeTp1EZ4FFt1OlQNU8mzViaK2ySP7I0Ew/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=wxpic#imgIndex=2)

**代码示例**：用户查询服务

```
// Service层  
public UserDTO getUserById(Long id) {  
   UserPO userPO = userDao.findById(id); // 从DAO获取PO  
   UserDTO dto = new UserDTO();  
   dto.setId(userPO.getId());  
   dto.setName(userPO.getName()); // 过滤敏感字段  
   return dto; // 返回DTO  
}  

// Controller层  
public UserVO getUser(Long id) {  
   UserDTO dto = userService.getUserById(id);  
   UserVO vo = new UserVO();  
   vo.setUserId(dto.getId());  
   vo.setUserName(dto.getName());  
   vo.setRegisterTime(formatDate(dto.getCreateTime())); // 格式化日期  
   return vo;  
}  
```

**优点**：简单直接，适合CRUD场景**缺点**：业务逻辑易泄漏到Service层

##### **场景2**

DDD架构（PO → DO → DTO → VO）。

**适用系统**：电商、金融等复杂业务系统**核心流程**：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ibJZVicC7nz5gIg2BDY56XC8PPKo6PVgk3cyYFS5TY72RnEibn8ZxLWr0L2BLsYbEwQUf2Epmq3nn9jaiaibmjc0SFw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=wxpic#imgIndex=3)

**关键角色**：DO（Domain Object）替代BO**代码示例**：订单支付域

```
// Domain层：订单领域对象  
publicclass OrderDO {  
   private OrderPO orderPO;  
   private PaymentPO paymentPO;  
   
   // 业务方法：支付校验  
   public void validatePayment() {  
     if (paymentPO.getAmount() < orderPO.getTotalAmount()) {  
        thrownew PaymentException("支付金额不足");  
      } 
   }  
}  

// App层：协调领域对象  
public OrderPaymentDTO pay(OrderPayCmd cmd) {  
    OrderDO order = orderRepo.findById(cmd.getOrderId());  
    order.validatePayment(); // 调用领域方法  return OrderConverter.toDTO(order); // 转DTO  
}  
```

**优点**：业务高内聚，适合复杂规则系统**缺点**：转换层级多，开发成本高

#### **三、高效转换工具**

手动转换对象？效率低且易错！

##### **3.1  MapStruct：编译期代码生成**

**原理**：APT注解处理器生成转换代码**示例**：PO转DTO

```
@Mapper  
publicinterface UserConverter {  
   UserConverter INSTANCE = Mappers.getMapper(UserConverter.class);  
   
   @Mapping(source = "createTime", target = "registerDate")  
   UserDTO poToDto(UserPO po);  
}  

// 编译后生成UserConverterImpl.java  
publicclass UserConverterImpl {  
   public UserDTO poToDto(UserPO po) {  
      UserDTO dto = new UserDTO();  
      dto.setRegisterDate(po.getCreateTime()); // 自动赋值！  
      return dto;  
  }  
}  
```

**优点**：零反射损耗，性能接近手写代码**开源地址**：https://github.com/mapstruct/mapstruct

##### **3.2 Dozer + Lombok：注解驱动转换**

**组合方案**：

- **Lombok**：自动生成getter/setter
- **Dozer**：XML/注解配置字段映射

```
// Lombok注解
@Data 
public class UserVO {  
   private String userId;  
   private String userName;  
}  

// 转换配置  
<field>  
  <a>userId</a>  
  <b>id</b>  
</field>  
```

**适用场景**：字段名不一致的复杂转换

##### **3.3 手动Builder模式：精细控制**

**适用场景**：需要动态构造的VO

```
public class OrderVOBuilder {  
   public OrderVO build(OrderDTO dto) {  
     return OrderVO.builder()  
        .orderNo(dto.getOrderNo())  
        .amount(dto.getAmount() + "元") // 动态拼接  
        .statusText(convertStatus(dto.getStatus()))  
        .build();  
    }  
}  
```

#### **四、避坑指南**

##### **坑1：PO直接返回给前端**

```
// 致命错误：暴露数据库敏感字段！  
public UserPO getUser(Long id) {  
  // 返回的PO包含password  
  return userDao.findById(id); 
}  
```

**解决方案**：

- 使用DTO过滤字段
- 注解屏蔽：`@JsonIgnore`

##### **坑2：DTO中嵌入业务逻辑**

```
public class OrderDTO {
    // 错误！DTO不应有业务方法  
    public void validate() { 
       if (amount < 0) 
          throw new Exception();  
    }  
}  
```

**本质错误**：混淆DTO与BO的职责

##### **坑3：循环嵌套转换**

```
// OrderVO中嵌套List<ProductVO>  
public class OrderVO {
   // 嵌套对象  
   private List<ProductVO> products; 
}  

// 转换时触发N+1查询  
orderVO.setProducts(order.getProducts()
       .stream()  
       .map(p -> convertToVO(p)) // 循环查询数据库  
       .collect(toList()));  
```

**优化方案**：批量查询 + 并行转换

#### **五、如何选择对象模型？**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/ibJZVicC7nz5gIg2BDY56XC8PPKo6PVgk3U2HvH9LESZ56nHI2uOtdicsW0uGibdBPzJy9s8Odg9hc6JZRuWsPt9rA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=wxpic#imgIndex=4)

#### **总结**

关于对象的4个核心原则：

1. **单一职责**： PO只存数据，BO只管业务，VO只负责展示——**绝不越界！**

2. **安全隔离**：

3. - PO永不出DAO层（防数据库泄露）
   - VO永不出Controller（防前端逻辑污染服务）

4. **性能优先**：

5. - 大对象转换用**MapStruct**（编译期生成代码）
   - 嵌套集合用**批量查询**（杜绝N+1）

6. **适度设计**：

7. - 10张表以内的系统：可用POJO一撸到底
   - 百张表以上核心系统：**必须严格分层**

对象设计没有银弹，**理解业务比套用模式更重要**！

## jar

### JAR 

> JAR（Java Archive）Java归档文件，后缀`.jar`，本质上就是一个 **zip压缩包**，里面存放编译后的`.class`字节码、配置文件、资源文件。
> 作用：把多个class、资源打包成一个文件，方便分发、引入、运行。

#### 1. JAR 的两种类型

##### ① 普通jar（类库jar）

普通依赖包，**不能直接java‑jar运行**，只用来给别的项目做依赖。
里面只有class、配置，没有指定主类。
> 例：`spring‑core.jar`、`mybatis.jar`，Maven/Gradle引入的依赖大多是普通jar。

##### ② 可执行jar（runable jar，可运行jar包）

可以直接命令运行：`java -jar xxx.jar`
**关键条件：META‑INF/MANIFEST.MF 文件里面配置 `Main‑Class: 全限定主类名`**，告诉JVM入口main方法在哪。
> SpringBoot打出来的jar就是可执行jar，但是SpringBoot是特殊的**fat‑jar（胖jar）**。

###### MANIFEST.MF（清单文件，META‑INF目录下）

是jar包的元信息文件，记录版本、主类、类路径等。
```manifest
Manifest-Version: 1.0
Main-Class: com.demo.Main  # 可执行jar必须配置，指定main入口
Class-Path: lib/a.jar lib/b.jar # 外部依赖jar路径
```

> 普通jar没有Main‑Class，直接`java -jar`会报错：`no main manifest attribute`。

#### 2. 常用命令：jdk自带jar命令

> jdk bin目录下的jar.exe，用来打包、查看、解压jar。

```shell
# 1.查看jar包内容，列出文件
jar -tvf demo.jar

# 2.解压jar到当前目录
jar -xvf demo.jar

# 3.打包普通jar
jar -cvf demo.jar ./bin/

# 4.打包可执行jar，指定主类
jar -cvfe run.jar com.demo.Main ./bin/
# e：指定入口主类
```

参数记忆：
- `c` create 创建jar
- `t` table 列表查看
- `x` extract 解压
- `f` file 指定jar文件名
- `e` entry 指定main入口类

#### 3. 什么是胖Jar(Fat‑Jar / Uber‑Jar)

> **普通jar：只有自己项目的class，第三方依赖jar不在里面。**
> **FatJar胖jar：把自己代码 + 所有第三方依赖全部打进一个jar包。**

SpringBoot Maven插件`spring‑boot‑maven‑plugin`打包出来就是胖jar。
打开SpringBoot jar包结构：
```
demo‑spring.jar
├─BOOT‑INF/
│   ├─classes/      # 自己项目编译后的class
│   └─lib/          # 所有第三方依赖jar全部放这里
├─META‑INF/
│   └─MANIFEST.MF   # 真正启动类是 org.springframework.boot.loader.JarLauncher
└─org/              # springboot loader相关class
```
> SpringBoot的MANIFEST.MF里的Main‑Class不是你写的业务main，而是`JarLauncher`；
> `Start‑Class`才是业务项目的main类。
> JarLauncher内部自定义类加载器去加载BOOT‑INF/lib下的一堆依赖jar。

> 普通JDK的`jar`命令做不到胖jar，需要插件（springboot插件、maven‑shade、maven‑assembly）。

##### 三个打包插件区别（面试高频）

1. **maven‑assembly‑plugin**：把依赖解压，把所有class全部打进同一个jar，类直接平铺。容易发生类冲突。
2. **maven‑shade‑plugin**：可以重命名包名，解决jar包类冲突；也可以打胖jar。
3. **spring‑boot‑maven‑plugin**：SpringBoot专用，生成SpringBoot专属FatJar，使用嵌套jar +自定义类加载器。

> 嵌套jar：jar里面还放一堆jar文件，JDK原生ClassLoader**不支持读取jar里面嵌套的jar**，所以SpringBoot自己实现JarClassLoader。

#### 4. jar包运行相关命令

```shell
# 运行可执行jar
java -jar app.jar

# 指定配置文件运行
java -jar app.jar --spring.profiles.active=prod

# 指定jvm参数，-JVM参数要写在‑jar前面！！
java -Xms256m -Xmx512m -jar app.jar

# 普通jar，没有Main‑Class，指定主类运行（需要把jar加入classpath）
java -cp demo.jar com.demo.Main
```

> 坑：JVM参数写在jar后面会变成程序入参，不是JVM参数。
> ❌错误：`java -jar app.jar -Xmx512m`
> ✅正确：`java -Xmx512m -jar app.jar`

#### 5. classpath（类路径）概念

JVM去哪些位置找class文件，这个路径集合就是classpath。
- `-cp / -classpath`：指定类路径，可以是目录、jar包；多个用分号`;`(windows)，冒号`:`(linux)分隔。
```shell
# linux
java -cp a.jar:b.jar com.demo.Main
# windows
java -cp a.jar;b.jar com.demo.Main
```
> 使用`java -jar`的时候，会忽略 `-cp` 参数，classpath由jar包MANIFEST.MF的Class‑Path或者SpringBoot自定义加载器接管。

#### 6. jar包常见问题&面试坑

##### 坑1：`no main manifest attribute, in xxx.jar`

原因：jar不是可执行jar，MANIFEST.MF没有Main‑Class。
解决：
1. 如果普通jar，不用`‑jar`，改用`java -cp xxx.jar 全类名`运行；
2. 打包时配置插件指定主类。

##### 坑2：SpringBoot jar里面BOOT‑INF/lib下嵌套jar，原生JDK无法直接读取

原生ClassLoader只能读取磁盘上的jar，**不支持jar包内部嵌套jar**，SpringBoot靠自定义`JarClassLoader`实现。

##### 坑3：jar包版本冲突

同一个类来自多个不同版本jar，出现`NoClassDefFoundError`、`ClassCastException`、`LinkageError`。Maven靠dependencyExclusion排除冲突jar。

##### 坑4：资源文件读取，打包jar后路径读取失败

开发时文件在resources目录，IDEA能读到；打成jar后，文件被压缩进jar内部，不能用File读取，要用类加载器`getResourceAsStream()`读取。
```java
// jar包内资源，不能new File，要用流
InputStream is = Demo.class.getClassLoader().getResourceAsStream("application.yml");
```

##### 坑5：jar包和jdk版本不匹配

编译使用高版本JDK，运行环境JDK版本低，报 `UnsupportedClassVersionError`。
class文件主版本号大于运行JDK支持版本。

#### 7. jar、war、ear对比

|包|后缀|用途|运行环境|
|---|---|---|---|
|jar|`.jar`|普通类库、SpringBoot可执行程序|内置tomcat，java‑jar直接运行|
|war|`.war`|传统web项目，放jsp、servlet|外部tomcat容器解压运行|
|ear|`.ear`|企业级，包含多个jar+war|JavaEE应用服务器(JBoss等)，现在几乎淘汰|

> SpringBoot项目可以打war包，放到外部tomcat；也可以打jar内置tomcat直接运行。

#### 8. 简单背诵总结

1. jar本质zip压缩包，存放class字节码、资源文件；分普通jar、可执行jar。
2. 可执行jar依靠META‑INF/MANIFEST.MF中`Main‑Class`指定入口。
3. Fat‑Jar胖jar：项目代码+所有第三方依赖全部打进一个包；SpringBoot jar属于胖jar，采用嵌套jar+自定义类加载器。
4. `java -jar`运行时，JVM参数必须写在‑jar前面；会忽略‑cp参数。
5. JDK原生ClassLoader不支持jar嵌套jar；jar内资源不能File读取，要用getResourceAsStream。
6. jar/war/ear区别；maven三个打包插件的用途。

### JAR 使用场景 + Maven 配置

> JAR：Java归档包，本质zip，存放class、配置、资源。
> Maven核心作用：**管理jar依赖、下载jar、打包生成jar/war、管理版本冲突**。

#### 一、Jar 的4大使用场景

##### 场景1：作为依赖库，供其他项目引用（最常见）

自己写一个工具项目，打成普通jar，别的项目引入这个jar，复用代码。

- 特点：普通jar，**没有Main‑Class，不能java‑jar直接运行**。
- 例子：commons‑lang3、mybatis、hutool工具包。

流程：

1. A工具项目 → Maven打包生成jar
2. 将jar安装到本地maven仓库 / 上传到私服Nexus
3. B项目pom.xml引入dependency，直接使用A的类。

> 
> 没有maven的老方式：手动复制jar到项目lib目录，手动构建classpath，版本管理麻烦。

##### 场景2：可执行Jar，独立运行Java应用（SpringBoot主流）

把业务代码+依赖打包，服务器直接`java -jar xxx.jar`启动，不需要额外tomcat。
SpringBoot项目就是这个场景，打**Fat‑Jar胖jar**（内置所有依赖、内置tomcat）。

##### 场景3：本地命令行工具，普通Java程序

简单main程序，打成可执行jar，服务器执行脚本运行。

> 
> 注意：原生JDK jar命令只能打简单可执行jar，不会把第三方依赖打进去。有第三方依赖一般用maven‑shade。

##### 场景4：Jar包作为模块，多模块Maven项目

Maven多模块项目：父工程 + 多个子module。

- 公共模块打成jar，其他子模块直接依赖这个子module，实现模块拆分。> 
> 微服务项目经常这样：common模块打成jar，所有服务模块引用common。

---

#### 二、Maven中与Jar相关核心配置

##### 1. 项目打包方式 `packaging`

放在pom.xml的project下：

```xml
<!-- 默认，不写就是jar -->
<packaging>jar</packaging>

<!-- web传统项目打war -->
<!-- <packaging>war</packaging> -->
```

- `jar`：打包输出jar包；
- `war`：输出war包；
- `pom`：父工程，只做版本管理，**不输出jar**。

##### 2. 引入外部Jar依赖 dependency

从maven中央仓库/私服下载jar到本地仓库，加入项目classpath。

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

###### scope（依赖范围，控制jar生效的生命周期）

| scope | 作用 |
| --- | --- |
| `compile`（默认） | 编译、测试、运行都有效；打包进jar。 |
| `test` | 仅测试生效；打包不会打入jar，如junit。 |
| `provided` | 编译测试有效，运行时由容器提供，**打包不打进jar**，如servlet-api。 |
| `runtime` | 运行、测试有效，编译不需要；打包进jar，如mysql驱动。 |
| `system` | 本地磁盘jar，不推荐，不走maven仓库。 |

> 
> provided非常重要：SpringBoot内置tomcat，servlet‑api是provided，不会打进最终jar。

##### 3. Maven核心命令（操作jar）

```xml
# 编译，生成class，不打包
mvn compile

# 打包，target目录生成jar/war
mvn package

# 将当前项目jar安装到本地maven仓库，供本机其他项目依赖
mvn install

# 部署，把jar上传到nexus私服，给团队其他人使用
mvn deploy

# clean 删除target目录
mvn clean package
```

> 
> `mvn install`：不仅打包，还复制到本地仓库。多模块项目经常使用。

#### 三、三种Maven打包插件，对应不同Jar场景

> 
> maven本身不会自动打胖jar，需要插件。

##### ① spring‑boot‑maven‑plugin 【SpringBoot项目首选】

场景：SpringBoot应用，打**SpringBoot专属Fat‑Jar**，可`java‑jar`运行。
pom配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

- 效果：生成BOOT‑INF结构的嵌套jar，内置所有依赖；MANIFEST.MF自动配置JarLauncher与业务Start‑Class。
- 使用：`mvn clean package`，target得到xxx.jar，直接`java -jar`。

##### ② maven‑shade‑plugin 打普通Java胖Jar（非SpringBoot）

场景：普通Java项目，main入口，需要把第三方依赖全部打进一个jar，服务器直接java‑jar运行。

> 
> 可以配置relocation重定位包名，解决jar包类冲突。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <!-- 指定main入口类，生成可执行jar -->
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.demo.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

##### ③ maven‑assembly‑plugin

场景：打包输出jar+脚本、配置文件、lib目录；打胖jar。
缺点：直接解压所有依赖class平铺，容易出现类冲突。多用于脚本分发包。

#### 四、Maven多模块场景（公共模块输出Jar）

结构示例

```
demo-parent (packaging=pom)
├─demo‑common  (packaging=jar) 公共工具模块
├─demo‑service (packaging=jar) SpringBoot业务模块
```

父pom：

```xml
<packaging>pom</packaging>
<modules>
    <module>demo-common</module>
    <module>demo-service</module>
</modules>
```

demo‑service模块依赖内部子模块demo‑common：

```xml
<dependency>
    <groupId>com.demo</groupId>
    <artifactId>demo-common</artifactId>
    <version>${project.version}</version>
</dependency>
```

执行`mvn clean install`：先编译打包common，安装到本地仓库，service就可以引用common的jar。

> 
> ⚠️注意：子模块要被别的模块依赖，必须执行install；只package只是生成target下jar，不会放到本地仓库，其他模块找不到。

#### 五、如何引入本地磁盘上的第三方jar（不进仓库，不推荐）

方式1：scope=system，不推荐，团队协作麻烦。

```xml
<dependency>
    <groupId>com.xxx</groupId>
    <artifactId>xxx</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/demo.jar</systemPath>
</dependency>
```

> 
> system scope的jar，默认不会打入最终jar，需要插件额外配置。
> 最佳实践：把jar上传私服nexus，正常dependency引入。

#### 六、面试高频问题

##### Q1：mvn package 和 mvn install区别

- package：只在当前项目target生成jar；不会放到本地maven仓库。
- install：执行package之后，把jar复制到本地maven仓库，本机其他maven项目可以依赖这个jar。

##### Q2：SpringBoot插件repackage做了什么？

先打出原始jar，再重新打包repackage，生成fat‑jar，把所有依赖放入BOOT‑INF/lib，修改MANIFEST.MF。

##### Q3：provided的依赖会不会打进最终jar？

不会。编译有效，打包排除。

##### Q4：shade插件和spring‑boot‑maven‑plugin区别

- shade：把所有依赖解压，class平铺到jar；适合普通Java main程序。
- springboot插件：嵌套jar，依赖仍然以jar形式放在BOOT‑INF/lib，自定义类加载器加载，SpringBoot专用。

##### Q5：多模块项目，子模块A依赖子模块B，打包报错找不到B？

原因：没有执行mvn install，B的jar没有安装到本地仓库。解决：根目录执行`mvn clean install`。

#### 七、场景速查表

| 业务场景 | packaging | 使用插件 | 运行方式 |
| --- | --- | --- | --- |
| 工具类项目，给别人依赖 | jar | 不需要特殊插件 | 作为dependency引入，不直接运行 |
| SpringBoot微服务 | jar | spring‑boot‑maven‑plugin | java‑jar xxx.jar |
| 普通Java main程序，带第三方依赖 | jar | maven‑shade‑plugin | java‑jar xxx.jar |
| 传统SSM web项目 | war | maven‑war‑plugin | 外部tomcat部署 |
| Maven父工程管理版本 | pom | 无 | 不输出jar |

#### 核心小结

1. jar主要场景：依赖库、可执行应用、命令行工具、多模块公共组件。
2. maven通过dependency引入jar；scope控制jar在哪个生命周期生效。
3. package生成target下jar；install把jar放入本地仓库；deploy上传私服。
4. SpringBoot用spring‑boot‑maven‑plugin；普通Java胖jar用maven‑shade。
5. 多模块公共模块要install，其他模块才能引用。

接下来可以讲Maven核心：依赖传递、依赖冲突、exclusion。

## 本地缓存

本地缓存（Local Cache）是把热点数据存在**当前应用进程内存**里的缓存，核心优势是**纳秒/微秒级访问、无网络IO、低延迟**，常用作**一级缓存（L1）** 配合 Redis 组成多级缓存。

---

### 一、核心特点（一眼看懂）

- ✅ 速度极快：内存直接读写，**μs/ns 级**；Redis 约 **ms 级**，DB 更慢。
- ✅ 无网络开销：进程内访问，不依赖外部服务。
- ✅ 容量有限：受 **JVM 堆/堆外内存**限制，不适合大数据量。
- ✅ 单机隔离：多实例间**不共享**，存在数据不一致风险。
- ✅ 重启丢失：默认**不持久化**（Ehcache 可配置磁盘）。

### 二、适用场景

- 读多写少、更新低频：系统配置、字典表、城市列表、基础资料。
- 极致性能链路：高并发热点（如商品详情、用户信息）。
- 多级缓存 L1：Caffeine → Redis → DB，**减轻 Redis 压力、降低延迟**。
- 防击穿/防雪崩：热点 Key 用本地缓存挡流量。

### 三、Java 主流实现对比

#### 1）ConcurrentHashMap（最简自研）

- 优点：JDK 原生、零依赖、线程安全。
- 缺点：**无过期、无淘汰、无统计**，需自行封装。
- 适用：简单小缓存、临时数据。

#### 2）Guava Cache（经典稳定）

- 特性：LRU 淘汰、写入/访问过期、最大容量、移除监听、命中率统计。
- 缺点：性能弱于 Caffeine，维护放缓。

#### 3）Caffeine（当前最优，推荐）

- 核心：**W-TinyLFU 淘汰算法**，命中率比 LRU 高 **10%–20%**。
- 特性：
  - 自动加载（LoadingCache）、异步刷新
  - 写入/访问过期、最大容量、权重控制
  - 堆内/堆外内存、统计监控
- 现状：Spring Boot 默认集成，高并发首选。

#### 4）Ehcache（全功能老牌）

- 特性：内存+磁盘持久化、堆外内存、集群、事务、过期/淘汰。
- 缺点：较重、配置复杂、性能低于 Caffeine。

| 方案              | 性能  | 过期 | 淘汰      | 持久化 | 推荐指数 |
| ----------------- | ----- | ---- | --------- | ------ | -------- |
| ConcurrentHashMap | ★★★★★ | ❌    | ❌         | ❌      | ★★☆      |
| Guava Cache       | ★★★★  | ✅    | LRU       | ❌      | ★★★☆     |
| Caffeine          | ★★★★★ | ✅    | W-TinyLFU | ❌      | ★★★★★    |
| Ehcache           | ★★★   | ✅    | 多种      | ✅      | ★★★      |

### 四、Caffeine 快速示例（Java）

#### 1）依赖（Maven）

```xml
<dependency>
    <groupId>com.github.benmanes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>
```

#### 2）基础用法（手动 put/get）

```java
Cache<Long, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000) // 最大1万条
    .expireAfterWrite(5, TimeUnit.MINUTES) // 写入5分钟过期
    .expireAfterAccess(2, TimeUnit.MINUTES) // 访问后2分钟过期
    .recordStats() // 开启统计（命中率等）
    .build();

// 存
cache.put(1L, "张三");
// 取（不存在返回 null）
String name = cache.getIfPresent(1L);
```

#### 3）自动加载（LoadingCache，常用）

```java
LoadingCache<Long, String> loadingCache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(5, TimeUnit.MINUTES)
    .refreshAfterWrite(1, TimeUnit.MINUTES) // 写入1分钟后异步刷新
    .build(id -> loadFromDb(id)); // 缓存未命中时执行

// 调用（自动加载）
String name = loadingCache.get(1L);

// 模拟DB查询
private static String loadFromDb(Long id) {
    return "用户" + id;
}
```

### 五、多级缓存架构（推荐）

**请求 → Caffeine（L1）→ Redis（L2）→ DB（L3）**

- L1：本地缓存，**μs 级**，命中率约 **80%**。
- L2：Redis，**ms 级**，分布式共享，命中率约 **15%**。
- L3：DB，持久化，命中率约 **5%**。

### 六、常见问题与解决

1. **数据不一致（多实例）**
   - 方案：本地缓存**短 TTL（1–5min）**；更新时**主动删除本地缓存+Redis**；或用消息队列广播失效。

2. **内存溢出（OOM）**
   - 方案：**严格设置 maximumSize**；用**软/弱引用**；监控内存与缓存大小。

3. **缓存穿透**
   - 方案：本地缓存**空值/默认值**；**布隆过滤器**前置拦截。

### 七、总结

- 本地缓存是**高并发高性能的利器**，优先选 **Caffeine**。
- 生产环境必做：**多级缓存（Caffeine+Redis）、合理 TTL、内存保护、一致性策略、监控告警**。

需要我给你一份可直接复制的 **Caffeine+Redis 多级缓存工具类（含过期删除、异步刷新、空值防穿透）** 吗？

### **Caffeine** 

**Caffeine** 是目前 **Java 生态性能最强、命中率最高**的本地缓存库，是 Guava Cache 的全面升级版，也是 Spring Boot 2.x+ 默认本地缓存实现。

---

#### 一、核心定位与优势

- **定位**：高性能、低延迟、高命中率的**进程内内存缓存**（JVM 本地缓存）
- **核心优势**
  1. **性能极致**：读性能接近 ConcurrentHashMap，写性能远超 Guava Cache（8~10 倍）
  2. **命中率最高**：自研 **W-TinyLFU** 淘汰算法，比 LRU/LFU 命中率高 10%~15%
  3. **功能全面**：过期、刷新、异步、引用回收、统计、监听器、权重控制
  4. **轻量无依赖**：核心包小、无第三方依赖，兼容 Java 8+

---

#### 二、核心原理：W-TinyLFU 算法（关键）

Caffeine 性能与命中率的核心是 **Window-TinyLFU** 双层淘汰机制：

1. **Window Cache（窗口区，约 1% 容量）**
   - 纯 LRU 策略
   - 接纳**新数据**，解决 LFU 冷启动问题（新热点不会刚进就被淘汰）

2. **Main Cache（主缓存区，99% 容量）**
   - TinyLFU 策略
   - 用 **Count-Min Sketch** 极低内存开销统计访问频率
   - 保留**长期高频**数据，淘汰低频数据

**解决两大经典问题**：

- **LRU 缺点**：避免冷数据扫描污染缓存、挤掉热点
- **LFU 缺点**：解决新热点冷启动、历史高频数据僵化

---

#### 三、Maven 依赖

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.8</version>
</dependency>
```

---

#### 四、三种核心使用方式

##### 1. 手动缓存（Manual Cache）

显式 put/get，适合主动控制缓存：

```java
Cache<String, String> cache = Caffeine.newBuilder()
    .initialCapacity(100)       // 初始容量
    .maximumSize(10_000)       // 最大条目数
    .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后过期
    .expireAfterAccess(5, TimeUnit.MINUTES) // 最后访问后过期
    .recordStats()             // 开启统计
    .build();

// 存
cache.put("key", "value");

// 取（不存在返回 null）
String val = cache.getIfPresent("key");

// 取（不存在则自动加载）
String val2 = cache.get("key2", k -> loadFromDB(k));

// 删
cache.invalidate("key");
```

##### 2. 自动加载缓存（LoadingCache）

缓存未命中时**自动加载**，适合 DB/接口查询缓存：

```java
LoadingCache<String, User> userCache = Caffeine.newBuilder()
    .maximumSize(1000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> userDao.findById(key)); // 加载函数

// 自动加载：不存在 → 调用 load → 缓存
User user = userCache.get("123");
```

##### 3. 异步加载缓存（AsyncLoadingCache）

I/O 密集场景，**异步加载不阻塞主线程**：

```java
AsyncLoadingCache<String, User> asyncCache = Caffeine.newBuilder()
    .buildAsync(key -> CompletableFuture.supplyAsync(() -> 
        userDao.findById(key)
    ));

// 返回 CompletableFuture
CompletableFuture<User> future = asyncCache.get("123");
User user = future.get();
```

---

#### 五、核心配置详解

##### 1. 容量控制

- `maximumSize(long)`：最大条目数
- `maximumWeight(long)` + `weigher((k,v)->weight)`：按**权重**（如内存大小）限制

##### 2. 过期策略

- `expireAfterWrite(duration)`：写入后固定过期
- `expireAfterAccess(duration)`：最后访问后过期
- `expireAfter(new Expiry<>())`：**自定义过期**（每个 key 独立过期时间）

##### 3. 自动刷新

```java
.refreshAfterWrite(5, TimeUnit.MINUTES)
```

- 后台异步刷新，**不阻塞读请求**
- 旧值可用时继续返回旧值

##### 4. 引用回收（防 OOM）

- `weakKeys()`：键用弱引用（GC 时自动回收）
- `weakValues()`：值弱引用
- `softValues()`：值软引用（内存不足时 GC 回收）

##### 5. 统计与监控

```java
.recordStats() // 开启
CacheStats stats = cache.stats();
stats.hitRate();   // 命中率
stats.missRate();  // 未命中率
stats.evictionCount(); // 淘汰次数
```

---

#### 六、与 Guava Cache / ConcurrentHashMap 对比

| 特性        | Caffeine      | Guava Cache | ConcurrentHashMap |
| ----------- | ------------- | ----------- | ----------------- |
| 淘汰算法    | **W-TinyLFU** | LRU         | 无淘汰            |
| 命中率      | **最高**      | 中          | 无                |
| 过期策略    | 全支持        | 有限        | 无                |
| 异步加载    | ✅             | ❌           | ❌                 |
| 性能        | **最高**      | 中          | 高（无淘汰）      |
| Spring 默认 | ✅ 2.x+        | ❌           | ❌                 |

---

#### 七、Spring Boot 集成（最常用）

1. 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

2. 开启缓存

```java
@EnableCaching
@SpringBootApplication
public class App {}
```

3. 注解使用

```java
@Cacheable(value = "userCache", key = "#id")
public User getUserById(Long id) {
    return userDao.findById(id);
}

@CacheEvict(value = "userCache", key = "#user.id")
public void updateUser(User user) {}
```

---

#### 八、最佳实践

- **读多写少、热点数据**：用 Caffeine
- **容量控制**：必设 `maximumSize`/`maximumWeight`
- **过期**：写多读少用 `expireAfterWrite`；读多写少用 `expireAfterAccess`
- **防雪崩**：配合 `refreshAfterWrite` + 随机过期
- **监控**：生产必开 `recordStats()` 监控命中率

---

#### 九、一句话总结

**Caffeine = 高性能 + 最高命中率 + 全功能**，是 Java 本地缓存**首选方案**。

要不要我给你一份可直接复制的 **Caffeine 生产级配置模板**（含过期、刷新、权重、统计、防 OOM）？

## 文件路径

### 1.三种路径

非常好的问题！理解 **当前工作目录（Current Working Directory）**、**项目根目录（Project Root）** 和 **类路径（Classpath）** 的区别，是 Java 开发中避免“文件找不到”错误的关键。下面为你详细解析三者的定义、区别、获取方式及使用场景。

------

#### 一、三者定义与本质区别

| 概念                    | 英文                      | 本质                                                         | 作用                                         |
| ----------------------- | ------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| **当前工作目录**        | Current Working Directory | **操作系统概念**：JVM 启动时所在的文件系统目录               | 相对路径（如 `new File("a.txt")`）的基准目录 |
| **类路径（Classpath）** | Classpath                 | **JVM 概念**：JVM 查找 `.class` 文件和资源（resources）的路径集合 | 加载类、读取 `src/main/resources` 下的资源   |
| **项目根目录**          | Project Root Directory    | **开发约定**：包含 `src/`、`pom.xml` 等的源码顶层目录        | 仅在开发阶段有意义，运行时无直接对应         |

> ✅ **关键区别**：
>
> - **工作目录 = 文件系统的起点**
>
> - **类路径 = class 和资源的查找路径**
>
> - **项目根目录 ≠ 工作目录 ≠ 类路径**（尽管开发时可能重合）
>
> - **“项目根目录”是你写代码时的家，“当前工作目录”是程序运行时站的地方——在家时两者一样，出门后就不一定了。**
>
>   如果你只是本地开发测试，放心用；如果要上线，请做好路径解耦 😊



#### **“当前工作目录”和“项目根目录”的区别**

**在概念上是两回事，但在某些开发场景下它们恰好相同。**

------

##### ✅ 简短回答：

> **不是一回事。**  
>
> - **当前工作目录（Current Working Directory）**：是 **操作系统层面** 的概念，指 JVM 启动时所在的目录。  
> - **项目根目录（Project Root Directory）**：是 **开发者/构建工具约定** 的概念，指包含 `src/`、`pom.xml` 等的源码顶层目录。

它们**可能重合，但不一定总是相等**。

------

##### 🔍 详细解释

###### 1. **当前工作目录（`user.dir`）**

- 由 **启动 Java 进程的位置** 决定。

- 可通过 `System.getProperty("user.dir")` 获取。

- 示例：

  ```bash
  # 在 /home/user 下运行
  cd /home/user
  java -jar /opt/myapp/app.jar
  ```

  → 此时 

  ```
  user.dir = /home/user
  ```

  ，

  即使 JAR 文件在 `/opt/myapp/`

###### 2. **项目根目录**

- 是你写代码时的工程结构顶层，比如：

  ```
  ~/projects/my-spring-app/   ← 这就是项目根目录
    ├── src/
    ├── pom.xml
    └── 中二知识笔记.pdf
  ```

- 它只存在于 **开发阶段** 或 **源码仓库中**。

- 一旦项目打包成 JAR/WAR，**“项目根目录”这个概念在运行时就消失了**（只剩文件系统路径或 classpath）。

------

##### 🆚 对比示例

| 场景                                           | 当前工作目录 (`user.dir`)       | 项目根目录                 | 是否相同？ |
| ---------------------------------------------- | ------------------------------- | -------------------------- | ---------- |
| 在 IDEA 中运行 main 方法                       | `~/projects/my-spring-app`      | `~/projects/my-spring-app` | ✅ 是       |
| 在项目根目录执行 `mvn spring-boot:run`         | `~/projects/my-spring-app`      | `~/projects/my-spring-app` | ✅ 是       |
| 打包后，在任意位置运行： `java -jar myapp.jar` | `/home/ops`（你执行命令的地方） | （已不存在）               | ❌ 否       |
| 用 systemd 服务启动 Java 应用                  | `/` 或 `/root`（默认）          | —                          | ❌ 否       |
| Docker 容器中运行（未指定 WORKDIR）            | `/`                             | —                          | ❌ 否       |

------

##### 💡 关键结论

- **开发阶段（IDE / 命令行在项目目录下运行）**：
  两者通常一致，所以 `new FileSystemResource("xxx.pdf")` 能工作。
- **部署/生产环境**：
  “项目根目录”不再存在，只有“当前工作目录”，而它可能是任意位置。此时硬编码相对路径会失败。

------

##### ✅ 最佳实践建议

1. **不要假设 `user.dir` 就是项目根目录**，尤其在生产代码中。

2. 如果需要访问外部文件：

   - 通过配置指定绝对路径（如 `app.document.path=/data/docs/xxx.pdf`）

   - 或明确基于 

     ```
     user.dir
     ```

      构建路径并加校验：

     ```java
     Path file = Paths.get(System.getProperty("user.dir")).resolve("中二知识笔记.pdf");
     if (!Files.exists(file)) {
         // 报错或使用默认路径
     }
     ```

3. **资源文件（如配置、小图片）应放 `src/main/resources`，用 `ClassPathResource` 加载**，与工作目录无关。

------

##### 🎯 一句话总结：

> **“项目根目录”是你写代码时的家，“当前工作目录”是程序运行时站的地方——在家时两者一样，出门后就不一定了。**

如果你只是本地开发测试，放心用；如果要上线，请做好路径解耦 😊

#### 二、如何获取它们？

##### 1. 获取 **当前工作目录**

```java
String workingDir = System.getProperty("user.dir");
System.out.println("当前工作目录: " + workingDir);
```

- 返回 JVM 启动时的目录。
- 示例输出（Linux/macOS）：`/home/user/my-project`
  （Windows）：`C:\Users\user\my-project`

> 📌 所有 **相对路径文件操作**（如 `new File("log.txt")`）都基于此目录。

------

##### 2. 获取 **类路径（Classpath）**

类路径是一个**路径列表**，可通过以下方式查看：

###### 方法 A：打印整个 classpath

```java
String classpath = System.getProperty("java.class.path");
System.out.println("类路径: " + classpath);
```

- 输出示例（Maven 项目）：

  ```
  /home/user/my-project/target/classes:
  /home/user/.m2/repository/org/springframework/spring-core/5.3.0/spring-core-5.3.0.jar:...
  ```

###### 方法 B：获取某个类或资源的实际位置

```java
// 获取当前类的 .class 文件位置
URL classUrl = MyClass.class.getProtectionDomain().getCodeSource().getLocation();
System.out.println("类所在位置: " + classUrl);

// 获取 classpath 下某资源的位置
URL resourceUrl = MyClass.class.getClassLoader().getResource("application.properties");
System.out.println("资源位置: " + resourceUrl);
```

> ⚠️ 注意：如果代码被打包进 JAR，`classUrl` 可能是 `file:/app.jar`，而资源 URL 可能是 `jar:file:/app.jar!/application.properties`。

------

##### 3. “获取项目根目录”？—— 它没有直接 API！

因为 **项目根目录是开发概念**，JVM 运行时并不知道它。

但你可以**间接推测**（仅限开发阶段）：

```java
// 假设你在 target/classes 下运行（Maven 默认）
URL classesUrl = Thread.currentThread().getContextClassLoader().getResource(".");
if (classesUrl != null) {
    try {
        Path classesDir = Paths.get(classesUrl.toURI()); // .../target/classes
        Path projectRoot = classesDir.getParent().getParent(); // .../my-project
        System.out.println("推测的项目根目录: " + projectRoot);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

> ❗ 此方法在 JAR 中失效（因为 `getResource(".")` 在 JAR 内返回 `jar:...!/.`，无法转为文件路径）。

✅ **更可靠的做法**：通过配置或 `user.dir`（见下文）。

------

#### 三、典型使用场景对比

| 需求                                           | 应该用                     | 示例                                              |
| ---------------------------------------------- | -------------------------- | ------------------------------------------------- |
| 读取 `src/main/resources/config.json`          | **类路径**                 | `new ClassPathResource("config.json")`            |
| 读取项目根目录下的 `data/input.csv`            | **当前工作目录**（开发时） | `new FileSystemResource("data/input.csv")`        |
| 写日志到 `logs/app.log`（与启动脚本同目录）    | **当前工作目录**           | `new FileOutputStream("logs/app.log")`            |
| 加载外部配置文件（如 `/etc/myapp/config.yml`） | **绝对路径**               | `new FileSystemResource("/etc/myapp/config.yml")` |

------

#### 四、常见误区与陷阱

##### ❌ 误区 1：认为 `getClass().getResource("/")` 是项目根目录

```java
// 错误！这返回的是 classpath 根（如 target/classes），不是项目根目录
URL url = getClass().getResource("/");
```

- 在 Maven 项目中，它指向 `target/classes/`，比项目根目录深两层。

##### ❌ 误区 2：用 `ClassPathResource` 读项目根目录的文件

```java
// 如果 "readme.txt" 在项目根目录（不在 resources/），这会失败！
new ClassPathResource("readme.txt"); // 找不到
```

→ 只有放在 `src/main/resources/` 的文件才会进入 classpath。

##### ❌ 误区 3：假设打包后还能访问项目结构

- JAR/WAR 包只包含编译后的 class 和 resources，**不包含 `pom.xml`、`src/`、项目根目录下的普通文件**。

------

#### 五、最佳实践建议

| 场景                         | 推荐做法                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| **开发阶段读项目根目录文件** | 确保文件存在 + 使用 `user.dir` + 加存在校验                  |
| **生产环境读外部文件**       | 通过配置指定**绝对路径**（如 `-Dconfig.path=/opt/app/data.cfg`） |
| **读应用配置/模板等资源**    | 放入 `src/main/resources`，用 `ClassPathResource`            |
| **不确定路径？**             | 打印 `user.dir` 和 `java.class.path` 调试                    |

------

#### 六、总结表

| 特性               | 当前工作目录                     | 类路径（Classpath）                     | 项目根目录                 |
| ------------------ | -------------------------------- | --------------------------------------- | -------------------------- |
| **本质**           | OS 概念                          | JVM 概念                                | 开发约定                   |
| **获取方式**       | `System.getProperty("user.dir")` | `System.getProperty("java.class.path")` | 无直接方式（需推测或配置） |
| **相对路径基准**   | 是                               | 否（classpath 资源用自己规则）          | 否                         |
| **打包后是否有效** | 取决于启动位置                   | 有效（JAR 内部）                        | 无效                       |
| **典型用途**       | 读写外部文件、日志               | 加载类、读 resources 资源               | 仅用于开发组织代码         |

------

如果你告诉我你的具体需求（比如：“我想在 Spring Boot 里读一个放在项目根目录的 PDF 文件”），我可以给出最合适的代码方案！

### 2.绝对路径和相对路径

在 Java（以及所有操作系统和编程语言）中，**绝对路径**和**相对路径**是文件系统操作的基础概念。理解它们对正确读写文件、避免 `FileNotFoundException` 至关重要。

------

#### 一、定义

##### ✅ 绝对路径（Absolute Path）

- **从文件系统的根目录开始**，完整描述文件或目录的位置。

- **不依赖当前工作目录**，在任何地方运行都指向同一个位置。

- 示例

  ：

  - Windows:  

    ```text
    C:\Users\Alice\Documents\notes.txt
    D:\myapp\config\app.conf
    ```

  - Linux / macOS:  

    ```text
    /home/alice/documents/notes.txt
    /opt/myapp/logs/app.log
    ```

> 🔑 特点：以盘符（Windows）或 `/`（Unix-like）开头。

------

##### ✅ 相对路径（Relative Path）

- **相对于“当前工作目录”（Current Working Directory）** 的路径。

- **会随着程序启动位置不同而指向不同文件**。

- 示例

  （假设当前工作目录是 

  ```
  /home/alice/myproject
  ```

  ）：

  ```text
  notes.txt           → /home/alice/myproject/notes.txt
  data/input.csv      → /home/alice/myproject/data/input.csv
  ../shared/utils.jar → /home/alice/shared/utils.jar
  ./config/app.conf   → /home/alice/myproject/config/app.conf
  ```

> 🔑 特点：不以 `/` 或盘符开头；常用 `.`（当前目录）、`..`（上级目录）。

------

#### 二、在 Java 中如何使用？

##### 1. 使用 `java.io.File`

```java
// 相对路径：相对于 user.dir
File relative = new File("data.txt");

// 绝对路径
File absolute = new File("/home/alice/data.txt");        // Linux/macOS
File absoluteWin = new File("C:\\Users\\Alice\\data.txt"); // Windows
```

##### 2. 使用 `java.nio.file.Path`（推荐）

```java
Path relative = Paths.get("data.txt");
Path absolute = Paths.get("/home/alice/data.txt");

// 将相对路径转为绝对路径（基于当前工作目录）
Path resolved = Paths.get(System.getProperty("user.dir")).resolve("data.txt");
```

##### 3. 在 Spring 中使用 `Resource`

```java
// 相对路径（基于工作目录）
Resource rel = new FileSystemResource("config/app.yml");

// 绝对路径
Resource abs = new FileSystemResource("/etc/myapp/app.yml");
```

------

#### 三、关键区别总结

| 特性                 | 绝对路径                          | 相对路径                               |
| -------------------- | --------------------------------- | -------------------------------------- |
| **起点**             | 文件系统根目录（如 `/` 或 `C:\`） | 当前工作目录（`user.dir`）             |
| **可移植性**         | ❌ 差（硬编码路径）                | ✅ 好（但依赖运行位置）                 |
| **是否受 `cd` 影响** | 否                                | 是                                     |
| **Java 中判断方法**  | `path.isAbsolute()`               | `!path.isAbsolute()`                   |
| **适用场景**         | 配置文件、日志目录（固定位置）    | 项目内资源、临时文件（与启动位置相关） |

------

#### 四、如何查看当前工作目录？

```java
String workingDir = System.getProperty("user.dir");
System.out.println("当前工作目录: " + workingDir);
```

- 这就是**所有相对路径的基准**！

------

#### 五、常见陷阱与最佳实践

##### ⚠️ 陷阱 1：在 IDE 能跑，打包后找不到文件

- **原因**：IDE 中工作目录 = 项目根目录，但生产环境不是。
- **解决**：用绝对路径，或通过配置指定路径。

##### ⚠️ 陷阱 2：混用 `/` 和 `\`

- 错误（Windows）

  ：

  ```java
  new File("C:\my folder\file.txt"); // \m 被当作转义字符！
  ```

- 正确做法

  ：

  ```java
  new File("C:\\my folder\\file.txt");     // 转义
  new File("C:/my folder/file.txt");       // Java 允许用 / 即使在 Windows
  Paths.get("C:", "my folder", "file.txt"); // 最安全
  ```

##### ✅ 最佳实践

1. **开发阶段**：可用相对路径（确保文件在工作目录下）。

2. 生产部署

   ：

   - 外部文件用**绝对路径**；
   - 或通过 `-Dconfig.path=...` 等 JVM 参数传入路径。

3. **资源文件**（如模板、配置）：放 `src/main/resources`，用 **classpath** 加载（与路径无关）。

4. 永远校验文件是否存在

   ：

   ```java
   if (!Files.exists(path)) {
       throw new FileNotFoundException("文件不存在: " + path.toAbsolutePath());
   }
   ```

------

#### 六、示例：相对路径 vs 绝对路径行为对比

假设项目结构：

```
/home/user/myapp/
├── app.jar
├── config/
│   └── settings.properties
└── data.txt
```

##### 场景 1：在 `/home/user/myapp` 下运行

```bash
cd /home/user/myapp
java -jar app.jar
```

- `new File("data.txt")` → ✅ 找到
- `new File("config/settings.properties")` → ✅ 找到

##### 场景 2：在 `/tmp` 下运行

```bash
cd /tmp
java -jar /home/user/myapp/app.jar
```

- `new File("data.txt")` → ❌ 找不到（会去 `/tmp/data.txt`）
- `new File("/home/user/myapp/data.txt")` → ✅ 找到

------

#### 七、一句话总结

> **绝对路径 = “身份证号”，全球唯一；
> 相对路径 = “我家楼下”，取决于你站在哪。**

在 Java 中：

- 用 **绝对路径** 确保稳定（适合生产）；
- 用 **相对路径** 方便开发（需控制运行目录）；
- 用 **classpath** 管理应用内资源（最可靠）。

## JAVA 基础

### 1. **Java中的注释有哪几种类型？它们各自的作用是什么？**

- **单行注释**：`//`，用于注释一行代码，简洁明了。
- **多行注释**：`/* ... */`，用于注释多行代码，常用于临时禁用代码块。
- **文档注释**：`/** ... */`，用于生成文档，尤其是通过 JavaDoc 工具生成 API 文档时使用。

------

### 2. **什么是JavaDoc？如何通过JavaDoc生成API文档？**

- **JavaDoc**：JavaDoc 是一种文档注释格式，使用 `/** ... */` 注释样式，可以为类、方法和字段添加描述。

- **生成API文档**：

  1. 在类或方法中使用 `@param`、`@return`、`@throws` 等标签进行注释。

  2. 使用命令行工具 `javadoc` 生成文档：

     ```bash
     javadoc -d doc MyClass.java
     ```

------

### 3. **什么是断言（assert）？它的使用场景和注意事项是什么？**

- **断言**：`assert` 是一种用于调试程序的工具，通常在开发和测试阶段用于验证假设。
- **使用场景**：验证程序运行时的假设条件。
- **注意事项**：断言默认在生产环境中禁用，需启用时加 `-ea` 参数。断言不应替代正常的错误处理。

## 常见业务场景

### 排序解决方案

Java 排序分为**基础数据排序**、**对象自定义排序**、**集合/数组排序**三大场景。

#### 一、核心工具类（必记）

Java 内置两个万能排序工具，性能最优（底层是快速排序/归并排序/Timsort），无需手写冒泡排序：

1. **`Arrays.sort()`**：用于**数组**排序（`int[]`、`String[]`、对象数组）
2. **`Collections.sort()`** / **`List.sort()`**：用于**集合**排序（`ArrayList`、`LinkedList`）

---

#### 二、场景1：基本数据类型/字符串排序

##### 1. 数组排序（int、double、String）

```java
import java.util.Arrays;

public class ArraySort {
    public static void main(String[] args) {
        // 1. 整数数组排序（默认升序）
        int[] arr = {5, 2, 9, 1, 5, 6};
        Arrays.sort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 5, 5, 6, 9]

        // 2. 字符串数组排序（按字母/Unicode升序）
        String[] strArr = {"banana", "apple", "orange"};
        Arrays.sort(strArr);
        System.out.println(Arrays.toString(strArr)); // [apple, banana, orange]
    }
}
```

##### 2. 降序排序（基本类型包装类）

基本类型`int[]`**不支持直接降序**，需要用包装类`Integer[]`：

```java
import java.util.Arrays;
import java.util.Collections;

public class ArrayDescSort {
    public static void main(String[] args) {
        Integer[] arr = {5, 2, 9, 1};
        // Collections.reverseOrder() 降序
        Arrays.sort(arr, Collections.reverseOrder());
        System.out.println(Arrays.toString(arr)); // [9, 5, 2, 1]
    }
}
```

---

#### 三、场景2：List集合排序

##### 1. 基础类型 List 排序

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

public class ListSort {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(3);
        list.add(1);
        list.add(4);
        list.add(2);

        // 方式1：升序
        Collections.sort(list);
        // 方式2：降序
        list.sort(Collections.reverseOrder());

        System.out.println(list); // [4, 3, 2, 1]
    }
}
```

---

#### 四、场景3：自定义对象排序（最常用）

比如排序`User`、`Student`、`Product`对象，有**两种方案**：

##### 方案1：实现 Comparable 接口（默认排序）

适用于**固定排序规则**（如默认按年龄升序）

```java
import java.util.Arrays;

// 1. 实现 Comparable 接口
class Student implements Comparable<Student> {
    private String name;
    private int age;

    // 2. 重写compareTo方法（定义排序规则）
    @Override
    public int compareTo(Student o) {
        // 升序：this.age - o.age
        // 降序：o.age - this.age
        return this.age - o.age;
    }

    // 构造器、getter、toString
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return name + ":" + age;
    }
}

// 使用
public class ObjectSort {
    public static void main(String[] args) {
        Student[] students = {
            new Student("张三", 20),
            new Student("李四", 18),
            new Student("王五", 22)
        };
        Arrays.sort(students);
        System.out.println(Arrays.toString(students));
        // [李四:18, 张三:20, 王五:22]
    }
}
```

##### 方案2：使用 Comparator 接口（灵活排序）

适用于**动态排序**（临时按年龄/姓名/成绩排序），**无需修改实体类**

```java
import java.util.ArrayList;
import java.util.Comparator;

public class ComparatorSort {
    public static void main(String[] args) {
        ArrayList<Student> list = new ArrayList<>();
        list.add(new Student("张三", 20));
        list.add(new Student("李四", 18));
        list.add(new Student("王五", 22));

        // 1. 按年龄 升序
        list.sort(Comparator.comparingInt(Student::getAge));
        // 2. 按年龄 降序
        list.sort((s1, s2) -> s2.getAge() - s1.getAge());
        // 3. 按姓名 字母排序
        list.sort(Comparator.comparing(Student::getName));

        System.out.println(list);
    }
}
```

---

#### 五、场景4：多字段排序（先按年龄，再按姓名）

```java
// 先按年龄升序，年龄相同按姓名降序
list.sort(Comparator.comparingInt(Student::getAge)
        .thenComparing(Comparator.comparing(Student::getName).reversed()));
```

---

#### 六、场景5：Java 8+ 流式排序（一行代码）

```java
// 排序后生成新集合，不修改原集合
List<Student> sortedList = list.stream()
        .sorted(Comparator.comparingInt(Student::getAge))
        .toList();
```

---

#### 七、完整方案对比（快速选型）

| 场景         | 方案                           | 代码示例                                       |
| ------------ | ------------------------------ | ---------------------------------------------- |
| 数组排序     | `Arrays.sort()`                | `Arrays.sort(arr)`                             |
| 数组降序     | `Arrays.sort(arr, 反向比较器)` | `Arrays.sort(arr, Collections.reverseOrder())` |
| List排序     | `List.sort()`                  | `list.sort(null)`                              |
| 对象默认排序 | 实现`Comparable`               | 重写`compareTo`                                |
| 对象灵活排序 | `Comparator`                   | `list.sort(Comparator.comparing())`            |
| 多字段排序   | `Comparator.thenComparing()`   | 链式调用                                       |

---

#### 八、手写经典排序算法（面试用）

如果面试要求手写，优先记**冒泡排序**和**快速排序**：

##### 1. 冒泡排序（简单）

```java
public static void bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        for (int j = 0; j < arr.length - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

##### 2. 快速排序（高效）

```java
public static void quickSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int pivot = arr[left];
    int i = left, j = right;
    while (i < j) {
        while (i < j && arr[j] >= pivot) j--;
        arr[i] = arr[j];
        while (i < j && arr[i] <= pivot) i++;
        arr[j] = arr[i];
    }
    arr[i] = pivot;
    quickSort(arr, left, i - 1);
    quickSort(arr, i + 1, right);
}
```

---

#### 总结

1. **开发用**：直接用 `Arrays.sort()` / `List.sort()` + `Comparator`，性能最优
2. **对象排序**：优先用 `Comparator` 灵活排序，不侵入实体类
3. **面试用**：掌握冒泡、快速排序手写
4. **降序/多字段**：用 `reversed()` / `thenComparing()` 链式实现