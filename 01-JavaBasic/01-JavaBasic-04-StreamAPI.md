## 十、Stream API

### 函数式编程/函数式接口/lambda/Stream

Stream API 是 Java 8 引入的核心特性之一，位于 `java.util.stream` 包下，其设计目标是**以声明式方式高效处理集合（或其他数据源）中的元素**，支持链式调用、并行处理和函数式编程风格，大幅简化数据过滤、转换、聚合等操作的代码。

```java
【Java 函数式编程】
├─ 核心基础：函数式接口 @FunctionalInterface
│    ├─ 四大内置核心接口
│    │   Consumer、Supplier、Function、Predicate
│    └─ 自定义函数式接口
├─ Lambda表达式
│    └─ Lambda = 函数式接口的实例(匿名实现类语法糖)
├─ 方法引用 ::
│    └─ Lambda 的简化写法
└─ Stream流式编程
     └─ 大量方法入参接收函数式接口，链式处理集合
```

#### 一、整体逻辑框架（先总述，再分述）

核心目标是**简化代码、提升开发效率，实现函数式编程风格**，其中

**函数式编程是思想，函数式接口是基础，Lambda 是语法糖，Stream 是函数式编程在集合处理上的落地实现**，四者层层递进、紧密关联。下面我分别解释：

##### 1. 函数式编程（核心思想）

函数式编程是一种**编程范式**（对比面向对象编程），核心特点是：

- 把“函数”作为一等公民（可作为参数传递、返回值返回、赋值给变量）；
- 强调“做什么”而非“怎么做”，关注结果而非执行步骤；
- 推崇无副作用（函数执行不修改外部状态）、不可变数据，便于并行处理。

Java 并非纯函数式语言，但 Java 8 引入函数式编程特性（Lambda、函数式接口、Stream），让开发者可以用函数式风格编写代码，弥补了传统命令式编程（逐行写执行步骤）的繁琐。

##### 2. 函数式接口（函数式编程的基础）

函数式接口是实现函数式编程的**载体**，定义很简单：

- 接口中**仅有一个抽象方法**（可包含多个默认方法/静态方法），用 `@FunctionalInterface` 注解标识（编译器会校验规则）；
- 核心作用：封装一段逻辑（如判断、转换、消费），让逻辑可以像“数据”一样传递（作为方法参数/返回值）。

Java 8 在 `java.util.function` 包中内置了核心函数式接口，覆盖绝大多数场景，无需自定义：

- `Predicate<T>`：接收参数返回布尔值（条件判断，如过滤集合）；
- `Function<T,R>`：接收 T 类型返回 R 类型（数据转换，如字符串转整数）；
- `Consumer<T>`：接收参数无返回值（消费数据，如遍历集合）；
- `Supplier<T>`：无参数返回数据（提供数据，如生成对象）。

##### 3. Lambda 表达式（函数式编程的语法糖）

Lambda 是简化函数式接口实现的**语法糖**，核心作用是：用简洁的语法替代匿名内部类，直接传递一段逻辑（函数）。

- 语法格式：`(参数列表) -> { 逻辑代码 }`（参数类型可省略，单参数可省略括号，单行逻辑可省略大括号/return）；
- 本质：Lambda 表达式会被编译为函数式接口的实现类实例，底层还是面向对象，但语法上更接近函数式编程。

示例对比（遍历集合）：

- 传统匿名内部类（繁琐）：

  ```java
  list.forEach(new Consumer<String>() {
      @Override
      public void accept(String s) {
          System.out.println(s);
      }
  });
  ```

- Lambda 表达式（简洁）：

  ```java
  list.forEach(s -> System.out.println(s));
  ```

##### 4. Stream（函数式编程的落地实现）

Stream 是 Java 8 提供的**集合处理工具**，基于函数式编程思想设计，核心作用是：以声明式方式（“做什么”）处理集合数据，替代传统的循环遍历（“怎么做”）。处理集合的过滤、排序等

- 核心特点：
  1. **流式处理**：数据像“流”一样依次经过过滤、映射、排序等操作，最终得到结果；
  2. **惰性执行**：中间操作（如 `filter`、`map`）仅记录逻辑，不执行，直到终止操作（如 `collect`、`forEach`）才触发执行；
  3. **不可变**：操作不会修改原集合，而是生成新的结果；
  4. **支持并行**：通过 `parallelStream()` 轻松实现并行处理，提升大数据量处理效率。

示例（过滤并转换集合）：

- 传统命令式（逐行写步骤）：

  ```java
  List<Integer> result = new ArrayList<>();
  for (String s : list) {
      if (s.length() > 3) { // 过滤
          result.add(s.length()); // 转换
      }
  }
  ```

- Stream 函数式（声明式，关注结果）：

  ```java
  List<Integer> result = list.stream()
          .filter(s -> s.length() > 3) // 中间操作：过滤
          .map(String::length) // 中间操作：转换
          .toList(); // 终止操作：收集结果
  ```

#### 二、四者关联总结（面试收尾，强化逻辑）

- 函数式编程是**思想**，强调“传递逻辑、声明式编程”；
- 函数式接口是**基础**，为逻辑传递提供载体；
- Lambda 是**语法糖**，简化函数式接口的实现，让逻辑传递更简洁；
- Stream 是**落地工具**，把函数式编程思想应用到集合处理中，大幅简化集合操作代码。

这一套特性结合使用，既能保持 Java 面向对象的本质，又能享受函数式编程的简洁、高效，是 Java 8 最核心的升级之一。

### 一、Stream 的核心概念与特性

首先需明确：**Stream 不是集合，也不是数据结构**，它更像一个“数据处理管道”——从数据源（如集合、数组、I/O 流）获取元素，通过一系列“中间操作”对元素进行处理，最终通过“终止操作”生成结果（如集合、数值、布尔值等）。

**Stream 的核心特性：**

1. **声明式编程**：关注“要做什么”（如“过滤偶数”“转换为字符串”），而非“怎么做”（无需手动写循环）；
2. **链式调用**：中间操作返回 Stream 本身，可串联多个操作，代码简洁；
3. **惰性求值**：中间操作仅“记录”处理逻辑，不立即执行；只有触发终止操作时，才会一次性执行所有中间操作（优化性能）；
4. **不可修改数据源**：Stream 操作不会改变原始数据源（如集合），只会生成新的结果；
5. **支持并行处理**：通过 `parallelStream()` 可轻松实现并行计算，底层依赖 Fork/Join 框架，无需手动处理线程。

### 二、Stream 的操作分类：中间操作 vs 终止操作

Stream 的操作严格分为两类，必须遵循“**中间操作可多个，终止操作仅一个（且触发执行）**”的规则：

| 操作类型     | 核心特点                                                     | 常见操作示例                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **中间操作** | 1. 不立即执行，仅记录操作逻辑；<br>2. 返回新的 Stream；<br>3. 可链式串联。 | 过滤（`filter`）、映射（`map`）、排序（`sorted`）、去重（`distinct`） |
| **终止操作** | 1. 触发所有中间操作执行；<br>2. 不返回 Stream（返回结果或无返回值）；<br>3. 执行后 Stream 不可再使用。 | 收集（`collect`）、遍历（`forEach`）、计数（`count`）、匹配（`anyMatch`） |

在 Java 8 引入的 **Stream API** 中，Stream 的操作被分为两大类：

------

#### 1. 中间操作（Intermediate Operations）

- **特点**：

  - **惰性求值（Lazy）**：不会立即执行，只有在遇到终止操作时才会触发整个流水线的执行。
  - **返回类型是 Stream**：可以链式调用多个中间操作。
  - **可重复连接**：多个中间操作可以串联形成一个处理流水线。

- **常见中间操作**：Stream API 大量使用内置函数式接口（Predicate、Function、Consumer 等）

  ```java
  filter(Predicate<T>)        // 过滤元素
  map(Function<T, R>)         // 映射转换
  flatMap(Function<T, Stream<R>>) // 扁平映射
  distinct()                  // 去重
  sorted() / sorted(Comparator) // 排序
  peek(Consumer<T>)           // 调试/查看元素（不修改）
  limit(long n)               // 截取前 n 个
  skip(long n)                // 跳过前 n 个
  ```

- **示例**：

  ```java
  List<String> result = list.stream()
      .filter(s -> s.length() > 3)   // 中间操作
      .map(String::toUpperCase)      // 中间操作
      .sorted()                      // 中间操作
      .collect(Collectors.toList()); // 终止操作
  ```

##### map

在 Java 的 **Stream API** 中，`map()` 是一个**中间操作（Intermediate Operation）**，用于对流中的每个元素应用一个函数，将其**转换为另一种类型或形式**，从而生成一个新的流。

------

###### 🧩 1. 基本语法

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

- **输入**：一个 `Function<T, R>` 函数式接口（接收 T，返回 R）
- **输出**：一个新的 `Stream<R>`
- **特点**：一对一映射（每个输入元素对应一个输出元素）

------

###### ✅ 2. 常见用法示例

🔹 示例 1：基本类型转换

```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

// 将每个单词转为大写
List<String> upper = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// 结果: ["APPLE", "BANANA", "CHERRY"]
```

🔹 示例 2：提取对象属性

```java
class Person {
    private String name;
    private int age;
    // constructor, getters...
}

List<Person> people = ...;

// 提取所有人的姓名
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

🔹 示例 3：数值计算

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

// 每个数平方
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
// 结果: [1, 4, 9, 16]
```

🔹 示例 4：类型转换（String → Integer）

```java
List<String> numStrs = Arrays.asList("1", "2", "3");

List<Integer> nums = numStrs.stream()
    .map(Integer::parseInt)  // 或 s -> Integer.valueOf(s)
    .collect(Collectors.toList());
```

------

###### ⚠️ 3. 注意事项

❌ 不要用于副作用（如打印）

虽然可以这么做，但不符合函数式编程原则：

```java
// 不推荐：map 不是用来做副作用的
list.stream().map(e -> {
    System.out.println(e); // 副作用
    return e;
});
```

✅ 正确做法：用 `peek()` 调试，用 `forEach()` 执行副作用：

```java
list.stream()
    .peek(System.out::println) // 仅用于调试
    .map(String::toUpperCase)
    .forEach(System.out::println); // 终止操作，执行打印
```

🔁 `map` vs `flatMap`

- `map`：一对一（`T → R`）
- `flatMap`：一对多（`T → Stream<R>`），然后**扁平化**为一个流

```java
List<List<Integer>> list = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);

// 使用 flatMap 展平
List<Integer> flat = list.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4]
```

------

###### 🧪 4. 与其他操作组合

```java
List<String> result = people.stream()
    .filter(p -> p.getAge() >= 18)   // 中间操作：过滤
    .map(Person::getName)            // 中间操作：映射
    .map(String::toUpperCase)        // 链式 map
    .sorted()                        // 中间操作：排序
    .collect(Collectors.toList());   // 终止操作
```

------

###### 📌 总结

| 特性         | 说明                                            |
| ------------ | ----------------------------------------------- |
| **类型**     | 中间操作（返回 `Stream`）                       |
| **作用**     | 元素转换（T → R）                               |
| **是否惰性** | 是（需终止操作触发）                            |
| **常见用途** | 提取属性、类型转换、计算新值                    |
| **替代方案** | 复杂转换用 `flatMap`，副作用用 `peek`/`forEach` |

> 💡 **最佳实践**：`map()` 应保持**纯函数**特性——无副作用、相同输入始终返回相同输出。

掌握 `map()` 是使用 Stream 进行数据转换的核心技能！



##### 5.Sort

在 Java Stream 中，`sort` 操作是通过 **`sorted()`** 方法实现的，属于 **中间操作（Intermediate Operation）**。

###### 1. **自然排序（元素需实现 `Comparable`）**

```java
List<String> list = Arrays.asList("banana", "apple", "cherry");
List<String> sorted = list.stream()
    .sorted()  // 按字典序升序
    .collect(Collectors.toList());
// 结果: ["apple", "banana", "cherry"]
```

###### 2. **自定义排序（使用 `Comparator`）**

```java
List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5);
List<Integer> sortedDesc = numbers.stream()
    .sorted(Comparator.reverseOrder())  // 降序
    .collect(Collectors.toList());
// 结果: [5, 4, 3, 1, 1]
```

###### 3. **对对象排序**

```java
class Person {
    String name;
    int age;
    // constructor, getters...
}

List<Person> people = ...;
List<Person> byAge = people.stream()
    .sorted(Comparator.comparingInt(p -> p.age))  // 按年龄升序
    .collect(Collectors.toList());

// 多级排序：先按年龄，再按姓名
List<Person> multiSorted = people.stream()
    .sorted(Comparator.comparingInt((Person p) -> p.age)
                     .thenComparing(p -> p.name))
    .collect(Collectors.toList());
```

------

###### ⚠️ 注意事项

- `sorted()` 是 **稳定排序**（stable），相等元素的相对顺序不变。

- 它是 **中间操作**，不会立即执行，需配合终止操作（如 `collect()`）。

- 对于并行流（`parallelStream()`），`sorted()` 仍能保证正确性，但可能影响性能。

- 如果流中包含 `null`，使用 `Comparator.naturalOrder()` 会抛出 `NullPointerException`。可改用：

  ```java
  .sorted(Comparator.nullsLast(Comparator.naturalOrder()))
  ```

------

###### 📌 总结

| 方法                                       | 类型     | 说明                                            |
| ------------------------------------------ | -------- | ----------------------------------------------- |
| `sorted()`                                 | 中间操作 | 使用元素的自然顺序排序（要求实现 `Comparable`） |
| `sorted(Comparator<? super T> comparator)` | 中间操作 | 使用自定义比较器排序                            |

> 💡 提示：`sorted()` 不会修改原集合，而是返回一个新的有序流。

------

#### 2. 终止操作（Terminal Operations）

- **特点**：

  - **触发执行**：一旦调用，会启动整个 Stream 流水线的计算。
  - **只执行一次**：一个 Stream 只能有一个终止操作；之后 Stream 就“消费”掉了，不能复用。
  - **返回非 Stream 类型**：如 `void`、`List`、`Integer`、`boolean` 等。

- **常见终止操作**：

  ```java
  forEach(Consumer<T>)        // 遍历
  collect(Collector<T, A, R>) // 收集结果（最常用）
  reduce(BinaryOperator<T>)   // 聚合归约
  count()                     // 元素数量
  findFirst() / findAny()     // 查找元素（返回 Optional）
  anyMatch(Predicate<T>)      // 是否存在匹配
  allMatch(Predicate<T>)      // 是否全部匹配
  noneMatch(Predicate<T>)     // 是否都不匹配
  min(Comparator) / max(Comparator) // 最值
  toArray()                   // 转为数组
  ```

- **示例**：

  ```java
  long count = list.stream()
      .filter(s -> s.startsWith("A"))
      .count(); // 终止操作，返回 long
  ```



##### collect

在 Java Stream API 中，**`collect()`** 是一个**终止操作（Terminal Operation）**，用于将流中的元素**汇聚（收集）成一个结果容器**，如 `List`、`Set`、`Map`、字符串，甚至自定义对象。

它是 Stream 流水线中**最常用、最强大的终止操作之一**。

------

###### 🧩 1. 基本语法

```java
<R, A> R collect(Collector<? super T, A, R> collector)
```

- **`Collector`**：定义了如何将流元素累积到结果容器中。
- Java 提供了 `Collectors` 工具类，包含大量预定义的收集器。

------

###### ✅ 2. 常见用法示例

🔹 1. 收集为 List / Set / Array

```java
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
String[] array = stream.toArray(String[]::new);
```

> 💡 `toList()` 和 `toSet()` 返回的是**不可变集合（Java 10+）** 或具体实现（如 `ArrayList`），具体行为取决于 JDK 版本。如需可变集合，可用：
>
> ```java
> Collectors.toCollection(ArrayList::new)
> ```

------

 🔹 2. 拼接为字符串（joining）

```java
List<String> words = Arrays.asList("Java", "Stream", "API");

String result = words.stream()
    .collect(Collectors.joining());          // "JavaStreamAPI"
String withDelimiter = words.stream()
    .collect(Collectors.joining(", "));      // "Java, Stream, API"
String withAll = words.stream()
    .collect(Collectors.joining(", ", "[", "]")); // "[Java, Stream, API]"
```

------

🔹 3. 转换为 Map

```java
List<Person> people = ...;

// key = name, value = age
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,      // keyMapper
        Person::getAge        // valueMapper
    ));

// 处理 key 冲突（如两个同名的人）
Map<String, Person> nameToPerson = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Function.identity(),  // 返回自身
        (existing, replacement) -> existing // 保留第一个
    ));
```

------

🔹 4. 分组（groupingBy）

```java
// 按年龄分组
Map<Integer, List<Person>> byAge = people.stream()
    .collect(Collectors.groupingBy(Person::getAge));

// 多级分组：先按年龄，再按性别
Map<Integer, Map<String, List<Person>>> multiGroup = people.stream()
    .collect(Collectors.groupingBy(Person::getAge,
             Collectors.groupingBy(Person::getGender)));
```

------

🔹 5. 分区（partitioningBy）

适用于**布尔条件**的分组（true / false 两组）：

```java
Map<Boolean, List<Person>> partition = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 18));
// partition.get(true) → 成年人列表
```

------

🔹 6. 统计（summarizing）

```java
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));

System.out.println(stats.getCount());  // 人数
System.out.println(stats.getSum());    // 年龄总和
System.out.println(stats.getAverage()); // 平均年龄
System.out.println(stats.getMin());    // 最小年龄
System.out.println(stats.getMax());    // 最大年龄
```

------

🔹 7. 自定义收集（reducing / collectingAndThen）

**Collectors.collectingAndThen:先用一个收集器收集结果，再对收集到的结果应用一个“finisher”函数进行后处理。**

```java
// 找到最长的名字
Optional<String> longest = people.stream()
    .map(Person::getName)
    .collect(Collectors.reducing((s1, s2) -> s1.length() > s2.length() ? s1 : s2));

// 先收集为 List，再转为不可变列表（Java 10+）
List<String> immutable = people.stream()
    .map(Person::getName)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));
```

------

###### 🛠️ 3. 自定义 Collector（高级）

你可以实现 `Collector` 接口来创建自己的收集逻辑，但大多数场景下 `Collectors` 已足够。

------

⚠️ 注意事项

| 问题               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| **流只能消费一次** | 调用 `collect()` 后，原 Stream 已关闭，不能再次使用          |
| **并发安全**       | 若使用并行流（`parallelStream()`），确保收集器是线程安全的（`Collectors` 提供的大多是安全的） |
| **空值处理**       | `toMap()` 中 key 或 value 为 `null` 可能抛异常（取决于 Map 实现，如 `HashMap` 允许，但某些场景会出错） |

------

###### 📌 总结：常用 `Collectors` 方法速查

| 目的       | 方法                                                |
| ---------- | --------------------------------------------------- |
| 转 List    | `Collectors.toList()`                               |
| 转 Set     | `Collectors.toSet()`                                |
| 转 Map     | `Collectors.toMap(key, value)`                      |
| 字符串拼接 | `Collectors.joining(delimiter)`                     |
| 分组       | `Collectors.groupingBy(classifier)`                 |
| 分区       | `Collectors.partitioningBy(predicate)`              |
| 统计       | `Collectors.summarizingInt/Long/Double()`           |
| 最值/求和  | `Collectors.maxBy()`, `summingInt()` 等             |
| 链式处理   | `Collectors.collectingAndThen(collector, finisher)` |

------

> 💡 **最佳实践**：优先使用 `Collectors` 提供的标准收集器，避免手动循环构建集合。`collect()` 是连接函数式流处理与传统集合/对象世界的桥梁。

掌握 `collect()`，你就掌握了 Stream 的“输出”能力！

#### Collectors

`Collectors.joining()` 是 Java Stream API 中一个非常实用的收集器（Collector），用于将流中的字符串元素拼接成一个单一的字符串。它支持指定分隔符、前缀和后缀，常用于将集合中的元素拼接成特定格式的字符串（如 CSV 格式、SQL 语句中的 IN 条件等）。

##### `Collectors.joining()`基本用法

`Collectors.joining()` 有三个重载方法，满足不同场景需求：

1. **无参版本**：直接拼接元素，无分隔符  

   ```java
   List<String> list = Arrays.asList("a", "b", "c");
   String result = list.stream().collect(Collectors.joining());
   // 结果："abc"
   ```

2. **单参数版本**：指定元素间的分隔符  

   ```java
   List<String> list = Arrays.asList("a", "b", "c");
   String result = list.stream().collect(Collectors.joining(", "));
   // 结果："a, b, c"
   ```

3. **三参数版本**：指定分隔符、前缀、后缀  

   ```java
   List<String> list = Arrays.asList("a", "b", "c");
   String result = list.stream().collect(Collectors.joining(", ", "(", ")"));
   // 结果："(a, b, c)" （常用于 SQL 的 IN 条件：WHERE id IN (1,2,3)）
   ```

###### 在分组场景中的应用

结合 `Collectors.groupingBy()` 时，`Collectors.joining()` 可快速实现“分组后拼接每组元素”的需求，例如题目中按首字母分组并拼接：

```java
List<String> words = Arrays.asList("apple", "banana", "avocado", "cherry");

// 按首字母分组，每组元素用 " | " 连接
Map<Character, String> result = words.stream()
    .collect(Collectors.groupingBy(
        // 分组键：首字母（转小写）
        word -> Character.toLowerCase(word.charAt(0)),
        // 收集每组元素：用 " | " 拼接
        Collectors.joining(" | ")
    ));

System.out.println(result); 
// 输出：{a=apple | avocado, b=banana, c=cherry}
```

这里 `groupingBy` 的第二个参数传入 `Collectors.joining(" | ")`，表示对每个分组内的字符串元素，用 `" | "` 拼接成一个字符串。

###### 注意点

- **流元素必须是字符串**：`joining()` 只能处理 `Stream<String>`，如果流元素是其他类型（如 Integer），需要先通过 `map(Object::toString)` 转换为字符串。  
  示例：  

  ```java
  List<Integer> nums = Arrays.asList(1, 2, 3);
  String result = nums.stream()
      .map(String::valueOf) // 转换为字符串流
      .collect(Collectors.joining("-")); 
  // 结果："1-2-3"
  ```

- **高效拼接**：内部使用 `StringBuilder` 实现拼接，性能优于手动循环拼接字符串（避免频繁创建字符串对象）。

通过 `Collectors.joining()`，可以简洁高效地完成字符串拼接操作，尤其在流处理和分组场景中非常实用。

------

###### 关键区别总结：

| 特性              | 中间操作             | 终止操作                      |
| ----------------- | -------------------- | ----------------------------- |
| 返回类型          | Stream               | 非 Stream（如 void、List 等） |
| 是否立即执行      | 否（惰性）           | 是（触发整个流水线）          |
| 可否链式调用多个  | 是                   | 否（只能有一个）              |
| Stream 是否可复用 | 是（直到终止操作前） | 否（执行后 Stream 关闭）      |

> ⚠️ 注意：**中间操作不会执行任何处理，除非后面跟了一个终止操作**。这是 Stream 实现高效流水线处理的核心机制。

------

这种设计使得 Stream API 既能表达清晰的数据处理逻辑，又能避免不必要的中间集合创建，提升性能。



### 三、Stream 的使用流程（3步）

使用 Stream 处理数据通常遵循“**获取 Stream → 中间操作链 → 终止操作**”的流程：

##### 1. 第一步：获取 Stream（4种常见方式）

从不同数据源创建 Stream：

- **从集合创建**：通过 `Collection` 接口的 `stream()`（串行）或 `parallelStream()`（并行）方法；

  ```java
  List<Integer> list = List.of(1, 2, 3, 4);
  Stream<Integer> stream = list.stream(); // 串行 Stream
  Stream<Integer> parallelStream = list.parallelStream(); // 并行 Stream
  ```

- **从数组创建**：通过 `Arrays.stream()` 方法；

  ```java
  Integer[] arr = {1, 2, 3, 4};
  Stream<Integer> arrStream = Arrays.stream(arr);
  ```

- **从值创建**：通过 `Stream.of()` 方法直接传入多个元素；

  ```java
  Stream<String> strStream = Stream.of("a", "b", "c");
  ```

- **从 I/O 流创建**：通过 `Files.lines()` 读取文件每行内容为 Stream（需处理 IO 异常）；

  ```java
  // 读取文件内容为 Stream<String>（每行一个元素）
  Stream<String> fileStream = Files.lines(Paths.get("test.txt"));
  ```

##### 2. 第二步：中间操作（常用操作详解）

中间操作的核心是“加工数据”，以下是开发中最常用的中间操作：

| 中间操作                             | 功能描述                                                    | 示例（以 `Stream<Integer>` 为例）                            |
| ------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
| `filter(Predicate<T>)`               | 过滤元素：保留满足 `Predicate` 条件的元素                   | `stream.filter(num -> num % 2 == 0)` → 保留偶数              |
| `map(Function<T,R>)`                 | 映射元素：将 T 类型转换为 R 类型                            | `stream.map(num -> num * 2)` → 每个元素乘以 2；<br>`stream.map(num -> String.valueOf(num))` → 转换为字符串 |
| `sorted()` / `sorted(Comparator<T>)` | 排序：默认自然排序（需元素实现 `Comparable`），或自定义排序 | `stream.sorted()` → 整数升序；<br>`stream.sorted((a, b) -> b - a)` → 整数降序 |
| `distinct()`                         | 去重：基于元素的 `equals()` 方法去重                        | `stream.distinct()` → 去除重复元素                           |
| `limit(long n)`                      | 限制数量：保留前 n 个元素                                   | `stream.limit(3)` → 只保留前 3 个元素                        |
| `skip(long n)`                       | 跳过元素：跳过前 n 个元素                                   | `stream.skip(2)` → 跳过前 2 个元素                           |

##### 3. 第三步：终止操作（常用操作详解）

终止操作触发执行所有中间操作，并返回最终结果，以下是常用终止操作：

| 终止操作                                    | 功能描述                                                 | 示例（以 `Stream<Integer>` 为例）                            |
| ------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| `forEach(Consumer<T>)`                      | 遍历元素：对每个元素执行 `Consumer` 逻辑                 | `stream.forEach(num -> System.out.println(num))` → 打印每个元素 |
| `collect(Collector<T,A,R>)`                 | 收集结果：将 Stream 转换为集合、Map 等                   | `stream.collect(Collectors.toList())` → 转换为 `List<Integer>`；<br>`stream.collect(Collectors.toSet())` → 转换为 `Set<Integer>` |
| `count()`                                   | 计数：返回元素个数（`long` 类型）                        | `long count = stream.filter(num -> num > 2).count()` → 统计大于 2 的元素个数 |
| `anyMatch(Predicate<T>)`                    | 任意匹配：是否存在至少一个元素满足条件（返回 `boolean`） | `boolean hasEven = stream.anyMatch(num -> num % 2 == 0)` → 是否有偶数 |
| `allMatch(Predicate<T>)`                    | 全部匹配：是否所有元素满足条件（返回 `boolean`）         | `boolean allPositive = stream.allMatch(num -> num > 0)` → 是否所有元素为正数 |
| `noneMatch(Predicate<T>)`                   | 无匹配：是否所有元素都不满足条件（返回 `boolean`）       | `boolean noZero = stream.noneMatch(num -> num == 0)` → 是否没有 0 |
| `findFirst()`                               | 查找第一个元素：返回 `Optional<T>`（避免空指针）         | `Optional<Integer> first = stream.findFirst()` → 获取第一个元素，用 `first.orElse(0)` 处理空值 |
| `max(Comparator<T>)` / `min(Comparator<T>)` | 求最大/最小值：返回 `Optional<T>`                        | `Optional<Integer> max = stream.max(Integer::compare)` → 求最大值 |

### 四、典型案例：Stream API 综合实战

通过一个案例串联常用操作，需求：**从列表中筛选出大于 3 的偶数，乘以 2 后排序，最终收集为 List 并打印**。

##### 传统循环实现（Java 8 前）：

```java
List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7, 8);
List<Integer> result = new ArrayList<>();

// 手动循环、筛选、转换、排序
for (int num : list) {
    if (num > 3 && num % 2 == 0) { // 筛选大于3的偶数
        result.add(num * 2); // 乘以2
    }
}
Collections.sort(result); // 排序

// 打印结果
for (int num : result) {
    System.out.println(num); // 输出：8, 12, 16
}
```

##### Stream API 实现（Java 8+）：

```java
List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7, 8);
List<Integer> result = list.stream()
                          .filter(num -> num > 3 && num % 2 == 0) // 筛选
                          .map(num -> num * 2) // 转换
                          .sorted() // 排序
                          .collect(Collectors.toList()); // 收集

// 遍历打印
result.forEach(System.out::println); // 方法引用简化，输出：8, 12, 16
```

对比可见：Stream 代码更简洁，逻辑连贯，无需手动处理循环和排序细节。

### 五、并行 Stream：轻松实现并行计算

Stream API 最强大的特性之一是**无需手动创建线程，即可实现并行处理**。只需将 `stream()` 替换为 `parallelStream()`，底层会自动分配线程处理数据（依赖 Fork/Join 框架）。

##### 示例：并行计算列表中元素的总和

```java
List<Integer> largeList = new ArrayList<>();
// 模拟一个包含100万条数据的大列表
for (int i = 0; i < 1_000_000; i++) {
    largeList.add(i);
}

// 并行计算总和（parallelStream() 开启并行）
long sum = largeList.parallelStream()
                   .filter(num -> num % 2 == 0) // 筛选偶数
                   .mapToLong(num -> num) // 转换为 long 类型（避免溢出）
                   .sum(); // 求和

System.out.println("偶数总和：" + sum);
```

##### 并行 Stream 注意事项：

1. **线程安全**：若中间操作涉及非线程安全的对象（如 `ArrayList` 的 `add` 方法），需手动保证线程安全（或使用 `collect` 而非 `forEach` 收集结果）；
2. **性能权衡**：数据量较小时，并行 Stream 的线程调度开销可能大于收益，适合**大数据量**（如万级以上）场景；
3. **避免状态依赖**：中间操作应尽量使用无状态函数（如 `num -> num * 2`），避免依赖外部变量（可能导致结果错乱）。

### 六、Stream API 的核心优势总结

1. **代码更简洁**：用链式调用替代嵌套循环和临时变量，可读性更高；
2. **开发效率更高**：内置大量现成操作（如排序、去重、聚合），无需重复造轮子；
3. **易于并行化**：一行代码实现并行处理，无需手动处理线程；
4. **函数式编程支持**：与 Lambda 表达式、方法引用无缝结合，符合现代 Java 编程风格。



### 七、数值流转换

#### 转换为普通类型

```java
Integer[] arr = {1,3,2};
int[] a = Arrays.stream(arr).mapToInt(Integer::intValue).toArray();
```

`mapToDouble()` 是 Stream API 中非常常用的一个中间操作，它属于 **数值流转换方法**，用于将普通的 `Stream<T>` 转换为 `DoubleStream`（专门处理 `double` 类型的流），方便进行数值计算（如求和、平均值、最大值等）。

##### `mapToDouble()` 的核心作用与特点：

1. **类型转换**：接收一个 `ToDoubleFunction<T>` 函数式接口（输入类型 `T`，返回 `double`），将流中的每个元素 `T` 转换为 `double` 类型，生成 `DoubleStream`。
2. **支持数值操作**：`DoubleStream` 提供了 `sum()`、`average()`、`max()`、`min()` 等专门的数值计算方法，比普通 `Stream` 更高效。
3. **链式调用**：转换后仍可进行其他中间操作（如 `filter`、`sorted`），最终通过终止操作得到结果。

##### 示例：`mapToDouble()` 的实际应用

以下示例通过 `mapToDouble()` 计算商品列表的总价和平均价格：

​    

```JAVA
import java.util.List;

// 商品类
class Product {
    private String name;
    private double price;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public double getPrice() {
        return price; // 提供获取价格的方法
    }
}

public class MapToDoubleDemo {
    public static void main(String[] args) {
        List<Product> products = List.of(
            new Product("手机", 3999.99),
            new Product("电脑", 5999.99),
            new Product("耳机", 799.99)
        );

        // 使用mapToDouble()将Stream<Product>转换为DoubleStream
        double totalPrice = products.stream()
                .mapToDouble(Product::getPrice) // 提取价格（转换为double）
                .sum(); // 求和（DoubleStream的终止操作）

        double averagePrice = products.stream()
                .mapToDouble(Product::getPrice)
                .average() // 求平均值（返回OptionalDouble）
                .orElse(0.0); // 处理空集合情况

        System.out.println("商品总价：" + totalPrice); // 输出：10799.97
        System.out.println("平均价格：" + averagePrice); // 输出：3599.99
    }
}

```





##### 类似的数值流转换方法：

除了 `mapToDouble()`，Stream API 还提供了另外两种常用的数值流转换：

- `mapToInt(ToIntFunction<T>)`：转换为 `IntStream`（处理 `int` 类型）；
- `mapToLong(ToLongFunction<T>)`：转换为 `LongStream`（处理 `long` 类型）。

这些方法的设计初衷是 **避免自动装箱/拆箱的性能损耗**，并提供更便捷的数值计算能力，是处理数值型数据时的首选。

总结

Stream API 是 Java 处理集合数据的“瑞士军刀”，其核心是“**以声明式方式串联中间操作，通过终止操作触发执行**”。掌握 `filter`、`map`、`collect` 等常用操作，结合并行 Stream，可大幅提升数据处理代码的简洁性和效率，是 Java 开发者必须掌握的核心技能之一。



#### 转换为包装类

```java
int[] a = {3,1,2};

// 把 int[] 转成 Integer[]
Integer[] arr = Arrays.stream(a)  // 得到 IntStream（int 流）
                     .boxed()    // 变成 Stream<Integer>（包装类流）
                     .toArray(Integer[]::new); // 转成数组
```



### `IntStream` 

`IntStream` 是 Java 8 引入的 **原始类型整数流**（`java.util.stream.IntStream`），专门用于处理 `int` 类型的元素，属于 Java Stream API 的一部分。它提供了丰富的中间操作和终端操作，用于高效处理整数序列，支持串行和并行处理。

#### 一、主要特点

1. **原始类型优化**：直接处理 `int` 类型，避免了自动装箱/拆箱（`int` ↔ `Integer`）的性能损耗，比 `Stream<Integer>` 更高效。
2. **流式操作**：支持链式调用的中间操作（如过滤、映射、排序）和终端操作（如统计、聚合、收集）。
3. **并行支持**：通过 `parallel()` 方法可轻松切换为并行流，利用多核 CPU 加速处理。

#### 二、创建 `IntStream` 的常用方式

1. **范围生成**：

   - `IntStream.range(int startInclusive, int endExclusive)`：生成 `[start, end)` 的整数流（不包含 `end`）。

     ```java
     IntStream.range(1, 5) // 生成 1,2,3,4
     ```

   - `IntStream.rangeClosed(int startInclusive, int endInclusive)`：生成 `[start, end]` 的整数流（包含 `end`）。

     ```java
     IntStream.rangeClosed(1, 5) // 生成 1,2,3,4,5
     ```

2. **数组转换**：

   ```java
   int[] arr = {1, 2, 3, 4};
   IntStream stream = Arrays.stream(arr); // 从数组创建流
   ```

3. **单个元素或多个元素**：

   ```java
   IntStream.of(1, 2, 3); // 直接传入元素
   IntStream.of(new int[]{1, 2, 3}); // 传入数组
   ```

4. **生成器/迭代器**：

   - `IntStream.generate(Supplier<Integer>)`：生成无限流（需配合 `limit()` 限制长度）。

   - `IntStream.iterate(int seed, IntUnaryOperator f)`：从初始值开始，通过函数迭代生成无限流。

     ```java
     IntStream.iterate(2, n -> n + 2) // 生成偶数：2,4,6,...
              .limit(5) // 限制前5个元素
              .forEach(System.out::println); // 2 4 6 8 10
     ```

#### 三、常用操作

#### 1. 中间操作（返回新的流，可链式调用）

- **过滤**：`filter(IntPredicate predicate)` 保留满足条件的元素。

  ```java
  IntStream.range(1, 10)
           .filter(n -> n % 2 == 0) // 保留偶数
           .forEach(System.out::print); // 2468
  ```

- **映射**：`map(IntUnaryOperator mapper)` 将元素转换为新的 `int` 值。

  ```java
  IntStream.range(1, 5)
           .map(n -> n * 2) // 每个元素乘以2
           .forEach(System.out::print); // 2468
  ```

- **排序**：`sorted()` 对元素自然排序（整数默认升序）。

  ```java
  IntStream.of(3, 1, 4, 2)
           .sorted()
           .forEach(System.out::print); // 1234
  ```

- **去重**：`distinct()` 去除重复元素。

  ```java
  IntStream.of(1, 2, 2, 3)
           .distinct()
           .forEach(System.out::print); // 123
  ```

- **并行化**：`parallel()` 转换为并行流（适合大数据量计算）。

##### 2. 终端操作（触发流处理，返回结果或副作用）

- **遍历**：`forEach(IntConsumer action)` 对每个元素执行操作（无返回值）。

- **统计计数**：`count()` 返回元素个数（`long` 类型）。

  ```java
  long count = IntStream.rangeClosed(1, 10).count(); // 10
  ```

- **求和/最值**：

  ```java
  int sum = IntStream.range(1, 5).sum(); // 1+2+3+4=10
  int max = IntStream.of(3, 1, 4).max().orElse(0); // 4
  int min = IntStream.of(3, 1, 4).min().orElse(0); // 1
  ```

- **聚合统计**：`summaryStatistics()` 返回包含多个统计信息的对象（总和、平均值、最值等）。

  ```java
  IntSummaryStatistics stats = IntStream.rangeClosed(1, 5).summaryStatistics();
  System.out.println("总和：" + stats.getSum()); // 15
  System.out.println("平均值：" + stats.getAverage()); // 3.0
  System.out.println("最大值：" + stats.getMax()); // 5
  ```

- **收集结果**：`toArray()` 转换为 `int[]` 数组；或通过 `collect()` 收集到集合（需装箱为 `Integer`）。

  ```java
  int[] arr = IntStream.range(1, 4).toArray(); // [1,2,3]
  
  List<Integer> list = IntStream.range(1, 4)
                               .boxed() // 转换为 Stream<Integer>
                               .collect(Collectors.toList()); // [1,2,3]
  ```

#### 四、与 `Stream<Integer>` 的区别

| 特性     | `IntStream`                      | `Stream<Integer>`              |
| -------- | -------------------------------- | ------------------------------ |
| 元素类型 | 原始类型 `int`                   | 包装类型 `Integer`             |
| 性能     | 无装箱/拆箱，效率更高            | 有装箱/拆箱开销                |
| 特有方法 | 提供 `sum()`、`max()` 等直接操作 | 需通过 `mapToInt()` 转换后使用 |
| 适用场景 | 纯整数处理场景                   | 需与其他对象流混合处理场景     |

#### 五、总结

`IntStream` 是处理整数序列的高效工具，尤其适合需要避免装箱开销、进行数值计算（如求和、统计、过滤）的场景。通过链式操作和并行支持，能简洁高效地实现复杂的整数流处理逻辑，是 Java 函数式编程中处理整数的核心 API 之一。、

### 函数式接口

在Java中，**函数式接口（Functional Interface）** 是指**只包含一个抽象方法的接口**，它是Java 8引入Lambda表达式和函数式编程的基础。Lambda表达式可以无缝适配函数式接口，使得代码更简洁、灵活。

函数式接口是Java函数式编程的核心，其单一抽象方法的特性使得Lambda表达式能够直接适配，极大简化了代码。通过`java.util.function`包的内置接口，可进一步提高开发效率，避免重复定义类似接口。

#### 核心特点：

1. **仅有一个抽象方法**：接口中只能有一个未实现的抽象方法（默认方法、静态方法、从`Object`继承的方法除外）。
2. **可被Lambda表达式实现**：函数式接口的唯一抽象方法可以通过Lambda表达式快速实现，无需传统的`implements`语法。
3. **`@FunctionalInterface`注解**：这是一个可选注解，用于显式声明接口为函数式接口。编译器会检查该接口是否符合规范（若不符合则报错），增强代码可读性。

#### 示例：

##### 1. 自定义函数式接口

```java
// 用注解声明为函数式接口
@FunctionalInterface
interface MyFunctionalInterface {
    // 唯一的抽象方法
    void doSomething(String input);
    
    // 允许有默认方法（不影响函数式接口特性）
    default void defaultMethod() {
        System.out.println("默认方法");
    }
    
    // 允许有静态方法
    static void staticMethod() {
        System.out.println("静态方法");
    }
}
```

##### 2. 用Lambda表达式实现

```java
public class Main {
    public static void main(String[] args) {
        // Lambda表达式实现函数式接口的抽象方法
        MyFunctionalInterface func = (input) -> System.out.println("处理：" + input);
        
        // 调用方法
        func.doSomething("测试"); // 输出：处理：测试
        func.defaultMethod();    // 输出：默认方法
        MyFunctionalInterface.staticMethod(); // 输出：静态方法
    }
}
```

#### Java内置的常用函数式接口：

Java 8的`java.util.function`包提供了大量预定义的函数式接口，覆盖常见场景，避免重复定义：

| 接口            | 抽象方法            | 用途                         |
| --------------- | ------------------- | ---------------------------- |
| `Runnable`      | `void run()`        | 无参数、无返回值的操作       |
| `Consumer<T>`   | `void accept(T t)`  | 接收一个参数，无返回值       |
| `Supplier<T>`   | `T get()`           | 无参数，返回一个结果         |
| `Function<T,R>` | `R apply(T t)`      | 接收T类型参数，返回R类型结果 |
| `Predicate<T>`  | `boolean test(T t)` | 接收参数，返回布尔值（判断） |

#### 内置接口示例：

```java
import java.util.function.Consumer;
import java.util.function.Predicate;

public class BuiltInExample {
    public static void main(String[] args) {
        // Consumer：消费一个字符串
        Consumer<String> printer = s -> System.out.println(s);
        printer.accept("Hello, Functional Interface!"); // 输出对应字符串
        
        // Predicate：判断数字是否为偶数
        Predicate<Integer> isEven = n -> n % 2 == 0;
        System.out.println(isEven.test(4)); // 输出：true
    }
}
```



### Lambda

在 Java 中，**Lambda 表达式**（Lambda Expression）是 **Java 8 引入的核心新特性**，用于**简化匿名内部类的写法**，使代码更简洁、可读，并支持函数式编程风格。

------

#### ✅ 一、什么是 Lambda 表达式？

Lambda 表达式是一种**匿名函数**（没有名称的方法），可以作为参数传递、赋值给变量，或直接执行。

> **本质**：提供了一种**更简洁的语法来实现函数式接口**（Functional Interface）。

------

#### ✅ 二、基本语法

```java
(参数列表) -> { 方法体 }
```

##### 常见形式：

| 场景         | Lambda 写法                         | 说明                    |
| ------------ | ----------------------------------- | ----------------------- |
| 无参无返回   | `() -> System.out.println("Hello")` |                         |
| 单参无返回   | `(x) -> System.out.println(x)`      | 可省略括号：`x -> ...`  |
| 多参有返回   | `(a, b) -> { return a + b; }`       |                         |
| 单表达式返回 | `(a, b) -> a + b`                   | 可省略 `{}` 和 `return` |

------

#### ✅ 三、使用前提：函数式接口

Lambda **只能用于函数式接口** —— 即**只有一个抽象方法的接口**。

Java 8 提供了常用函数式接口（在 `java.util.function` 包中）：

| 接口             | 抽象方法            | 用途       |
| ---------------- | ------------------- | ---------- |
| `Runnable`       | `void run()`        | 无参无返回 |
| `Consumer<T>`    | `void accept(T t)`  | 消费一个值 |
| `Supplier<T>`    | `T get()`           | 生成一个值 |
| `Function<T, R>` | `R apply(T t)`      | 转换类型   |
| `Predicate<T>`   | `boolean test(T t)` | 判断条件   |

##### 示例：

```java
// 传统写法
Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Old way");
    }
};

// Lambda 写法
Runnable r2 = () -> System.out.println("Lambda way");

// Predicate 示例
List<String> list = Arrays.asList("apple", "bat", "cat");
list.stream()
    .filter(s -> s.length() > 3)   // Predicate<String>
    .forEach(System.out::println); // Consumer<String>
```

------

#### ✅ 四、Lambda 的核心优势

1. **代码更简洁**：减少模板代码（如 `new XXX() { ... }`）
2. **支持函数式编程**：便于集合操作（配合 `Stream API`）
3. **提升可读性**：逻辑聚焦于“做什么”，而非“怎么实现”
4. **便于并行处理**：`stream().parallel()` 可轻松并行化

------

#### ✅ 五、注意事项

- ❌ 不能访问非 final 的局部变量

  （实际上是“effectively final”）

  ```java
int x = 10;
  // x = 20; // 如果取消注释，下面 Lambda 编译报错
  Runnable r = () -> System.out.println(x); // OK，x 是 effectively final
  ```
  
- ✅ 可以访问成员变量和 `this`

- ⚠️ Lambda **不是内部类**，编译后生成的是 `invokedynamic` 字节码指令（性能更好）

------

#### ✅ 六、典型应用场景

1. **集合遍历与过滤**

   ```java
   users.stream()
        .filter(u -> u.getAge() >= 18)
        .map(User::getName)
        .collect(Collectors.toList());
   ```

2. **事件监听（GUI/Android）**

   ```java
   button.setOnClickListener(v -> toast("Clicked!"));
   ```

3. **多线程**

   ```java
   new Thread(() -> {
       System.out.println("Running in thread: " + Thread.currentThread().getName());
   }).start();
   ```

------

#### ✅ 总结（面试/笔试一句话回答）

> **Lambda 表达式是 Java 8 引入的语法糖，用于简洁地实现函数式接口，使代码更紧凑、可读，并支持函数式编程和 Stream API，大幅提升集合操作和并发编程的开发效率。**

------

掌握 Lambda 是现代 Java 开发的必备技能，也是面试高频考点！如需深入 `Stream`、方法引用（`::`）等内容，可继续提问 😊



### Stream 并行流 

#### 一、核心概念

- 普通流：**串行流**，单线程逐个执行
- 并行流：**多线程并行执行**，底层默认使用 **ForkJoinPool 线程池**
- 开启方式：
  1. 集合直接调用 `.parallelStream()`
  2. 普通流调用 `.parallel()` 转为并行流

---

#### 二、基础用法

##### 1. 两种创建方式

```java
List<Integer> list = List.of(1,2,3,4,5,6,7,8,9,10);

// 方式1：直接获取并行流
list.parallelStream()
    .forEach(System.out::println);

// 方式2：串行流转并行流
list.stream()
    .parallel()
    .forEach(System.out::println);
```

##### 2. 关闭并行（转回串行）

```java
list.parallelStream()
    .sequential() // 变回串行
    .forEach(System.out::println);
```

---

#### 三、并行流特点

1. 自动多线程，无需手动创建线程
2. 基于 **Fork/Join 框架**，任务拆分+合并
3. 默认线程数 = **CPU核心数**
4. 执行顺序**完全无序**

---

#### 四、代码演示：串行 vs 并行

##### 串行（有序、单线程）

```java
// 顺序输出 1~10
list.stream()
    .forEach(System.out::print);
```

##### 并行（无序、多线程）

```java
// 乱序输出
list.parallelStream()
    .forEach(System.out::print);
```

> 若要并行且保证顺序：使用 `forEachOrdered`

```java
list.parallelStream()
    .forEachOrdered(System.out::print);
```

---

#### 五、常用并行流场景

适合：**无状态、无共享变量、密集型计算**

```java
// 求和
int sum = list.parallelStream()
              .mapToInt(Integer::intValue)
              .sum();

// 过滤+收集
List<Integer> evenList = list.parallelStream()
        .filter(x -> x % 2 == 0)
        .collect(Collectors.toList());
```

---

#### 六、高频大坑（面试必问）

##### 坑1：操作**有状态/共享变量** → 线程安全问题

并行流多线程同时操作同一个集合/变量，会并发异常

```java
// ❌ 错误：ArrayList 线程不安全
List<Integer> resList = new ArrayList<>();
list.parallelStream().forEach(resList::add);

// ✅ 正确：用线程安全集合 / collect 收集
List<Integer> res = list.parallelStream()
        .collect(Collectors.toList());
```

##### 坑2：包含 IO、数据库、Redis 操作 → 不建议

并行流**默认共用全局 ForkJoinPool**，IO 阻塞会耗尽公共线程，影响全局所有并行任务。

##### 坑3：顺序依赖业务不能用

例如：需要按顺序累加、排序强依赖顺序，并行会错乱。

##### 坑4：简单任务并行反而更慢

线程创建、任务拆分有开销，**小数据量、简单计算**，串行更快。

---

#### 七、底层原理（面试）

1. 并行流底层：`ForkJoinPool` + `ForkJoinTask`
2. 任务拆分：大数据集合拆分成多个小任务
3. 多线程并行执行
4. 最后合并结果
5. 默认线程数：`Runtime.getRuntime().availableProcessors()` CPU核心数

---

#### 八、规范使用总结

✅ 适合使用并行流

- 大数据量、纯计算密集型
- 无共享变量、无状态
- 不依赖执行顺序

❌ 禁止使用

- 集合增删改、共享变量
- 数据库、MQ、网络IO、文件IO
- 事务、缓存操作
- 小数据量简单逻辑

---

#### 九、扩展：自定义并行流线程池

解决公共线程池被耗尽问题（企业开发常用）

```java
// 自定义 ForkJoinPool
ForkJoinPool pool = new ForkJoinPool(10);
pool.submit(() -> {
    list.parallelStream().forEach(xxx);
}).join();
```

需要我给你写一段**并行流线程安全报错+修复**的完整测试代码吗？



### ForkJoin

**默认不用自定义 ForkJoinPool**，`parallelStream` 底层**自带公共ForkJoin池**，开箱即用。

---

#### 1. 默认底层用什么池

```java
list.parallelStream()
```

- 底层默认使用：**`ForkJoinPool.commonPool()`**
- 线程数默认：**CPU核心数 - 1**
- 全局共享一个公共池，**所有并行流共用**
- 无需自己创建、初始化、管理线程池

---

#### 2. 什么时候才需要**自定义 ForkJoinPool**

3种必自定义场景：

1. **不想共用全局公共池**，怕被其他并行流挤占线程
2. 需要**修改并行度（线程数）**
3. 并行流里有**阻塞IO、sleep、慢查询**，防止拖垮全局公共池

---

#### 3. 如何自定义 ForkJoinPool 执行 parallelStream

##### 原理

并行流会**继承当前线程的ForkJoin池**，所以把流放进自定义池里执行即可。

```java
// 1. 自定义池，指定并行度（线程数）
ForkJoinPool customPool = new ForkJoinPool(10); // 10个线程

// 2. 在自定义池中执行并行流
customPool.submit(() -> {
    list.parallelStream()
        .forEach(...);
}).join();

// 用完关闭
customPool.shutdown();
```

---

#### 4. 关键区别

| 方式               | 线程池         | 并行度   | 优缺点                         |
| ------------------ | -------------- | -------- | ------------------------------ |
| 直接parallelStream | 公共commonPool | CPU核-1  | 最简单，全局共享，易被阻塞拖慢 |
| 自定义ForkJoinPool | 自己新建池     | 自己指定 | 隔离任务、可控并发、不影响全局 |

---

#### 5. 开发规范

- **纯CPU密集、无阻塞**：直接用默认 `parallelStream`，**不用自定义池**
- **有IO阻塞、接口调用、sleep**：**必须自定义ForkJoinPool**，避免占满公共池
- 业务核心并行计算：建议自定义池隔离，互不干扰

---

#### 6. 面试高频答案

1. `parallelStream` 默认使用 **ForkJoinPool.commonPool**，**不需要手动定义**。
2. 默认并行度由JVM参数 `common.parallelism` 决定，默认CPU核心数减一。
3. 存在阻塞任务、需要调并发数、任务隔离时，**手动创建ForkJoinPool包裹并行流执行**。
4. 不建议在并行流里写阻塞代码，会耗尽公共池线程。

### for、Stream、parallelStream()

#### 一、总结

1. **普通操作（for/foreach）**：基础循环，单线程，简单直接，适合小数据量
2. **Stream**：流式操作，单线程，代码优雅，支持链式过滤/映射/聚合
3. **parallelStream()**：并行流，**多线程**，利用多核CPU，大数据量下速度最快

---

#### 二、核心区别对比表

| 特性               | 普通for/foreach  | Stream()               | parallelStream()     |
| ------------------ | ---------------- | ---------------------- | -------------------- |
| **线程模型**       | 单线程           | 单线程                 | **多线程（自动）**   |
| **代码简洁度**     | 繁琐             | 优雅、链式             | 同Stream             |
| **性能（小数据）** | 最快             | 稍慢                   | 最慢（线程开销大）   |
| **性能（大数据）** | 慢               | 中等                   | **最快**             |
| **线程安全**       | 安全             | 安全                   | **不安全（需注意）** |
| **顺序**           | 严格有序         | 严格有序               | **无序**             |
| **适用场景**       | 简单逻辑、小数据 | 优雅处理集合、过滤排序 | 大数据量、无状态计算 |

---

#### 三、代码示例对比（同一个需求：筛选+计算）

需求：从 1~10000 的数字中，找出偶数，然后求和。

##### 1. 普通操作（for 循环）

```java
List<Integer> list = new ArrayList<>();
// 造数据
for (int i = 1; i <= 10000; i++) list.add(i);

// 普通循环
int sum = 0;
for (int num : list) {
    if (num % 2 == 0) {
        sum += num;
    }
}
System.out.println(sum);
```

✅ 优点：最简单、无学习成本  
❌ 缺点：代码长，复杂逻辑会写成“面条代码”

---

##### 2. Stream 流式操作

```java
int sum = list.stream()
              .filter(num -> num % 2 == 0)
              .mapToInt(Integer::intValue)
              .sum();
```

✅ 优点：**一行链式搞定，可读性极强**  
✅ 支持：filter、map、sorted、collect、reduce 等

---

##### 3. parallelStream 并行流

```java
int sum = list.parallelStream()
              .filter(num -> num % 2 == 0)
              .mapToInt(Integer::intValue)
              .sum();
```

✅ 优点：**自动多线程，大数据量速度碾压前两者**  
⚠️ 底层自动使用 ForkJoinPool 线程池

---

#### 四、性能到底差多少？

我给你一个直观结论：

##### 1. **数据量小（<1000）**

**普通 for > Stream > parallelStream**  
原因：并行流创建线程、切换线程的开销，比计算本身还大。

##### 2. **数据量大（>1万 ~ 100万+）**

**parallelStream >>> Stream > 普通 for**  
多核CPU下，并行流能把速度提升 **N 倍**（N≈CPU核心数）。

---

#### 五、最关键的坑：parallelStream 必须注意

##### 1. 线程不安全

不要在并行流里修改**外部变量**（如集合、计数器），会出现数据错乱。

❌ 错误写法：

```java
List<Integer> res = new ArrayList<>();
list.parallelStream().forEach(num -> res.add(num)); // 线程不安全！
```

✅ 正确写法：

```java
List<Integer> res = list.parallelStream().collect(Collectors.toList());
```

##### 2. 不保证顺序

parallelStream 是**无序**的，依赖顺序的业务不要用。

##### 3. 不要用在阻塞/IO操作

并行流默认使用公共线程池，IO 阻塞会拖慢整个应用。

---

#### 六、开发中怎么选？（最实用指南）

##### 1. 用 **普通for/foreach**

- 逻辑极简单
- 数据量很小
- 追求极致性能（极小数据）
- 新手、团队不熟悉Stream

##### 2. 用 **Stream()**

- 想代码优雅
- 需要过滤、映射、排序、分组、聚合
- 数据量中等
- 要求顺序

##### 3. 用 **parallelStream()**

- **数据量大**
- **无状态计算**（不依赖前后元素）
- 不要求顺序
- 纯CPU计算（数学运算、集合处理）

---

#### 七、最终一句话口诀

```
小数据用普通循环，
中等数据用Stream优雅，
大数据计算用并行流狂飙！
```

#### 总结

1. **普通循环**：简单、单线程、小数据最快
2. **Stream**：优雅、单线程、适合集合链式处理
3. **parallelStream**：多线程、大数据性能王者，但要注意线程安全与无序
4. 选择核心：**看数据量 + 看是否需要并行 + 看是否要求有序**

需要我给你做一个**100万数据的真实性能测试代码**，你直接运行就能看到三者速度差距吗？

