

## 九、数学公式

在 Java 中，**没有直接的“数学公式”语法**（像数学课本那样写 `x² + y² = r²`），但 Java 提供了强大的 **数学函数库**（主要是 `java.lang.Math` 类）来实现各种数学运算。

**常用运算API**

1. 取整
   - round()：四舍五入
   - ceil()：向上取整
   - floor()：向下取整
2. 最值
   - max(a,b)、min(a,b)
3. 绝对值 abs()
4. 幂次、开方
   - pow(a,b) a的b次方
   - sqrt() 平方根
   - cbrt() 立方根
5. 三角函数 sin、cos、tan、弧度运算
6. 随机数 Math.random() 返回 [0,1) double数值

---

### 一、基本算术运算（直接支持）

Java 原生支持：

```java
int a = 10, b = 3;
int sum = a + b;      // 加
int diff = a - b;     // 减
int prod = a * b;     // 乘
int div = a / b;      // 除（整数除法）
int mod = a % b;      // 取模（余数）
```

> ⚠️ 注意：`/` 对整数是**截断除法**（如 `5 / 2 = 2`），对浮点数是精确除法（`5.0 / 2 = 2.5`）。

---

### 二、`Math` 类：核心数学函数库

`java.lang.Math` 是 **静态工具类**，无需导入（属于 `java.lang` 包）。

#### 1. 常用常量

```java
double pi = Math.PI;        // π ≈ 3.141592653589793
double e = Math.E;          // 自然对数底 e ≈ 2.718281828459045
```

#### 2. 幂运算与开方

| 数学公式      | Java 写法                     |
| ------------- | ----------------------------- |
| $x^y$         | `Math.pow(x, y)`              |
| $\sqrt{x}$    | `Math.sqrt(x)`                |
| $\sqrt[3]{x}$ | `Math.cbrt(x)`                |
| $x^2$         | `x * x`（比 `pow(x,2)` 更快） |

```java
double x = 4.0;
System.out.println(Math.pow(x, 3)); // 64.0
System.out.println(Math.sqrt(x));   // 2.0
```

#### 3. 对数与指数

| 数学公式                         | Java 写法                   |
| -------------------------------- | --------------------------- |
| $e^x$                            | `Math.exp(x)`               |
| $\ln x$（自然对数）              | `Math.log(x)`               |
| $\log_{10} x$                    | `Math.log10(x)`             |
| $\log_b x = \frac{\ln x}{\ln b}$ | `Math.log(x) / Math.log(b)` |

```java
System.out.println(Math.log(Math.E));    // 1.0
System.out.println(Math.log10(1000));    // 3.0
```

#### 4. 三角函数（**参数是弧度！**）

| 数学公式         | Java 写法                    |
| ---------------- | ---------------------------- |
| $\sin x$         | `Math.sin(x)`                |
| $\cos x$         | `Math.cos(x)`                |
| $\tan x$         | `Math.tan(x)`                |
| $\arcsin x$      | `Math.asin(x)`               |
| $\arccos x$      | `Math.acos(x)`               |
| $\arctan x$      | `Math.atan(x)`               |
| $\arctan2(y, x)$ | `Math.atan2(y, x)`（更安全） |

> 🔁 角度转弧度：`Math.toRadians(degrees)`  
> 🔁 弧度转角度：`Math.toDegrees(radians)`

```java
double angle = 90.0;
double rad = Math.toRadians(angle);
System.out.println(Math.sin(rad)); // ≈1.0
```

#### 5. 取整与舍入

| 功能     | Java 方法                             | 说明                             |
| -------- | ------------------------------------- | -------------------------------- |
| 向下取整 | `Math.floor(x)`                       | ≤ x 的最大整数（返回 double）    |
| 向上取整 | `Math.ceil(x)`                        | ≥ x 的最小整数                   |
| 四舍五入 | `Math.round(x)`                       | 返回 `long`（对 float 返回 int） |
| 截断小数 | `(int)x` 或 `Math.floor(x)`（正数时） | 不是 `Math` 方法                 |
| 取绝对值 | `Math.abs(x)`                         | 支持 int/long/float/double       |

```java
System.out.println(Math.floor(3.7));  // 3.0
System.out.println(Math.ceil(3.2));   // 4.0
System.out.println(Math.round(3.6));  // 4 (long)
System.out.println(Math.abs(-5));     // 5
```

#### 6. 最大值、最小值、符号

```java
Math.max(a, b);   // 返回较大值
Math.min(a, b);   // 返回较小值
Math.signum(x);   // 返回符号：-1.0, 0.0, 或 1.0
```

#### 7. 随机数

```java
Math.random(); // 返回 [0.0, 1.0) 的 double 随机数
// 生成 [a, b) 的整数： (int)(Math.random() * (b - a)) + a
1. Random固定种子每次生成随机序列一模一样
2. Math.random()线程竞争，高并发优先ThreadLocalRandom
```

> 更推荐使用 `java.util.Random` 或 `ThreadLocalRandom`（功能更强）。

1. java.util.Random
   - 伪随机、可设置种子；多线程环境性能差
2. ThreadLocalRandom（JDK7，推荐并发场景）
   - 线程隔离、高性能
3. SecureRandom 密码加密、验证码、安全场景强随机数

---

### 三、大数运算（超出 `long` 范围）

当数字超过 `long`（约 9e18）时，用：

- `BigInteger`：任意精度整数
- `BigDecimal`：任意精度小数（适合金融计算）

```java
import java.math.BigInteger;

BigInteger a = new BigInteger("123456789012345678901234567890");
BigInteger b = new BigInteger("987654321098765432109876543210");
BigInteger sum = a.add(b); // 加法
BigInteger pow = a.pow(10); // 幂运算
```

#### 一、核心定位

1. **BigInteger**
   处理**超大整数**，超出 `long` 范围（`long` 最大：`9223372036854775807`）。
2. **BigDecimal**
   处理**高精度小数**，解决 `double/float` 浮点丢失精度问题，**金融、金额、计算必用**。

---

#### 二、为什么不用 double？

```java
System.out.println(0.1 + 0.2);
// 输出：0.30000000000000004
```

二进制浮点存储缺陷，**绝对不能用于金额计算**。

---

#### 一、BigInteger 超大整数

##### 1. 创建对象

```java
// 方式1：字符串构造（推荐）
BigInteger num1 = new BigInteger("99999999999999999999");
// 方式2：数值转
BigInteger num2 = BigInteger.valueOf(100);
// 常量
BigInteger zero = BigInteger.ZERO;
BigInteger one = BigInteger.ONE;
BigInteger ten = BigInteger.TEN;
```

##### 2. 常用运算（**不可变，运算返回新对象**）

```java
BigInteger a = new BigInteger("100");
BigInteger b = new BigInteger("30");

BigInteger add = a.add(b);        // 加
BigInteger sub = a.subtract(b);   // 减
BigInteger mul = a.multiply(b);    // 乘
BigInteger div = a.divide(b);      // 除
BigInteger rem = a.remainder(b);   // 取余
BigInteger abs = a.abs();          // 绝对值
```

##### 3. 比较

```java
// 返回：-1 小于  0 等于  1 大于
int compare = a.compareTo(b); 

if (a.compareTo(b) > 0) {
    System.out.println("a > b");
}
```

---

#### 二、BigDecimal 高精度小数（重点）

##### 1. 创建对象（避坑）

```java
// ✅ 推荐：字符串构造，无精度丢失
BigDecimal d1 = new BigDecimal("0.1");
// ❌ 禁止：double构造，自带精度问题
BigDecimal d2 = new BigDecimal(0.1);
// 快速创建
BigDecimal d3 = BigDecimal.valueOf(0.2);
```

##### 2. 基础运算

```java
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");

BigDecimal add = a.add(b);
BigDecimal sub = a.subtract(b);
BigDecimal mul = a.multiply(b);
// 除法必须指定 保留位数+舍入模式，否则报错
BigDecimal div = a.divide(b, 2, RoundingMode.HALF_UP);
```

##### 3. 重点：除法必写舍入模式

不写会抛出：`ArithmeticException: Non-terminating decimal expansion`
常用舍入模式：

- `RoundingMode.HALF_UP`：**四舍五入**（最常用，金额）
- `RoundingMode.FLOOR`：向下取整
- `RoundingMode.CEILING`：向上取整

##### 4. 比较大小（禁止用 ==、equals）

```java
// 正确：compareTo
if (a.compareTo(b) > 0) {
    // a 大于 b
}
if (a.compareTo(BigDecimal.ZERO) == 0) {
    // 判断是否为0
}
```

- `equals` 会比较**小数位数**，`new BigDecimal("1.0").equals(new BigDecimal("1")) → false`

##### 5. 保留小数、格式化

```java
// 保留2位小数，四舍五入
BigDecimal num = new BigDecimal("1.2345");
BigDecimal res = num.setScale(2, RoundingMode.HALF_UP);
// 结果：1.23
```

##### 6. 类型转换

```java
int intVal = bigDecimal.intValue();
double doubleVal = bigDecimal.doubleValue();
String strVal = bigDecimal.toString();
```

---

#### 三、核心特性（面试常问）

1. **不可变类**
   每次加减乘除都会**生成新对象**，原对象不变。
2. 性能比基础类型慢，适合**金额、超大数**，普通计算不用。
3. BigDecimal 解决浮点精度丢失，是金融项目强制规范。

---

#### 四、高频工具写法（开发直接用）

```java
/**
 * 金额相加
 */
public static BigDecimal add(BigDecimal a, BigDecimal b) {
    if (a == null) a = BigDecimal.ZERO;
    if (b == null) b = BigDecimal.ZERO;
    return a.add(b);
}

/**
 * 除法，默认保留2位小数 四舍五入
 */
public static BigDecimal div(BigDecimal a, BigDecimal b) {
    if (b.compareTo(BigDecimal.ZERO) == 0) {
        throw new RuntimeException("除数不能为0");
    }
    return a.divide(b, 2, RoundingMode.HALF_UP);
}
```

---

#### 五、总结速记

1. **BigInteger**：超大整数，替代 long。
2. **BigDecimal**：高精度小数，金额专用，解决 double 精度丢失。
3. 构造优先用**字符串**，禁止 double 直接构造。
4. 除法必须指定**保留位数+舍入模式**。
5. 比较用 `compareTo`，不要用 `equals`、`==`。
6. 都是不可变类，运算结果需接收返回值。

需要我给你写一段 **0.1+0.2 用BigDecimal正确计算** 的完整测试代码吗？

#### BigInteger & BigDecimal 底层原理

核心共性：**不可变、数组存数据、十进制精准运算、无二进制浮点误差**

---

##### 一、共同底层核心

1. **不可变类**
   所有运算 `add/subtract/multiply` 不会修改原对象，**每次返回全新对象**。
   原因：内部数值私有+无set方法，线程安全、适合缓存。

2. **不使用二进制存储**
   int/long/double 都是**二进制存储**，导致小数精度丢失；
   BigInteger/BigDecimal 基于**十进制数组**存储，天然精准。

---

##### 二、BigInteger 底层原理（超大整数）

###### 1. 核心存储结构

```java
// 源码核心属性
final int[] mag;   // 存储大数字【绝对值】，按高位→低位分段存储
final int signum;  // 符号位：1正数  -1负数  0=数字0
```

- 超大数字拆分成**int数组**分段保存，突破 long 最大值限制；
- 不再用基本类型存数值，理论无限大。

###### 2. 运算原理

加减乘除、取模：
完全模拟**手写数学计算逻辑**：

- 加法：数组逐位进位
- 减法：数组逐位借位
- 乘法：大数分段相乘+移位合并
- 除法：循环试商

###### 3. 为什么不会溢出？

基本类型有固定bit长度；
BigInteger 靠 **动态int数组** 扩容存储，无上限。

---

##### 三、BigDecimal 底层原理（高精度小数）

###### 1. 核心存储结构

```java
// 源码核心属性
private final BigInteger intVal; // 完整无缩放整数
private final int scale;         // 小数位数（缩放值）
private final int precision;     // 总有效位数
```

###### 存储逻辑（关键）

把**小数 → 纯整数**存储，彻底避开二进制浮点：
例：`3.14`

- intVal = 314
- scale = 2
  真实值计算公式：
  \[
- 数值 = $$\frac{intVal}{10^{scale}}$$
  \]

###### 2. 精度丢失根源对比

- double：二进制科学计数法存储 → 0.1 无法精准表示
- BigDecimal：十进制整数+缩放 → 0.1 就是 1 / 10，**完全精准**

###### 3. 运算底层逻辑

1. 加减法：统一对齐 scale 小数位，转成大整数运算，再还原缩放
2. 乘法：整数相乘，scale 相加
3. 除法：**无限循环小数必须指定舍入模式**，否则直接抛异常

---

##### 四、高频面试点

###### 1. 为什么 BigDecimal 不用 equals 比较？

`equals` 同时比较 **数值 + scale**

- new BigDecimal("1.0")  scale=1
- new BigDecimal("1")    scale=0
  👉 equals = false，但数值相等
  ✅ 正确：`compareTo()` 只比数值

###### 2. 为什么除法必须加舍入模式？

二进制/十进制无限小数（1÷3），无法用有限数组存储，
不指定舍入策略 → 直接抛出 `ArithmeticException`

###### 3. 为什么不推荐 new BigDecimal(double)？

double 本身已经二进制失真，传入时误差已经存在，无法挽回；
✅ 推荐：`new BigDecimal("0.1")` / `BigDecimal.valueOf(0.1)`

---

##### 五、底层性能缺点

1. 基于数组+对象运算，**远慢于 int/long/double**
2. 每次运算新建对象，内存开销大
3. 只适合：金额、高精度计算、超大数；普通业务禁用

---

##### 六、一句话速记

1. **BigInteger**：int数组存大数+符号位，模拟手工数学运算，解决超大整数。
2. **BigDecimal**：大整数+小数位数scale，小数转整数存储，根治浮点精度丢失。
3. 两者都是**不可变**，运算产生新对象，适合金融、高精度场景。

#### BigDecimal/BigInteger 应用场景



##### 一、一句话区分

- **BigDecimal**：**金融小数**（金额、利率、汇率、手续费）
- **BigInteger**：**超大整数**（无小数、远超 long 范围）

---

##### 二、long 上限先记住

```java
// long 最大值
Long.MAX_VALUE = 9223372036854775807
```

一旦数字**超过这个长度**，普通整型全部存不下，只能用 `BigInteger`。

---

##### 三、BigInteger 真实业务用途（面试+开发）

###### 1. 超大序列号/流水号/唯一ID

- 订单号、设备唯一编码、雪花算法极端扩容场景
- 物联网设备ID、硬件序列号，数字超长
- 银行/政务超大流水编号，纯整数、无小数

###### 2. 密码学、加密算法（高频）

加密底层全是**超大整数运算**：

- RSA、MD5、SHA、国密算法
- 密钥、模运算、大数取余、幂运算
- 加密里的 大整数相乘、取模 全靠 BigInteger

###### 3. 科学计算、数学场景

- 高精度阶乘、超大次方
- 统计、天文、地理、科研数据
- 竞赛算法、大数计算题

###### 4. 积分、票数、数量级极大的整数

- 平台积分、用户累计签到次数、超大点赞量
- 库存、票数、投票统计，数值不断累加怕溢出
- 电商秒杀超大并发计数

###### 5. 数据库超大整型字段映射

- 数据库 `NUMERIC / DECIMAL` 存超大整数
- 后端不用 Long，用 BigInteger 接收，防止溢出

###### 6. 区块链、数字货币底层

- 链上余额、区块高度、哈希值运算
- 虚拟币数量（无小数部分场景）

---

##### 四、举个直观例子

###### 场景：计算 1000 的阶乘

数字爆炸式增长，**long 直接溢出**，只能 BigInteger

```java
BigInteger result = BigInteger.ONE;
for (int i = 1; i <= 1000; i++) {
    result = result.multiply(BigInteger.valueOf(i));
}
System.out.println(result);
```

---

##### 五、两者对比总结（超好记）

| 类型       | 适用             | 核心场景                               |
| ---------- | ---------------- | -------------------------------------- |
| BigDecimal | 带小数、高精度   | 支付、账单、利率、财务、金融计算       |
| BigInteger | 超大整数、无小数 | 加密、大数ID、阶乘、科学计算、超大计数 |

---

##### 六、面试标准答案

1. `BigDecimal` 解决 **double 浮点精度丢失**，专门用于金融金额、小数运算。
2. `BigInteger` 是**任意长度大整数**，解决 long 溢出问题，
   多用于**加密算法、超大序号、大数数学运算、海量计数**等纯整数场景。

需要我给你写一段 **RSA 大数取模 + BigInteger 实操代码** 吗？一看就懂。

---

### 四、常见数学公式 Java 实现示例

| 数学公式                                     | Java 代码                                                    |
| -------------------------------------------- | ------------------------------------------------------------ |
| 两点距离：$\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$ | `Math.sqrt(Math.pow(x2-x1,2) + Math.pow(y2-y1,2))`           |
| 阶乘（递归/循环）                            | `for (int i=1; i<=n; i++) res *= i;`                         |
| 判断素数                                     | `for (int i=2; i*i <= n; i++) if (n%i==0) return false;`     |
| 最大公约数（GCD）                            | `BigInteger.valueOf(a).gcd(BigInteger.valueOf(b)).intValue()` 或用欧几里得算法 |
| 排列组合                                     | `C(n,k) = n! / (k! * (n-k)!)`（注意溢出）                    |

---

### 五、注意事项

1. **浮点精度问题**：  
   `0.1 + 0.2 != 0.3`（因为二进制浮点表示限制）  
   → 金融计算用 `BigDecimal`

2. **三角函数单位是弧度**，不是角度！

3. `Math` 方法都是 **静态的**，直接 `Math.xxx()` 调用。

4. `Math` 不会抛出异常，但可能返回 `NaN` 或 `Infinity`（如 `sqrt(-1)` → `NaN`）。

---

### 六、总结

| 需求                       | 推荐方式                    |
| -------------------------- | --------------------------- |
| 基础运算                   | `+ - * / %`                 |
| 高等函数（幂、对数、三角） | `Math` 类                   |
| 大整数                     | `BigInteger`                |
| 高精度小数                 | `BigDecimal`                |
| 随机数                     | `Math.random()` 或 `Random` |

> ✅ 记住：**Java 没有“公式解析器”**，你必须把数学公式“翻译”成代码逻辑。







## Java日期时间

> Java 日期分两大体系：
1. **旧API（JDK1.0/1.1）**：`Date`、`SimpleDateFormat`、`Calendar`，**线程不安全，不推荐业务使用**。
2. **新API（JDK8+ java.time包）**：`LocalDate / LocalTime / LocalDateTime / Instant / Duration / Period / DateTimeFormatter`，**线程安全，推荐优先使用**。

> 面试高频考点：旧API线程安全问题、新旧API对比、时间戳、时区、格式化、时间间隔。

### 一、旧版API（不推荐）

#### 1. java.util.Date

代表**时刻（毫秒时间戳）**，内部维护从`1970‑01‑01 00:00:00 UTC`到现在的毫秒数。
```java
// 当前系统时间
Date date = new Date();
System.out.println(date);

// 指定毫秒时间戳构造Date
Date date2 = new Date(1750000000000L);

// 获取时间戳毫秒
long time = date.getTime();
// 设置时间戳
date.setTime(time);
```
⚠️坑：
1. `new Date(year,month,day)`过时；**month从0开始，0代表1月，11代表12月，极易踩坑**。
2. Date同时包含日期+时间，没有单独的日期、时间类型。
3. `Date`对象本身可变，`setTime()`修改内部值。

#### 2. Calendar 日历类（抽象类）

用来做日期加减、年月日获取，替代Date过时构造。
```java
Calendar cal = Calendar.getInstance();
// 获取年
int year = cal.get(Calendar.YEAR);
// 月！注意：0=1月
int month = cal.get(Calendar.MONTH)+1;
int day = cal.get(Calendar.DAY_OF_MONTH);

//日期加减
cal.add(Calendar.DAY_OF_MONTH, 3); //加3天

//转Date
Date date = cal.getTime();
```
> 坑：月份从0开始；对象可变；线程不安全。

#### 3. SimpleDateFormat 格式化

**线程不安全！严禁定义为static全局变量！**
```java
//格式化：date → 字符串
SimpleDateFormat sdf = new SimpleDateFormat("yyyy‑MM‑dd HH:mm:ss");
String str = sdf.format(new Date());

//解析：字符串 → Date
Date parseDate = sdf.parse("2026‑08‑27 12:00:00");
```
模式符号：
- `yyyy` 4位年；`MM`月份；`dd`日
- `HH`24小时制；`hh`12小时制；`mm`分钟；`ss`秒；`SSS`毫秒

> 重大坑：`SimpleDateFormat`不能static共享，多线程并发会出现异常、时间错乱。
> 解决方案：每次new对象；或者使用ThreadLocal；或者直接用JDK8 `DateTimeFormatter`。



#### **4.强烈推荐用 `DateTimeFormatter` 彻底替换 `SimpleDateFormat`**

`DateTimeFormatter(JDK8+)`：**不可变、线程安全，可以定义为 static final 全局常量，多线程直接复用**，解决 SimpleDateFormat 的线程安全大坑。

> SimpleDateFormat 问题根源：对象内部持有可变的 Calendar 成员变量，并发 format / parse 会修改同一个对象状态，出现时间错乱、数字异常、解析报错。 DateTimeFormatter 所有内部字段 final，无可变状态，多线程安全。

##### 1. DateTimeFormatter 格式化示例（等价上面SimpleDateFormat）

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

// ✅可以static final全局复用！线程安全
private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

public static void main(String[] args) {
    //1.格式化：时间对象 → 字符串
    LocalDateTime now = LocalDateTime.now();
    String dateStr = now.format(FORMATTER);
    System.out.println(dateStr);

    //2.解析：字符串 → LocalDateTime
    LocalDateTime parseTime = LocalDateTime.parse("2026-08-27 12:00:00", FORMATTER);
    System.out.println(parseTime);
}
```

> 模式符号大部分和SimpleDateFormat一样： `yyyy`年、`MM`月、`dd`日、`HH`24小时、`hh`12小时、`mm`分钟、`ss`秒、`SSS`毫秒。

##### 2. 如果老项目还在用 Date，怎么搭配 DateTimeFormatter？

DateTimeFormatter 不直接操作 `java.util.Date`，需要中转 Instant+时区：

```
Date oldDate = new Date();
// Date → String
String str = oldDate.toInstant()
        .atZone(java.time.ZoneId.systemDefault())
        .format(FORMATTER);

// String → Date
LocalDateTime ldt = LocalDateTime.parse("2026-08-27 12:00:00", FORMATTER);
Date date = Date.from(ldt.atZone(java.time.ZoneId.systemDefault()).toInstant());
```

##### 三种解决方案对比（面试常问）

| 方案                           | 说明                    | 优缺点                                       |
| ------------------------------ | ----------------------- | -------------------------------------------- |
| 每次new SimpleDateFormat       | 每次调用新建实例        | 线程安全；频繁创建对象，有GC开销             |
| SimpleDateFormat + ThreadLocal | 线程绑定sdf实例         | 线程安全；代码啰嗦，要注意remove防止内存泄漏 |
| **DateTimeFormatter【首选】**  | JDK8+，static final复用 | 线程安全，API简洁，性能好，推荐业务全部迁移  |

##### ⚠️注意两个容易踩坑的区别

1. **SimpleDateFormat 解析宽松模式** `SimpleDateFormat` 默认宽松解析，比如 `"2026‑02‑30"` 会自动偏移计算成合法日期； `DateTimeFormatter` 默认**严格解析**，非法日期直接抛异常。

```
//如果需要宽松模式（极少场景）需要手动设置解析器
DateTimeFormatter formatter = new DateTimeFormatterBuilder()
        .appendPattern("yyyy-MM-dd HH:mm:ss")
        .toFormatter();
```

1. 不要混淆：

- `SimpleDateFormat` 处理旧 `Date`
- `DateTimeFormatter` 处理 `LocalDate/LocalTime/LocalDateTime/ZonedDateTime`

##### 面试简答

> Q：能不能使用DateTimeFormatter替换SimpleDateFormat？ A：可以，JDK8及以上优先使用DateTimeFormatter。它不可变、线程安全，支持static全局常量复用；SimpleDateFormat因为内部Calendar可变，多线程并发会时间错乱，不适合static。老项目的Date对象，可以通过Instant+时区做转换适配新API。

> 生产规范：新项目禁止使用SimpleDateFormat，统一使用`java.time`包 + `DateTimeFormatter`。

如果你需要，我可以给你一份工具类模板：日期格式化、字符串互转、时间戳互转的工具代码，直接复制到项目使用。

---

### 二、JDK8 java.time 新时间API【重点，开发必用】

包：`java.time.*`，底层**不可变对象，线程安全**，所有修改操作返回新对象，原对象不变。

#### 五大核心类

|类|含义|示例|
|---|---|---|
|`LocalDate`|本地日期，只有年月日，**没有时间、无时区**|`2026‑08‑27`|
|`LocalTime`|本地时间，只有时分秒毫秒，没有日期|`14:30:55`|
|`LocalDateTime`|本地日期+时间，**不带时区**|`2026‑08‑27 14:30:55`|
|`Instant`|时间戳，UTC标准时刻，存毫秒/纳秒|时间线的一个点|
|`ZonedDateTime`|带时区的完整日期时间|带Asia/Shanghai时区|

#### 1. LocalDate 只存日期

```java
//获取当前本地日期
LocalDate nowDate = LocalDate.now();
//指定年月日
LocalDate date = LocalDate.of(2026,8,27);

//获取字段
int year = date.getYear();
int month = date.getMonthValue(); //1‑12，不再0开始！！
int day = date.getDayOfMonth();

//加减时间，返回新对象，原对象不变
LocalDate plus3Day = date.plusDays(3); //加3天
LocalDate minusMonth = date.minusMonths(1); //减1月

//修改
LocalDate withYear = date.withYear(2027);

//比较
boolean isBefore = date.isBefore(LocalDate.now());
boolean isAfter = date.isAfter(LocalDate.now());
```

#### 2. LocalTime 只存时间

```java
LocalTime nowTime = LocalTime.now();
LocalTime time = LocalTime.of(12,30,10);
LocalTime plusHour = time.plusHours(2);
```

#### 3. LocalDateTime 日期+时间（业务最常用）

```java
//当前本地日期时间
LocalDateTime ldt = LocalDateTime.now();
LocalDateTime ldt2 = LocalDateTime.of(2026,8,27,15,20);

//LocalDate + LocalTime合成
LocalDateTime ldt3 = LocalDate.now().atTime(LocalTime.now());

//拆分成Date、Time
LocalDate localDate = ldt.toLocalDate();
LocalTime localTime = ldt.toLocalTime();

//加减
ldt.plusDays(1);
ldt.plusHours(2);
ldt.minusMinutes(10);
```
> ⚠️注意：**LocalDateTime不带时区信息**，不能直接代表时间戳。同一时刻，不同时区LocalDateTime值不一样。

#### 4. Instant 时间戳（UTC）

内部保存从1970‑01‑01 UTC的秒+纳秒。
```java
//当前UTC时刻
Instant instant = Instant.now();
//获取毫秒时间戳，等价 System.currentTimeMillis()
long milli = instant.toEpochMilli();

//时间戳转Instant
Instant instant2 = Instant.ofEpochMilli(1750000000000L);
```

> Instant是UTC；LocalDateTime是本地视图，二者不能直接互转，**必须经过时区ZoneId**。

#### 5. ZonedDateTime 带时区时间

```java
//上海时区
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));

//LocalDateTime加上时区得到ZonedDateTime
LocalDateTime ldt = LocalDateTime.now();
ZonedDateTime zoned = ldt.atZone(ZoneId.of("Asia/Shanghai"));
```

#### 时间间隔计算

1. `Period`：计算**年月日**间隔，用于LocalDate
```java
LocalDate d1 = LocalDate.of(2025,1,1);
LocalDate d2 = LocalDate.of(2026,3,5);
Period period = Period.between(d1,d2);
int years = period.getYears();
int months = period.getMonths();
int days = period.getDays();
```

2. `Duration`：计算**时分秒毫秒**间隔，用于Instant/LocalDateTime
```java
LocalDateTime t1 = LocalDateTime.of(2026,1,1,10,0);
LocalDateTime t2 = LocalDateTime.of(2026,1,1,12,30);
Duration duration = Duration.between(t1,t2);
long hours = duration.toHours();
long minutes = duration.toMinutes();
```

> 坑：Duration不能用于LocalDate；Period不能用于LocalTime。

#### DateTimeFormatter 格式化&解析（线程安全！）

> **DateTimeFormatter是不可变，static全局可以安全复用**，对比SimpleDateFormat。

```java
//定义格式，可static final
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy‑MM‑dd HH:mm:ss");

//格式化：LocalDateTime → String
LocalDateTime ldt = LocalDateTime.now();
String dateStr = ldt.format(formatter);

//解析：字符串 → LocalDateTime
LocalDateTime parseLdt = LocalDateTime.parse("2026‑08‑27 10:20:30",formatter);
```

#### 新旧API互相转换（项目迁移经常用到）

```java
// Date → LocalDateTime
Date oldDate = new Date();
Instant instant = oldDate.toInstant();
LocalDateTime ldt = instant.atZone(ZoneId.systemDefault()).toLocalDateTime();

// LocalDateTime → Date
ZonedDateTime zdt = ldt.atZone(ZoneId.systemDefault());
Date newDate = Date.from(zdt.toInstant());
```

#### 时间戳 ↔ LocalDateTime

```java
//时间戳毫秒 → LocalDateTime
long ts = System.currentTimeMillis();
LocalDateTime ldt = Instant.ofEpochMilli(ts)
        .atZone(ZoneId.systemDefault())
        .toLocalDateTime();

//LocalDateTime → 毫秒时间戳
long timeStamp = ldt.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
```

### 三、核心面试高频题

#### Q1：SimpleDateFormat为什么线程不安全？

A：内部对象`Calendar`是可变成员变量，多线程并发调用format/parse会并发修改内部状态，出现时间错乱、异常。**不能static**。
> DateTimeFormatter不可变，线程安全，可以static final。

#### Q2：LocalDateTime可以直接转时间戳吗？

不能。LocalDateTime不带时区，不知道是哪个地区的时间，必须加上时区ZoneId，转Instant，再拿毫秒。

#### Q3：Period 和 Duration区别

- Period：基于**年月日**，处理LocalDate。
- Duration：基于**时分秒纳秒**，处理Instant/LocalDateTime。

#### Q4：LocalDate、LocalTime、LocalDateTime是线程安全吗？

是的，对象不可变，所有加减修改返回新实例，原对象不变。

#### Q5：Calendar月份为什么坑？

Calendar月份`0‑11`，0代表一月；新API `getMonthValue()`返回1‑12，符合人类习惯。

#### Q6 Instant.now() 和 LocalDateTime.now()区别

- Instant.now()：UTC零时区的时刻。
- LocalDateTime.now()：本机系统默认时区的本地时间。

### 四、开发最佳实践

1. **业务优先使用java.time包，杜绝SimpleDateFormat静态实例**。
2. 数据库存储时间：
   - MySQL datetime：映射`LocalDateTime`
   - MySQL date：映射`LocalDate`
   - MySQL time：映射`LocalTime`
   - 存时间戳：存BIGINT毫秒，使用Instant。
3. `DateTimeFormatter`定义为`static final`常量复用。
4. 跨时区业务，一定要使用`ZonedDateTime`，不要直接LocalDateTime。
5. 旧Date尽量不要在新业务使用，做兼容时做好转换。

### 快速背诵总结

1. 旧API：Date、Calendar、SimpleDateFormat；可变，线程不安全，月份从0开始。
2. JDK8新时间包java.time，对象不可变，线程安全。
   - LocalDate：年月日；LocalTime：时分秒；LocalDateTime：日期时间（无时区）；Instant：UTC时间戳。
3. Period算年月日间隔；Duration算时分秒间隔。
4. DateTimeFormatter线程安全，可以static；SimpleDateFormat禁止static。
5. LocalDateTime转时间戳必须带上时区ZoneId。
6. 新旧API转换靠Instant + ZoneId做桥梁。

如果你需要，我继续讲集合（List Set Map），Java后端面试重中之重。



---

## JDK 原生高频通用工具类

### 3.1 对象工具：java.util.Objects（JDK7）
1. equals(a,b)：规避a == null时 NPE
2. isNull()、nonNull() 对象判空
3. requireNonNull() 参数校验，为空抛出空指针，参数校验首选
4. hash() 生成hash值
5. toString() 空安全的对象toString

### 3.2 Optional 空指针防护（JDK8，重点）

Java 8 引入的 `Optional<T>` 是一个容器类，用于解决 **空指针异常（NullPointerException）** 问题。它可以包含一个非 `null` 的值，或者表示“不存在值”（即空状态），通过明确的 API 设计强制开发者处理空值情况，避免隐性的空指针风险。

所属：java.util
1. Optional<T> 封装可空对象，用于规避NPE
2. 静态方法：ofNullable()、of()、empty()
3. 实例API
    - isPresent() / isEmpty(JDK11) 判断有无值
    - orElse、orElseGet、orElseThrow 获取默认值或者抛出异常
    - map、flatMap 链式调用
4. 编码坑
    - 禁止直接 get()
    - 不要把Optional当做方法入参、成员变量

#### **核心思想**

`Optional<T>` 不存储 `null`，而是用 `Optional.empty()` 表示空状态。通过它提供的方法，开发者可以更优雅地处理“值可能存在或不存在”的场景，替代传统的 `if (obj != null)` 判断。

#### **常用方法**

##### 1. 创建 `Optional` 实例

- `Optional.empty()`：创建一个空的 `Optional` 实例（值不存在）。

- `Optional.of(T value)`：创建一个包含非 `null` 值的 `Optional`，若 `value` 为 `null` 则直接抛 `NullPointerException`。

- `Optional.ofNullable(T value)`：创建一个包含值的 `Optional`，若 `value` 为 `null` 则返回空实例（推荐使用，更安全）。

  ```java
  Optional<String> emptyOpt = Optional.empty();
  Optional<String> nonEmptyOpt = Optional.of("hello"); // 若传入null，立即报错
  Optional<String> nullableOpt = Optional.ofNullable(null); // 安全处理null，返回empty
  ```

##### 2. 判断值是否存在

- `isPresent()`：返回 `true` 表示值存在（非空），`false` 表示为空。

- `isEmpty()`（Java 11+）：与 `isPresent()` 相反，返回 `true` 表示为空。

  ```java
  Optional<String> opt = Optional.of("test");
  if (opt.isPresent()) {
      System.out.println(opt.get()); // 输出 "test"
  }
  ```

##### 3. 获取值

- `get()`：若值存在则返回该值，否则抛 `NoSuchElementException`（**不推荐直接使用**，需先判断 `isPresent()`）。

- `orElse(T other)`：若值存在则返回该值，否则返回指定的默认值 `other`（`other` 无论是否使用都会被创建）。

- `orElseGet(Supplier<? extends T> supplier)`：若值存在则返回该值，否则通过 `supplier` 生成默认值（**延迟创建**，更高效）。

- `orElseThrow(Supplier<? extends X> exceptionSupplier)`：若值存在则返回该值，否则抛出 `supplier` 生成的异常（推荐用于必须有值的场景）。

  ```java
  Optional<String> opt = Optional.ofNullable(null);
  
  String v1 = opt.orElse("default"); // 返回 "default"
  String v2 = opt.orElseGet(() -> "generated by supplier"); // 返回 "generated by supplier"
  String v3 = opt.orElseThrow(() -> new IllegalArgumentException("值不存在")); // 抛异常
  ```

##### 4. 条件处理与转换

- `ifPresent(Consumer<? super T> action)`：若值存在，则执行 `action` 操作（替代 `if (obj != null) { ... }`）。

- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`（Java 9+）：若值存在执行 `action`，否则执行 `emptyAction`。

- `map(Function<? super T, ? extends U> mapper)`：若值存在，对其执行 `mapper` 转换，返回新的 `Optional<U>`；若为空则返回空 `Optional`。

- `flatMap(Function<? super T, Optional<U>> mapper)`：与 `map` 类似，但 `mapper` 直接返回 `Optional<U>`（避免嵌套 `Optional`）。

- `filter(Predicate<? super T> predicate)`：若值存在且满足 `predicate`，则返回当前 `Optional`，否则返回空。

  ```java
  Optional<String> opt = Optional.of("hello");
  
  // 若存在则打印
  opt.ifPresent(s -> System.out.println(s.length())); // 输出 5
  
  // 转换为长度的Optional
  Optional<Integer> lengthOpt = opt.map(String::length); // Optional[5]
  
  // 过滤长度大于3的值
  Optional<String> filtered = opt.filter(s -> s.length() > 3); // Optional["hello"]
  ```

#### **使用场景示例**

传统处理空值的方式可能嵌套多层 `if (obj != null)`，而 `Optional` 可以简化为链式调用：

```java
// 传统方式
String userName = null;
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        userName = address.getCity();
    }
}

// Optional方式
String city = Optional.ofNullable(user)
    .flatMap(User::getAddress) // 假设getAddress返回Optional<Address>
    .map(Address::getCity)
    .orElse("默认城市");
```

#### **注意事项**

1. 不要用 `Optional` 作为类的字段、方法参数或集合元素（增加复杂度，违背设计初衷）。
2. 避免过度使用 `isPresent()` + `get()`，这与传统 `null` 判断并无本质区别，应优先使用 `ifPresent`、`orElse` 等方法。
3. `Optional` 是不可变的，所有转换方法（如 `map`、`filter`）都会返回新实例。

`Optional` 的核心价值在于 **强制显式处理空值**，使代码更健壮、可读性更高，尤其适合链式调用场景。

### 3.3 数组工具 java.util.Arrays
1. asList() 数组转List（固定长度集合，add/remove报错，经典坑）
2. sort() 数组排序，双轴快速排序
3. binarySearch() 二分查找（数组必须有序）
4. copyOf() 数组拷贝扩容
5. equals 深度比较数组元素；deepEquals多维数组对比
6. fill() 数组填充
7. toString() 打印数组内容

### 3.4 集合工具 java.util.Collections
1. 空集合：emptyList() emptySet() emptyMap() 推荐返回空集合代替null
2. 只读集合 unmodifiableList/Set/Map，防止外部修改内部集合
3. synchronizedXxx 线程安全包装集合（性能差，不推荐）
4. sort、max、min、shuffle打乱顺序、swap交换元素

### 3.5 字符串原生工具（String自带API）
length、isEmpty、charAt、substring、split、replace、replaceAll、trim、intern
坑点：
- substring JDK6存在字符数组内存泄漏
- trim只能去除半角空格，无法清除全角空格

### 3.6 线程相关工具类
1. ThreadLocal：线程本地副本、用完必须remove，防止内存泄漏
2. TimeUnit（java.util.concurrent）
    - 时间单位枚举；sleep、线程等待、时间转换
```java
TimeUnit.SECONDS.sleep(3);
```

### 3.7 比较器工具
1. Comparable 内部比较接口
2. Comparator 外部比较器，JDK8支持链式 thenComparing

### 3.8 Base64（JDK8‑java.util.Base64）
- 编码器、解码器；URL安全模式Base64

### 3.9 UUID
java.util.UUID，生成唯一ID；
version‑4随机UUID；缺点无序，数据库索引性能差

---

## 四、主流第三方常用工具类（工作必用，面试经常问到）
### 1. Spring‑Framework 工具包
1. org.springframework.util.StringUtils
    - hasLength()、hasText()、isEmpty、逗号分割字符串
2. CollectionUtils：集合判空
3. ObjectUtils、FileUtils、ResourceUtils
4. Assert：参数断言校验

### 2. Hutool（国内最常用）
1. StrUtil：字符串判空 isBlank、isNotBlank、去除空格
2. DateUtil：日期格式化、日期加减、获取时间戳
3. ArrayUtil、CollUtil 数组集合工具
4. SecureUtil、IdUtil雪花ID
5. BeanUtil 对象属性拷贝

### 3. Apache‑Commons 全家桶
1. commons‑lang3
    - StringUtils（最经典）、ArrayUtils、CollectionUtils
    - DateUtils、StopWatch计时器
    - Pair、Triple
2. commons‑io：IOUtils、FileUtils 文件流操作
3. commons‑beanutils 对象拷贝（性能较差）
4. commons‑collections4 高级集合工具

### 4. Guava谷歌工具库
1. Strings、Preconditions参数校验
2. ImmutableList 不可变集合
3. Splitter字符串分割、Joiner拼接
4. RateLimiter限流、LoadingCache本地缓存

---

## 五、该模块常见面试题汇总
1. Date和LocalDateTime区别，为什么JDK8废弃旧日期API
2. SimpleDateFormat为什么线程不安全，解决方案
3. Duration和Period区别
4. Objects.equals 和 a.equals(b)区别
5. Arrays.asList 坑点
6. Optional的使用规范、禁止使用场景
7. ThreadLocal内存泄漏原因
8. Math.round(-1.5)结果
9. 随机数 Random、ThreadLocalRandom、SecureRandom区别
10. 业务判空：!=null / isEmpty / isBlank / hasText区别

