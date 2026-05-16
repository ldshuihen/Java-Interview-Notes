

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

ConcurrentHashMap 中 CAS 和 synchronized 配合使用：
1. 桶为空时，使用 CAS 无锁插入，避免加锁开销，提高并发；
2. 桶不为空时，才使用 synchronized 锁住头节点，保证安全；
3. CAS 是 CPU 原子指令，不加锁、不阻塞，比 synchronized 更轻量；
4. 设计思想：能无锁则无锁，只在冲突时加锁，从而达到极高并发性能。

---

#### 五、终极一句话（最精髓）

##### CAS 用于无锁快速写入，synchronized 用于冲突时保证安全。

CAS 提升速度，synchronized 保证安全，二者缺一不可。


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

### 查找元素时间复杂度 + 底层原理

#### Set  **查找元素时间复杂度 + 底层原理**

（Java 高频面试题，必须背死）

##### 一句话结论

1. **HashSet**：O(1) 最快
2. **LinkedHashSet**：O(1) 快
3. **TreeSet**：O(log n) 最慢

---

##### 1. HashSet

**底层：HashMap（哈希表 + 链表/红黑树）**
- **查找时间复杂度：O(1)**
- 原理：计算 `hashCode()` 直接定位数组下标，几乎一步找到
- 极端冲突（链表）：O(n)
- 转红黑树：O(log n)
- **平均情况稳定 O(1)**

---

##### 2. LinkedHashSet

**底层：LinkedHashMap（哈希表 + 双向链表）**
- **查找时间复杂度：O(1)**
- 继承 HashSet，查找逻辑一样
- 只是多了链表保持**插入顺序**
- 性能几乎 = HashSet

---

##### 3. TreeSet

**底层：TreeMap（红黑树 有序二叉树）**
- **查找时间复杂度：O(log n)**
- 原理：从根节点二分查找，每次排除一半
- 数据越多，越比 HashSet 慢
- 作用：**自动排序**

---

##### 最直观对比（背这个）

| 集合          | 底层            | 查找元素时间复杂度 | 特点             |
| ------------- | --------------- | ------------------ | ---------------- |
| HashSet       | HashMap         | O(1)               | 最快、无序       |
| LinkedHashSet | LinkedHashMap   | O(1)               | 快、保持插入顺序 |
| TreeSet       | TreeMap(红黑树) | O(log n)           | 慢、自动排序     |

---

##### 面试标准答案

- 想**最快**：用 **HashSet** O(1)
- 想**保持插入顺序**：用 **LinkedHashSet** O(1)
- 想**自动排序**：用 **TreeSet** O(log n)

需要我给你讲 **红黑树为什么是 O(log n)** 吗？

#### Map 查找时间复杂度

##### 一、核心结论

1. **HashMap** → 平均：$\boldsymbol{O(1)}$
2. **LinkedHashMap** → 平均：$\boldsymbol{O(1)}$
3. **TreeMap** → 稳定：$\boldsymbol{O(\log n)}$

---

##### 二、逐个拆解

###### 1. HashMap

- 底层：**数组 + 链表 + 红黑树**
- 查找原理：
  1. key 计算 hashCode → 定位数组下标
  2. 下标位置直接命中：$O(1)$
  3. 哈希冲突链表：最坏 $O(n)$
  4. 树化后：最坏 $O(\log n)$
- 日常平均：**$O(1)$**

---

###### 2. LinkedHashMap

- 底层：继承 HashMap，额外维护**双向链表**
- 查询逻辑和 HashMap 完全一致
- 只是多了顺序维护（插入/访问顺序）
- 查找复杂度：**平均 $O(1)$**

---

###### 3. TreeMap

- 底层：**红黑树（自平衡二叉搜索树）**
- 按键大小排序，查找走二分遍历
- 每次比较减半，树高度 $\log n$
- 稳定查找：**$O(\log n)$**

---

##### 三、汇总对照表

| 实现类        | 底层结构           | 查询复杂度  | 特点              |
| ------------- | ------------------ | ----------- | ----------------- |
| HashMap       | 哈希表+链表+红黑树 | 平均 $O(1)$ | 无序、查询最快    |
| LinkedHashMap | 哈希表+双向链表    | 平均 $O(1)$ | 有序（插入/访问） |
| TreeMap       | 红黑树             | $O(\log n)$ | 按键自动排序      |

---

##### 四、面试加分补充

1. 为什么 TreeMap 慢？
需要**比较大小、遍历树节点**，没有哈希直接寻址快。
2. 为什么 HashMap 最快？
哈希散列直接定位数组索引，**随机访问**。
3. 对应 Set 关系：
- HashSet → 依赖 HashMap
- LinkedHashSet → 依赖 LinkedHashMap
- TreeSet → 依赖 TreeMap

### Comparable 

最全详解（用法、区别、底层、面试必背）

#### 一、是什么

`Comparable` 是**内部比较器**，**实体类自己实现**比较规则，自带排序能力。

位于包：`java.lang.Comparable`

```java
public interface Comparable<T> {
    // 返回值：负数 / 0 / 正数
    public int compareTo(T o);
}
```

---

#### 二、返回值规则（死记）

假设：`this` 当前对象，`o` 传入对象

- `this > o`  return **正数**
- `this == o` return **0**
- `this < o`  return **负数**

口诀：
**自己减别人，升序；别人减自己，降序**

---

#### 三、实战写法（实体类实现）

##### 1. 实体类实现 Comparable

```java
@Data
public class User implements Comparable<User> {
    private Integer age;
    private String name;

    // 按年龄 升序
    @Override
    public int compareTo(User o) {
        // 核心：this.age - o.age
        return this.age - o.age;
    }
}
```

##### 2. 直接排序

```java
List<User> list = new ArrayList<>();
list.add(new User(20,"张三"));
list.add(new User(18,"李四"));

// 直接排序，不用额外规则
Collections.sort(list);
```

---

#### 四、常用场景

1. `Collections.sort(集合)`
2. `Arrays.sort(数组)`
3. **TreeSet / TreeMap** 无指定比较器时，默认用 `Comparable`

> TreeSet 元素必须实现 Comparable，否则直接报类型转换异常

---

#### 五、Comparable 和 Comparator 区别（面试必考）

| 特性     | Comparable     | Comparator                    |
| -------- | -------------- | ----------------------------- |
| 名称     | 内部比较器     | 外部比较器                    |
| 位置     | 实体类实现接口 | 单独写一个比较类 / 匿名内部类 |
| 侵入性   | 改实体类代码   | 不改动实体类                  |
| 排序规则 | 固定一套       | 可灵活多套排序规则            |
| 方法     | compareTo()    | compare()                     |

##### 简单一句话

- **Comparable**：我自己跟别人比，写死在实体类里
- **Comparator**：第三方帮两个对象比，灵活临时排序

---

#### 六、Lambda 简化（JDK8）

不用改实体类，临时排序用 Comparator：
```java
// 年龄升序
list.sort(Comparator.comparing(User::getAge));
// 年龄降序
list.sort(Comparator.comparing(User::getAge).reversed());
```

---

#### 七、底层原理

1. 调用 `sort` 时，内部遍历集合
2. 不断调用 `compareTo()` 两两比较
3. 根据返回正负值决定交换位置
4. TreeMap/TreeSet 底层红黑树插入时，依赖 `compareTo` 定位节点

---

#### 八、背诵总结

1. `Comparable` 是**内部比较器**，接口只有 `compareTo()`
2. 规则：**自己减别人升序**
3. 实体类实现后，可直接 `Collections.sort`、放入 TreeSet
4. 固定排序用 Comparable，**临时多规则排序用 Comparator**

需要我给你写 **Comparable 升序、降序、多字段排序（年龄+姓名）** 完整示例吗？

### Comparator 

详细讲解（用法、区别、Lambda、多字段排序、面试必背）

#### 一、是什么

`Comparator`：**外部比较器**
- 不用修改实体类源码
- 临时定义**灵活排序规则**
- JDK 1.2 接口，JDK8 后大量默认方法、支持 Lambda

接口核心抽象方法：
```java
int compare(T o1, T o2);
```

#### 二、返回值规则（和 Comparable 一样）

- 返回 **负数**：o1 放前面
- 返回 **0**：相等
- 返回 **正数**：o2 放前面

口诀：
**o1 - o2 升序**
**o2 - o1 降序**

---

#### 三、三种写法演示

##### 1. 匿名内部类（传统写法）

```java
List<User> list = new ArrayList<>();

Collections.sort(list, new Comparator<User>() {
    @Override
    public int compare(User o1, User o2) {
        // 按年龄升序
        return o1.getAge() - o2.getAge();
    }
});
```

##### 2. Lambda 写法（JDK8 推荐）

```java
// 年龄升序
list.sort((o1, o2) -> o1.getAge() - o2.getAge());

// 年龄降序
list.sort((o1, o2) -> o2.getAge() - o1.getAge());
```

##### 3. 静态工厂方法（最简洁 企业常用）

```java
// 年龄升序
list.sort(Comparator.comparingInt(User::getAge));

// 年龄降序
list.sort(Comparator.comparingInt(User::getAge).reversed());
```

---

#### 四、多字段组合排序（重点）

**先按年龄升序，年龄相同按姓名字典序升序**
```java
list.sort(
    Comparator.comparingInt(User::getAge)
              .thenComparing(User::getName)
);
```

先按**年龄倒序（降序）**

年龄相同，再按**姓名倒序（降序）**

```java
list.sort(
    Comparator.comparingInt(User::getAge).reversed()
              .thenComparing(User::getName).reversed()
);
```



链式连用：

- `comparingXXX`：第一排序字段
- `thenComparingXXX`：等值时第二排序字段

---

#### 五、Comparator 适用场景

1. 实体类**不能改源码**，又要排序
2. 同一实体**多种排序规则**（年龄、姓名、薪资灵活切换）
3. `TreeSet / TreeMap` 自定义排序，不依赖实体 `Comparable`
4. Stream 流排序：
```java
list.stream()
    .sorted(Comparator.comparingInt(User::getAge).reversed())
    .collect(Collectors.toList());
```

---

#### 六、Comparable vs Comparator 面试必背对比

| 维度     | Comparable         | Comparator       |
| -------- | ------------------ | ---------------- |
| 类型     | 内部比较器         | 外部比较器       |
| 所在包   | java.lang          | java.util        |
| 方法     | compareTo(this, o) | compare(o1, o2)  |
| 侵入性   | 要修改实体类代码   | 不改动实体       |
| 排序规则 | 固定一套           | 可多套、随时切换 |
| 适用     | 默认固定排序       | 临时/多规则排序  |

一句话总结：
- **Comparable**：我自己自带排序规则，写死在类里
- **Comparator**：外人帮我比，灵活换规则、不改实体

---

#### 七、开发规范用法

优先顺序：
1. 默认固定排序 → 用 **Comparable**
2. 临时、多规则、不想改实体 → 用 **Comparator + comparing + thenComparing**

---

需要我给你写一份 **TreeSet 用 Comparator 自定义排序** 的完整可运行代码吗？

#### Comparator 排序 4 个方法

---

##### 一、Comparator 排序 4 个核心方法

###### 1. `comparingInt()`

专门用于：**int 类型字段**（年龄、序号、数量）
```java
// 按年龄 int 排序
list.sort(Comparator.comparingInt(User::getAge));
```

###### 2. `comparingLong()`

专门用于：**long 类型字段**（ID、时间戳）
```java
// 按 ID long 排序
list.sort(Comparator.comparingLong(User::getId));
```

###### 3. `comparingDouble()`

专门用于：**double 类型字段**（金额、分数）
```java
// 按余额 double 排序
list.sort(Comparator.comparingDouble(User::getBalance));
```

###### 4. `comparing()`

通用型，用于：**包装类型、String、对象**
```java
// 字符串姓名
list.sort(Comparator.comparing(User::getName));
```

---

##### 二、最常用组合：多字段排序（企业最常用）

```java
list.sort(
    Comparator.comparingInt(User::getAge)    // int 年龄
              .thenComparing(User::getName)   // String 姓名
);
```

---

##### 三、快速记忆表（背这个就够）

| 方法名              | 适用类型              | 例子             |
| ------------------- | --------------------- | ---------------- |
| **comparingInt**    | int                   | age、status、num |
| **comparingLong**   | long                  | id、createTime   |
| **comparingDouble** | double                | money、score     |
| **comparing**       | String、Integer、对象 | name、title      |

---

##### 四、为什么要用 comparingInt，不直接用 comparing？

因为：
- **comparingInt 避免装箱拆箱**
- **性能更好**
- **代码更清晰**

比如：
```java
// 不推荐：Integer 装箱拆箱，性能差
comparing(User::getAge)

// 推荐：int 原生类型，性能高
comparingInt(User::getAge)
```

---

##### 总结

你看到的 **`comparingInt`** 是专门给 **int 字段** 用的。

对应的还有：
- **comparingLong**
- **comparingDouble**
- **comparing（通用）**

这四个就是 Java 排序**最标准、最常用、企业真实写法**。

需要我给你演示 **降序 + 多字段组合 + 空值安全排序** 吗？

#### 对象排序 两种标准写法（Comparable + Comparator）

给 **自定义对象 List<User> 排序**，就两种方案，直接记模板就能用。

先准备实体：
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
    private Integer age;
    private BigDecimal salary;
}
```

---

##### 一、方式一：实现 Comparable（内部比较器）

**实体类自己定义默认排序规则**，改实体代码。

###### 1. 实现接口重写 compareTo

```java
@Data
public class User implements Comparable<User> {
    private Integer id;
    private String name;
    private Integer age;

    // 规则：按年龄升序，年龄相同按姓名升序
    @Override
    public int compareTo(User o) {
        // 先比年龄
        int res = this.age.compareTo(o.age);
        // 年龄相等再比姓名
        if (res == 0) {
            res = this.name.compareTo(o.name);
        }
        return res;
    }
}
```

###### 2. 使用排序

```java
List<User> list = new ArrayList<>();
// 直接排，不用写比较规则
Collections.sort(list);
```

适用：**对象有固定默认排序**。

---

##### 二、方式二：Comparator 外部比较器（推荐）

**不改实体类**，灵活随时换排序规则，企业最常用。

###### 1. JDK8 最简链式写法（必背）

单字段排序

```java
// 年龄升序
list.sort(Comparator.comparingInt(User::getAge));

// 年龄降序
list.sort(Comparator.comparingInt(User::getAge).reversed());

// 姓名字符串升序
list.sort(Comparator.comparing(User::getName));

// 姓名降序
list.sort(Comparator.comparing(User::getName).reversed());
```

多字段组合排序（高频）

需求：**年龄降序 → 同年龄姓名升序**
```java
list.sort(
    Comparator.comparingInt(User::getAge).reversed()
              .thenComparing(User::getName)
);
```

需求：**年龄升序 → 同年龄薪资降序**
```java
list.sort(
    Comparator.comparingInt(User::getAge)
              .thenComparing(User::getSalary, Comparator.reverseOrder())
);
```

###### 2. 老式匿名内部类（了解即可）

```java
Collections.sort(list, new Comparator<User>() {
    @Override
    public int compare(User o1, User o2) {
        // 年龄降序
        return o2.getAge() - o1.getAge();
    }
});
```

###### 3. Lambda 简写

```java
list.sort((o1, o2) -> o1.getAge() - o2.getAge());
```

---

##### 三、BigDecimal 对象排序

不能用 `-` 减法，只能用 `compareTo`
```java
// 薪资升序
list.sort(Comparator.comparing(User::getSalary));

// 薪资降序
list.sort(Comparator.comparing(User::getSalary).reversed());
```

---

##### 四、核心区别总结

1. **Comparable**
- 实体类实现接口，写死默认排序
- 侵入实体代码，只能一套规则

2. **Comparator**
- 外部临时规则，**不改实体**
- 支持多套规则、多字段链式排序
- 开发首选

---

##### 五、万能模板直接抄

```java
// 单字段升序
list.sort(Comparator.comparingInt(User::getAge));

// 单字段降序
list.sort(Comparator.comparingInt(User::getAge).reversed());

// 多字段：A降序，B升序
list.sort(
    Comparator.comparingInt(User::getAge).reversed()
              .thenComparing(User::getName)
);
```