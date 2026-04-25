### Java 中的 Builder 模式详解

Builder 模式（建造者模式）是 Java 中一种常用的创建型设计模式，核心目的是**解决复杂对象的构建问题**——尤其是当一个类有大量可选参数、多层嵌套属性，或需要保证对象不可变性时，替代臃肿的构造器（如 telescoping constructor）或繁琐的 setter 方法。

#### 一、为什么需要 Builder？
先看一个反例：如果一个 `User` 类有多个可选参数，用传统构造器/setter 会有问题：
```java
// 问题1：构造器参数多，易混淆顺序；参数可选时需重载多个构造器
public User(String name, Integer age, String phone, String address) { ... }

// 问题2：setter 方法破坏对象不可变性（对象创建后仍可被修改）
User user = new User();
user.setName("张三");
user.setAge(20);
user.setPhone(null); // 可选参数需手动处理 null
```

而 Builder 模式可以：
1. 链式调用，代码更简洁易读；
2. 控制对象创建过程，保证参数合法性；
3. 支持不可变对象（创建后无 setter）；
4. 清晰区分必选/可选参数。

#### 二、Builder 模式的核心实现方式
##### 1. 手动实现 Builder（最基础）
以 `User` 类为例，完整实现如下：
```java
// 不可变对象（所有属性私有 + final，无 setter）
public class User {
    // 必选参数
    private final String name;
    private final Integer age;
    // 可选参数
    private final String phone;
    private final String address;

    // 私有构造器：仅允许 Builder 内部调用
    private User(Builder builder) {
        // 校验必选参数（提前暴露错误）
        if (builder.name == null || builder.name.isEmpty()) {
            throw new IllegalArgumentException("姓名不能为空");
        }
        if (builder.age == null || builder.age < 0) {
            throw new IllegalArgumentException("年龄必须为非负数");
        }
        // 赋值（从 Builder 拷贝到 User）
        this.name = builder.name;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    // 静态内部 Builder 类
    public static class Builder {
        // 必选参数（非 final，需先赋值）
        private String name;
        private Integer age;
        // 可选参数（非 final，默认值可自定义）
        private String phone;
        private String address;

        // Builder 构造器：仅接收必选参数
        public Builder(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        // 可选参数的链式方法（返回 Builder 自身，支持链式调用）
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        // 构建最终对象
        public User build() {
            return new User(this);
        }
    }

    // 仅提供 getter（无 setter，保证不可变）
    public String getName() { return name; }
    public Integer getAge() { return age; }
    public String getPhone() { return phone; }
    public String getAddress() { return address; }
}
```

**使用方式（链式调用）：**
```java
// 创建必选参数 + 部分可选参数的对象
User user = new User.Builder("张三", 20)
        .phone("13800138000")
        .build();

// 验证不可变性：无 setter，无法修改属性
System.out.println(user.getName()); // 张三
System.out.println(user.getPhone()); // 13800138000
```

##### 2. 自动生成 Builder（Lombok 简化）
手动写 Builder 代码冗余，实际开发中常用 Lombok 的 `@Builder` 注解自动生成：
```java
import lombok.Builder;
import lombok.Getter;

@Getter // 自动生成 getter
@Builder // 自动生成 Builder 内部类、链式方法、build() 方法
public class User {
    // 必选参数（可通过 @NonNull 校验非空）
    private final String name;
    private final Integer age;
    // 可选参数
    private final String phone;
    private final String address;
}
```

**使用方式：**
```java
// Lombok 生成的 Builder 支持链式调用
User user = User.builder()
        .name("李四") // 必选（实际无强制校验，需手动加 @NonNull）
        .age(25)
        .address("北京市")
        .build();
```

> 补充：如果需要校验必选参数，可给字段加 `@NonNull`，Lombok 会在 `build()` 时自动校验空值。

##### 3. 第三方库中的 Builder（如 Spring、MyBatis）
很多框架也内置了 Builder 模式，比如：
- `StringBuilder`/`StringBuffer`：构建可变字符串（基础 Builder 实现）；
- MyBatis 的 `SqlSessionFactoryBuilder`：构建 SqlSessionFactory；
- Spring 的 `UriComponentsBuilder`：构建 URL 参数。

示例（`StringBuilder`）：
```java
// 链式构建字符串，避免字符串拼接的性能损耗
String str = new StringBuilder()
        .append("Hello")
        .append(" ")
        .append("World")
        .toString();
System.out.println(str); // Hello World
```

#### 三、Builder 模式的核心特点
1. **分离构建与表示**：Builder 类负责对象的构建逻辑，目标类（如 User）只负责数据存储，职责单一；
2. **链式调用**：每个 Builder 方法返回自身（`return this`），支持 `builder.a().b().c()` 形式；
3. **不可变性支持**：目标类属性可设为 `final`，仅通过 Builder 赋值，创建后无法修改；
4. **参数校验**：可在 `build()` 方法中统一校验参数合法性（如非空、范围），避免无效对象。

#### 四、Builder 模式的适用场景
- 类有 **3 个及以上可选参数**，或参数类型相似（如多个 String/Integer，易混淆顺序）；
- 需要创建 **不可变对象**（无 setter）；
- 构建过程需分步、分阶段，或需要生成不同配置的对象（如一个 User 有基础信息/扩展信息）；
- 替代 `Map`/`JSONObject` 传递多参数（更类型安全）。

#### 五、Builder 模式的优缺点
| 优点                     | 缺点                                   |
| ------------------------ | -------------------------------------- |
| 代码可读性高（链式调用） | 增加类的复杂度（多一个 Builder 类）    |
| 支持不可变对象           | 简单对象使用会“过度设计”               |
| 参数校验集中，易维护     | 手动实现代码冗余（可通过 Lombok 解决） |

---

#### 总结

1. **核心作用**：Builder 模式解决复杂对象的构建问题，替代臃肿的构造器/setter，保证代码可读性和对象不可变性；
2. **实现方式**：手动实现（静态内部类 + 链式方法 + 私有构造器）、Lombok `@Builder` 自动生成、框架内置 Builder（如 StringBuilder）；
3. **核心特点**：链式调用、分离构建与表示、支持参数校验和不可变对象创建。

Builder 是 Java 开发中处理复杂对象创建的“标配”，尤其是在业务实体类（POJO/DO/DTO）设计中，优先使用 Builder 而非多层构造器或全量 setter。

#### `.builder()`

在 Java 中，`.builder()` 本质上是**一个静态方法**（而非属性），调用它会返回一个 Builder 模式的实例（Builder 内部类对象），后续链式调用的才是 Builder 实例的**成员方法**（也不是属性）。

##### 一、先明确核心概念：方法 vs 属性

在 Java 中：
- **属性（字段/成员变量）**：是类存储数据的变量，调用形式为 `对象.属性名`（如 `user.name`，但通常属性会私有化，不会直接暴露）；
- **方法**：是类的行为/操作，调用形式为 `对象.方法名()`（必须带括号），`.builder()` 带括号，因此是方法。

##### 二、拆解 `.builder()` 及后续链式调用的本质

以最常见的 Lombok `@Builder` 生成的代码为例，我们还原底层逻辑，你就能清晰理解：

###### 1. `.builder()` 是静态方法

Lombok 为 `User` 类自动生成一个**静态方法 `builder()`**，作用是创建并返回 `User.Builder` 实例：
```java
public class User {
    // 静态方法：builder()，返回 Builder 实例
    public static User.Builder builder() {
        return new User.Builder();
    }

    // 内部 Builder 类
    public static class Builder {
        // Builder 的成员变量（对应 User 的属性）
        private String name;
        private Integer age;

        // Builder 的成员方法（链式设置参数）
        public User.Builder name(String name) {
            this.name = name;
            return this; // 返回自身，支持链式调用
        }

        public User.Builder age(Integer age) {
            this.age = age;
            return this;
        }

        // 构建最终对象的方法
        public User build() {
            return new User(this.name, this.age);
        }
    }
}
```

###### 2. 完整调用链的拆解

```java
User user = User.builder() // 1. 调用静态方法 builder()，返回 Builder 实例
        .name("张三")      // 2. 调用 Builder 实例的 name() 方法，返回自身
        .age(20)          // 3. 调用 Builder 实例的 age() 方法，返回自身
        .build();         // 4. 调用 Builder 实例的 build() 方法，返回 User 实例
```
- `User.builder()`：静态方法调用，返回 `Builder` 对象；
- `.name("张三")`/`.age(20)`：`Builder` 实例的成员方法（非静态），返回 `this`（自身），因此能链式调用；
- `.build()`：`Builder` 实例的成员方法，返回最终的 `User` 对象。

##### 三、特殊场景：手动实现的 Builder

如果你手动写 Builder 模式（比如之前的 `User.Builder` 构造器接收必选参数），可能没有 `builder()` 静态方法，而是直接 `new User.Builder(...)`，但核心逻辑一致：
```java
// 手动实现的 Builder：无 builder() 静态方法，直接 new Builder 实例
User user = new User.Builder("张三", 20) // new 出来的是 Builder 实例
        .phone("13800138000")           // 调用 Builder 的成员方法
        .build();                       // 调用 build() 生成 User
```

##### 四、易混淆的“属性”误区

新手容易把 `.builder()` 误认为属性，核心原因：
1. 有些语言（如 Kotlin）支持“属性访问器”，可省略括号，但 Java 中**方法必须带括号**，属性直接访问（无括号）；
2. Lombok 隐藏了 `builder()` 方法的实现，看起来像“属性”，但底层仍是方法。

---

##### 总结

1. `.builder()` 是**静态方法**（带括号是关键特征），调用后返回 Builder 实例；
2. `.builder()` 之后的 `.name()`/`.age()`/`.build()` 是 Builder 实例的**成员方法**，而非属性；
3. Java 中“属性”无括号（如 `obj.field`），“方法”必须带括号（如 `obj.method()`），这是区分两者的核心标志。