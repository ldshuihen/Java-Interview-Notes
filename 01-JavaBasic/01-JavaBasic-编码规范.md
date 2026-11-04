# 十二、Java编码规范

## 1、空指针 NPE 相关（最高频）
1. `str != null` 和 `isEmpty()` 的区别
2. `isEmpty()`、`isBlank()`、`hasLength()`、`hasText()` 四者区别
3. Objects.nonNull、Optional判空优缺点
4. 为什么调用方法前必须先判断对象不等于 null
5. `String str = null; "test".equals(str)` 为什么不会空指针，常量前置规范
6. List集合判空：`list != null` 和 `!list.isEmpty()`
7. Map判空、数组判空
8. 基本类型和包装类默认值引发NPE（Integer默认null、int默认0）
9. 方法返回包装类、自动拆箱空指针
10. 三元运算符包装类隐式拆箱坑
```java
Integer a = null;
Integer b = true ? a : 1; // NPE
```

## 2、String 字符串踩坑
1. String、StringBuilder、StringBuffer区别、线程安全
2. String 不可变原理，final字节数组
3. `new String("a")` 创建几个对象
4. intern() 常量池机制、JDK7之后变化
5. 字符串+=拼接底层编译后是StringBuilder，循环拼接性能灾难
6. 空格、全角空格、空白字符判空坑
7. equals 和 == 在字符串对比区别
8. String长度上限

## 3、基本数据类型 & 包装类大坑
1. 8种基础类型默认值、包装类默认null
2. 装箱、拆箱底层原理、性能损耗
3. Integer缓存池 -128~127，==比较陷阱
```java
Integer i1 = 127;
Integer i2 = 127;
Integer i3 = 128;
Integer i4 = 128;
// i1==i2 true；i3==i4 false
```
4. Long、Short、Byte缓存池
5. Double、Float没有缓存池
6. 包装类之间 == 和 equals
7. int和Integer互相比较规则
8. 基础类型优先使用，包装类适合允许null的场景

## 4、浮点数 BigDecimal 精度坑
1. double、float精度丢失原因
2. `0.1+0.2 !=0.3`
3. BigDecimal构造禁止直接传入double，要用字符串入参
4. BigDecimal比较必须用compareTo，禁止 ==、equals
5. 除法必须设置舍入模式，否则除不尽抛出异常
6. 金额业务禁止double，强制BigDecimal

## 5、equals、hashCode 编码规范坑
1. == 和 equals 区别
2. 重写equals必须重写hashCode
3. hashSet存入对象不重写hashCode造成重复数据
4. 原生Object‑equals地址对比
5. 字符串equals、包装类equals
6. 自定义实体类equals容易漏掉字段
7. 避免null调用equals

## 6、运算符、逻辑、三元运算符坑
1. && 短路与、||短路或，& | 非短路区别
2. `i++` 和 `++i`
3. 移位运算 >> 算术右移、>>>逻辑无符号右移
4. int溢出问题，int最大值+1变成负数
5. long赋值不加L后缀默认int溢出
```java
long num = 1000 * 1000 * 1000 * 1000; // int溢出
```
6. 三元运算符自动类型转换
7. 浮点数 == 判断，禁止直接对比两个double

## 7、数组编码坑
1. 数组长度固定，初始化之后不能扩容
2. 数组默认初始化值
3. 二维数组
4. 数组判空：数组对象 != null && array.length>0
5. Arrays.equals 和 ==
6. 数组不能直接转为集合之后add报错（asList返回固定大小列表）
```java
List<Integer> list = Arrays.asList(1,2,3);
list.add(4); // 抛出UnsupportedOperationException
```

## 8、Collection、Map 集合全套坑点
### List
1. ArrayList默认初始化容量、扩容机制、线程不安全
2. LinkedList双向链表，随机访问慢
3. 遍历ArrayList时for‑remove并发修改异常
4. 增强for循环删除元素报错原因
5. CopyOnWriteArrayList写时复制原理与缺点
6. subList子列表强关联原集合，原集合改动子列表异常

### Set
1. HashSet底层HashMap、元素无序
2. LinkedHashSet有序、TreeSet可排序
3. TreeSet元素必须实现Comparable接口，否则报错

### Map
1. HashMap JDK7与8区别、头插法死循环、线程不安全
2. HashMap允许key、value null；HashTable不允许
3. ConcurrentHashMap线程安全
4. HashMap扩容、红黑树转换阈值
5. foreach遍历map过程修改元素抛出并发修改异常
6. LinkedHashMap实现LRU缓存
7. key为自定义对象必须重写equals、hashCode

### 通用集合规范
1. 集合判空优先 isEmpty()，效率高于size()>0
2. 集合尽量指定初始化容量，减少频繁扩容
3. 不要返回集合引用造成外部篡改内部成员变量
4. 使用Collections.unmodifiableList返回只读集合

## 9、枚举Enum编码坑
1. 枚举本质继承Enum、不能继承其他类
2. 枚举单例是最佳单例实现
3. 枚举使用==对比，不需要equals
4. switch支持枚举
5. 枚举构造方法默认私有

## 10、日期时间类经典坑（大厂高频）
1. SimpleDateFormat非线程安全，全局静态对象会报错
2. Date过时，JDK8推荐 LocalDateTime / LocalDate / LocalTime
3. java.sql.Date 和 java.util.Date
4. 时间戳毫秒、秒混淆
5. 时区问题，LocalDateTime不带时区，ZonedDateTime带时区
6. 日期格式化、月份MM和分钟mm大小写错误
7. 闰年、月末日期转换异常

## 11、异常 try‑catch‑finally 坑点
1. finally一定会执行吗？System.exit直接终止不会执行
2. try、catch里面return，finally执行顺序
3. finally修改返回值坑
4. 受检异常Exception、运行时异常RuntimeException、Error
5. 不要catch顶级Exception粗暴捕获所有异常
6. 禁止在finally块抛出异常
7. 业务异常自定义异常，不要直接抛出RuntimeException
8. 异常不要吞掉，必须打印日志

## 12、IO、序列化坑点
1. IO流用完必须close；try‑with‑resources自动关闭资源
2. Serializable序列化、transient关键字字段不会序列化
3. serialVersionUID不定义，类微小改动反序列化失败
4. 静态成员变量不会参与序列化
5. 对象循环引用序列化报错

## 13、内部类、匿名内部类、Lambda坑
1. 匿名内部类只能访问final修饰局部变量（JDK8隐式final）
2. 静态内部类不持有外部类对象；成员内部类持有外部类引用容易内存泄漏
3. Lambda不能修改外部局部变量
4. this在lambda、匿名内部类指向问题

## 14、static、final、this、super编码坑
1. static属于类、不属于实例对象
2. static代码块仅类加载执行一次
3. 实例代码块每次new对象执行，早于构造方法
4. final修饰变量必须初始化且不可二次赋值；final方法不能重写、final类禁止继承
5. 父类构造方法优先子类
6. 构造方法不能使用this调用自身；super必须放在构造方法第一行

## 15、线程、并发日常编码坑
1. 实例变量非线程安全，多线程共享成员变量出错
2. ThreadLocal内存泄漏、用完remove
3. sleep不会释放锁、wait释放锁
4. synchronized锁字符串常量池陷阱
5. 线程池禁止使用Executors快捷创建，容易OOM
6. 不要直接调用run()，必须start()启动线程
7. volatile不能保证原子性

## 16、重载Overload、重写Override坑
1. 重载：方法名相同、参数列表不同，和返回值无关
2. 重写：子类方法修饰符权限不能比父类严格、返回值协变、不能抛出更大受检异常
3. 静态方法不能被重写，只是隐藏
4. private方法不能重写

## 17、自动类型转换、强制转换坑
1. 大容量转小容量强制转换溢出
2. char和int转换
3. 父类引用指向子类对象向下转型异常ClassCastException
4. instanceof类型判断规范

## 18、内存泄漏日常编码坑（代码层面）
1. 静态集合持有大量对象
2. ThreadLocal未清理
3. 长生命周期对象持有短生命周期引用
4. 匿名内部类持有外部类
5. 未关闭socket、io、数据库连接

## 19、JDK8+新特性编码坑
1. Lambda表达式禁止修改外部局部变量
2. Stream流式操作只能消费一次，关闭之后不可复用
3. Optional使用误区，直接get()依然会空指针
4. Optional不要作为方法参数
5. 接口default方法，多接口相同default方法子类必须重写解决冲突

## 20、阿里开发手册硬性编码规范类考题
1. 所有POJO属性使用包装类型
2. 常量全部大写，下划线分割
3. 方法参数超过5个封装DTO
4. 禁止魔法值，定义常量
5. 循环内不要创建对象
6. 字符串拼接循环优先StringBuilder
7. 日期禁止使用SimpleDateFormat静态实例
8. 集合初始化指定容量
9. 重写toString、equals、hashCode
10. 枚举存放固定状态码
11. 注释规范、方法javadoc
12. 禁止返回null集合，返回空集合
```java
return Collections.emptyList();
```

## 21、if‑else、switch、分支坑
1. switch‑case 记得break穿透问题
2. if判断顺序，先判空再取属性
3. 浮点数禁止switch
4. 过多if‑else优先策略模式、枚举、Map路由

## 22、传参、值传递坑
1. Java只有值传递
2. 基础类型传参拷贝副本，方法内修改不会影响原值
3. 对象参数传递引用副本，方法修改对象内部属性会生效
4. 方法内直接new对象无法修改外部原引用

## 23、方法返回值坑
1. 返回集合、数组警惕外部修改内部数据
2. 禁止返回null，优先空容器、Optional
3. 包装类返回值自动拆箱NPE

## 24、标识符命名规范坑
1. 大驼峰类名、小驼峰方法变量、全大写常量
2. 包名小写
3. 布尔变量禁止is开头造成序列化、getter陷阱
