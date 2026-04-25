面试中关于 **SQL 语句** 的考察非常常见，通常会涉及从基础语法到复杂查询和数据库优化多个层面。下面我帮你分类总结常见的 SQL 相关面试题目及知识点，方便你系统准备。

好的！以下是 **数据库建表时常用字段类型的分类介绍**，包括基础类型、枚举类型、时间日期类型等，帮助你理解并在设计表结构时合理选择。

------

## 数据库建表字段类型分类介绍



### 数据库字段类型

以 MySQL 为例，其他数据库（PostgreSQL、Oracle、SQL Server）略有差异，但分类类似。

------

#### 1. 数值类型（Numeric Types）

| 类型            | 大小（字节） | 说明                                   | 备注                          |
| --------------- | ------------ | -------------------------------------- | ----------------------------- |
| `TINYINT`       | 1            | -128 到 127（有符号），0~255（无符号） | 常用于布尔（0/1）或小范围数字 |
| `SMALLINT`      | 2            | -32768 到 32767                        | 小范围整数                    |
| `MEDIUMINT`     | 3            | -8388608 到 8388607                    | 中等范围整数                  |
| `INT`/`INTEGER` | 4            | -2^31 到 2^31-1                        | 常用整数类型                  |
| `BIGINT`        | 8            | -2^63 到 2^63-1                        | 超大整数                      |
| `DECIMAL(M,D)`  | 变动         | 精确小数，M 总位数，D 小数位数         | 金额、财务数值推荐用它        |
| `FLOAT`         | 4            | 单精度浮点数                           | 近似数值，精度有限            |
| `DOUBLE`        | 8            | 双精度浮点数                           | 精度更高，近似数值            |

------

#### 2. 字符串类型（String Types）

| 类型                | 大小（最大字节）  | 说明                                   | 备注                       |
| ------------------- | ----------------- | -------------------------------------- | -------------------------- |
| `CHAR(N)`           | 固定 N 字节       | 定长字符串，N 最大 255                 | 适合长度固定字段，如性别   |
| `VARCHAR(N)`        | 变长，最多 N 字节 | 可变长字符串，N 最大 65535             | 常用文本类型               |
| `TEXT`              | 65,535 字节       | 长文本数据                             | 不能有默认值               |
| `MEDIUMTEXT`        | 16 MB             | 更大文本                               | 比 TEXT 更大               |
| `LONGTEXT`          | 4 GB              | 超大文本                               | 超大数据存储               |
| `ENUM('a','b',...)` | 1-2 字节          | 枚举类型，字段值只能是预定义值之一     | 用于状态、类别等有限值字段 |
| `SET('a','b',...)`  | 1-8 字节          | 多值集合，字段可存储预定义值的任意组合 | 用于多选标签               |

------

#### 3. 日期和时间类型（Date and Time Types）

| 类型        | 大小（字节） | 说明                           | 格式示例                |
| ----------- | ------------ | ------------------------------ | ----------------------- |
| `DATE`      | 3            | 仅日期，YYYY-MM-DD             | 2025-06-08              |
| `DATETIME`  | 8            | 日期+时间，YYYY-MM-DD HH:MM:SS | 2025-06-08 19:00:00     |
| `TIMESTAMP` | 4            | 时间戳，UTC时间，受时区影响    | 2025-06-08 11:00:00 UTC |
| `TIME`      | 3            | 仅时间，HH:MM:SS               | 19:00:00                |
| `YEAR`      | 1            | 年份，YYYY                     | 2025                    |

在 SQL（尤其是 MySQL）中，`NOW()`, `INTERVAL`, `DAY` 是进行**时间日期操作**的核心语法组件，常用于筛选、计算时间范围等逻辑，下面为你详细讲解：

------

##### ✅ 1. `NOW()` 函数

- **作用**：获取当前的 **日期和时间**（当前时间戳）。
- **返回值**：`DATETIME` 类型，例如：`2025-06-09 20:30:00`

###### 示例：

```sql
SELECT NOW();  -- 2025-06-09 20:30:00
```

##### ✅ 1.`CURDATE()` 函数

------

###### 1. **`CURDATE()` 是什么？**

- `CURDATE()`：返回 **当前日期（不含时间）**
- 返回值类型：`DATE`，格式为：`'YYYY-MM-DD'`
- 等价于 `CURRENT_DATE`

------

###### 2. 示例对比：`NOW()` vs `CURDATE()`

| 函数        | 返回值类型 | 示例返回值              |
| ----------- | ---------- | ----------------------- |
| `NOW()`     | `DATETIME` | `'2025-06-09 21:00:00'` |
| `CURDATE()` | `DATE`     | `'2025-06-09'`          |

**示例：**

```sql
SELECT NOW();       -- 2025-06-09 21:00:00
SELECT CURDATE();   -- 2025-06-09
```

------

###### 3. 使用场景举例

🔸 查询**今天的订单**

```sql
SELECT * FROM orders
WHERE DATE(create_time) = CURDATE();
```

 🔸 查询**昨天的订单**

```sql
SELECT * FROM orders
WHERE DATE(create_time) = CURDATE() - INTERVAL 1 DAY;
```

 🔸 查询**本月订单**

```sql
SELECT * FROM orders
WHERE MONTH(create_time) = MONTH(CURDATE())
  AND YEAR(create_time) = YEAR(CURDATE());
```

------

###### 4. 常见面试问法

| 问题                                       | 回答提示                                                     |
| ------------------------------------------ | ------------------------------------------------------------ |
| `CURDATE()` 与 `NOW()` 的区别？            | `CURDATE()` 只返回日期；`NOW()` 返回完整的时间戳（含时分秒） |
| 怎么筛选今天、昨天、本月的记录？           | 配合 `CURDATE()` 使用 `DATE()`，或使用 `YEAR()`、`MONTH()` 等函数 |
| 查询当天记录为什么用 `DATE(create_time)`？ | 防止时间精度影响匹配（`= '2025-06-09'` 不包含 `21:00:00` 之后的记录） |

------

需要我出一些基于 `CURDATE()` 的实战题或者优化建议吗？比如：**按天统计注册量**、**当天订单汇总**等。

------

##### ✅ 2. `INTERVAL` 关键字

- **作用**：用于在日期计算中**添加或减去时间单位**。
- **常见用法**：`NOW() - INTERVAL 30 DAY` 表示 30 天前的当前时刻。

###### 语法格式：

```sql
时间值 [+/-] INTERVAL 数值 时间单位
```

###### 常见单位（MySQL 支持）：

| 单位     | 含义 | 示例                 |
| -------- | ---- | -------------------- |
| `DAY`    | 天   | `INTERVAL 7 DAY`     |
| `HOUR`   | 小时 | `INTERVAL 2 HOUR`    |
| `MINUTE` | 分钟 | `INTERVAL 30 MINUTE` |
| `SECOND` | 秒   | `INTERVAL 10 SECOND` |
| `MONTH`  | 月   | `INTERVAL 1 MONTH`   |
| `YEAR`   | 年   | `INTERVAL 5 YEAR`    |

------

##### ✅ 3. 示例组合用法

###### 🔸 查询近 30 天的订单

```sql
SELECT * FROM orders
WHERE create_time >= NOW() - INTERVAL 30 DAY;
```

###### 🔸 查询 1 小时内登录的用户

```sql
SELECT * FROM login_log
WHERE login_time >= NOW() - INTERVAL 1 HOUR;
```

###### 🔸 查询昨天的数据

```sql
SELECT * FROM orders
WHERE DATE(create_time) = CURDATE() - INTERVAL 1 DAY;
```

------

##### ✅ 面试延伸提问（高频）：

| 面试问题                       | 回答提示                                                  |
| ------------------------------ | --------------------------------------------------------- |
| `NOW()` 和 `CURDATE()` 区别？  | `NOW()` 返回日期+时间，`CURDATE()` 只返回日期（00:00:00） |
| 如果查询当天的数据应该怎么写？ | `WHERE DATE(create_time) = CURDATE()`                     |
| 查询最近 7 天的用户注册量？    | `WHERE register_time >= NOW() - INTERVAL 7 DAY`           |
| 如何统计每月注册用户数？       | `GROUP BY DATE_FORMAT(register_time, '%Y-%m')`            |

------

如需我帮你出一套**时间范围相关的 SQL 练习题与面试题**，也可以继续告诉我。

------

#### 4. 布尔类型（Boolean）

- MySQL 没有真正的 `BOOLEAN` 类型，通常用 `TINYINT(1)` 代替，0 表示 `false`，1 表示 `true`。

------

#### 5. 二进制类型（Binary Types）

| 类型           | 说明               | 用途                     |
| -------------- | ------------------ | ------------------------ |
| `BINARY(N)`    | 固定长度二进制数据 | 存储二进制短数据         |
| `VARBINARY(N)` | 变长二进制数据     | 存储变长二进制数据       |
| `BLOB`         | 二进制大对象       | 图片、音频、视频文件存储 |
| `MEDIUMBLOB`   | 中等大小二进制对象 |                          |
| `LONGBLOB`     | 超大二进制对象     |                          |

------

#### 6. 特殊类型

- **JSON**（MySQL 5.7+ 支持）：存储 JSON 格式数据，支持 JSON 查询与索引。
- **空间类型**（如 `GEOMETRY`、`POINT`）：用于地理信息系统（GIS）数据存储。

------

#### 7. 常见设计建议

| 场景                 | 建议类型                           | 备注                    |
| -------------------- | ---------------------------------- | ----------------------- |
| 用户ID、主键         | `BIGINT` 或 `INT`                  | 推荐使用无符号，且自增  |
| 性别、状态、类别字段 | `ENUM`                             | 限定固定值，减少出错    |
| 用户名、邮件等字符串 | `VARCHAR(50~255)`                  | 适度限制长度            |
| 密码哈希存储         | `CHAR(60)`                         | bcrypt 哈希一般固定长度 |
| 备注、描述           | `TEXT` 或 `VARCHAR`                | 根据长度选择            |
| 创建/更新时间        | `DATETIME` 或 `TIMESTAMP`          | 推荐统一时区或 UTC      |
| 金额                 | `DECIMAL(10,2)`                    | 精确存储货币金额        |
| 头像图片             | `VARCHAR` 存储 URL，图片不存数据库 | 避免大数据量二进制存储  |

------

#### 示例表结构片段

```sql
CREATE TABLE user (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password CHAR(60) NOT NULL,
    gender ENUM('male', 'female', 'other') DEFAULT 'other',
    phone VARCHAR(20),
    email VARCHAR(100),
    avatar_url VARCHAR(255),
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

------

如果需要，我可以帮你生成不同数据库（PostgreSQL、Oracle、SQL Server）对应的建表字段类型对照和范例。需要吗？

表中除了常见的字段类型定义之外，**还可以添加各种约束（Constraints）**来保证数据的完整性和正确性。下面是约束的分类介绍及作用说明。

------

### 数据库表字段约束分类介绍

------

#### 1. 主键约束（Primary Key）

- **作用**：唯一标识表中每一行记录，不能重复且不能为空（隐含 `NOT NULL`）。
- **使用**：一般用于唯一标识行的字段，如 `id`。
- **示例**：

```sql
id INT PRIMARY KEY AUTO_INCREMENT
```

------

#### 2. 唯一约束（Unique Constraint）

- **作用**：保证字段值在表中唯一，但允许空值（NULL）存在（依数据库而定）。
- **使用**：限制某字段（或字段组合）唯一，如用户名、邮箱等。
- **示例**：

```sql
username VARCHAR(50) UNIQUE
```

------

#### 3. 非空约束（NOT NULL）

- **作用**：字段必须有值，不能为 NULL。
- **使用**：关键字段必填，如密码、用户名。
- **示例**：

```sql
email VARCHAR(100) NOT NULL
```

------

#### 4. 默认值约束（DEFAULT）

- **作用**：字段没有赋值时，自动赋默认值。
- **使用**：状态字段、时间字段等常用默认值。
- **示例**：

```sql
status ENUM('active','inactive') DEFAULT 'active'
created_at DATETIME DEFAULT CURRENT_TIMESTAMP
```

------

#### 5. 外键约束（Foreign Key）

- **作用**：保证字段值必须存在于另一个表的主键或唯一键中，实现表之间的数据关联。
- **使用**：保证数据引用完整性，如订单表的用户ID必须是用户表中存在的。
- **示例**：

```sql
user_id INT,
FOREIGN KEY (user_id) REFERENCES user(id)
```

- **特性**：
  - 可以设置级联删除、级联更新。
  - 保证跨表数据一致。

------

#### 6. 检查约束（CHECK）

- **作用**：限制字段的取值范围或格式，确保数据合理性。
- **使用**：如年龄必须大于0，状态字段只能是特定值。
- **示例**：

```sql
age INT CHECK (age >= 0)
```

- **注意**：MySQL 在5.7以前版本对 `CHECK` 支持有限。

------

#### 7. 自增约束（AUTO_INCREMENT / SERIAL）

- **作用**：数字字段自动递增，一般用作主键。
- **使用**：主键ID常用。
- **示例**：

```sql
id INT PRIMARY KEY AUTO_INCREMENT
```

------

#### 8. 索引约束（INDEX）

- **作用**：虽然不是严格意义上的约束，但通过索引加速查询。
- **使用**：对查询频繁字段创建索引，提升性能。
- **示例**：

```sql
CREATE INDEX idx_username ON user(username);
```

------

#### 9. 组合约束（复合主键、复合唯一）

- **作用**：多字段联合保证唯一性或作为联合主键。
- **示例**：

```sql
PRIMARY KEY (order_id, product_id)
UNIQUE (email, phone)
```

------

#### 约束总结表

| 约束类型       | 功能                       | 允许 NULL | 唯一性   | 备注                 |
| -------------- | -------------------------- | --------- | -------- | -------------------- |
| PRIMARY KEY    | 唯一标识、非空             | 不允许    | 唯一     | 一张表只能有一个主键 |
| UNIQUE         | 唯一，但允许空（视DB而定） | 允许      | 唯一     | 可多个唯一索引       |
| NOT NULL       | 必须有值                   | 不允许    | 无       | 约束字段不能为空     |
| DEFAULT        | 默认值                     | 可有可无  | 无       | 字段无赋值时生效     |
| FOREIGN KEY    | 参照完整性                 | 视定义    | 视定义   | 外键约束关联其它表   |
| CHECK          | 值范围限制                 | 视定义    | 无       | 限制字段取值符合条件 |
| AUTO_INCREMENT | 自增字段                   | 不允许    | 通常唯一 | 适合主键、自增ID     |
| INDEX          | 提升查询性能               | 可有可无  | 可唯一   | 非严格约束           |

------

#### 小结

- **约束是保证数据完整性的重要手段**，避免脏数据。
- 常用的是主键、唯一、非空、外键、默认值。
- 设计数据库时，应尽量利用约束来减少程序层校验。
- 不同数据库对约束支持程度略有不同，如 `CHECK`。

------

需要的话，我可以帮你写示例的 SQL 建表语句，展示各种约束怎么写，你要吗？

### SQL 中关键字

还有很多，以下是一些**经典且高频的 SQL 关键字**分类及简要介绍：

------

#### 1. **数据查询相关**

- `SELECT` ：查询字段或表达式
- `FROM` ：指定查询的表
- `WHERE` ：过滤条件
- `GROUP BY` ：分组
- `HAVING` ：分组后的过滤条件
- `ORDER BY` ：排序
- `LIMIT` / `TOP` ：限制返回行数
- `DISTINCT` ：去重



##### `DISTINCT` 关键字

在 SQL 中用于去除查询结果中的重复行，确保返回的每一行都是唯一的。

------

###### 1. 基本用法

```sql
SELECT DISTINCT column1, column2
FROM table_name;
```

- 结果中只返回不同的 `(column1, column2)` 组合，不重复。
- 如果只写 `DISTINCT column1`，表示以 `column1` 去重，其他字段随对应 `column1` 返回。

------

###### 2. 例子

假设有个 `orders` 表：

| order_id | user_id | amount |
| -------- | ------- | ------ |
| 1        | 101     | 50     |
| 2        | 101     | 75     |
| 3        | 102     | 20     |
| 4        | 101     | 50     |

- 查询所有用户 ID（含重复）：

```sql
SELECT user_id FROM orders;
```

结果：

```
101
101
102
101
```

- 查询所有不同的用户 ID：

```sql
SELECT DISTINCT user_id FROM orders;
```

结果：

```
101
102
```

------

###### 3. 多列去重

```sql
SELECT DISTINCT user_id, amount FROM orders;
```

结果是去重后的 `(user_id, amount)` 组合，比如：

| user_id | amount |
| ------- | ------ |
| 101     | 50     |
| 101     | 75     |
| 102     | 20     |

注意：如果 `(user_id, amount)` 重复的行，只保留一条。

------

###### 4. 与 `GROUP BY` 的区别

- `DISTINCT` 是针对整行的唯一性进行去重。
- `GROUP BY` 是将结果按指定列分组，并可以配合聚合函数。

两者在某些场景下效果类似，但用法和目的不同。

------

###### 5. 性能考虑

- `DISTINCT` 会增加排序和去重开销，尤其大数据量时。
- 对大表建议结合索引或改用聚合函数等方式优化。

------

如果需要更多示例或复杂场景说明，可以告诉我！

------

#### 2. **表连接相关**

- `JOIN` / `INNER JOIN` ：内连接
- `LEFT JOIN` / `LEFT OUTER JOIN` ：左连接
- `RIGHT JOIN` / `RIGHT OUTER JOIN` ：右连接
- `FULL JOIN` / `FULL OUTER JOIN` ：全连接（部分数据库支持）
- `ON` ：连接条件
- `USING` ：连接条件简写（同名列）

------

#### 3. **数据操作**

- `INSERT INTO` ：插入数据
- `UPDATE` ：更新数据
- `DELETE` ：删除数据
- `VALUES` ：插入数据的值列表

------

#### 4. **数据定义（DDL）**

- `CREATE` ：创建表、索引、视图、数据库等
- `ALTER` ：修改表结构
- `DROP` ：删除表、索引、视图、数据库
- `TRUNCATE` ：清空表数据（高效删除）

------

#### 5. **事务控制**

- `BEGIN` / `START TRANSACTION` ：开始事务
- `COMMIT` ：提交事务
- `ROLLBACK` ：回滚事务
- `SAVEPOINT` ：保存点（部分数据库支持）

------

#### 6. **约束关键字**

- `PRIMARY KEY` ：主键
- `FOREIGN KEY` ：外键
- `UNIQUE` ：唯一约束
- `CHECK` ：检查约束
- `NOT NULL` ：非空约束
- `DEFAULT` ：默认值

------

#### 7. **函数及表达式**

- `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` ：聚合函数
- `CASE WHEN THEN ELSE END` ：条件表达式
- `CAST()` / `CONVERT()` ：类型转换
- `COALESCE()` ：空值替代
- `NULLIF()` ：比较空值

------

#### 8. **高级查询**

- `WITH` ：公共表表达式（CTE）
- `UNION` / `UNION ALL` ：集合并
- `EXISTS` ：判断子查询是否返回结果
- `IN` ：集合包含判断
- `LIKE` ：模糊匹配
- `BETWEEN` ：区间判断
- `IS NULL` / `IS NOT NULL` ：空值判断

------

#### 9. **索引和性能相关**

- `EXPLAIN` ：查询执行计划分析
- `ANALYZE` ：收集统计信息



是的，你说得非常对！在之前的关键字分类中，**确实遗漏了部分常用于条件表达式的操作符关键字**，下面我补充并归纳如下 👇：

------

#### 10. 条件判断类关键字（逻辑与比较运算）

这些关键字是 SQL 中 `WHERE`、`HAVING`、`CASE WHEN` 等语句常用的逻辑和条件判断工具：

##### 🔹 1. **逻辑运算关键字**

| 关键字        | 含义                | 示例                                          |
| ------------- | ------------------- | --------------------------------------------- |
| `AND`         | 逻辑与              | `WHERE age > 18 AND status = 'active'`        |
| `OR`          | 逻辑或              | `WHERE city = 'Beijing' OR city = 'Shanghai'` |
| `NOT`         | 逻辑非，取反        | `WHERE NOT status = 'inactive'`               |
| `IS`          | 判断空/布尔等       | `IS NULL`, `IS NOT TRUE`                      |
| `IS NULL`     | 判断值是否为 NULL   | `WHERE phone IS NULL`                         |
| `IS NOT NULL` | 判断值是否不为 NULL | `WHERE email IS NOT NULL`                     |

------

##### 🔹 2. **区间与集合判断关键字**

| 关键字                    | 含义                               | 示例                                  |
| ------------------------- | ---------------------------------- | ------------------------------------- |
| `BETWEEN ... AND ...`     | 判断是否在某个范围之间（包含边界） | `WHERE salary BETWEEN 3000 AND 6000`  |
| `NOT BETWEEN ... AND ...` | 判断是否不在某个范围之间           | `WHERE score NOT BETWEEN 60 AND 80`   |
| `IN (...)`                | 判断是否属于某个集合               | `WHERE status IN ('PAID', 'SHIPPED')` |
| `NOT IN (...)`            | 判断是否不属于某个集合             | `WHERE user_id NOT IN (1, 2, 3)`      |

------

##### 🔹 3. **模糊与模式匹配关键字**

| 关键字     | 含义                | 示例                                |
| ---------- | ------------------- | ----------------------------------- |
| `LIKE`     | 模糊匹配            | `WHERE name LIKE 'A%'`              |
| `NOT LIKE` | 非模糊匹配          | `WHERE name NOT LIKE '%test%'`      |
| `ESCAPE`   | 转义字符，用于 LIKE | `WHERE path LIKE '%\_%' ESCAPE '\'` |

------

这些关键字在**面试中非常高频**，尤其是涉及：

- 复杂条件组合（AND + OR + NOT）
- 区间过滤（BETWEEN AND）
- 空值判断（IS NULL）
- 模糊搜索（LIKE）

### SQL别名

SQL 中的 **别名（Alias）** 是一种简洁可读性更强的写法，常用于给表名或字段名 **临时命名**，帮助我们：

- 简化字段引用（特别是字段名较长或存在函数表达式时）
- 避免同名字段冲突（特别是在多表连接中）
- 提高查询语句的可读性

------

#### ✅ 1. 字段别名（Column Alias）

##### 语法：

```sql
SELECT column_name AS alias_name
FROM table_name;
```

- `AS` 关键字可省略（推荐保留以增强可读性）
- 常用于聚合函数、计算字段、长字段名

##### 示例：

```sql
SELECT 
  name AS username, 
  salary * 12 AS annual_salary 
FROM employees;
```

> 上例中：`salary * 12` 是表达式，起了一个别名叫 `annual_salary`。

------

#### ✅ 2. 表别名（Table Alias）

##### 语法：

```sql
SELECT t.column_name 
FROM table_name AS t;
```

- 一般在多表连接、多次引用一个表或写子查询时使用
- 只在当前 SQL 语句执行期间有效

##### 示例：

```sql
SELECT e.name, d.name AS department_name
FROM employees AS e
JOIN departments AS d ON e.department_id = d.id;
```

> 表别名 `e` 和 `d` 让连接中的字段引用更清晰。

------

#### 🧠 面试/实战注意点：

| 场景                 | 建议做法                             |
| -------------------- | ------------------------------------ |
| 多表 JOIN            | 强烈建议给每张表设置别名             |
| 聚合函数、表达式     | 建议加别名（如 `COUNT(*) AS total`） |
| 子查询               | 子查询必须起别名，否则报错           |
| 别名中含空格或关键词 | 用引号包裹，例如 `AS "user count"`   |

##### ❗错误示例：

```sql
SELECT salary * 12 annual_salary   -- ✅ 正确，但不推荐省略 AS（尤其在复杂表达式中）
SELECT * FROM (SELECT * FROM users)  -- ❌ 错误：子查询必须有别名
```

------

##### ✅ 子查询中别名举例：

```sql
SELECT avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg
WHERE avg_salary > 10000;
```

> 子查询必须起名为 `dept_avg`，否则会报错。

#### 别名的使用范围

SQL 中**别名（Alias）\**的使用虽然非常常见且灵活，但在不同上下文中也存在一定的\**使用范围和限制**，下面从字段别名、表别名、作用范围及注意事项几个方面详细介绍：

------

##### ✅ 1. 字段别名的使用范围

| 特性                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| ✅ 可在 `SELECT` 中使用                     | 字段别名通常用于 `SELECT` 语句中给输出列起别名               |
| ❌在 `WHERE` 中不可用                       | `WHERE` 是在字段别名还未生效前执行的，不能引用别名           |
| ✅在 `GROUP BY`、`ORDER BY` 中使用不受限    | MYSQL完全支持，其他数据库有的支持部分(这里不做阐述)          |
| ❌ 在 `GROUP BY`、`ORDER BY` 中使用受限     | 不同数据库支持不同，推荐直接使用原字段或表达式               |
| ✅ 在 `ORDER BY` 中使用（大多数数据库支持） | 如 `ORDER BY total_salary` 是 `SELECT salary * 12 AS total_salary` 定义的别名 |
| ✅ 可用于外层查询（子查询中定义）           | 外层可以使用子查询定义的别名字段                             |



###### 示例说明：

```sql
SELECT salary * 12 AS total_salary
FROM employees
WHERE total_salary > 100000; -- ❌ 报错，因为 WHERE 中不能使用别名
```

应改为：

```sql
SELECT salary * 12 AS total_salary
FROM employees
WHERE salary * 12 > 100000; -- ✅ 正确
```

------

##### ✅ 2. 表别名的使用范围

| 特性                               | 描述                                       |
| ---------------------------------- | ------------------------------------------ |
| ✅ 在整个 SQL 查询中使用            | 表别名一旦声明，可用于该语句中所有字段引用 |
| ✅ 多表 JOIN 时必须用表别名限定字段 | 否则字段名重复或不明确                     |
| ✅ 必须出现在 `FROM` 或 `JOIN` 后面 | 且别名后不能再使用原表名                   |
| ✅ 子查询别名是必须的               | 无子查询别名将报错                         |

###### 示例：

```sql
-- 正确：子查询必须起别名
SELECT *
FROM (
    SELECT id, name FROM users
) AS u;
```

------

##### ✅ 3. 子查询中的别名

###### 子查询结果字段别名：

- 可以在子查询中定义字段别名，外层查询可使用
- 外层查询中再次使用字段别名可覆盖

```sql
SELECT u.name_alias
FROM (
    SELECT name AS name_alias FROM users
) AS u;
```

------

##### ⚠️ 4. 使用限制 & 注意事项

| 限制/问题               | 原因或解决方案                                         |
| ----------------------- | ------------------------------------------------------ |
| `WHERE` 不能使用别名    | 因为 `WHERE` 在 SQL 执行的较早阶段（在 `SELECT` 之前） |
| `HAVING` 可以使用别名   | 因为 `HAVING` 是在 `GROUP BY` 后、`SELECT` 后执行的    |
| `ORDER BY` 可以使用别名 | 多数数据库支持在 `ORDER BY` 中使用 SELECT 中定义的别名 |
| 子查询必须命名          | 子查询不命名会导致 SQL 语法错误                        |
| 别名含特殊字符或空格    | 需要加引号（`"别名"` 或 `[别名]`，视数据库而定）       |

------

##### ✅ SQL 执行顺序简要（便于理解为什么 WHERE 中不能用别名）

```text
FROM → ON → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

> 因此：
>
> - `SELECT` 中定义的别名，在 `WHERE` 阶段并不可见
> - 但 `ORDER BY` 是在 `SELECT` 之后，因此可见

------

##### ✅ 总结

| 别名类型   | 是否必须               | 使用范围                       | 注意事项         |
| ---------- | ---------------------- | ------------------------------ | ---------------- |
| 字段别名   | 可选                   | `SELECT`、`ORDER BY`、`HAVING` | `WHERE` 不可用   |
| 表别名     | 推荐（多表时几乎必须） | 整个 SQL 查询中字段前缀        | 子查询必须有别名 |
| 子查询别名 | 必须                   | `FROM` 子句                    | 否则 SQL 无效    |

------





### SQL基本数学函数

在 SQL 中，常用的**数学函数**可以帮助我们对数值数据进行取整、四舍五入、取绝对值、取对数等各种计算，**在面试和实际开发中经常出现**。下面是对这些函数的系统整理和示例：

------

#### ✅ 一、常用数学函数汇总（以 MySQL 为例）

| 函数名                   | 作用                        | 示例                                            | 说明                              |
| ------------------------ | --------------------------- | ----------------------------------------------- | --------------------------------- |
| `ROUND(x, d)`            | **四舍五入到小数点后 d 位** | `ROUND(123.4567, 2)` → `123.46`                 | 默认 d = 0，即取整                |
| `FLOOR(x)`               | **向下取整**                | `FLOOR(123.9)` → `123`                          | 永远不大于原数                    |
| `CEIL(x)` / `CEILING(x)` | **向上取整**                | `CEIL(123.1)` → `124`                           | 永远不小于原数                    |
| `ABS(x)`                 | **取绝对值**                | `ABS(-10)` → `10`                               |                                   |
| `MOD(x, y)`              | **取模（余数）**            | `MOD(10, 3)` → `1`                              | 有些数据库也用 `%`                |
| `POWER(x, y)`            | **x 的 y 次方**             | `POWER(2, 3)` → `8`                             | 等价于 `2^3`                      |
| `SQRT(x)`                | **平方根**                  | `SQRT(16)` → `4`                                |                                   |
| `RAND()`                 | **生成 0~1 之间的随机数**   | `RAND()` → `0.726381`（每次不同）               | 可以用于生成测试数据              |
| `TRUNCATE(x, d)`         | **截断小数**                | `TRUNCATE(123.4567, 2)` → `123.45`              | 和 `ROUND()` 不同，**不四舍五入** |
| `LOG(x)`                 | **自然对数 ln(x)**          | `LOG(2.71828)` → `1`                            |                                   |
| `LOG10(x)`               | **以 10 为底对数**          | `LOG10(1000)` → `3`                             |                                   |
| `EXP(x)`                 | **e 的 x 次幂**             | `EXP(1)` → `2.71828`                            | 计算 e^x                          |
| `SIGN(x)`                | 返回数值的符号              | `SIGN(-8)` → `-1``SIGN(0)` → `0``SIGN(5)` → `1` | 判断正负                          |

------

#### ✅ 二、实战例子

##### 1. 查询订单金额保留两位小数（四舍五入）：

```sql
SELECT id, ROUND(amount, 2) AS rounded_amount
FROM orders;
```

##### 2. 查询每个订单金额向下取整值：

```sql
SELECT id, FLOOR(amount) AS floor_amount
FROM orders;
```

##### 3. 查询金额为负数的订单的绝对值：

```sql
SELECT id, ABS(amount) AS abs_amount
FROM orders
WHERE amount < 0;
```

##### 4. 随机抽取 5 个用户：

```sql
SELECT * FROM users
ORDER BY RAND()
LIMIT 5;
```

------

#### ✅ 三、和其他数据库的兼容性（简述）

| 函数             | MySQL | PostgreSQL     | SQL Server               | Oracle              |
| ---------------- | ----- | -------------- | ------------------------ | ------------------- |
| `ROUND`          | ✅     | ✅              | ✅                        | ✅                   |
| `FLOOR` / `CEIL` | ✅     | ✅              | ✅                        | ✅                   |
| `MOD`            | ✅     | ✅              | ✅（使用 `%` 或 `MOD()`） | ✅                   |
| `RAND()`         | ✅     | `RANDOM()`     | `RAND()`                 | `DBMS_RANDOM.VALUE` |
| `TRUNCATE()`     | ✅     | ✅（`TRUNC()`） | ❌                        | ✅（`TRUNC()`）      |

------

#### ✅ 四、典型面试题示例

> 💡 **题目：查询每位用户的平均订单金额，保留 2 位小数，并对结果从高到低排序**

```sql
SELECT u.id, u.name, ROUND(AVG(o.amount), 2) AS avg_amount
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
ORDER BY avg_amount DESC;
```

------

如果你希望我：

- 出几道关于 `ROUND / FLOOR / MOD` 等函数的面试题
- 或者给出一张测试表用于练习这些函数
   请随时告诉我！

### SQL字符串函数

SQL 中的**字符串函数**是面试和日常开发中的常用考点，常用于字符串的处理、截取、替换、拼接、搜索等操作。下面是分类整理的常用字符串函数，适用于 MySQL（其他数据库如 PostgreSQL / Oracle / SQL Server 语法略有不同）。

------

#### ✅ 一、常用字符串函数总览（MySQL）

| 函数                       | 说明                               | 示例                                   |
| -------------------------- | ---------------------------------- | -------------------------------------- |
| `LENGTH(str)`              | 返回字节长度（英文=1字节，中文=3） | `LENGTH('abc') → 3`                    |
| `CHAR_LENGTH(str)`         | 返回字符数（英文、中文都=1字符）   | `CHAR_LENGTH('你好') → 2`              |
| `CONCAT(s1, s2,…)`         | 字符串拼接                         | `CONCAT('abc', '123') → abc123`        |
| `SUBSTRING(str, pos, len)` | 截取子串                           | `SUBSTRING('abcdef', 2, 3) → bcd`      |
| `LEFT(str, n)`             | 从左边取 n 个字符                  | `LEFT('abcdef', 3) → abc`              |
| `RIGHT(str, n)`            | 从右边取 n 个字符                  | `RIGHT('abcdef', 2) → ef`              |
| `INSTR(str, sub)`          | 返回子串第一次出现位置             | `INSTR('abcabc', 'b') → 2`             |
| `REPLACE(str, a, b)`       | 替换子串 a 为 b                    | `REPLACE('abcabc', 'a', 'X') → XbcXbc` |
| `LOWER(str)`               | 转小写                             | `LOWER('ABC') → abc`                   |
| `UPPER(str)`               | 转大写                             | `UPPER('abc') → ABC`                   |
| `TRIM(str)`                | 去除前后空格                       | `TRIM(' abc ') → 'abc'`                |
| `LTRIM(str)`               | 去除左侧空格                       | `LTRIM(' abc') → 'abc'`                |
| `RTRIM(str)`               | 去除右侧空格                       | `RTRIM('abc ') → 'abc'`                |
| `REPEAT(str, n)`           | 重复字符串 n 次                    | `REPEAT('ab', 3) → ababab`             |
| `REVERSE(str)`             | 字符串反转                         | `REVERSE('abc') → cba`                 |
| `FORMAT(num, d)`           | 保留小数点并加千位分隔             | `FORMAT(12345.6789, 2) → 12,345.68`    |

------

#### ✅ 二、典型面试题型示例

##### ❓题 1：查询员工表中所有姓名为三个字的员工（汉字）

```sql
SELECT * FROM employees
WHERE CHAR_LENGTH(name) = 3;
```

##### ❓题 2：将手机号中间四位脱敏

```sql
SELECT CONCAT(LEFT(phone, 3), '****', RIGHT(phone, 4)) AS masked_phone
FROM users;
```

##### ❓题 3：查询 email 中包含 "admin" 的用户

```sql
SELECT * FROM users
WHERE INSTR(email, 'admin') > 0;
```

##### ❓题 4：将每个用户名转为大写展示

```sql
SELECT UPPER(username) FROM users;
```

##### ❓题 5：去掉用户名前后的空格

```sql
SELECT TRIM(username) FROM users;
```

------

#### ✅ 三、TRIM 的更多形式（MySQL）

```sql
TRIM([LEADING | TRAILING | BOTH] str FROM source)
```

##### 示例：

```sql
SELECT TRIM(BOTH 'x' FROM 'xxxabcxxx'); -- abc
SELECT TRIM(LEADING 'x' FROM 'xxxabcxxx'); -- abcxxx
SELECT TRIM(TRAILING 'x' FROM 'xxxabcxxx'); -- xxxabc
```

------

#### ✅ 四、字符串函数的综合应用（模拟题）

**题目：** 查询用户名 + 邮箱（用中横线拼接），并全部转为小写：

```sql
SELECT LOWER(CONCAT(username, '-', email)) AS user_info
FROM users;
```

------

如果你想要我帮你设计字符串处理专项练习题（含难度、答案、解析），或结合实际业务场景（脱敏、日志处理、搜索优化），也可以继续告诉我。

### SQL日期函数

SQL 中的**日期函数**是处理时间数据（如注册时间、订单时间、日志时间戳等）必不可少的工具，尤其在数据分析、统计报表、数据筛选等面试题中频繁出现。

------

#### ✅ 一、常用日期函数（MySQL 版）

| 函数                                                   | 功能                            | 示例                                                 |
| ------------------------------------------------------ | ------------------------------- | ---------------------------------------------------- |
| `NOW()`                                                | 当前日期和时间（带时分秒）      | `2025-06-09 19:30:00`                                |
| `CURDATE()`                                            | 当前日期（无时分秒）            | `2025-06-09`                                         |
| `CURTIME()`                                            | 当前时间（无日期）              | `19:30:00`                                           |
| `DATE()`                                               | 只取日期部分                    | `DATE('2025-06-09 10:20:30') → 2025-06-09`           |
| `YEAR(date)`                                           | 提取年份                        | `YEAR(NOW()) → 2025`                                 |
| `MONTH(date)`                                          | 提取月份                        | `MONTH('2025-06-09') → 6`                            |
| `DAY(date)`                                            | 提取“日”                        | `DAY('2025-06-09') → 9`                              |
| `HOUR(date)`                                           | 提取小时                        | `HOUR(NOW()) → 19`                                   |
| `MINUTE(date)`                                         | 提取分钟                        | `MINUTE(NOW()) → 30`                                 |
| `SECOND(date)`                                         | 提取秒                          | `SECOND(NOW()) → 15`                                 |
| `DATE_FORMAT(date, format)`                            | 格式化日期输出                  | `DATE_FORMAT(NOW(), '%Y-%m-%d') → '2025-06-09'`      |
| `STR_TO_DATE(str, format)`                             | 将字符串转为日期                | `STR_TO_DATE('2025-06-09', '%Y-%m-%d')`              |
| `DATEDIFF(d1, d2)`                                     | 返回两个日期之间的天数（d1-d2） | `DATEDIFF('2025-06-09', '2025-06-01') → 8`           |
| `TIMESTAMPDIFF(unit, d1, d2)`                          | 指定单位计算间隔                | `TIMESTAMPDIFF(DAY, '2025-06-01', '2025-06-09') → 8` |
| `ADDDATE(date, n)` / `DATE_ADD(date, INTERVAL n unit)` | 加时间                          | `DATE_ADD(NOW(), INTERVAL 7 DAY)`                    |
| `SUBDATE(date, n)` / `DATE_SUB(date, INTERVAL n unit)` | 减时间                          | `DATE_SUB(NOW(), INTERVAL 1 MONTH)`                  |

------

#### ✅ 二、典型格式化字符串

| 格式符 | 含义             | 示例   |
| ------ | ---------------- | ------ |
| `%Y`   | 4位年份          | `2025` |
| `%y`   | 2位年份          | `25`   |
| `%m`   | 两位月份         | `06`   |
| `%d`   | 两位日           | `09`   |
| `%H`   | 小时（24小时制） | `19`   |
| `%i`   | 分钟             | `30`   |
| `%s`   | 秒               | `59`   |

```sql
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日 %H:%i:%s');
-- 2025年06月09日 19:30:59
```

------

#### ✅ 三、面试常考日期 SQL 示例

##### ❓ 1. 查询最近 7 天的订单

```sql
SELECT * FROM orders
WHERE create_time >= NOW() - INTERVAL 7 DAY;
```

##### ❓ 2. 统计每月订单数量

```sql
SELECT DATE_FORMAT(create_time, '%Y-%m') AS month, COUNT(*) AS order_count
FROM orders
GROUP BY month
ORDER BY month DESC;
```

##### ❓ 3. 查询生日在本月的用户

```sql
SELECT * FROM users
WHERE MONTH(birthday) = MONTH(CURDATE());
```

##### ❓ 4. 查询年龄大于 30 岁的用户

```sql
SELECT * FROM users
WHERE TIMESTAMPDIFF(YEAR, birthday, CURDATE()) > 30;
```

------

#### ✅ 四、进阶面试题

##### 🔍 查询各用户的最近一次登录时间

```sql
SELECT user_id, MAX(login_time) AS last_login
FROM login_logs
GROUP BY user_id;
```

##### 🔍 查询每天新增用户数（近 7 天）

```sql
SELECT DATE(create_time) AS day, COUNT(*) AS new_users
FROM users
WHERE create_time >= CURDATE() - INTERVAL 7 DAY
GROUP BY day
ORDER BY day DESC;
```

------

如果你需要 PostgreSQL / SQL Server / Oracle 的日期函数对比，或需要我出**日期处理专项训练题 + 答案解析**，也可以继续告诉我。

### UNION & UNION ALL

在 SQL 中，`UNION` 和 `UNION ALL` 是用于**合并两个或多个 `SELECT` 查询结果**的重要关键字，常考于面试，尤其是涉及性能、去重、字段对齐等问题。

------

#### ✅ 一、UNION 与 UNION ALL 的区别

| 对比点   | `UNION`                 | `UNION ALL`                        |
| -------- | ----------------------- | ---------------------------------- |
| 功能     | 合并结果并 **自动去重** | 合并结果，**不去重**（保留重复行） |
| 性能     | 较慢，需要排序去重      | 较快，无需去重操作                 |
| 使用场景 | 需要去重的统计类查询    | 大多数实际场景推荐使用，提升性能   |

------

#### ✅ 二、使用要求（字段数量、类型必须一致）

1. **字段数量必须一致**：
   - 两个 `SELECT` 子句中 **字段个数必须相同**。
2. **字段类型兼容**：
   - 对应位置的字段类型要能够进行隐式转换（如 `INT` 与 `VARCHAR` 不能直接混用）。

------

#### ✅ 示例：字段必须匹配（数量和类型）

##### 表结构

```sql
users(id INT, name VARCHAR, city VARCHAR)
customers(id INT, full_name VARCHAR, region VARCHAR)
```

##### 正确用法

```sql
SELECT id, name, city FROM users
UNION
SELECT id, full_name, region FROM customers;
```

- 字段数量：3 个
- 字段类型：INT, VARCHAR, VARCHAR — 对应类型匹配

------

#### ✅ 三、示例对比

##### 1. `UNION`（去重）

```sql
SELECT city FROM users
UNION
SELECT region FROM customers;
```

🔎 会**去掉重复的城市/区域名称**。

##### 2. `UNION ALL`（不去重）

```sql
SELECT city FROM users
UNION ALL
SELECT region FROM customers;
```

🔍 **保留重复项**，更快，但可能包含重复数据。

------

#### ✅ 四、UNION 的排序与别名使用

可以在 **最后一个 `SELECT` 之后使用 `ORDER BY` 排序整个结果集**：

```sql
SELECT name FROM users
UNION ALL
SELECT full_name FROM customers
ORDER BY name;
```

> 💡 注意：如果两个字段名不一致，只能使用第一个 `SELECT` 的字段别名或顺序号排序，如 `ORDER BY 1`

------

#### ✅ 五、典型面试题：UNION 相关

##### 📌 题目：

查询所有注册用户和访客用户的手机号（去重），并按手机号排序。

```sql
SELECT phone FROM registered_users
UNION
SELECT phone FROM guest_users
ORDER BY phone;
```

##### 📌 题目：

查询注册用户和访客用户的手机号及来源（不去重）

```sql
SELECT phone, 'registered' AS source FROM registered_users
UNION ALL
SELECT phone, 'guest' AS source FROM guest_users;
```

------

### 结果集组合操作符

、在 MySQL 等关系型数据库中，**`UNION` 属于“结果集组合操作符”（Result Set Combination Operator）**，核心功能是将两个或多个独立 `SELECT` 查询的结果集合并为一个单一的结果集，同时自动去除合并后结果中的重复记录。

#### 1. 本质与核心特性

- **操作对象**：两个或多个结构兼容的 `SELECT` 结果集（而非直接操作表）。
- **核心行为**：
  1. **合并结构**：将多个 `SELECT` 的结果“纵向拼接”（行数增加，列数不变），要求所有 `SELECT` 返回的**列数一致**、对应列的**数据类型兼容**（如第一列都是字符串、第二列都是数值）。
  2. **自动去重**：合并后会扫描整个结果集，删除完全重复的行（即所有列值都相同的行），这是 `UNION` 与 `UNION ALL` 的关键区别。

#### 2. 与其他操作符的归类对比

为更清晰理解其定位，可将其与数据库中其他常见操作符对比：

| 操作符类别           | 代表操作符           | 核心作用                     | 操作对象                |
| -------------------- | -------------------- | ---------------------------- | ----------------------- |
| **结果集组合操作符** | `UNION`、`UNION ALL` | 合并多个 `SELECT` 的结果集   | 独立的 `SELECT` 结果集  |
| 关系运算操作符       | `JOIN`（内/左/右）   | 基于关联字段“横向关联”多个表 | 数据库表                |
| 逻辑运算操作符       | `AND`、`OR`、`NOT`   | 筛选满足逻辑条件的记录       | 查询条件中的表达式      |
| 比较运算操作符       | `=`、`>`、`<`、`IN`  | 比较字段值与目标值的关系     | 字段值与常量/其他字段值 |

#### 3. 关键使用规则（确保合法性）

`UNION` 并非可以随意合并结果，需遵循严格规则：

- **列数一致**：所有 `SELECT` 语句返回的列数必须相同（如第一个 `SELECT` 查 2 列，后续所有 `SELECT` 也必须查 2 列）。  
  示例（错误）：  

  ```sql
  SELECT name, age FROM student  -- 2列
  UNION
  SELECT id, name, gender FROM teacher;  -- 3列（列数不匹配，报错）
  ```

- **列类型兼容**：对应位置的列数据类型需可兼容（无需完全一致，但需能隐式转换，如 `INT` 和 `DECIMAL` 可兼容，`INT` 和 `VARCHAR` 可能不兼容）。  
  示例（合法）：  

  ```sql
  SELECT id (INT), create_time (DATETIME) FROM order_2023
  UNION
  SELECT id (INT), create_date (DATE) FROM order_2024;  -- DATETIME与DATE兼容
  ```

- **列名由第一个 `SELECT` 决定**：最终合并结果的列名，以第一个 `SELECT` 语句的列名为准，后续 `SELECT` 的列名会被忽略。  
  示例：  

  ```sql
  SELECT name AS 姓名, age AS 年龄 FROM student  -- 列名：姓名、年龄
  UNION
  SELECT teacher_name, teacher_age FROM teacher;  -- 列名仍为“姓名、年龄”
  ```

#### 4. 典型使用场景

- **合并同结构的分表数据**：例如将按年份拆分的订单表（`order_2023`、`order_2024`）合并查询，同时去除重复订单（若存在）。  

  ```sql
  SELECT order_id, user_id, amount FROM order_2023
  UNION
  SELECT order_id, user_id, amount FROM order_2024;
  ```

- **合并不同表的相似数据**：例如从“学生表”和“教师表”中合并查询所有人员的姓名和年龄，且确保无重复（如同一人可能既是学生又是兼职教师）。

总结：`UNION` 是专门用于“纵向合并多个 `SELECT` 结果集并去重”的操作符，属于结果集层面的组合工具，需严格遵循列数、列类型的兼容规则，在分表查询、多表相似数据合并场景中广泛使用。

#### 5.实例：union / unionall



在 MySQL 中，`UNION ALL` 是用于合并两个或多个 `SELECT` 查询结果集的操作符。它会将所有查询的结果组合成一个单一的结果集，与 `UNION` 相比，`UNION ALL` 不会去除重复记录，因此执行效率更高。

##### **`UNION ALL` 的基本语法**

```sql
SELECT 列1, 列2, ... FROM 表1
UNION ALL
SELECT 列1, 列2, ... FROM 表2
[UNION ALL
SELECT 列1, 列2, ... FROM 表3
...];
```

##### **使用条件**

1. **列数必须一致**：每个 `SELECT` 语句返回的列数必须相同（例如，第一个查询返回 3 列，后续所有查询也必须返回 3 列）。
2. **列类型兼容**：对应位置的列的数据类型必须兼容（例如，第一列都是字符串类型，第二列都是数值类型等）。
3. **列名可不同**：最终结果集的列名由第一个 `SELECT` 语句的列名决定。

#### **`UNION ALL` 与 `UNION` 的区别**

| 特性         | `UNION ALL`                  | `UNION`                        |
| ------------ | ---------------------------- | ------------------------------ |
| 重复记录处理 | 保留所有重复记录             | 自动去除重复记录（相当于去重） |
| 执行效率     | 更高（无需去重操作）         | 较低（需要额外的去重步骤）     |
| 适用场景     | 确认无重复记录或需要保留重复 | 需要合并并去重的场景           |

##### **示例说明**

假设有两个表：

- **表A（学生表）**

  | id   | name | age  |
  | ---- | ---- | ---- |
  | 1    | 张三 | 20   |
  | 2    | 李四 | 22   |

- **表B（教师表）**

  | id   | name | age  |
  | ---- | ---- | ---- |
  | 1    | 王五 | 35   |

  | 2   | 李四  | 22  |  -- 与表A中的“李四”重复

##### 1. 使用 `UNION ALL`

```sql
SELECT name, age FROM 表A
UNION ALL
SELECT name, age FROM 表B;
```

**结果**（保留重复记录）：

| name | age  |
| ---- | ---- |
| 张三 | 20   |
| 李四 | 22   |
| 王五 | 35   |

| 李四  | 22  |  -- 重复记录被保留

##### 2. 使用 `UNION`（对比）

```sql
SELECT name, age FROM 表A
UNION
SELECT name, age FROM 表B;
```

**结果**（自动去重）：

| name | age  |
| ---- | ---- |
| 张三 | 20   |

| 李四  | 22  |  -- 重复记录只保留一条
| 王五  | 35  |

##### **使用场景**

- 当需要合并多个表的结果，且确认数据无重复，或明确需要保留重复记录时（例如，合并“2023年订单”和“2024年订单”，允许同一订单号出现在不同年份）。
- 对查询效率要求较高的场景（因无需去重，`UNION ALL` 性能优于 `UNION`）。

#### **注意事项**

- 如果要对合并后的结果排序，需在最后一个 `SELECT` 后添加 `ORDER BY`（排序作用于整个合并结果）：

  ```sql
  SELECT name, age FROM 表A
  UNION ALL
  SELECT name, age FROM 表B
  ORDER BY age;  -- 对合并后的所有记录按age排序
  ```

- 避免在有大量重复数据的场景下盲目使用 `UNION ALL`，否则可能导致结果集过大，影响性能。



### 逻辑运算符

在 MySQL 中，逻辑运算符用于连接或判断条件表达式的真假，返回 `TRUE`（1）、`FALSE`（0）或 `NULL`（未知），主要用于 `WHERE` 子句（筛选数据）、`HAVING` 子句（筛选聚合结果）、`IF()`/`CASE` 等条件判断场景。

逻辑运算符是 MySQL 条件判断的核心，需重点掌握：  

- 基础运算符（`AND`/`OR`/`NOT`）用于简单多条件组合，注意短路求值和 `NULL` 交互；  
- 特殊运算符（`ALL`/`ANY`）用于与子查询结果比较，需理解“全量”和“任一”的逻辑；  
- 优先级和括号的使用，避免条件判断顺序错误。  

合理使用逻辑运算符能让筛选条件更精准，提升查询效率和可读性。

#### 一、基础逻辑运算符（常用）

基础逻辑运算符用于直接连接两个或多个条件，是日常查询中最频繁使用的类型。

| 运算符 | 名称   | 作用说明                                                     | 示例（基于 `Students` 表：id, name, score, age）             |
| ------ | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `AND`  | 逻辑与 | 所有条件**同时为真**时，结果才为真；只要有一个条件为假，结果为假。 | 筛选“分数>80 且 年龄>18”：<br>`WHERE score>80 AND age>18`    |
| `OR`   | 逻辑或 | 只要有一个条件**为真**，结果就为真；所有条件都为假时，结果才为假。 | 筛选“分数>90 或 年龄=20”：<br>`WHERE score>90 OR age=20`     |
| `NOT`  | 逻辑非 | 对单个条件取反（真变假，假变真）；若条件为 `NULL`，结果仍为 `NULL`。 | 筛选“分数不大于80”：<br>`WHERE NOT (score>80)`（等价于 `score<=80`） |

##### 关键特性：

1. **短路求值**：  
   - `AND`：如果第一个条件为假，后续条件不再判断（因为整体已确定为假）；  
   - `OR`：如果第一个条件为真，后续条件不再判断（因为整体已确定为真）。  
     示例：`WHERE age>100 AND score>90`，若 `age>100` 为假，`score>90` 不会被判断，提升效率。

2. **与 `NULL` 的交互**：  
   `NULL` 代表“未知”，与逻辑运算符结合时结果特殊：  
   - `TRUE AND NULL` → `NULL`（未知）；  
   - `FALSE AND NULL` → `FALSE`（已确定为假）；  
   - `TRUE OR NULL` → `TRUE`（已确定为真）；  
   - `FALSE OR NULL` → `NULL`（未知）；  
   - `NOT NULL` → `NULL`（无法对未知取反）。

#### 二、特殊逻辑运算符（与子查询结合）

这类运算符用于 **单个值与子查询返回的多个值** 进行比较，判断是否满足“全部”或“任一”条件，核心是 `ALL` 和 `ANY`（`SOME` 与 `ANY` 功能完全相同）。

| 运算符 | 名称     | 作用说明                                                     | 示例（基于 `Courses` 表：course, min_score）                 |
| ------ | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ALL`  | 逻辑全量 | 单个值与子查询返回的**所有值**都满足比较条件时，结果为真。   | 筛选“分数高于所有课程最低分”：<br>`WHERE score > ALL (SELECT min_score FROM Courses)` |
| `ANY`  | 逻辑任一 | 单个值与子查询返回的**任一值**满足比较条件时，结果为真（`SOME` 是别名）。 | 筛选“分数高于任一课程最低分”：<br>`WHERE score > ANY (SELECT min_score FROM Courses)`（等价于 `SOME`） |

##### 示例解析：

假设 `Courses` 表的 `min_score` 为 `[80, 75, 85]`：

- `score > ALL (...)`：要求分数 > 80 **且** > 75 **且** > 85 → 即分数 > 85（需大于所有值的最大值）；  
- `score > ANY (...)`：要求分数 > 80 **或** > 75 **或** > 85 → 即分数 > 75（需大于所有值的最小值）。

##### 注意事项：

1. **子查询要求**：`ALL`/`ANY` 后面的子查询必须返回 **单列数据**（多行），若返回多列会报错；  
2. **空结果集处理**：若子查询返回空集，`ALL` 条件默认返回 `TRUE`（SQL 标准），`ANY` 条件默认返回 `FALSE`；  
3. **与聚合函数的等价性**：  
   - `score > ALL (SELECT min_score ...)` 等价于 `score > (SELECT MAX(min_score) ...)`；  
   - `score > ANY (SELECT min_score ...)` 等价于 `score > (SELECT MIN(min_score) ...)`；  
     （聚合函数性能通常更优，复杂场景建议优先用聚合函数）。

#### 三、逻辑运算符的优先级

当一个条件中包含多个逻辑运算符时，优先级决定判断顺序（优先级高的先执行），若优先级相同则从左到右执行：  

1. **最高**：`NOT`（逻辑非）；  
2. **中间**：`AND`（逻辑与）；  
3. **最低**：`OR`（逻辑或）。  

##### 示例：

`WHERE score>80 AND age>18 OR name='张三'`  

- 实际执行顺序：先判断 `score>80 AND age>18`，再将结果与 `name='张三'` 做 `OR` 判断；  
- 若需调整顺序，用括号 `()` 强制优先级：`WHERE score>80 AND (age>18 OR name='张三')`。

#### 四、常见使用场景

1. **多条件筛选**：`WHERE` 子句中组合多个条件，例如“筛选分数80-90分且年龄18-22岁的学生”：  

   ```sql
   SELECT * FROM Students 
   WHERE score BETWEEN 80 AND 90 
     AND age BETWEEN 18 AND 22;
   ```

2. **聚合结果筛选**：`HAVING` 子句中筛选分组后的结果，例如“筛选平均分>85且人数>3的班级”：  

   ```sql
   SELECT class, AVG(score) AS avg_score, COUNT(*) AS num 
   FROM Students 
   GROUP BY class 
   HAVING avg_score>85 AND num>3;
   ```

3. **条件判断函数**：`IF()`/`CASE` 中使用逻辑运算符，例如“根据分数判断等级”：  

   ```sql
   SELECT name, score,
     CASE 
       WHEN score>=90 THEN '优秀'
       WHEN score>=80 AND score<90 THEN '良好'
       ELSE '及格'
     END AS grade
   FROM Students;
   ```

# MYSQL

| 分类               | 常考重点                             |
| ------------------ | ------------------------------------ |
| 基础语法           | `SELECT`、`WHERE`、`ORDER BY`        |
| 多表查询           | 各种连接，子查询                     |
| 聚合与分组         | `GROUP BY`、`HAVING`、聚合函数       |
| 数据操作           | `INSERT`、`UPDATE`、`DELETE`         |
| 索引               | 创建、使用场景、失效原因             |
| SQL 优化           | 执行计划、索引优化、分页优化、锁机制 |
| 数据库设计         | 范式、主键外键、约束                 |
| 复杂查询           | 窗口函数、递归查询、视图             |
| 实际应用题         | 业务场景查询、排行榜、层级关系       |
| 数据库相关基础知识 | 事务、复制、分库分表                 |

## SQL 查询

**可以，而且这是 SQL 查询中最经典、最常用的组合之一。**

在实际业务开发中，几乎 80% 以上的复杂查询都会同时用到这三个子句。

### 1. 执行顺序（关键）

理解它们的**执行顺序**对于写出正确的查询至关重要：

```sql
FROM / JOIN      -- 1. 连接表
   ↓
ON               -- 2. 指定连接条件
   ↓
WHERE            -- 3. 过滤行（在分组前）
   ↓
GROUP BY         -- 4. 分组
   ↓
HAVING           -- 5. 过滤组（在分组后）
   ↓
SELECT           -- 6. 选择列
   ↓
ORDER BY         -- 7. 排序

COUNT()是聚合函数，不能直接写在WHERE子句中（WHERE用于筛选行，聚合函数的筛选需用HAVING COUNT()）；    

-- MySQL 查询语句的正确执行顺序
FROM(including JOINs) ---> WHERE ---> GROUP BY ---> HAVING ---> SELECT ---> DISTINCT ---> ORDER BY ---> LIMIT/OFFSET
正确的执行顺序应该是
先找到要查询表格或连接要查询的表格，因此 FROM 才是第一步；
接下来是进行条件筛选，所以是 WHERE 紧随其后；
然后如果遇到表格有分组的需要，则需要先 GROUP BY；
分组时如果也存在筛选条件，这里就要用 HAVING 进行分组筛选；
这些执行过后才是查询操作 SELECT；
SELECT 的时候如果遇到重复数据，就需要去重，即使用 DISTINCT;
接下来如果要对查询后的数据进行排序，会用到 ORDER BY；
最后如果要指定返回的查询数据范围、条数则要用 LIMIT/OFFSET 函数。
```

**简单记忆：** 先连表 (`JOIN ON`) → 再过滤 (`WHERE`) → 最后分组 (`GROUP BY`)。

---

### 2. 标准语法结构

```sql
SELECT 
    表 1. 列，
    聚合函数 (表 2. 列)
FROM 表 1
JOIN 表 2 ON 表 1. 关联键 = 表 2. 关联键
WHERE 过滤条件
GROUP BY 分组字段;
```

---

### 3. 实战示例

#### 场景：统计 2024 年每个用户的“已完成”订单总金额

假设两张表：
- `users` (用户表：`id`, `name`)
- `orders` (订单表：`id`, `user_id`, `amount`, `status`, `created_at`)

```sql
SELECT 
    u.name AS user_name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
JOIN orders o ON u.id = o.user_id          -- 1. 连接表
WHERE o.status = 'completed'               -- 2. 过滤行（只统计已完成的）
  AND o.created_at >= '2024-01-01'         --    过滤行（只统计 2024 年）
GROUP BY u.id, u.name;                     -- 3. 按用户分组
```

---

### 4. 三个子句的具体职责

| 子句         | 职责                                               | 执行阶段 | 能否用聚合函数     |
| :----------- | :------------------------------------------------- | :------- | :----------------- |
| **JOIN ON**  | **定义表之间的关系**<br>决定哪些行能匹配在一起     | 最早     | ❌ 不能             |
| **WHERE**    | **过滤原始数据**<br>减少参与分组的数据量，提升性能 | 分组前   | ❌ 不能             |
| **GROUP BY** | **定义统计维度**<br>决定按什么字段进行汇总         | 过滤后   | N/A (它是分组依据) |

---

### 5. 特别注意：LEFT JOIN 中的 WHERE 陷阱

这是最容易出错的地方！**在 `LEFT JOIN` 中，`ON` 和 `WHERE` 的过滤逻辑完全不同。**

##### 场景：统计所有用户（包括没有订单的）的订单数

###### ❌ 错误写法（变成了 INNER JOIN）

如果把对右表（orders）的过滤放在 `WHERE` 中，会过滤掉没有订单的用户（因为 orders 字段为 NULL）。
```sql
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'  -- ⚠️ 这里会排除掉 o 为 NULL 的行（即无订单用户）
GROUP BY u.id;
```

###### ✅ 正确写法（保留 LEFT JOIN 特性）

将对右表的过滤条件移到 `ON` 中。
```sql
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id 
  AND o.status = 'completed'  -- ✅ 连接时就过滤，保留无订单用户
GROUP BY u.id;
```
*(注：如果不需要过滤订单状态，只是想统计所有订单，则不需要加 status 条件)*

---

### 6. 性能优化建议

1.  **WHERE 尽量前置**：`WHERE` 在 `GROUP BY` 之前执行，能减少参与分组的数据量，性能更好。
2.  **索引优化**：
    - `JOIN ON` 的关联字段最好有索引。
    - `WHERE` 中的过滤字段最好有索引。
    - `GROUP BY` 的字段最好有索引（避免使用临时表排序）。
3.  **避免 SELECT ***：只查询需要的字段，减少内存消耗。

### 总结

- **可以同时使用**，且非常普遍。
- **顺序**：`JOIN ON` → `WHERE` → `GROUP BY`。
- **核心区别**：`ON` 管连接，`WHERE` 管行过滤，`GROUP BY` 管数据聚合。
- **小心陷阱**：`LEFT JOIN` 时，对右表的过滤条件尽量写在 `ON` 里，除非你确实想把它变成内连接。

## MYSQL视图（VIEW）

视图是虚拟表，不存储实际数据，只存储查询定义。

对于简单视图（如无聚合、分组或连接的视图），数据库系统支持更新操作，并能将这些操作直接应用于基表。

视图可以简化复杂查询的书写，但如果视图定义复杂或涉及大数据集，执行时可能增加开销并降低性能。

创建视图并不总会提高查询速度；视图本身不优化查询，性能取决于视图定义的复杂性和数据库的优化能力，有时甚至因额外解析层而降低性能或保持原样。

### 一、核心定义

视图是基于一个或多个表（或其他视图）的**虚拟表**，本身不存储实际数据，仅存储查询语句（SELECT 语句），查询视图时会动态执行底层 SQL 并返回结果，可理解为“预定义的查询语句别名”。

### 二、核心语法

#### 1. 创建视图

```sql
-- 基础语法
CREATE [OR REPLACE] VIEW 视图名 [(列名1, 列名2, ...)]
AS SELECT 语句
[WITH [CASCADED | LOCAL] CHECK OPTION]; -- 插入/更新时的校验规则

-- 示例：创建员工薪资视图（仅展示部分字段）
CREATE OR REPLACE VIEW v_emp_salary AS
SELECT emp_id, emp_name, salary 
FROM emp 
WHERE salary > 5000;
```
- `OR REPLACE`：视图已存在时覆盖（避免先删除再创建）；
- `WITH CHECK OPTION`：限制通过视图插入/更新的数据必须满足视图的 WHERE 条件（下文详解）。

#### 2. 查询视图

```sql
-- 与查询普通表完全一致
SELECT * FROM v_emp_salary;
```

#### 3. 修改视图

```sql
-- 方式1：CREATE OR REPLACE 覆盖（推荐）
-- 方式2：ALTER 语句
ALTER VIEW v_emp_salary AS
SELECT emp_id, emp_name, salary, dept_id FROM emp WHERE salary > 4000;
```

#### 4. 删除视图

```sql
DROP VIEW [IF EXISTS] 视图名;
-- 示例
DROP VIEW IF EXISTS v_emp_salary;
```

### 三、核心特性

#### 1. 虚拟性（核心）

- 视图不存储数据，仅存储查询逻辑，每次查询视图都会重新执行底层 SELECT 语句；
- 底层表数据变更后，视图查询结果会实时同步（无数据一致性问题）。

#### 2. 简化复杂查询

将多表关联、复杂条件的查询封装为视图，后续只需查询视图即可，无需重复编写复杂 SQL：
```sql
-- 复杂多表查询封装为视图
CREATE VIEW v_emp_dept AS
SELECT e.emp_id, e.emp_name, d.dept_name 
FROM emp e
JOIN dept d ON e.dept_id = d.dept_id
WHERE e.salary > 5000;

-- 简化查询
SELECT * FROM v_emp_dept;
```

#### 3. 数据安全（权限控制）

- 可给用户授予视图的查询权限，而不授予底层表的权限，限制用户只能访问视图暴露的字段，隐藏敏感数据：
  ```sql
  -- 仅授予视图查询权限，隐藏薪资字段
  GRANT SELECT ON v_emp_name TO test_user;
  ```

#### 4. 可更新性（有条件）

视图支持 INSERT/UPDATE/DELETE 操作，但需满足严格条件（本质是操作底层表）：
##### 可更新条件（缺一不可）：

1. 视图的 SELECT 语句无 `DISTINCT`、`GROUP BY`、`HAVING`、`UNION` 等聚合/去重操作；
2. 视图无嵌套查询（FROM 子句中无其他视图）；
3. 视图的列全部来自底层表的列（无计算列、函数列，如 `salary+1000` 不可更新）；
4. 底层表有主键，且视图包含主键列。

##### 示例（可更新视图）：

```sql
-- 可更新视图（仅简单查询）
CREATE VIEW v_emp_basic AS
SELECT emp_id, emp_name, salary FROM emp;

-- 通过视图更新底层表数据
UPDATE v_emp_basic SET salary = 6000 WHERE emp_id = 1;
```

##### `WITH CHECK OPTION` 校验规则：

- 作用：限制通过视图插入/更新的数据，必须满足视图的 WHERE 条件；
- `CASCADED`（默认）：递归检查所有嵌套视图的条件；
- `LOCAL`：仅检查当前视图的条件。
```sql
-- 视图条件：salary > 5000
CREATE VIEW v_emp_salary AS
SELECT emp_id, emp_name, salary FROM emp WHERE salary > 5000
WITH CHECK OPTION;

-- 报错：更新后 salary=4000 不满足视图条件
UPDATE v_emp_salary SET salary = 4000 WHERE emp_id = 1;
```

### 四、视图 vs 表（核心对比）

| 维度     | 视图（VIEW）              | 普通表（TABLE）            |
| -------- | ------------------------- | -------------------------- |
| 数据存储 | 不存储实际数据（虚拟）    | 存储实际数据（物理）       |
| 占用空间 | 仅存储 SQL 语句，占用极小 | 存储数据，占用磁盘空间     |
| 数据更新 | 有严格条件限制            | 无额外限制（满足约束即可） |
| 性能     | 查询时动态执行 SQL，略低  | 直接读取数据，性能更高     |
| 核心用途 | 简化查询、权限控制        | 存储原始数据               |

### 五、使用注意事项

1. **性能问题**：复杂视图（多表关联、聚合函数）查询性能较低，避免在视图上做二次聚合/关联；
2. **索引失效**：视图本身无索引，查询视图的性能依赖底层表的索引，需优化底层表索引；
3. **避免嵌套视图**：多层嵌套视图会大幅降低查询效率，且难以调试；
4. **不可更新场景**：含聚合函数（SUM/COUNT）、计算列、DISTINCT 的视图不可更新；
5. **视图重命名**：MySQL 无直接重命名视图的语句，需先 DROP 再 CREATE（或用 `RENAME TABLE` 间接实现）。

### 六、核心总结

1. 视图是虚拟表，仅存储查询逻辑，不存储数据，查询时动态执行底层 SQL；
2. 核心价值：简化复杂查询、控制数据访问权限，部分简单视图支持更新；
3. 可更新视图需满足无聚合/去重/计算列等条件，`WITH CHECK OPTION` 可限制视图更新的数据范围；
4. 视图性能依赖底层表，避免复杂嵌套和高频查询复杂视图。

## 表结构（Table Structure）

### 一、核心定义

MySQL 表结构是数据库中对表的**元数据定义**，包含表的名称、列（字段）的属性（名称、类型、约束）、索引、存储引擎、字符集等核心信息，是存储和管理数据的基础框架。

### 二、表结构的核心组成部分

#### 1. 列（字段）属性（最核心）

每一列的定义包含以下关键属性，语法模板：
```sql
列名 数据类型 [约束条件] [默认值] [注释] [AUTO_INCREMENT] [NULL | NOT NULL];
```
##### （1）数据类型（基础）

决定列能存储的数据类型，分为三大类：
| 类型分类   | 常用类型                    | 适用场景               |
| ---------- | --------------------------- | ---------------------- |
| 数值型     | INT、BIGINT、FLOAT、DECIMAL | 年龄、ID、金额、分数等 |
| 字符串型   | VARCHAR、CHAR、TEXT         | 姓名、地址、备注等     |
| 日期时间型 | DATETIME、DATE、TIMESTAMP   | 订单时间、创建时间等   |

##### （2）约束条件（保证数据完整性）

| 约束类型 | 关键字      | 作用                               |
| -------- | ----------- | ---------------------------------- |
| 主键约束 | PRIMARY KEY | 唯一标识行，非空且唯一             |
| 非空约束 | NOT NULL    | 列值不能为 NULL                    |
| 唯一约束 | UNIQUE      | 列值唯一（可多个 NULL）            |
| 外键约束 | FOREIGN KEY | 关联另一张表的主键，保证数据一致性 |
| 检查约束 | CHECK       | 限制列值的范围（MySQL 8.0+ 生效）  |

##### （3）其他属性

- `DEFAULT`：列的默认值，插入数据时未指定则使用默认值；
- `AUTO_INCREMENT`：自增属性，仅适用于数值型主键，插入时自动生成唯一递增值；
- `COMMENT`：列的注释，便于理解字段含义；
- `NULL / NOT NULL`：指定列是否允许存储 NULL 值（默认允许 NULL）。

#### 2. 索引（优化查询性能）

依附于表结构的特殊数据结构，用于加速查询，常见类型：
- 主键索引（PRIMARY KEY）：自动创建，唯一且非空；
- 唯一索引（UNIQUE）：保证列值唯一；
- 普通索引（INDEX）：无唯一性约束，仅加速查询；
- 联合索引：基于多列创建的索引（如 `INDEX idx_name_age (name, age)`）；
- 全文索引（FULLTEXT）：适用于TEXT类型，加速文本检索。

#### 3. 存储引擎（决定表的存储特性）

MySQL 表的存储方式和特性由存储引擎决定，通过 `ENGINE` 指定：
| 常用存储引擎 | 核心特性                | 适用场景                  |
| ------------ | ----------------------- | ------------------------- |
| InnoDB       | 支持事务、外键、行级锁  | 业务表（订单、用户）      |
| MyISAM       | 不支持事务/外键，查询快 | 只读/少写表（日志、统计） |

#### 4. 字符集与排序规则

- `CHARSET`：指定表的字符集（如 `utf8mb4`，支持所有中文和emoji）；
- `COLLATE`：排序规则（如 `utf8mb4_general_ci`，不区分大小写；`utf8mb4_bin`，区分大小写）。

### 三、表结构的常用操作（DDL）

#### 1. 创建表（CREATE TABLE）

```sql
-- 完整示例：创建用户表
CREATE TABLE `user` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '用户ID（主键）',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `age` TINYINT UNSIGNED DEFAULT 0 COMMENT '年龄',
  `phone` CHAR(11) UNIQUE COMMENT '手机号（唯一）',
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `dept_id` INT UNSIGNED COMMENT '部门ID（外键）',
  PRIMARY KEY (`id`), -- 主键索引
  INDEX idx_create_time (`create_time`), -- 普通索引
  FOREIGN KEY (`dept_id`) REFERENCES `dept`(`id`) -- 外键关联部门表
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
```

#### 2. 查看表结构（核心调试操作）

```sql
-- 方式1：查看列的详细属性（最常用）
DESC / DESCRIBE 表名;
DESC `user`;

-- 方式2：查看创建表的完整SQL（含索引、引擎、字符集等）
SHOW CREATE TABLE 表名;
SHOW CREATE TABLE `user`;

-- 方式3：查询系统元数据表（information_schema）
SELECT * FROM information_schema.COLUMNS WHERE TABLE_NAME = 'user';
```

#### 3. 修改表结构（ALTER TABLE）

```
ALTER TABLE可以用于:
\- 添加列:ALTER TABLE 表名 ADD 列名 数据类型
\- 修改列:ALTER TABLE 表名 MODIFY 列名 新数据类型
\- 删除列:ALTER TABLE 表名 DROP COLUMN 列名
\- 添加约束:ALTER TABLE 表名 ADD CONSTRAINT 约束名 约束定义
```



##### （1）添加列

```sql
ALTER TABLE `user` ADD COLUMN `email` VARCHAR(100) DEFAULT NULL COMMENT '邮箱' AFTER `phone`;
```

##### （2）修改列属性

```sql
-- 修改列类型/约束/默认值
ALTER TABLE `user` MODIFY COLUMN `age` INT UNSIGNED DEFAULT 18 COMMENT '年龄（修改后）';
-- 重命名列
-- 修改列定义：将 phone 列的数据类型调整为 CHAR(11)。重命名列：将列名从 phone 更改为 mobile。
-- 添加唯一约束：确保手机号在表中是唯一的。更新注释：修改列的描述信息。
ALTER TABLE `user` CHANGE COLUMN `phone` `mobile` CHAR(11) UNIQUE COMMENT '手机号（重命名）';
```

##### （3）删除列

```sql
ALTER TABLE `user` DROP COLUMN `email`;
```

##### （4）添加/删除索引

```sql
-- 添加普通索引
ALTER TABLE `user` ADD INDEX idx_age (`age`);
-- 删除索引
ALTER TABLE `user` DROP INDEX idx_age;
-- 添加主键（若未设置）
ALTER TABLE `user` ADD PRIMARY KEY (`id`);
```

##### （5）修改存储引擎/字符集

```sql
-- 修改存储引擎
ALTER TABLE `user` ENGINE = MyISAM;
-- 修改字符集
ALTER TABLE `user` DEFAULT CHARSET = utf8;
```

#### 4. 删除表（DROP TABLE）

```sql
-- 安全删除（先判断是否存在）
DROP TABLE IF EXISTS `user`;
```

##### 5. 重命名表

```sql
RENAME TABLE `user` TO `user_info`;
```

### 四、表结构设计的最佳实践

1. **数据类型选型**：
   - 优先选占用空间小的类型（如年龄用 TINYINT 而非 INT）；
   - 金额用 DECIMAL 而非 FLOAT/DOUBLE（避免精度丢失）；
   - 变长字符串用 VARCHAR，定长用 CHAR（如手机号用 CHAR(11)）。
2. **约束设计**：
   - 每张表必须设置主键（推荐自增 INT/BIGINT 或 UUID）；
   - 非空字段显式设置 NOT NULL，避免 NULL 值引发查询异常；
   - 外键约束谨慎使用（可通过业务逻辑保证一致性，提升性能）。
3. **索引设计**：
   - 索引不是越多越好（增删改会变慢）；
   - 高频查询的字段创建索引，避免给大文本/频繁更新的字段建索引；
   - 联合索引遵循“最左前缀原则”。
4. **注释规范**：
   - 表和列都添加清晰的 COMMENT，便于后期维护；
   - 字段命名见名知意（如 `create_time` 而非 `ct`）。

### 五、核心总结

1. MySQL 表结构包含列属性、索引、存储引擎、字符集等核心部分，列属性是基础，约束保证数据完整性，索引优化查询性能；
2. 常用操作：CREATE（创建）、DESC/SHOW CREATE TABLE（查看）、ALTER（修改）、DROP（删除）、RENAME（重命名）；
3. 表结构设计需遵循“精简、规范、高效”原则：合理选型数据类型，按需设置约束和索引，添加清晰注释。

## MySQL 常考语句分类整理

### 一、数据查询（SELECT）

| 类型     | 示例                                                       |
| -------- | ---------------------------------------------------------- |
| 基础查询 | `SELECT * FROM users;`                                     |
| 指定列   | `SELECT id, name FROM users;`                              |
| 条件查询 | `SELECT * FROM users WHERE age > 25;`                      |
| 排序     | `SELECT * FROM users ORDER BY age DESC;`                   |
| 去重     | `SELECT DISTINCT city FROM users;`                         |
| 分页     | `SELECT * FROM users LIMIT 10 OFFSET 20;`                  |
| 别名     | `SELECT name AS username FROM users;`                      |
| 模糊匹配 | `SELECT * FROM users WHERE name LIKE 'A%';`                |
| 范围查询 | `SELECT * FROM users WHERE age BETWEEN 20 AND 30;`         |
| 多条件   | `SELECT * FROM users WHERE age > 20 AND city = 'Beijing';` |

------

### 二、多表操作（连接与子查询）

| 类型              | 示例                                                         |
| ----------------- | ------------------------------------------------------------ |
| 内连接            | `SELECT u.name, o.amount FROM users u JOIN orders o ON u.id = o.user_id;` |
| 左连接            | `SELECT u.name, o.amount FROM users u LEFT JOIN orders o ON u.id = o.user_id;` |
| 右连接            | `SELECT u.name, o.amount FROM users u RIGHT JOIN orders o ON u.id = o.user_id;` |
| 子查询（WHERE中） | `SELECT name FROM users WHERE id IN (SELECT user_id FROM orders);` |
| 子查询（FROM中）  | `SELECT t.user_id, COUNT(*) FROM (SELECT * FROM orders WHERE amount > 100) t GROUP BY t.user_id;` |

------

### 三、聚合与分组

| 类型        | 示例                                                         |
| ----------- | ------------------------------------------------------------ |
| 总数        | `SELECT COUNT(*) FROM users;`                                |
| 求和        | `SELECT SUM(amount) FROM orders;`                            |
| 平均值      | `SELECT AVG(age) FROM users;`                                |
| 最大/最小值 | `SELECT MAX(age), MIN(age) FROM users;`                      |
| 分组统计    | `SELECT city, COUNT(*) FROM users GROUP BY city;`            |
| 分组条件    | `SELECT city, COUNT(*) FROM users GROUP BY city HAVING COUNT(*) > 10;` |

------

### 四、数据更新与写入

| 类型       | 示例                                                         |
| ---------- | ------------------------------------------------------------ |
| 插入       | `INSERT INTO users(name, age) VALUES('Alice', 30);`          |
| 批量插入   | `INSERT INTO users(name, age) VALUES ('Bob', 25), ('Cathy', 22);` |
| 插入或更新 | `INSERT INTO users(id, name) VALUES (1, 'Tom') ON DUPLICATE KEY UPDATE name='Tom';` |
| 更新数据   | `UPDATE users SET age = 28 WHERE id = 1;`                    |
| 删除数据   | `DELETE FROM users WHERE age < 18;`                          |

------

### 五、索引与性能优化相关

| 类型           | 示例                                                        |
| -------------- | ----------------------------------------------------------- |
| 查看索引       | `SHOW INDEX FROM users;`                                    |
| 创建索引       | `CREATE INDEX idx_age ON users(age);`                       |
| 删除索引       | `DROP INDEX idx_age ON users;`                              |
| 使用执行计划   | `EXPLAIN SELECT * FROM users WHERE age > 30;`               |
| 查看慢查询日志 | `SHOW VARIABLES LIKE 'slow_query_log%';`                    |
| 强制使用索引   | `SELECT * FROM users FORCE INDEX (idx_age) WHERE age > 25;` |

------

### 六、事务与锁操作

| 类型             | 示例                                                         |
| ---------------- | ------------------------------------------------------------ |
| 开启事务         | `START TRANSACTION;` 或 `BEGIN;`                             |
| 提交事务         | `COMMIT;`                                                    |
| 回滚事务         | `ROLLBACK;`                                                  |
| 设置隔离级别     | `SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;`   |
| 查看当前隔离级别 | `SELECT @@tx_isolation;`                                     |
| 悲观锁           | `SELECT * FROM users WHERE id = 1 FOR UPDATE;`               |
| 乐观锁           | `UPDATE users SET age = 26, version = version + 1 WHERE id = 1 AND version = 2;` |

------

### Bonus：面试高频场景题

| 问题                       | 示例                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 查询每个用户的最后一笔订单 | 使用 `GROUP BY` + `MAX(order_time)` 或 `子查询 + JOIN` 或 窗口函数 ROW_NUMBER () |
| 查询重复数据               | `SELECT name, COUNT(*) FROM users GROUP BY name HAVING COUNT(*) > 1;` |
| 查询某字段为空             | `SELECT * FROM users WHERE phone IS NULL;`                   |
| 查询某天注册的用户         | `SELECT * FROM users WHERE DATE(register_time) = '2024-01-01';` |
| 分页优化                   | `SELECT * FROM users WHERE id > ? LIMIT 10;` 替代 OFFSET     |

------

## 一、SQL 基础语法

- **数据查询**
  - `SELECT` 基础查询
  - 字段筛选、排序 (`ORDER BY`)
  - 限制返回行数 (`LIMIT` / `TOP`)
- **条件过滤**
  - `WHERE` 语句及条件表达式
  - 逻辑运算符（AND、OR、NOT）
  - 模糊查询（`LIKE`）
  - 范围查询（`BETWEEN`）
  - 空值处理（`IS NULL` / `IS NOT NULL`）

以下是对你列出的 SQL 基础语法知识点的**详细说明与示例**，适合作为系统复习材料或面试准备：



------

### 1. 数据查询（`SELECT`）

#### **基本格式**：

```sql
SELECT column1, column2, ...
FROM table_name;
```

#### **示例：**

```sql
SELECT name, age FROM employees;
```

------

### 2. 字段筛选与排序（`ORDER BY`）

#### **语法**：

```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC];
```

#### **示例：**

```sql
SELECT name, salary FROM employees ORDER BY salary DESC;
```

------

### 3. 限制返回行数（`LIMIT` 或 `TOP`）

- **MySQL / PostgreSQL / SQLite：**

```sql
SELECT * FROM employees LIMIT 5;
```

- **SQL Server：**

```sql
SELECT TOP 5 * FROM employees;
```

------

### 4. 条件过滤（`WHERE`）

#### **语法：**

```sql
SELECT * FROM employees WHERE age > 30;
```

------

### 5. 逻辑运算符（`AND` / `OR` / `NOT`）

#### **示例：**

```sql
SELECT * FROM employees
WHERE age > 30 AND department = 'HR';
SELECT * FROM employees
WHERE department = 'IT' OR department = 'Finance';
SELECT * FROM employees
WHERE NOT department = 'Sales';
```

------

### 6. 模糊查询（`LIKE`）

#### **语法：**

```sql
SELECT * FROM employees WHERE name LIKE 'A%';
-- 以A开头
SELECT * FROM employees WHERE name LIKE '%son';
-- 以son结尾
SELECT * FROM employees WHERE name LIKE '%ar%';
-- 包含“ar”
```

------

### 7. 范围查询（`BETWEEN`）

#### **语法：**

```sql
SELECT * FROM employees WHERE salary BETWEEN 5000 AND 10000;
```

------

### 8. 空值判断（`IS NULL` / `IS NOT NULL`）

#### **示例：**

```sql
SELECT * FROM employees WHERE email IS NULL;
SELECT * FROM employees WHERE phone IS NOT NULL;
```

------

### ✅ 小结图表：

| 操作类型 | 关键字/语法          | 示例                           |
| -------- | -------------------- | ------------------------------ |
| 基本查询 | `SELECT`             | `SELECT name FROM employees;`  |
| 条件筛选 | `WHERE`              | `WHERE age > 30`               |
| 排序     | `ORDER BY`           | `ORDER BY salary DESC`         |
| 限制行数 | `LIMIT` / `TOP`      | `LIMIT 5` / `TOP 5`            |
| 多条件   | `AND` / `OR` / `NOT` | `WHERE age > 30 AND dept='HR'` |
| 模糊匹配 | `LIKE`               | `LIKE 'A%'`                    |
| 范围查询 | `BETWEEN`            | `BETWEEN 5000 AND 10000`       |
| 空值判断 | `IS NULL`            | `email IS NULL`                |

------

如果你需要我继续扩展下一章（如**分组、聚合函数、连接、窗口函数、事务**等），可以直接告诉我章节名或说“继续”。

------

## 二、多表查询与连接

- **连接类型**
  - 内连接 (`INNER JOIN`)
  - 左连接 (`LEFT JOIN`)、右连接 (`RIGHT JOIN`)
  - 全连接 (`FULL OUTER JOIN`)（部分数据库支持）
  - 自连接（表自身连接）
- **连接条件**
  - 使用 `ON` 或 `USING` 指定连接条件
- **子查询**
  - 标量子查询、行子查询、表子查询
  - 相关子查询与非相关子查询
- **嵌套查询**（在 `WHERE`、`FROM` 中使用子查询）

------

### 1. **连接类型（Join Types）**

#### ✅ 1.1 内连接（`INNER JOIN`）

仅返回两张表中满足连接条件的记录。inner join时只会对非NULL的记录做join,

```sql
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

##### 内连接写法

是的，**在 SQL 中 `INNER JOIN` 的 `INNER` 关键字是可以省略的**，省略后默认就是内连接。

------

###### ✅ 举例说明：

这两个语句是完全等价的：

```sql
-- 写法1：标准写法，明确指定 INNER
SELECT * 
FROM orders 
INNER JOIN customers ON orders.customer_id = customers.id;

-- 写法2：省略 INNER，默认就是 INNER JOIN
SELECT * 
FROM orders 
JOIN customers ON orders.customer_id = customers.id;
```

------

###### ✅ 总结：

| 写法         | 连接类型   | 是否推荐         |
| ------------ | ---------- | ---------------- |
| `INNER JOIN` | 内连接     | ✅ 推荐写法，清晰 |
| `JOIN`       | 默认内连接 | ✅ 可用，简洁     |

出现在面试或实际开发中时，为了 **提高可读性和清晰性**，建议在正式 SQL 中写明 `INNER JOIN`，尤其当和 `LEFT JOIN`、`RIGHT JOIN` 混用时，避免歧义。

##### SQL语句解释

这条 SQL 是一条 **内连接（INNER JOIN）查询语句**，它的功能是从 `employees` 表和 `departments` 表中查询出每个员工的姓名及其所属部门名称。下面是对这条语句的详细解释：

------

###### ✅ **SQL 语句**

```sql
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

------

###### 🔍 **语句解释**

| 部分                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `SELECT e.name, d.name AS department` | 选择结果中的两列：1. `e.name` 是员工姓名2. `d.name` 是部门名称，被起别名为 `department` |
| `FROM employees e`                    | 指定主表 `employees`，并起别名为 `e`                         |
| `INNER JOIN departments d`            | 将 `employees` 表和 `departments` 表进行 **内连接**，并将 `departments` 起别名为 `d` |
| `ON e.department_id = d.id`           | 连接条件：员工表的 `department_id` 与部门表的 `id` 匹配      |

------

##### **前提假设**

假设数据表结构如下：

###### `employees` 表

| id   | name  | department_id |
| ---- | ----- | ------------- |
| 1    | Alice | 1             |
| 2    | Bob   | 2             |
| 3    | Carol | NULL          |

###### `departments` 表

| id   | name        |
| ---- | ----------- |
| 1    | HR          |
| 2    | Engineering |

------

#####  **执行结果（示例）**

| name  | department  |
| ----- | ----------- |
| Alice | HR          |
| Bob   | Engineering |

> 注意：Carol 没有部门（`department_id` 为 `NULL`），所以在 INNER JOIN 中会被排除。

------

###### ✅ 关键点总结

- `INNER JOIN` 只保留两边表都有匹配的数据行。
- 若某员工没有对应部门（如 `department_id` 是 `NULL`），该员工不会出现在结果中。
- `AS department` 是起别名，让结果更语义化。

------

如需保留未分配部门的员工，也可改用 `LEFT JOIN`，如下所示：

```sql
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

这时 `Carol` 也会出现在结果中，`department` 为 `NULL`。

如需更多 JOIN 的例子或练习题，也可以继续问我！

------

#### ✅ 1.2 左连接（`LEFT JOIN`）

返回左表所有记录，右表匹配不到则为 `NULL`。

```sql
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

------

#### ✅ 1.3 右连接（`RIGHT JOIN`）

返回右表所有记录，左表匹配不到则为 `NULL`。适用于需要展示所有右表数据的场景。

```sql
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

------

#### ✅ 1.4 全连接（`FULL OUTER JOIN`）

返回所有记录，不论两边是否匹配（MySQL 不直接支持，可用 `UNION` 模拟）。

```sql
-- PostgreSQL / SQL Server 支持
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

------

#### ✅ 1.5 自连接（Self Join）

同一张表与自己连接，常用于上下级结构、员工推荐人等。

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

#### CROSS JOIN（交叉连接）

##### 一、核心定义

`CROSS JOIN`（交叉连接）是 MySQL 中一种表连接方式，也叫**笛卡尔积连接**，会将左表的每一行与右表的每一行进行组合，最终结果集的行数 = 左表行数 × 右表行数，无任何连接条件（或连接条件恒成立）。

##### 二、语法与基础示例

###### 1. 基础语法（两种等价写法）

```sql
-- 写法1：显式 CROSS JOIN（推荐，语义清晰）
SELECT 列名 FROM 表1 CROSS JOIN 表2;

-- 写法2：隐式笛卡尔积（省略 CROSS JOIN，仅用逗号分隔表）
SELECT 列名 FROM 表1, 表2;
```

###### 2. 实战示例

**（1）准备测试表**

```sql
-- 表1：商品分类（category）
CREATE TABLE category (
  cid INT,
  cname VARCHAR(20)
);
INSERT INTO category VALUES (1, '电子产品'), (2, '生活用品');

-- 表2：店铺（shop）
CREATE TABLE shop (
  sid INT,
  sname VARCHAR(20)
);
INSERT INTO shop VALUES (101, '京东'), (102, '淘宝');
```

**（2）执行 CROSS JOIN**

```sql
-- 显式交叉连接
SELECT c.cid, c.cname, s.sid, s.sname 
FROM category CROSS JOIN shop;
```

**（3）结果集（共 2×2=4 行）**

| cid  | cname    | sid  | sname |
| ---- | -------- | ---- | ----- |
| 1    | 电子产品 | 101  | 京东  |
| 1    | 电子产品 | 102  | 淘宝  |
| 2    | 生活用品 | 101  | 京东  |
| 2    | 生活用品 | 102  | 淘宝  |

##### 三、核心特性

###### 1. 无连接条件（核心）

`CROSS JOIN` 不需要 `ON` 子句指定连接条件（加 `ON 1=1` 等价于无条件），如果强行加有效条件（如 `ON c.cid = s.sid`），会退化为 `INNER JOIN`。

###### 2. 结果集大小

结果集行数是参与连接的所有表行数的乘积，若表数据量大（如左表1万行、右表1万行），会生成1亿行数据，**性能极低**，需谨慎使用。

###### 3. 与其他连接的区别

| 连接类型   | 核心特征                   | 结果集逻辑              |
| ---------- | -------------------------- | ----------------------- |
| CROSS JOIN | 无连接条件，笛卡尔积       | 左表×右表所有行组合     |
| INNER JOIN | 有 ON 连接条件             | 仅满足条件的行组合      |
| LEFT JOIN  | 左表全保留，右表匹配补NULL | 左表所有行 + 右表匹配行 |

##### 四、适用场景（极少，需严格限制）

###### 1. 生成测试数据

需要批量生成组合数据时（如生成“分类-店铺”全组合测试用例），是 `CROSS JOIN` 最常见的合法场景。

###### 2. 多表全组合查询

极少数业务场景需要全组合结果（如所有日期与所有商品的匹配），但需先确认数据量，避免性能问题。

##### 五、注意事项

1. **性能风险**：`CROSS JOIN` 是性能最差的连接方式，禁止对大表使用（会耗尽数据库资源）；
2. **隐式笛卡尔积**：开发中误写 `SELECT * FROM a, b`（省略连接条件）会等价于 `CROSS JOIN`，是高频错误，需避免；
3. **条件过滤**：若需缩小结果集，需在 `WHERE` 子句中添加过滤条件（但优先用 `INNER JOIN + ON` 替代）：
   ```sql
   -- 不推荐：CROSS JOIN + WHERE（先笛卡尔积再过滤，性能差）
   SELECT * FROM category CROSS JOIN shop WHERE c.cid = 1;
   
   -- 推荐：INNER JOIN + ON（先过滤再连接，性能优）
   SELECT * FROM category INNER JOIN shop ON c.cid = 1;
   ```

##### 六、核心总结

1. `CROSS JOIN` 即笛卡尔积连接，结果集行数 = 左表行数 × 右表行数，无连接条件；
2. 语法上 `CROSS JOIN` 与 `表1, 表2` 等价，显式写法更易读，但均需警惕性能问题；
3. 仅适用于生成测试数据、小表全组合查询等极少场景，大表禁止使用，优先用 `INNER JOIN` 替代；
4. 开发中需避免误写无连接条件的多表查询，防止隐式笛卡尔积。

在 SQL 中，`INNER JOIN` 和 `JOIN` 是**完全相同的**，二者作用一致，均表示“内连接”，仅返回两个表中满足连接条件的匹配行。

#### INNER JOIN 与 其他连接

##### 1. 语法本质：JOIN 是 INNER JOIN 的简写

SQL 标准中，`JOIN` 关键字默认就是“内连接”的含义，`INNER` 是可选的冗余关键字，用于明确标注连接类型（提升代码可读性）。  
二者执行逻辑、返回结果完全一致，以下两条查询语句效果相同：
```sql
-- 用 INNER JOIN 明确标注内连接
SELECT a.列1, b.列2
FROM 表A a
INNER JOIN 表B b 
  ON a.关联列 = b.关联列; -- 连接条件（必须满足才返回）

-- 用 JOIN 简写（默认内连接）
SELECT a.列1, b.列2
FROM 表A a
JOIN 表B b 
  ON a.关联列 = b.关联列;
```
返回结果：仅包含表A和表B中“关联列值匹配”的行，不匹配的行（如仅在表A存在、或仅在表B存在的行）会被过滤掉。

---

##### 2. 关键区分：避免与其他连接类型混淆

虽然 `JOIN` 和 `INNER JOIN` 相同，但需注意：**其他连接类型（如左连接、右连接）必须加前缀，不能简写**，否则会被默认视为内连接，导致结果错误。  
常见连接类型对比如下：

| 连接类型         | 关键字写法                | 核心作用（返回结果）                          | 能否简写 |
|------------------|---------------------------|-----------------------------------------------|----------|
| 内连接（重点）   | `INNER JOIN` 或 `JOIN`    | 仅返回两表中匹配的行                          | 能（`JOIN`） |
| 左连接           | `LEFT JOIN` 或 `LEFT OUTER JOIN` | 返回左表所有行 + 右表匹配行（右表不匹配则补NULL） | 能（`LEFT JOIN`） |
| 右连接           | `RIGHT JOIN` 或 `RIGHT OUTER JOIN` | 返回右表所有行 + 左表匹配行（左表不匹配则补NULL） | 能（`RIGHT JOIN`） |
| 全连接（部分数据库支持） | `FULL JOIN` 或 `FULL OUTER JOIN` | 返回两表所有行（不匹配则补NULL） | 能（`FULL JOIN`） |

> 注意：MySQL 不支持 `FULL JOIN`，需通过 `LEFT JOIN + UNION RIGHT JOIN` 实现类似效果。

---

##### 3. 建议：推荐使用 INNER JOIN 提升可读性

虽然 `JOIN` 简写更简洁，但在团队协作或复杂SQL中，**推荐显式使用 `INNER JOIN`**，原因如下：
- 避免歧义：明确标注“内连接”，让其他开发者（或未来的自己）快速理解连接逻辑，无需依赖“默认规则”。
- 一致性：与 `LEFT JOIN`、`RIGHT JOIN` 等其他连接类型的写法保持一致（均带前缀），代码结构更统一。

---

要不要我帮你整理一份**4种常见连接类型的对比表+实操SQL示例**？包含创建测试表、插入数据、不同连接的查询语句及结果分析，方便你直观理解差异。

### 2. **连接条件（ON / USING）**

- `ON` 用于指定任意连接条件（更常用、更灵活）；
- `USING(column_name)` 用于两个表中有相同列名时，简化写法：

```sql
SELECT * FROM orders
JOIN customers USING(customer_id);
```

你这部分关于 `ON` 和 `USING` 的介绍已经非常准确了，为了让理解更完整，我可以补充一下它们的区别、适用场景和注意事项，并加上对比示例。

------

##### ✅ `ON` 与 `USING` 的区别详解

| 特性     | `ON`                             | `USING`                                  |
| -------- | -------------------------------- | ---------------------------------------- |
| 灵活性   | ⭐⭐⭐⭐⭐ 高，可任意指定连接条件     | ⭐⭐⭐ 低，仅适用于同名列                   |
| 使用场景 | 可用于任何连接（列名不同也可以） | 仅当两个表中存在相同列名时使用           |
| 列名控制 | 保留两表的列名                   | 返回的结果中只保留一份该列名             |
| 可读性   | 较低，需手动匹配                 | 较高，适合简化语句                       |
| 多列连接 | 支持                             | 支持多个同名列（用 `USING(col1, col2)`） |

------

##### 📌 示例对比

###### 1. 使用 `ON`

```sql
SELECT o.id, c.name
FROM orders o
JOIN customers c
ON o.customer_id = c.id;
```

- 两个表的列名不同（`o.customer_id` vs `c.id`）
- 必须使用 `ON` 指定连接条件

###### 2. 使用 `USING`

```sql
SELECT o.id, c.name
FROM orders o
JOIN customers c
USING (customer_id);
```

- 两个表都有 `customer_id` 字段，**列名相同**
- 结果中只有一个 `customer_id` 字段
- SQL 更简洁，自动处理相同字段匹配

------

##### ⚠️ 注意事项

1. `USING` 后的字段名必须在两个表中都存在。
2. 使用 `USING` 后，该字段在返回结果中只出现一次（会自动消除歧义）。
3. 如果你想保留两个表的字段名，必须用 `ON` 明确指定。

------

##### ✅ 小结

- **开发中建议**：大多数情况下使用 `ON`，因为它更灵活、适用于复杂连接逻辑。
- **如果两个表字段名一致且连接逻辑简单**，可以用 `USING` 简化写法。

如你正在准备 SQL 面试或实战项目，也可以问我要相关练习题或更多 JOIN 场景案例。

------

### 3. **子查询（Subqueries）**

------

#### ✅ 3.1 标量子查询（返回单个值）

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

------

#### ✅ 3.2 行子查询（返回单行多列）

```sql
SELECT *
FROM employees
WHERE (department_id, job_id) =
      (SELECT department_id, job_id FROM employees WHERE id = 1001);
```

------

#### ✅ 3.3 表子查询（作为一张虚拟表参与查询）

```sql
SELECT d.dept_name, t.avg_salary
FROM departments d
JOIN (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) t ON d.id = t.department_id;
```

------

### 4. **相关子查询（correlated）与非相关子查询**

- **非相关子查询**：子查询先执行，不依赖外层查询。
- **相关子查询**：子查询依赖外层查询的每一行执行一次。

✅ 非相关子查询：

```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'Beijing'
);
```

✅ 相关子查询：

```sql
SELECT name
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

------

### 5. 嵌套查询（WHERE、FROM 中）

- **在 WHERE 中使用子查询（常见）**

```sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id FROM departments WHERE name = 'HR'
);
```

- **在 FROM 中嵌套子查询**

```sql
SELECT t.department_id, t.avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) t
WHERE t.avg_salary > 10000;
```

### 6.CTE（WITH 子句）

#### 一、定义

**CTE = Common Table Expression，公共表表达式**
就是 **WITH 子句**：
可以把**一段查询结果定义成一个临时结果集**，像临时表一样使用，**只在当前 SQL 中有效**。

#### 二、基本语法

```sql
WITH 临时名称 AS (
    SELECT ...  -- 子查询
)
SELECT * FROM 临时名称;
```

等价于：**把子查询提到前面，起个名字，后面随便用**。

#### 三、作用（重点）

1. **让 SQL 更清晰、易读**
2. 替代复杂子查询、嵌套子查询
3. 可以**多次引用**同一个临时结果集
4. 可以写**递归 SQL**（树形结构查询：部门、菜单、分类）

---

#### 四、和普通子查询的区别

##### 普通子查询（嵌套）

```sql
SELECT * FROM (
    SELECT user_id, MAX(order_time) FROM order GROUP BY user_id
) t;
```

##### CTE（WITH）写法

```sql
WITH t AS (
    SELECT user_id, MAX(order_time) FROM order GROUP BY user_id
)
SELECT * FROM t;
```

**逻辑完全一样，只是结构更干净。**

---

#### 五、你刚才那句 SQL 里的 CTE 是什么意思？

你问的是这句：
```sql
WITH user_last_order AS (
    SELECT user_id, MAX(order_time) AS last_time
    FROM `order`
    GROUP BY user_id
)
SELECT o.*
FROM `order` o
JOIN user_last_order ulo
ON o.user_id = ulo.user_id AND o.order_time = ulo.last_time;
```

这里的 CTE 作用：
- 先查询出**每个用户最后下单时间**
- 给它起名 `user_last_order`
- 后面直接 JOIN 使用，**结构清晰、可读性极高**

这就是 **GROUP BY + MAX + CTE**。

---

#### 六、CTE 优点总结（面试回答）

1. **提高可读性**：复杂 SQL 分层清晰
2. **可重复使用**：一个 WITH 定义，后面可多次调用
3. **可写递归查询**（树结构必备）
4. **不产生真实表**，执行完自动消失
5. MySQL 8.0+、PostgreSQL、Oracle、SQL Server 全部支持

---

#### 七、一句话记住

**WITH 子句 = CTE = 临时查询结果集 = 让 SQL 更干净的子查询升级版**

---

如果你愿意，我可以教你 **CTE 递归查询（树形结构 SQL）**，这是**面试超级高频压轴题**。

------

### 面试高频问题总结（配合上方内容）

| 问题                 | 涉及知识点   | 解答要点                                                    |
| -------------------- | ------------ | ----------------------------------------------------------- |
| 多表如何连接？       | JOIN 类型    | INNER / LEFT / RIGHT / FULL JOIN                            |
| 自连接场景？         | self join    | 组织结构、员工上级                                          |
| 子查询和 JOIN 区别？ | 性能与可读性 | JOIN 更高效，子查询更直观                                   |
| 什么是相关子查询？   | 子查询分类   | 子查询依赖外层字段                                          |
| SQL 执行顺序？       | SQL 执行逻辑 | FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY |



------

## 三、聚合与分组

- **聚合函数**
  - `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`
- **分组查询**
  - `GROUP BY`
  - `HAVING` 用于分组后条件过滤
- **多级分组（ROLLUP、CUBE）**（部分数据库支持）

------

### ✅ 1. 聚合函数（Aggregate Functions）

聚合函数用于对一组数据进行计算，常见的有：

| 函数                          | 说明                    |
| ----------------------------- | ----------------------- |
| `COUNT(*)`                    | 统计记录数（包括 NULL） |
| `COUNT(column)`               | 统计非 NULL 值数量      |
| `SUM(column)`                 | 计算总和                |
| `AVG(column)`                 | 计算平均值              |
| `MIN(column)` / `MAX(column)` | 最小 / 最大值           |

 count(*)包括了所有列，相当于行数，在统计结果的时候，不会忽略某些列值为NULL的行。 

count(1)用1代表代码行，忽略所有列的值，在统计结果的时候，统计所有行，效果同count(*)。 

count(列名)只包括列名对应一列，在统计结果的时候，会忽略列值为空的行。 

📌 示例：

```sql
SELECT 
    COUNT(*) AS total_count,
    COUNT(salary) AS salary_count,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary
FROM employees;
```

------

### ✅ 2. 分组查询（GROUP BY）

用于对记录分组，并配合聚合函数统计每组的结果。

```sql
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id;
```

------

#### 🚩 面试陷阱：GROUP BY 后必须 SELECT 分组字段或聚合函数！

```sql
-- 错误写法（不允许非聚合非分组字段）
SELECT department_id, name 
FROM employees 
GROUP BY department_id;
```

#### `GROUP BY` 的 进一步解释

你总结得很准确！下面是对 `GROUP BY` 的 **进一步解释和面试补充说明**：

------

##### ✅ `GROUP BY` 使用说明：

- **语法规则**：

  ```sql
  SELECT 分组字段, 聚合函数(...)
  FROM 表名
  GROUP BY 分组字段;
  ```

- **注意点**：

  - `SELECT` 中必须是：
    - 出现在 `GROUP BY` 子句中的字段，或
    - 对其他字段使用聚合函数（如 `COUNT()`、`SUM()`、`AVG()` 等）

------

##### ❌ 错误示例（经典面试陷阱）：

```sql
SELECT department_id, name 
FROM employees 
GROUP BY department_id;
```

- ❗ 错误原因：`name` 既不是 `GROUP BY` 字段，也不是聚合函数。

- ✅ 正确写法之一（使用聚合）：

  ```sql
  SELECT department_id, MAX(name) 
  FROM employees 
  GROUP BY department_id;
  ```

------

##### ✅ 配合 `HAVING` 做分组过滤：

```sql
SELECT department_id, COUNT(*) AS num
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

- `WHERE`：过滤原始记录；
- `HAVING`：过滤聚合后的结果（**只能与 `GROUP BY` 一起用**）。

------

##### 🧠 面试高频问法：

| 问题                                                         | 面试官想考察 |
| ------------------------------------------------------------ | ------------ |
| `GROUP BY` 后面必须写哪些字段？                              | SQL 语法理解 |
| `HAVING` 和 `WHERE` 区别？                                   | 聚合前后逻辑 |
| 能不能在 `GROUP BY` 后 `SELECT` 一个没聚合也没出现在 `GROUP BY` 的字段？ | 错误陷阱识别 |

------

需要我出几道 `GROUP BY` 的面试题和练习 SQL 吗？

当然可以！以下是对前面 **面试高频问法的详细回答**，并附带几道 **经典 GROUP BY 面试题** 和 **实际 SQL 练习题**：

------

#### ✅ 面试高频问法详解

##### 💬 问题 1：`GROUP BY` 后面必须写哪些字段？

**答：**

- `SELECT` 语句中，**除了聚合函数外的字段，必须全部出现在 `GROUP BY` 子句中**。
- 否则 SQL 语句在多数数据库（如 MySQL 设置为 SQL_MODE=STRICT）中会报错。

**例：**

```sql
-- 正确：name 出现在 GROUP BY
SELECT department_id, name, COUNT(*) FROM employees GROUP BY department_id, name;
```

------

##### 💬 问题 2：`HAVING` 和 `WHERE` 的区别？

| 项目                | `WHERE`              | `HAVING`                     |
| ------------------- | -------------------- | ---------------------------- |
| 作用阶段            | 聚合前过滤原始数据   | 聚合后过滤分组结果           |
| 能否用聚合函数      | ❌ 不行（如 COUNT()） | ✅ 可以（如 COUNT(*) > 5）    |
| 是否依赖 `GROUP BY` | ❌ 可单独使用         | ✅ 一般和 `GROUP BY` 一起使用 |

------

##### 💬 问题 3：能否在 `GROUP BY` 后 `SELECT` 一个没聚合也没出现在 `GROUP BY` 的字段？

**答：**

- ❌ **不能。**这样会导致语义不明确（每组内多个值，数据库无法确定返回哪一个）。

**错误示例：**

```sql
SELECT department_id, name FROM employees GROUP BY department_id;
-- name 无法聚合且未在 GROUP BY 中，报错。
```

------

#### 📌 面试题 + SQL 实践题（含答案）

------

##### ✅ 题目 1：查询每个部门的员工数

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;
```

------

##### ✅ 题目 2：查询员工人数超过 5 人的部门

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

------

##### ✅ 题目 3：找出每个部门中工资最高的员工姓名和工资

```sql
SELECT e.name, e.salary, e.department_id
FROM employees e
INNER JOIN (
    SELECT department_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY department_id
) t ON e.department_id = t.department_id AND e.salary = t.max_salary;
```

------

##### ✅ 题目 4：统计每个月订单数量

```sql
SELECT DATE_FORMAT(order_date, '%Y-%m') AS month, COUNT(*) AS order_count
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m');
```

------

##### ✅ 题目 5：统计每个客户的总消费金额（假设每笔订单有 `customer_id` 和 `amount` 字段）

```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id;
```

------

如果你还想要 **按不同难度层级分组的面试题、或实战真题汇总（如阿里、美团、字节）**，我也可以继续补充。需要吗？

------

### ✅ 3. 分组过滤（HAVING）

- `WHERE`：在分组前过滤记录；
- `HAVING`：在分组后过滤分组。

📌 示例：查询人数 > 3 的部门：

```sql
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 3;
```

------

### ✅ 4. 多级分组（ROLLUP、CUBE）👑（MySQL / SQL Server / Oracle）

用于生成小计、合计。常用于报表类查询。

#### 4.1 ROLLUP：逐级合计

```sql
SELECT department_id, job_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id, job_id WITH ROLLUP;
```

输出示例：

| department_id | job_id  | emp_count |
| ------------- | ------- | --------- |
| 10            | MANAGER | 3         |
| 10            | NULL    | 3         |
| NULL          | NULL    | 10        |

------

#### 4.2 CUBE：对所有组合做聚合（不常见，支持较少）

```sql
SELECT department_id, job_id, COUNT(*) 
FROM employees 
GROUP BY CUBE(department_id, job_id);
```

------

### 🎯 面试高频问题及典型 SQL 示例

| 面试问题                            | 涉及概念    | 答题要点 / 示例                                         |
| ----------------------------------- | ----------- | ------------------------------------------------------- |
| COUNT(*) 和 COUNT(col) 区别？       | 聚合函数    | `COUNT(*)` 计所有行，`COUNT(col)` 忽略 NULL             |
| GROUP BY 和 HAVING 的区别？         | 分组过滤    | `WHERE` 是原始记录过滤，`HAVING` 是分组过滤             |
| 分组统计平均工资大于 10000 的部门？ | 聚合 + 分组 | 见下示例                                                |
| 如何查询每个部门的最高工资？        | 聚合        | `SELECT dept_id, MAX(salary) FROM ... GROUP BY dept_id` |
| ROLLUP 用在哪？                     | 多级分组    | 报表统计、小计合计                                      |

------

📌 综合示例：查询各部门平均工资，筛选出平均工资高于 10000 的部门，并按平均工资降序排列

```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 10000
ORDER BY avg_salary DESC;
```

------

📌 示例：统计每个部门、岗位人数，并添加小计

```sql
SELECT department_id, job_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id, job_id WITH ROLLUP;
```

------

如果你需要第四章（窗口函数与排名）或帮你生成面试题题库练习，随时告诉我！

------

## 四、数据操作语句（DML）

- **数据插入**
  - `INSERT INTO`
  - 多行插入
- **数据更新**
  - `UPDATE` 结合 `WHERE` 条件
- **数据删除**
  - `DELETE` 结合 `WHERE` 条件
- **批量操作**



第四部分内容是 SQL 中非常常用的「数据操作语句（DML）」部分，几乎在所有实际开发和面试场景中都会用到。以下是该部分的详细介绍和常见面试题示例。

------

### ✅ 1. 数据插入（`INSERT INTO`）

#### 1.1 单行插入

```sql
-- 可以指定插入哪些字段，字段名顺序与字段值顺序一致即可，可以给部分或所有字段名加``，不能加''，加''时会执行报错。
-- INSERT INTO employees (`id`, `name`, department, salary)
INSERT INTO employees (id, name, department, salary)
VALUES (101, 'Alice', 'HR', 6000);


```

#### 1.2 多行插入（MySQL、PostgreSQL 支持）

```sql
INSERT INTO employees (id, name, department, salary)
VALUES 
(102, 'Bob', 'IT', 8000),
(103, 'Carol', 'Finance', 9000);
```

#### 1.3 插入部分字段（其他为默认值或 NULL）

```sql
INSERT INTO employees (name, department) 
VALUES ('David', 'Sales');
```

------

### ✅ 2. 数据更新（`UPDATE`）

> ⚠️ 更新操作必须加 `WHERE` 条件，否则会更新整张表！

```sql
-- 修改员工工资
UPDATE employees 
SET salary = salary + 500
WHERE department = 'HR';
```

#### 示例：根据条件修改多个字段

```sql
UPDATE employees
SET salary = 10000, department = 'R&D'
WHERE id = 101;
```

------

### ✅ 3. 数据删除（`DELETE`）

> ⚠️ 不加 `WHERE` 会删除整表！

```sql
DELETE FROM employees 
WHERE salary < 3000;
```

#### 删除整表所有数据：

```sql
DELETE FROM employees; -- 留结构
TRUNCATE TABLE employees; -- 更高效，适合清空测试数据
```

------

### ✅ 4. 批量操作

SQL 本身不支持「for 循环」，但可以借助：

- 多行 `INSERT`；
- 多条件 `WHERE` 语句；
- 在程序代码中循环执行；
- 使用事务包裹多条操作：

```sql
BEGIN;

INSERT INTO orders (id, user_id, total) VALUES (201, 1, 120.5);
UPDATE products SET stock = stock - 1 WHERE id = 3;
DELETE FROM cart WHERE user_id = 1;

COMMIT;
```

------

### 🎯 面试高频题型 & 示例

| 问题                          | 涉及知识点            | 示例/答题要点                                    |
| ----------------------------- | --------------------- | ------------------------------------------------ |
| 如何插入多条数据？            | `INSERT` 多行         | 多行 `VALUES` 或循环                             |
| 如何避免误操作更新全表？      | `UPDATE` 必加 `WHERE` | 强调限制条件                                     |
| `DELETE` 与 `TRUNCATE` 区别？ | 删除数据              | `TRUNCATE` 不能回滚、效率高、不触发触发器        |
| 如何批量更新工资？            | 批量更新              | 结合条件表达式或 `CASE WHEN`                     |
| 删除某一部门所有员工？        | 条件删除              | `DELETE FROM employees WHERE department = 'xxx'` |

------

### 📌 示例：使用 `CASE` 实现多条件批量更新

```sql
UPDATE employees
SET salary = CASE 
    WHEN department = 'HR' THEN salary + 500
    WHEN department = 'IT' THEN salary + 1000
    ELSE salary
END;
```

------

### 📌 示例：批量插入并更新库存（伪代码）

```sql
BEGIN;

INSERT INTO orders (user_id, product_id, quantity) VALUES (1, 3, 2);
UPDATE products SET stock = stock - 2 WHERE id = 3;

COMMIT;
```

------

## DDL/DML/DQL/DCL/TCL

MySQL 的 SQL 语句除了 DML（数据操作语言）和 DDL（数据定义语言），还有另外三类核心分类，总共五大类，每类负责不同的数据库操作场景。我会用通俗易懂的方式帮你梳理清楚，包括定义、常用命令和使用场景。

### 一、MySQL SQL 语句的五大分类

#### 1. DDL（Data Definition Language）：数据定义语言
**作用**：定义/修改数据库、表、索引等**数据库结构**，操作会立即生效且不可回滚（MySQL 8.0+ 支持 DDL 事务，但默认不开启）。
**常用命令**：
```sql
CREATE  -- 创建（库、表、索引）：CREATE DATABASE test; CREATE TABLE user(id INT);
ALTER   -- 修改（表结构）：ALTER TABLE user ADD COLUMN age INT;
DROP    -- 删除（库、表）：DROP TABLE user;
TRUNCATE-- 清空表（属于 DDL，会重置自增ID）：TRUNCATE TABLE user;
```

#### 2. DML（Data Manipulation Language）：数据操作语言
**作用**：操作表中的**数据行**（增删改），默认支持事务回滚（需在事务中执行）。
**常用命令**：
```sql
INSERT  -- 插入数据：INSERT INTO user(name) VALUES('张三');
UPDATE  -- 更新数据：UPDATE user SET age=20 WHERE id=1;
DELETE  -- 删除数据：DELETE FROM user WHERE id=1;
```
⚠️ 注意：`SELECT` 常被归为 DQL（而非 DML），不要混淆。

#### 3. DQL（Data Query Language）：数据查询语言
**作用**：查询表中的数据，是日常使用最频繁的语句。
**核心命令**：
```sql
SELECT  -- 基础查询：SELECT id,name FROM user WHERE age>18;
-- 进阶：联表、分组、排序等
SELECT o.id,u.name FROM order o JOIN user u ON o.user_id=u.id ORDER BY o.create_time DESC;
```

#### 4. DCL（Data Control Language）：数据控制语言
**作用**：管理数据库的**权限和事务**，针对用户/角色操作。
**常用命令**：
```sql
GRANT   -- 授权：GRANT SELECT,INSERT ON test.* TO 'user1'@'%';
REVOKE  -- 撤销权限：REVOKE INSERT ON test.* FROM 'user1'@'%';
COMMIT  -- 提交事务：COMMIT;
ROLLBACK-- 回滚事务：ROLLBACK;
SET     -- 设置权限/变量：SET PASSWORD FOR 'user1'@'%' = 'new_pwd';
```

#### 5. TCL（Transaction Control Language）：事务控制语言
**作用**：专门管理事务（可看作 DCL 的子集，部分资料会单独归类）。
**核心命令**：
```sql
START TRANSACTION; -- 开启事务
COMMIT;            -- 提交事务（确认修改）
ROLLBACK;          -- 回滚事务（撤销修改）
SAVEPOINT;         -- 设置保存点：SAVEPOINT sp1; ROLLBACK TO sp1;
```

### 二、快速区分技巧（新手友好）
| 分类 | 核心操作     | 关键词               | 操作对象     |
| ---- | ------------ | -------------------- | ------------ |
| DDL  | 改结构       | CREATE/ALTER/DROP    | 库、表、索引 |
| DML  | 改数据（行） | INSERT/UPDATE/DELETE | 表中的数据行 |
| DQL  | 查数据       | SELECT               | 表中的数据行 |
| DCL  | 改权限/事务  | GRANT/REVOKE/COMMIT  | 用户、事务   |
| TCL  | 纯事务控制   | COMMIT/ROLLBACK      | 事务         |

### 总结
1. MySQL SQL 核心分五类：**DDL（改结构）、DML（改数据）、DQL（查数据）、DCL（管权限）、TCL（管事务）**，其中 DQL/DDL/DML 是新手最常接触的三类。
2. 关键区分点：DDL 操作“结构”（不可逆），DML 操作“数据行”（可回滚），DQL 仅“查询”不修改数据。
3. TCL 常被归为 DCL 的子集，核心记住 `COMMIT/ROLLBACK` 是事务控制的核心即可。

------

## 五、索引相关

- 索引的概念和作用
- 索引的类型（B-Tree、哈希索引、全文索引等）
- 什么时候使用索引、索引失效原因
- 主键索引与唯一索引区别
- 复合索引及其使用原则

非常好！第五章「索引相关」内容在 SQL 优化与性能调试面试中极其重要。下面是该部分的详细介绍，包括概念讲解、常见面试问题和 SQL 示例。



------

### ✅ 1. 索引的概念和作用

**索引（Index）** 类似于书的目录，是数据库系统用于提高**查询速度**的数据结构。

**作用：**

- 加快查询速度（如 `SELECT`）
- 加快表连接速度（如 `JOIN`）
- 用于排序（`ORDER BY`）、分组（`GROUP BY`）
- 加速 `WHERE` 条件过滤、`LIKE` 精确前缀匹配
- 实现唯一性约束（如唯一索引）

------

### ✅ 2. 索引的类型

| 类型                     | 简介                             | 场景                                 |
| ------------------------ | -------------------------------- | ------------------------------------ |
| **B-Tree索引**           | 默认索引类型，适用于范围、排序等 | `=`, `<`, `>`, `BETWEEN`, `ORDER BY` |
| **哈希索引**             | 快速定位，但不支持范围查找       | 精确匹配场景，如 `=`                 |
| **全文索引**（Fulltext） | 用于文本搜索，支持分词           | 搜索引擎、文章匹配                   |
| **空间索引**             | 用于 GIS 数据                    | 地理空间数据处理                     |

> 📌 InnoDB 支持 B+Tree 索引，不支持哈希索引（除非手动构建）。

------

### ✅ 3. 索引使用与失效

#### ✅ 建议添加索引的情况：

- 经常作为 `WHERE` 条件的字段
- 经常用于 `ORDER BY`、`GROUP BY` 的字段
- 表连接时的连接字段（`JOIN ON`）
- `DISTINCT` 语句字段
- 高频被访问的列（但不是频繁更新）

#### ❌ 索引失效的常见原因：

| 原因                               | 示例                                    |
| ---------------------------------- | --------------------------------------- |
| 在索引列上使用函数                 | `WHERE DATE(created_at) = '2023-01-01'` |
| 类型不一致                         | `WHERE id = '100'`（id 是 int）         |
| 前置字段未使用                     | 索引 (`a`, `b`) 中仅使用 `b`            |
| 使用 `OR` 导致部分失效             | `WHERE a = 1 OR b = 2`                  |
| 模糊匹配不在前缀                   | `LIKE '%abc'` 无法用索引                |
| 数据量太小，优化器认为全表扫描更快 |                                         |

------

### ✅ 4. 主键索引 vs 唯一索引

| 对比项       | 主键索引           | 唯一索引      |
| ------------ | ------------------ | ------------- |
| 唯一性       | 必须唯一           | 必须唯一      |
| 空值         | 不允许 NULL        | 允许一个 NULL |
| 一个表中数量 | 只能有一个         | 可以有多个    |
| 默认索引类型 | 聚簇索引（InnoDB） | 非聚簇索引    |

------

### ✅ 5. 复合索引（联合索引）

> 一个索引包含多个字段，例如 `(a, b, c)`

#### ✅ 最左前缀原则

只会命中从**最左开始的连续字段**：

```sql
CREATE INDEX idx_ab ON user(name, age);

-- 可以用到索引：
WHERE name = 'Tom'
WHERE name = 'Tom' AND age = 20

-- 无法用到索引：
WHERE age = 20
```

#### ✅ 索引选择顺序建议

- 选择区分度高的字段在前面（如 `user_id`）
- 经常使用范围查询的字段放后面（如 `create_time`）

------

### 🎯 面试经典题汇总

| 面试题                   | 涉及考点         | 示例答案要点                     |
| ------------------------ | ---------------- | -------------------------------- |
| 什么是索引？有什么作用？ | 索引作用         | 加速查询、排序、过滤等           |
| 什么情况下索引会失效？   | 索引失效机制     | 函数、数据类型不匹配、模糊查询等 |
| 主键索引与唯一索引区别？ | 唯一性、空值限制 | 主键唯一且非空                   |
| 如何设计复合索引？       | 最左匹配原则     | `(a, b)` 只在使用 a 时才起作用   |
| 什么时候不适合加索引？   | 负面影响场景     | 小表、低频访问字段、频繁更新字段 |

------

### 📌 实操示例 SQL

```sql
-- 创建索引
CREATE INDEX idx_user_name ON users(name);

-- 创建联合索引
CREATE INDEX idx_user_name_age ON users(name, age);

-- 查看索引
SHOW INDEX FROM users;

-- 删除索引
DROP INDEX idx_user_name ON users;
```

------

需要我为你整理「事务与锁机制」、「SQL 优化」、「实战题目」等后续章节内容吗？或者你想练习一套索引相关的 SQL 题？

------

## 六、SQL 优化相关

- **执行计划分析**
  - `EXPLAIN` 语法使用
- **查询优化**
  - 避免全表扫描
  - 利用索引优化查询条件
  - 避免 `SELECT *`
  - 减少子查询，改用连接
- **分页优化**
  - 分页查询的性能陷阱和优化方案
- **锁机制**
  - 悲观锁与乐观锁实现
- **事务与并发**
  - 事务隔离级别
  - 死锁及避免方案

非常好！下面是 **第六章：SQL 优化相关** 的完整知识点总结，适用于笔试面试，也便于你作为复习笔记或讲解材料使用。

------

### ✅ 1. 执行计划分析

#### 📌 使用 `EXPLAIN` 查看 SQL 执行计划：

```sql
EXPLAIN SELECT * FROM users WHERE id = 1;
```

| 常见字段      | 含义说明                                               |
| ------------- | ------------------------------------------------------ |
| id            | 查询序列编号                                           |
| select_type   | 查询类型（SIMPLE、PRIMARY、SUBQUERY）                  |
| table         | 查询的表                                               |
| type          | 连接类型（ALL > index > range > ref > eq_ref > const） |
| possible_keys | 可能用到的索引                                         |
| key           | 实际用到的索引                                         |
| rows          | 预估扫描的行数                                         |
| Extra         | 额外信息，如 `Using index`、`Using filesort` 等        |

> **优化目标：** 尽量让 `type` 为 `ref` 或更优，避免 `ALL`（全表扫描）。

------

### ✅ 2. 查询优化技巧

#### 🚫 避免全表扫描（`type=ALL`）

- 建立合适索引
- 避免函数操作字段：`WHERE DATE(time_column)` ❌
- 避免隐式转换：`WHERE id = '123'` ❌（id 为 int）

#### ✅ 利用索引优化查询条件

- 使用 `=`, `<`, `>`, `BETWEEN`, `LIKE 'prefix%'`
- 联合索引要用到**最左前缀**

#### 🚫 避免 `SELECT *`

- 只查询需要的字段，减小 I/O、网络开销

#### 🚫 减少子查询，改用 JOIN

```sql
-- ❌ 子查询方式
SELECT name FROM user WHERE id IN (SELECT user_id FROM order);

-- ✅ JOIN 优化
SELECT u.name FROM user u JOIN order o ON u.id = o.user_id;
```

------

### ✅ 3. 分页优化

#### 分页查询性能陷阱：

```sql
-- ❌ 慢：LIMIT 越大越慢（如 10万偏移）
SELECT * FROM articles LIMIT 100000, 10;
```

#### ✅ 优化方式：

```sql
-- 利用索引优化分页（基于主键）
SELECT * FROM articles WHERE id > 100000 LIMIT 10;
```

> 建议返回翻页的主键/时间戳，下次分页基于条件翻页，而非 `OFFSET`

你总结得非常精准，下面我将为这段内容进行 **补充性讲解 + 常见面试点解析 + 进阶优化方案**，帮助你在面试中进一步加分。

------

#### ✅ 分页优化详解

##### 📌 1. **性能陷阱来源**

分页语句：

```sql
SELECT * FROM articles LIMIT 100000, 10;
```

问题在于：

- 数据库需要扫描 **前 100000 行数据**，即使只返回 10 条。
- 所以 **OFFSET 越大，性能越差**。
- 尤其是用户“翻到第10000页”时，查询几乎不可接受。

------

##### ✅ 2. **优化方案：基于索引的 Keyset Pagination（游标分页）**

###### ✅ 推荐写法：

```sql
SELECT * FROM articles
WHERE id > 上一页最后一条记录的ID
ORDER BY id ASC
LIMIT 10;
```

好处：

- 避免 OFFSET 扫描开销，直接从上一次的位置跳过。
- 查询速度与页数无关，**只与 LIMIT 大小有关**，性能稳定。

------

##### 🚩 面试高频问法 & 答案

###### 💬 Q1：LIMIT OFFSET 为什么性能差？

> 因为 OFFSET 会导致数据库遍历并丢弃前 N 条记录，虽然不返回，但扫描成本仍然存在，尤其在数据量大时显著降低性能。

------

###### 💬 Q2：如何优化高偏移量分页？

> 使用 **基于唯一索引的游标分页**（如主键ID或时间戳），只查询大于/小于上一页最后一条记录的条件，跳过扫描和排序。

------

##### ✅ 3. **其他进阶技巧**

##### ✅ 时间戳分页

```sql
SELECT * FROM orders
WHERE created_at < '2024-01-01 00:00:00'
ORDER BY created_at DESC
LIMIT 10;
```

适用于：

- 按时间倒序显示记录（如订单、日志等）
- 数据是 append-only 的场景（不修改历史）

------

##### ✅ 复合索引分页（用于多条件）

```sql
SELECT * FROM blog
WHERE category = 'Java'
  AND id > 上次最后一条ID
ORDER BY id
LIMIT 10;
```

需要联合建立索引：(category, id)

------

##### ✅ 最后一页优化技巧

分页时如果用户跳转“最后一页”，常规 LIMIT + OFFSET 性能极差：

```sql
SELECT * FROM table ORDER BY id DESC LIMIT 10;
```

反转排序方向，取前 10 条，再手动反转展示顺序。

------

#### 🎯 总结面试亮点句式

- **OFFSET 会跳过大量记录，占用 I/O 和 CPU，性能随页数线性变差。**
- **游标分页是一种常用的大数据分页优化手段，适合 ID/时间线性增长的场景。**
- **结合主键或唯一索引字段分页，可以做到高性能、稳定响应。**

------

如果你需要我帮你补充 **MyBatis-Plus分页工具优化实践、Redis分页、或Elasticsearch分页设计方案**，也可以继续说。

非常好的问题！



#### 在 SQL 中，`LIMIT` 和 `OFFSET` 

是两个不同的语句组件，它们**不是绑定使用的**。你看到的“只有 `LIMIT` 没有 `OFFSET`”的情况，**并不是默认省略，而是有意省略，用来实现性能优化**。

------

##### ✅ 基本语法对比

| 语法形式                           | 含义                               |
| ---------------------------------- | ---------------------------------- |
| `LIMIT N`                          | 取前 N 条记录（默认从第 0 行开始） |
| `LIMIT N OFFSET M` 或 `LIMIT M, N` | 跳过 M 条记录后，取 N 条           |

------

##### ✅ 为什么很多优化写法只用 `LIMIT` 不用 `OFFSET`？

这是因为：

- `OFFSET` 会强制数据库扫描并跳过前 M 行，即使不返回数据，**性能代价也很大**。
- 所以高级分页优化方案（如基于 ID 的 **游标分页**），直接通过 `WHERE id > ?` 来跳过前面的数据，不使用 `OFFSET`：

```sql
SELECT * FROM articles WHERE id > 100000 LIMIT 10;
```

👉 相当于：

- 用 `WHERE` 精准定位起始位置（代替了 OFFSET）
- 用 `LIMIT` 控制返回数量

------

##### ✅ 不是默认省略，是逻辑替代

总结：`LIMIT` 没有配合 `OFFSET` 使用，**不是因为系统默认省略了 OFFSET**，而是因为：

- 我们 **显式用 `WHERE` 替代了 OFFSET 的作用**
- 这是更优的分页方式

------

##### 🚩注意点：

- `LIMIT N` 默认是 `OFFSET 0`，从第 1 条开始。
- `LIMIT N OFFSET M` 等价于 `LIMIT M, N`（MySQL 写法）
- 游标分页不需要 OFFSET，因为我们从逻辑上“跳过了”前面的记录（如 `id > ?`）

------

如果你希望我举出具体分页例子（如文章表、订单表的游标分页 vs 普通分页对比），可以继续提问。

### ✅ 4. 锁机制

| 锁类型     | 说明                            | 示例                    |
| ---------- | ------------------------------- | ----------------------- |
| **行锁**   | 只锁一行，事务间并发性高        | `SELECT ... FOR UPDATE` |
| **表锁**   | 锁整张表，效率低                | MyISAM                  |
| **悲观锁** | 操作前加锁（例如 `FOR UPDATE`） | 多用于高并发写场景      |
| **乐观锁** | 不加锁，操作前判断版本          | 需要 `version` 字段     |

#### 乐观锁实现（伪代码）：

```sql
UPDATE product
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 当前版本号;
```

------

### ✅ 5. 事务与并发控制

#### 四种事务隔离级别：

| 级别                         | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED             | ✅    | ✅          | ✅    |
| READ COMMITTED               | ❌    | ✅          | ✅    |
| REPEATABLE READ（MySQL默认） | ❌    | ❌          | ✅    |
| SERIALIZABLE                 | ❌    | ❌          | ❌    |

#### 幻读解决方式：

- MySQL InnoDB 默认使用 **间隙锁（Gap Lock）** 防止幻读

#### 死锁场景：

```sql
-- 事务 A：先锁表1，再锁表2
-- 事务 B：先锁表2，再锁表1
```

> 解决办法：保持一致的**加锁顺序**；设置事务超时；使用小事务

------

### 🎯 面试经典题

| 问题                        | 涉及知识点         | 回答要点                            |
| --------------------------- | ------------------ | ----------------------------------- |
| 如何查看 SQL 是否使用索引？ | `EXPLAIN` 执行计划 | 查看 key 字段、rows 数量、type 类型 |
| SQL 中如何优化分页？        | 分页优化           | 主键偏移、游标分页                  |
| 什么是悲观锁与乐观锁？      | 并发控制           | 使用场景、区别、实现方式            |
| 什么情况下索引失效？        | 查询优化           | 函数、类型不匹配、like 后缀等       |
| 事务有哪些隔离级别？        | 并发事务           | 读一致性、脏读、幻读区别            |
| 什么是幻读？如何避免？      | 间隙锁、事务       | 间隙锁机制、MVCC                    |

------

### 📌 实操示例

```sql
-- 查询执行计划
EXPLAIN SELECT * FROM user WHERE id = 1;

-- 使用乐观锁更新（带 version）
UPDATE product SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 10;

-- 悲观锁
SELECT * FROM user WHERE id = 1 FOR UPDATE;

-- 分页优化
SELECT * FROM orders WHERE id > 100000 LIMIT 10;
```

------

如果你需要我为「SQL 优化实战题」「数据库锁面试题」「事务一致性场景题」再展开一章内容，我可以继续为你整理。需要吗？

------

## 七、数据库设计与范式

- **表设计**
  - 主键、外键设计
  - 约束（`NOT NULL`, `UNIQUE`，默认值）
- **范式**
  - 第一范式（1NF）
  - 第二范式（2NF）
  - 第三范式（3NF）
  - 反范式设计（实际项目中的性能权衡）
- **ER 图与关系模型**

这是你整理的第七部分内容：《数据库设计与范式》。下面我会详细展开每一小节内容，结合解释、规范与实战应用，适合用于：

- 技术面试准备；
- 系统架构设计参考；
- 数据库建模与优化指导。

------

### ✅ 1. 表设计原则

#### 📌 主键设计

- 每个表必须有主键，唯一标识每条记录。
- 可选：
  - **自然主键**（如身份证号、学号）
  - **代理主键**（如自增 id，更常用）
- 主键应：
  - 尽量简单；
  - 不变性强；
  - 查询频繁字段优先。

#### 📌 外键设计（用于表示表之间的关系）

- 约束子表记录必须引用父表存在的主键。
- 示例：

```sql
CREATE TABLE orders (
  id BIGINT PRIMARY KEY,
  user_id BIGINT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

> 注意：实际系统中很多高并发项目不使用外键约束，而是**应用层维护约束逻辑**，提高灵活性。

#### 📌 常用约束：

| 约束          | 含义                      |
| ------------- | ------------------------- |
| `NOT NULL`    | 不允许空值                |
| `UNIQUE`      | 值唯一                    |
| `DEFAULT`     | 默认值                    |
| `CHECK`       | 条件限制（MySQL 8+ 支持） |
| `PRIMARY KEY` | 主键                      |
| `FOREIGN KEY` | 外键                      |

------

### ✅ 2. 三大范式（NF）

#### 📌 第一范式（1NF）：字段不可再分

- 每列都是**原子值**
- 不允许有数组、列表、结构体等复合类型
- ❌ 错误设计：

```sql
hobbies = "游泳,骑车,游戏"
```

- ✅ 正确做法：分表或规范化字段

#### 📌 第二范式（2NF）：完全依赖主键

- 消除**部分依赖**
- 表中字段应依赖主键的全部，而不是主键的一部分
- 多见于复合主键场景

> 如果主键是 `(student_id, course_id)`，其他字段不能只依赖 `student_id`

#### 📌 第三范式（3NF）：消除传递依赖

- 非主键字段**只能依赖主键**
- ❌ 错误示例：

```sql
学生表：id, 班级id, 班级名
→ 班级名依赖班级id，而班级id依赖学生id（传递依赖）
```

- ✅ 正确做法：拆表

------

### ✅ 3. 反范式设计（打破范式以提升性能）

> 在满足业务功能的前提下，适当牺牲范式原则以换取性能。

#### 反范式应用场景：

| 场景                   | 做法                | 理由                    |
| ---------------------- | ------------------- | ----------------------- |
| 高并发读写             | 增加冗余字段        | 减少 join，提高响应速度 |
| 日志、报表类           | 不拆表              | 历史记录存档            |
| 查询频繁但字段来自多表 | 建物化视图/冗余字段 | 减少连接查询            |

------

### ✅ 4. ER 图（实体-关系图）

ER 图用于系统建模阶段，表示实体之间的关系。

| 元素                 | 说明                            |
| -------------------- | ------------------------------- |
| 实体（Entity）       | 表（如：User、Order）           |
| 属性（Attribute）    | 字段（如：username, age）       |
| 关系（Relationship） | 外键连接、1对1，1对多，多对多等 |

**工具推荐**：

- PowerDesigner
- draw.io
- Navicat、DataGrip 内置 ER 图
- dbdiagram.io（在线建模）

------

### ✅ 面试经典题举例

| 问题                          | 涉及点   | 简要思路                               |
| ----------------------------- | -------- | -------------------------------------- |
| 什么是三大范式？              | 范式     | 原子性、完全依赖、无传递依赖           |
| 你在项目中如何建表？          | 表设计   | 主键选择、外键是否使用、字段约束等     |
| 为什么在项目中要打破范式？    | 反范式   | 为了性能，例如减少 JOIN                |
| 主键使用 UUID 好还是自增 id？ | 主键策略 | UUID 去中心化、自增更快                |
| 外键是否建议使用？            | 外键设计 | 小型系统用外键，大型系统多在应用层控制 |

------

### ✅ 示例：用户与订单系统 ER 建模

- 实体：`User`, `Order`, `Product`
- 关系：
  - 一个用户可以有多个订单（1:N）
  - 一个订单中包含多个商品（M:N，需中间表）

------

如果你需要我基于这个 ER 模型帮你设计出完整的 SQL 表结构语句，我可以继续为你生成。是否需要？

------

## 八、复杂查询和高级特性

- **窗口函数**
  - `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`, `LEAD()`
- **递归查询**
  - 递归CTE (`WITH RECURSIVE`)
- **视图**
  - 视图的创建和使用
  - 物化视图（部分数据库支持）
- **存储过程、触发器**
  - 基本用法和优缺点
- **事务控制语句**
  - `BEGIN`, `COMMIT`, `ROLLBACK`

你整理的第八部分《复杂查询和高级特性》涉及数据库中最具深度和挑战性的内容，在**高级开发岗位**或**数据库岗位**面试中极易涉及。下面是该部分的详细讲解、面试常问点和 SQL 示例：

------

### ✅ 1. 窗口函数（Window Function）

> 在每一行上进行聚合，不影响结果行数，常用于排名、累计、前后值比较等。

#### 📌 语法结构：

```sql
函数名(...) OVER (
  PARTITION BY 分组字段
  ORDER BY 排序字段
  [ROWS BETWEEN ...]
)
```

#### 📌 常用函数：

| 函数           | 说明                         |
| -------------- | ---------------------------- |
| `ROW_NUMBER()` | 全表唯一编号，按排序给行编号 |
| `RANK()`       | 排名，遇到并列会跳过后续排名 |
| `DENSE_RANK()` | 紧凑排名，不跳过             |
| `LAG(col, n)`  | 获取前第 n 行的值            |
| `LEAD(col, n)` | 获取后第 n 行的值            |
| `SUM() OVER()` | 滚动累加                     |

#### 📌 示例：用户按分数排名

```sql
SELECT id, username, score,
       RANK() OVER (PARTITION BY class_id ORDER BY score DESC) AS rank
FROM student;
```

------

### ✅ 2. 递归查询（CTE）

用于处理层级数据，如评论树、组织架构、菜单嵌套等。

#### 📌 基础语法（以 PostgreSQL、MySQL 8+、SQL Server 为例）：

```sql
WITH RECURSIVE category_path AS (
    SELECT id, name, parent_id
    FROM category
    WHERE id = 1

    UNION ALL

    SELECT c.id, c.name, c.parent_id
    FROM category c
    JOIN category_path cp ON c.parent_id = cp.id
)
SELECT * FROM category_path;
```

> 自引用 `JOIN` 自己，终止条件避免死循环。

------

### ✅ 3. 视图（View）

> 视图是一个虚拟表，本质是保存的 SQL 查询。

#### 📌 创建视图：

```sql
CREATE VIEW active_users AS
SELECT id, username FROM user WHERE status = 'active';
```

- 可用于封装复杂逻辑，提升代码复用性。

#### 📌 物化视图（Materialized View）

- 会**存储查询结果**，不是所有数据库支持（如 PostgreSQL 支持，MySQL 不支持）。
- 适合读多写少的场景。

------

### ✅ 4. 存储过程 & 触发器（Procedure & Trigger）

#### 📌 存储过程（Stored Procedure）

- 用于封装业务逻辑（例如批量操作、事务）
- 带参数、可以使用控制语句（IF、WHILE）

```sql
DELIMITER //

CREATE PROCEDURE add_user(IN uname VARCHAR(50))
BEGIN
    INSERT INTO user(username) VALUES (uname);
END //

DELIMITER ;
```

调用：

```sql
CALL add_user('jack');
```

#### 📌 触发器（Trigger）

> 触发器是在 `INSERT/UPDATE/DELETE` 操作前后自动执行的 SQL 语句块。

```sql
CREATE TRIGGER log_user_update
AFTER UPDATE ON user
FOR EACH ROW
INSERT INTO user_log(user_id, old_name, new_name)
VALUES (OLD.id, OLD.username, NEW.username);
```

> 注意：**触发器隐藏性高**，建议控制使用。

------

### ✅ 5. 事务控制（Transaction Control）

#### 📌 基本语法：

```sql
BEGIN;
UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

- `BEGIN`/`START TRANSACTION`: 开启事务
- `COMMIT`: 提交事务
- `ROLLBACK`: 回滚事务
- `SAVEPOINT`/`ROLLBACK TO`: 部分回滚

#### 📌 面试延伸：

- **事务四大特性（ACID）**
- **事务隔离级别（READ COMMITTED 等）**
- 并发问题（幻读、脏读、不可重复读）

------

### ✅ 面试常问题 & 答案简要

| 面试题                     | 考察点   | 简要说明                                |
| -------------------------- | -------- | --------------------------------------- |
| 窗口函数和聚合函数的区别？ | 窗口函数 | 窗口函数不收缩行数，聚合函数会          |
| 递归查询怎么实现？         | CTE      | `WITH RECURSIVE`，层级数据常用          |
| 什么是视图，有什么优劣？   | View     | 优点封装查询，缺点可能降低性能          |
| 触发器什么时候用？         | Trigger  | 日志记录/约束控制，不宜过多             |
| 如何实现一个事务转账？     | 事务控制 | `BEGIN + UPDATE + COMMIT`，防止中途失败 |

------

如果你希望我提供一套综合示例，比如：**递归查询菜单结构 + 窗口函数实现分页排名 + 存储过程处理订单逻辑**，可以继续提出，我会打包设计出完整的实战案例。需要吗？



------

## 九、实际场景题型示例

- 查找重复数据
- 求某个字段的最大/最小值对应的整行数据
- 统计某一时间段的订单量、用户活跃度等
- 实现分页查询
- 实现排行榜功能
- 业务复杂的多条件查询
- 实现员工层级关系查询（递归）

以下是第九部分《实际场景题型示例》的经典 SQL 题型详细讲解、面试思路与代码范例，适用于中高级岗位面试、实战笔试和数据库设计考核。

------

### ✅ 1. 查找重复数据（找出重复的用户名）

```sql
SELECT username, COUNT(*) AS cnt
FROM user
GROUP BY username
HAVING COUNT(*) > 1;
```

📌 **面试考点**：

- 聚合 + 分组筛选
- `HAVING` vs `WHERE` 区别

------

### ✅ 2. 查询某字段最大值对应的整行（订单金额最大的一笔）

```sql
SELECT *
FROM orders
WHERE amount = (
  SELECT MAX(amount) FROM orders
);
```

📌 **变式**：取每个用户的最大订单（按用户分组）

```sql
SELECT *
FROM orders o
WHERE amount = (
  SELECT MAX(amount)
  FROM orders
  WHERE user_id = o.user_id
);
```

------

### ✅ 3. 统计某时间段的订单量（按天分组）

```sql
SELECT DATE(create_time) AS day, COUNT(*) AS order_count
FROM orders
WHERE create_time BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY DATE(create_time)
ORDER BY day;
```

📌 可扩展：

- 用户活跃数：`COUNT(DISTINCT user_id)`
- 成交金额：`SUM(amount)`

------

### ✅ 4. 分页查询（MySQL）

```sql
-- 查询第 2 页，每页 10 条
SELECT * FROM product
ORDER BY create_time DESC
LIMIT 10 OFFSET 10;


```

📌 **分页优化**：

- 避免大 OFFSET
- 用主键游标式分页：`WHERE id < ? ORDER BY id DESC LIMIT 10`

#### 使用游标分页

如果你要将传统的分页写法：

```sql
SELECT * FROM product
ORDER BY create_time DESC
LIMIT 10 OFFSET 10;
```

改成 **主键游标式分页（基于索引优化）**，应该这样做：

------

##### ✅ 主键游标分页写法：

```sql
SELECT * FROM product
WHERE id < ?      -- 上一页最后一条的 id（或下一页的起始 id）
ORDER BY id DESC
LIMIT 10;
```

------

##### 📌 示例说明：

假设你拿到了上一页最后一条记录的 `id = 200`，那么下一页查询如下：

```sql
SELECT * FROM product
WHERE id < 200
ORDER BY id DESC
LIMIT 10;
```

------

##### ✅ 优点：

- 避免大偏移量 `OFFSET N` 导致的慢查询。
- 通过索引快速定位 `id`，提高分页性能。
- 更适合做“无限下拉”、“前端继续加载下一页”式的查询。

------

##### 🔄 补充：时间游标分页也可以：

```sql
SELECT * FROM product
WHERE create_time < '2024-06-01 12:00:00'
ORDER BY create_time DESC
LIMIT 10;
```

------

##### ⚠️ 注意事项：

- 保证 `id` 或 `create_time` 是 **递增** 且有索引字段；
- 前端需记录上一次的 `id`（或时间戳）作为下次分页起点；
- 分页方向变更（如“上一页”）实现稍复杂，需额外维护逻辑。

如果你用 Spring Boot + MyBatis 或 JPA 查询，我也可以提供分页接口的完整写法。需要可以告诉我。

------

### ✅ 5. 实现排行榜（总成交金额前 10 用户）

```sql
SELECT user_id, SUM(amount) AS total_amount
FROM orders
GROUP BY user_id
ORDER BY total_amount DESC
LIMIT 10;
```

📌 **拓展**：每个商品下的用户购买排行

```sql
SELECT product_id, user_id, RANK() OVER (PARTITION BY product_id ORDER BY SUM(amount) DESC) AS rank
FROM orders
GROUP BY product_id, user_id;
```

------

### ✅ 6. 业务复杂的多条件查询

**查询：近 30 天内，手机型号为 iPhone 且订单金额大于 100 的已支付订单**

```sql
SELECT *
FROM orders
WHERE create_time >= NOW() - INTERVAL 30 DAY
  AND phone_model LIKE '%iPhone%'
  AND amount > 100
  AND status = 'PAID';
```

📌 **考察点**：

- 多条件组合
- 时间、模糊、数值、枚举等类型的处理

#### **多条件复合查询**

这是一个非常典型的 **多条件复合查询**，它综合考察了多个 SQL 能力点：

------

##### ✅ 面试考察点解析：

| 条件类型       | 表现语法                                 | 考察知识点                       |
| -------------- | ---------------------------------------- | -------------------------------- |
| **时间范围**   | `create_time >= NOW() - INTERVAL 30 DAY` | 日期函数、动态时间范围写法       |
| **模糊匹配**   | `phone_model LIKE '%iPhone%'`            | `LIKE` 模糊匹配                  |
| **数值范围**   | `amount > 100`                           | 数值条件判断                     |
| **枚举值判断** | `status = 'PAID'`                        | 状态字段判断，通常是枚举或字典值 |

------

##### ✅ SQL 优化建议：

1. ✅ **创建索引**（常用字段建议建复合索引）：

   ```sql
   CREATE INDEX idx_orders_query
   ON orders (create_time, phone_model, amount, status);
   ```

2. ✅ **模糊查询改进**（避免 `%前缀`）：

   - 若 `phone_model` 存储格式规范（如全称、版本），建议避免前缀 `%`。
   - 可配合全文索引（MySQL 支持 `FULLTEXT`）。

3. ✅ **分页加载大数据量**（如加 LIMIT）：

   ```sql
   SELECT *
   FROM orders
   WHERE create_time >= NOW() - INTERVAL 30 DAY
     AND phone_model LIKE '%iPhone%'
     AND amount > 100
     AND status = 'PAID'
   ORDER BY create_time DESC
   LIMIT 20;
   ```

------

#### 🧠 面试延伸提问（附答案提示）：

| 面试追问                                       | 回答提示                                                     |
| ---------------------------------------------- | ------------------------------------------------------------ |
| 如果 orders 表数据量上千万，这条查询怎么优化？ | 建组合索引、优化 phone_model 查询（避免 `%前缀`）、加 LIMIT  |
| LIKE 查询很慢怎么办？                          | 使用全文索引、改成等值匹配、标准化存储字段（如型号统一格式） |
| `status` 字段是否适合建索引？                  | 是。因为是常用的过滤字段，且区分度有限，适合建索引           |
| 怎么判断这条 SQL 是否用了索引？                | 使用 `EXPLAIN` 查看执行计划                                  |

------

#### ✅ 衍生练习题（可用于巩固）：

1. **查询近 7 天内，微信支付订单金额在 500~1000 之间的用户手机号**

   ```sql
   SELECT user_phone
   FROM orders
   WHERE create_time >= NOW() - INTERVAL 7 DAY
     AND pay_method = 'WeChat'
     AND amount BETWEEN 500 AND 1000;
   ```

2. **查找所有包含 "Pro" 的安卓设备订单，状态为待发货**

   ```sql
   SELECT *
   FROM orders
   WHERE phone_model LIKE '%Pro%'
     AND system_type = 'Android'
     AND status = 'TO_SHIP';
   ```

如需我再帮你出几道类似题或转化为代码接口实现，也可以告诉我。

------

### ✅ 7. 员工层级关系查询（递归查询）

```sql
-- 表结构：employee(id, name, manager_id)

WITH RECURSIVE emp_tree AS (
  SELECT id, name, manager_id, 1 AS level
  FROM employee
  WHERE id = 1  -- CEO

  UNION ALL

  SELECT e.id, e.name, e.manager_id, et.level + 1
  FROM employee e
  JOIN emp_tree et ON e.manager_id = et.id
)
SELECT * FROM emp_tree;
```

📌 **用途**：

- 企业组织结构
- 评论树
- 菜单嵌套

------

### ✅ 总结：面试场景建议

| 类型           | 考点              | 建议准备              |
| -------------- | ----------------- | --------------------- |
| 查重、筛选     | GROUP BY + HAVING | 理解聚合函数使用      |
| 最大行记录     | 子查询            | 注意结果多行处理      |
| 时间段统计     | DATE() + GROUP BY | 常用于报表            |
| 排名/分页      | LIMIT / 窗口函数  | 建议掌握 ROW_NUMBER() |
| 层级查询       | WITH RECURSIVE    | 多为加分项            |
| 多条件复杂过滤 | WHERE 组合        | 真实业务经常考察      |

------

如果你希望我**打包成一份 PDF 面试复习资料**或按照「企业真题 → SQL 分析 → 结果解释」的格式整理，可继续告诉我，我可以为你生成一份完整版本。是否需要？



当然，这里给你准备几道**SQL实际场景的经典面试题**，覆盖多种常见考点，非常适合面试准备。每题后面我会简要说明考点和难度。

------

## 九、经典SQL面试题（实际场景）

------

### 1. 查找表中重复数据

**题目**
 找出 `users` 表中重复的邮箱（email）及重复次数。

```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**考点**：聚合函数、分组查询、HAVING过滤
 **难度**：简单

------

### 2. 查询某个字段最大值对应的整行数据

**题目**
 查询 `orders` 表中，订单金额 `amount` 最大的订单详细信息。

```sql
SELECT *
FROM orders
WHERE amount = (SELECT MAX(amount) FROM orders);
```

**考点**：子查询、最大值查询
 **难度**：中等

------

### 3. 统计某时间段内每天的订单量

**题目**
 统计 `orders` 表中，2023年5月1日至5月7日每天的订单数量。

```sql
SELECT DATE(order_date) AS order_day, COUNT(*) AS order_count
FROM orders
WHERE order_date BETWEEN '2023-05-01' AND '2023-05-07'
GROUP BY order_day
ORDER BY order_day;
```

**考点**：日期处理、分组聚合
 **难度**：中等

------

### 4. 实现分页查询

**题目**
 查询 `products` 表，按价格降序分页，每页显示 10 条，查询第 3 页数据。

```sql
SELECT *
FROM products
ORDER BY price DESC
LIMIT 10 OFFSET 20;
```

（MySQL、PostgreSQL适用）

**考点**：分页查询、排序
 **难度**：简单

------

### 5. 查询用户排行榜（按积分排序）

**题目**
 查询 `users` 表中积分（points）排名前10的用户及排名。

```sql
SELECT username, points,
       RANK() OVER (ORDER BY points DESC) AS rank
FROM users
LIMIT 10;
```

**考点**：窗口函数、排名
 **难度**：中等偏上（要求熟悉窗口函数）

------

### 6. 多条件复杂查询

**题目**
 查询满足以下条件的用户：注册时间在2022年之后，状态为“active”，且邮箱或手机号不为空。

```sql
SELECT *
FROM users
WHERE register_date > '2022-01-01'
  AND status = 'active'
  AND (email IS NOT NULL OR phone IS NOT NULL);
```

**考点**：多条件过滤、逻辑运算符
 **难度**：简单

------

### 7. 员工层级关系递归查询

**题目**
 查询员工及其所有下属（多层级），`employees` 表有字段 `id` 和 `manager_id`。

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, name, manager_id
  FROM employees
  WHERE id = 1  -- 查询员工ID=1的所有下属
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employees e
  INNER JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

**考点**：递归CTE、层级关系
 **难度**：高级

------

### 8. 查找两张表中存在但不匹配的数据（反连接）

**题目**
 找出 `orders` 表中没有对应 `users` 表中用户的订单。

```sql
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

**考点**：左连接、空值判断、反连接查询
 **难度**：中等

------

### 9. 统计每个用户的订单总金额和订单数（连接与聚合）

**题目**
 查询每个用户的订单总金额和订单数量。

```sql
SELECT u.id, u.username, COUNT(o.id) AS order_count, SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;
```

**考点**：连接查询、聚合函数、分组
 **难度**：中等

------

### 10. 删除重复记录（保留一条）

**题目**
 假设 `users` 表中有重复的邮箱，删除重复数据，只保留id最小的那条。

```sql
DELETE FROM users
WHERE id NOT IN (
  SELECT MIN(id)
  FROM users
  GROUP BY email
);
```

**考点**：删除操作、子查询、重复数据处理
 **难度**：中等

------

### 总结

- 这些题目覆盖了常见SQL知识点：聚合、子查询、多表连接、窗口函数、递归、分页等。
- 难度从简单到高级都有，适合面试全方位准备。
- 实践时建议自己写几遍，理解思路和原理。



当然！再给你补充一批经典且代表性的SQL面试题，涵盖更丰富的实战场景和难点：

------

### 11. 查找最近N天内没有登录的用户

**题目**
 找出最近30天内没有登录过的用户。

```sql
SELECT *
FROM users
WHERE last_login < NOW() - INTERVAL 30 DAY
   OR last_login IS NULL;
```

**考点**：日期函数、条件过滤
 **难度**：简单

------

### 12. 查询每月新增用户数

**题目**
 统计每个月新增注册用户数（按年月分组）。

```sql
SELECT DATE_FORMAT(register_date, '%Y-%m') AS month, COUNT(*) AS new_users
FROM users
GROUP BY month
ORDER BY month;
```

（MySQL语法，Postgres可以用 `to_char`）

**考点**：日期函数、分组统计
 **难度**：中等

------

### 13. 分组排名内取Top N

**题目**
 查询每个部门（department_id）销售额排名前3的员工。

```sql
SELECT *
FROM (
  SELECT employee_id, department_id, sales,
         ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY sales DESC) AS rn
  FROM sales_data
) t
WHERE rn <= 3;
```

**考点**：窗口函数、分区排名
 **难度**：中高级

------

### 14. 实现动态列透视（Pivot）

**题目**
 将订单表按月份显示每月订单量（动态列），MySQL无内置PIVOT，需要用条件聚合。

```sql
SELECT user_id,
       SUM(CASE WHEN MONTH(order_date) = 1 THEN 1 ELSE 0 END) AS Jan,
       SUM(CASE WHEN MONTH(order_date) = 2 THEN 1 ELSE 0 END) AS Feb,
       -- ...依次写12个月
       SUM(CASE WHEN MONTH(order_date) = 12 THEN 1 ELSE 0 END) AS Dec
FROM orders
GROUP BY user_id;
```

**考点**：条件聚合、动态透视表
 **难度**：高级

------

### 15. 查找两张表共有的记录（交集）

**题目**
 找出两个表 `tableA` 和 `tableB` 中都存在的用户ID。

```sql
SELECT user_id
FROM tableA
INTERSECT
SELECT user_id
FROM tableB;
```

（PostgreSQL支持，MySQL可用内连接实现）

```sql
SELECT a.user_id
FROM tableA a
INNER JOIN tableB b ON a.user_id = b.user_id;
```

**考点**：集合操作、连接
 **难度**：中等

------

### 16. 找出连续登录天数最多的用户

**题目**
 求用户连续登录天数最长的记录（较难，涉及日期差和窗口函数）。

思路：利用日期减去行号，分组连续天数段，统计最大长度。

```sql
WITH login_days AS (
  SELECT user_id, login_date,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS rn
  FROM user_logins
),
groups AS (
  SELECT user_id, login_date, rn,
         DATE_SUB(login_date, INTERVAL rn DAY) AS grp
  FROM login_days
)
SELECT user_id, COUNT(*) AS consecutive_days
FROM groups
GROUP BY user_id, grp
ORDER BY consecutive_days DESC
LIMIT 1;
```

**考点**：窗口函数、日期处理、分组
 **难度**：高级

------

### 17. 利用CTE实现员工薪资等级划分

**题目**
 假设员工表有字段`salary`，对员工划分薪资等级（低、中、高），分组统计人数。

```sql
WITH salary_grade AS (
  SELECT *,
    CASE
      WHEN salary < 3000 THEN '低'
      WHEN salary BETWEEN 3000 AND 7000 THEN '中'
      ELSE '高'
    END AS grade
  FROM employees
)
SELECT grade, COUNT(*) AS count
FROM salary_grade
GROUP BY grade;
```

**考点**：CTE、条件表达式
 **难度**：中等

------

### 18. 删除表中重复数据（用ROW_NUMBER实现）

**题目**
 删除 `users` 表中email重复的行，只保留每组的第一条。

```sql
WITH ranked_users AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
  FROM users
)
DELETE FROM users
WHERE id IN (
  SELECT id FROM ranked_users WHERE rn > 1
);
```

**考点**：窗口函数、CTE、删除操作
 **难度**：高级

------

### 19. 批量更新多条记录不同字段

**题目**
 批量修改多条商品价格，每条价格不一样。

```sql
UPDATE products
SET price = CASE id
    WHEN 1 THEN 99.9
    WHEN 2 THEN 199.9
    WHEN 3 THEN 299.9
    ELSE price
END
WHERE id IN (1, 2, 3);
```

**考点**：批量更新、CASE表达式
 **难度**：中等

------

### 20. 查询订单金额超过平均值的订单

**题目**
 找出订单金额超过所有订单平均值的订单。

```sql
SELECT *
FROM orders
WHERE amount > (SELECT AVG(amount) FROM orders);
```

**考点**：聚合函数、子查询比较
 **难度**：简单

------

------

如果你还想要某个专题的更多题目，或者想要详细解答和代码示例，随时告诉我！

------

需要我帮你写题解讲解或者模拟答案代码吗？

------

## 十、数据库相关基础

- SQL 和 NoSQL 区别
- 主从复制
- 分库分表设计思路
- 备份和恢复策略

以下是**第十部分：数据库相关基础**的详细讲解，适合数据库理论理解与中高级面试准备使用：

------

### ✅ 1. SQL 与 NoSQL 区别

| 对比项     | SQL（关系型）                | NoSQL（非关系型）                |
| ---------- | ---------------------------- | -------------------------------- |
| 数据结构   | 表结构（行列）               | 文档、键值、图、列族等多样化     |
| 事务支持   | 支持 ACID                    | 一般不强制支持事务（BASE）       |
| 查询语言   | 使用结构化查询语言 SQL       | 通常使用 API、JSON 查询          |
| 典型数据库 | MySQL、PostgreSQL、Oracle    | MongoDB、Redis、Cassandra、Neo4j |
| 适用场景   | 强一致性、关系复杂的数据系统 | 海量、快速变化、低一致性容忍系统 |

📌 **总结**：

- SQL 强调结构、规范、安全性（如银行、电商）。
- NoSQL 强调灵活、性能、横向扩展（如社交媒体、日志系统）。

------

### ✅ 2. 主从复制（Master-Slave Replication）

> 数据同步机制：主库负责写操作，从库负责读操作。

- **主库（Master）**：
  - 负责处理写请求。
  - 将操作日志（如 binlog）同步给从库。
- **从库（Slave）**：
  - 只读，定时从主库拉取日志并执行。
  - 可读写分离、提高读性能。

#### 使用场景：

- 数据备份
- 增强读能力
- 容灾恢复

#### 注意问题：

- 延迟问题（主从数据不一致）
- 写入主库失败后主从切换

------

### ✅ 3. 分库分表设计思路

> 应对单表数据量过大、读写压力瓶颈。

#### ✅ 分库：

- 按模块（用户库、订单库）
- 按业务（商户库、日志库）
- 按时间（按月分库）

#### ✅ 分表：

- 按用户 ID 哈希分表
- 按时间（如 order_2024_01, order_2024_02）
- 按地理区域、商户 ID

#### ✅ 路由策略：

- 维护分片路由配置表
- 通过中间件如 ShardingSphere、MyCat 实现自动路由

#### 注意点：

- 跨表查询性能差
- 聚合统计需汇总多个分片
- 分布式事务问题（需二阶段提交、消息队列等辅助）

------

### ✅ 4. 数据库备份与恢复策略

#### 📌 备份方式：

| 类型     | 说明                         | 工具              |
| -------- | ---------------------------- | ----------------- |
| 逻辑备份 | 导出 SQL 文件（结构 + 数据） | `mysqldump`       |
| 物理备份 | 拷贝数据文件（更快）         | `XtraBackup`      |
| 增量备份 | 只备份新增或变动的数据       | binlog + 定期快照 |

#### 📌 恢复方式：

- 导入 SQL 文件（适合小数据量）
- 使用二进制日志还原到指定时间点
- 通过备份 + 增量合并恢复

#### 📌 生产建议：

- 保证每日全量备份 + 多点存储（云 OSS、本地）
- 保留 7 天以上备份
- 定期演练恢复流程（防“备份等于没备”）

------

### 🔚 总结梳理

| 知识点       | 面试关键词                    | 实践建议                       |
| ------------ | ----------------------------- | ------------------------------ |
| SQL vs NoSQL | CAP、ACID vs BASE、场景选择   | 掌握 MongoDB、Redis 的基本操作 |
| 主从复制     | binlog、延迟、读写分离        | 需掌握从库故障恢复策略         |
| 分库分表     | 哈希路由、雪花 ID、事务拆分   | 推荐 ShardingSphere 实战       |
| 备份恢复     | `mysqldump`、binlog、增量恢复 | 写脚本定期备份 + 异地容灾      |

------

如果你需要我整理一份《数据库高频面试真题 + 模拟解答》，也可以继续告诉我，我可以生成一份适配岗位的复习文档 PDF。是否需要？



------



## MySQL函数分类

### 逻辑查询的量词

在 SQL（特别是 MySQL）的逻辑查询中，所谓的“量词”其实是指**用于集合比较的逻辑运算符**。它们主要用于 `WHERE` 子句中，配合子查询来判断某个条件是否对集合中的**部分**或**全部**元素成立。

根据离散数学和 SQL 标准的对应关系，逻辑查询中的量词主要分为两大类：**存在量词**和**全称量词**[[source_group_web_1]]。

以下是这两类量词的详细聚合对比，以及它们在 SQL 中的具体实现形式：

#### 1. 存在量词（Existential Quantifier）

**数学符号：** $\exists$
**核心逻辑：** “只要有一个满足，即为真”[[source_group_web_2]]。

这类量词用于判断**是否存在**至少一条记录满足条件。

| SQL 关键字 | 含义   | 逻辑规则                                                     | 常用等价形式 | 适用场景                                                     |
| ---------- | ------ | ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
| **`ANY`**  | 任意   | 只要子查询结果中有**任意一个**值满足条件，就返回 `TRUE`[[source_group_web_3]]。 | -            | 通常配合比较运算符使用（如 `> ANY`）。                       |
| **`SOME`** | 有些   | 与 `ANY` 完全同义，意思一样，可以互换[[source_group_web_4]]。 | -            | `SOME` 是 SQL 标准中更正式的术语，但 `ANY` 更常用。          |
| **`IN`**   | 在其中 | 判断值是否**等于**子查询结果中的**某一个**[[source_group_web_5]]。 | **`= ANY`**  | 最常用的“存在”判断，例如查询某部门的所有员工[[source_group_web_6]]。 |

**💡 逻辑示例：**
假设子查询结果为 `[10, 20, 30]`。

- `x > ANY (子查询)`：如果 `x = 15`，因为 15 > 10（存在一个），所以为 `TRUE`。
- `x IN (子查询)`：如果 `x = 20`，因为 20 等于集合中的 20，所以为 `TRUE`。

------

#### 2. 全称量词（Universal Quantifier）

**数学符号：** $\forall$
**核心逻辑：** “所有都满足，才为真”[[source_group_web_7]]。

这类量词用于判断是否**所有**记录都满足条件。在 SQL 中，直接使用 `ALL` 的逻辑比较反直觉，通常建议用 `NOT EXISTS` 逻辑来理解。

| SQL 关键字   | 含义     | 逻辑规则                                                     | 常用等价形式 | 适用场景                                                     |
| ------------ | -------- | ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
| **`ALL`**    | 全部     | 只有当子查询结果中的**每一个**值都满足条件时，才返回 `TRUE`[[source_group_web_8]]。 | -            | 配合比较运算符使用（如 `> ALL`）。                           |
| **`NOT IN`** | 不在其中 | 判断值是否**不等于**子查询结果中的**任何一个**[[source_group_web_9]]。 | **`!= ALL`** | 排除子查询中的所有结果，例如查询不在黑名单中的用户[[source_group_web_10]]。 |

**💡 逻辑示例：**
假设子查询结果为 `[10, 20, 30]`。

- `x > ALL (子查询)`：只有当 `x = 31` 时（31 大于集合中所有值），才为 `TRUE`。如果 `x = 25`，因为 25 < 30，所以为 `FALSE`。
- `x NOT IN (子查询)`：如果 `x = 25`，因为 25 不在集合中，所以为 `TRUE`。

------

#### 3. 特殊逻辑：空集（Empty Set）行为

这是一个非常重要的“坑”。当子查询返回**空结果集**（0 条记录）时，`ANY` 和 `ALL` 的行为是不同的：

- `ANY` / `SOME`

  ：在空集上恒为 

  `FALSE`

  。

  - *逻辑解释：* “存在一个满足条件的”在没有任何元素的情况下，当然是假的。

- `ALL`

  ：在空集上恒为 

  `TRUE`

  。

  - *逻辑解释：* “所有的都满足条件”。因为没有反例（没有任何元素不满足），所以在逻辑上被视为真（这叫“空真”）[[source_group_web_11]]。

#### 4. 总结对照表（速查）

| 数学概念               | SQL 关键字            | 通俗理解                      | 等价转换 (推荐写法)             |
| ---------------------- | --------------------- | ----------------------------- | ------------------------------- |
| **存在量词** $\exists$ | **`ANY`**, **`SOME`** | 只要有一个就行                | -                               |
| **存在量词** $\exists$ | **`IN`**              | 在列表里能找到                | `= ANY`                         |
| **全称量词** $\forall$ | **`ALL`**             | 必须比最大的还大/比最小的还小 | -                               |
| **全称量词** $\forall$ | **`NOT IN`**          | 在列表里都找不到              | `!= ALL`                        |
| **否定存在**           | **`NOT EXISTS`**      | 一个都找不到                  | `!= ALL` (逻辑等价，但写法不同) |

**建议：**
虽然 SQL 支持 `ANY`/`ALL`，但在实际开发中：
\1. **用 `IN` 代替 `= ANY`**：可读性更好[[source_group_web_13]]。
\2. **用 `NOT IN` 代替 `!= ALL`**：可读性更好[[source_group_web_14]]。
\3. **复杂的 `ALL` 比较（如 `> ALL`）**：建议直接改写为 `MIN()` 或 `MAX()`，或者使用 `NOT EXISTS` 逻辑，以免产生逻辑错误[[source_group_web_15]]。

### 1. **字符串函数**

这些函数用于操作和处理字符串数据。

- `CONCAT(str1, str2, ...)`: 将多个字符串连接成一个字符串。
- `LENGTH(str)`: 返回字符串的字节长度。
- `CHAR_LENGTH(str)`: 返回字符串的字符长度（考虑多字节字符）。
- `SUBSTRING(str, start, length)`: 从字符串中提取子串。
- `REPLACE(str, old_str, new_str)`: 替换字符串中的某个子串。
- `LOWER(str)`, `UPPER(str)`: 将字符串转换为小写或大写。
- `INSERT(字符串, 起始位, 删除长度, 替换串)`。SELECT INSERT('lo ina',3,1,'ve ch')` → `love china。
- `LEFT(字符串, 截取长度)`从字符串**左侧开头**截取指定长度字符。`LEFT('lo ina', 2)` → `'lo'`（取前 2 个字符）。
- `RIGHT(字符串, 截取长度)`从字符串**右侧末尾**截取指定长度字符。`RIGHT('lo ina', 3)` → `'ina'`（取最后 3 个字符）。

#### `SUBSTRING_INDEX()` 

是 MySQL 中一个非常实用的字符串处理函数，专门用来**按指定的分隔符截取字符串**[[source_group_web_1]]。

它在处理包含分隔符的文本数据（如 URL、文件路径、CSV 标签、IP 地址等）时特别好用[[source_group_web_2]]。

以下是它的核心用法、实战示例以及注意事项：

##### 1. 语法结构

```sql
SUBSTRING_INDEX(str, delim, count)
```

- **`str`**: 原始字符串（或字段名）。

- **`delim`**: 分隔符（例如 `.`、`,`、`/`、`@` 等）。

- `count`

  : 截取的方向和段数。

  - **正数**：从左往右数，返回第 `count` 个分隔符**左边**的所有内容。
  - **负数**：从右往左数，返回第 `|count|` 个分隔符**右边**的所有内容[[source_group_web_3]]。

##### 2. 核心逻辑演示

假设我们有一个字符串：`'www.mysql.com'`

| SQL 语句                                    | 结果                               | 解释                         |
| ------------------------------------------- | ---------------------------------- | ---------------------------- |
| `SUBSTRING_INDEX('www.mysql.com', '.', 1)`  | **www**                            | 取第 1 个 `.` 左边的内容     |
| `SUBSTRING_INDEX('www.mysql.com', '.', 2)`  | **[www.mysql](http://www.mysql/)** | 取第 2 个 `.` 左边的内容     |
| `SUBSTRING_INDEX('www.mysql.com', '.', -1)` | **com**                            | 取倒数第 1 个 `.` 右边的内容 |
| `SUBSTRING_INDEX('www.mysql.com', '.', -2)` | **mysql.com**                      | 取倒数第 2 个 `.` 右边的内容 |

##### 3. 常见实战场景

###### 🌐 场景一：URL 或路径处理

- 提取域名：

  ```sql
  SELECT SUBSTRING_INDEX('https://www.example.com/page', '/', 3);
  -- 结果: https://www.example.com
  ```

- 提取文件名：

  ```sql
  SELECT SUBSTRING_INDEX('/var/www/html/index.php', '/', -1);
  -- 结果: index.php
  ```

- 提取二级路径：

  ```sql
  SELECT SUBSTRING_INDEX(SUBSTRING_INDEX('/a/b/c/d', '/', 3), '/', -1);
  -- 结果: b
  -- 原理：先截取前3段(/a/b/c)，再从右边截取1段，得到中间的b[[source_group_web_5]]。
  ```

###### 📧 场景二：邮箱或数据清洗

- 提取邮箱用户名：

  ```sql
  SELECT SUBSTRING_INDEX('user@example.com', '@', 1);
  -- 结果: user
  ```

- 提取邮箱域名：

  ```sql
  SELECT SUBSTRING_INDEX('user@example.com', '@', -1);
  -- 结果: example.com
  ```

- 去除括号内容：

   (例如 `张三(管理员)`变为 `张三`)

  ```sql
  SELECT SUBSTRING_INDEX('张三(管理员)', '(', 1);
  -- 结果: 张三
  ```

###### 📊 场景三：处理逗号分隔的值 (CSV)

假设有一个字段存储了 `语文,数学,英语`：

- 获取第一门科目：

  ```sql
  SELECT SUBSTRING_INDEX('语文,数学,英语', ',', 1); -- 语文
  ```

- 获取最后一门科目：

  ```sql
  SELECT SUBSTRING_INDEX('语文,数学,英语', ',', -1); -- 英语
  ```

- 获取第二门科目 (数学)：

  ```sql
  SELECT SUBSTRING_INDEX(SUBSTRING_INDEX('语文,数学,英语', ',', 2), ',', -1);
  -- 原理：先取前2段("语文,数学")，再取最后一段。
  ```

##### 4. ⚠️ 注意事项与坑点

1. **分隔符不存在：**
   如果字符串中不包含指定的分隔符，函数会直接返回**整个原始字符串**[[source_group_web_6]]。
   - `SUBSTRING_INDEX('hello', '.', 1)` 的结果是 `'hello'`。
2. **`count` 超出范围：**
   如果 `count` 的绝对值大于分隔符出现的次数，同样会返回**整个原始字符串**[[source_group_web_7]]。
   - `SUBSTRING_INDEX('a,b', ',', 5)` 的结果是 `'a,b'`。
3. **区分大小写：**
   分隔符是**区分大小写**的[[source_group_web_8]]。
   - `SUBSTRING_INDEX('A-B-C', 'b', 1)` 会返回 `A-B-C`（因为找不到小写 b）。
4. **NULL 值处理：**
   如果任一参数为 `NULL`，结果通常为 `NULL`[[source_group_web_9]]。
5. **性能问题：**
   在 `WHERE` 条件中使用 `SUBSTRING_INDEX` 进行筛选通常会导致**索引失效**，引发全表扫描[[source_group_web_10]]。如果需要频繁查询，建议在插入数据时就将字段拆分存储，或者使用生成列（Generated Columns）。
6. **空值占位：**
   如果分隔符连续出现（如 `'a,,b'`），空的部分也会被算作一个“段”[[source_group_web_11]]。
   - `SUBSTRING_INDEX('a,,b', ',', 2)` 的结果是 `'a,'`。



`TRIM` 是一个在多种编程语言和数据库（尤其是 SQL）中广泛使用的字符串函数，它的核心作用是**移除字符串开头和/或结尾的指定字符**（默认通常是空格）[[source_group_web_1]]。

#### MySQL / SQL 中的 TRIM 函数

在 SQL 中，`TRIM` 是数据清洗（Data Cleaning）的利器，用于处理用户输入或导入数据时产生的多余空格或特殊符号[[source_group_web_2]]。

##### 📝 语法结构

```sql
TRIM([{BOTH | LEADING | TRAILING} [remstr] FROM] str)
```

- **`str`**: 要处理的原始字符串或字段名。
- **`remstr`**: （可选）要移除的字符。如果不指定，默认移除**空格**。
- **`BOTH`**: 移除两端的字符（默认值）。
- **`LEADING`**: 仅移除开头（左侧）的字符。
- **`TRAILING`**: 仅移除结尾（右侧）的字符。

##### 🧪 常见用法示例

| SQL 语句                             | 原始字符串  | 结果    | 说明                    |
| ------------------------------------ | ----------- | ------- | ----------------------- |
| `TRIM(' hello ')`                    | `hello`     | `hello` | 默认移除两端空格        |
| `TRIM(LEADING 'x' FROM 'xxhello')`   | `xxhello`   | `hello` | 仅移除左侧的 'x'        |
| `TRIM(TRAILING '!' FROM 'hello!!!')` | `hello!!!`  | `hello` | 仅移除右侧的 '!'        |
| `TRIM(BOTH '-' FROM '--test--')`     | `--test--`  | `test`  | 移除两端的 '-'          |
| `TRIM('x' FROM 'xxhelloxx')`         | `xxhelloxx` | `hello` | 简写形式（等同于 BOTH） |

##### 💡 配套函数（MySQL 特有简写）

MySQL 还提供了两个专门用于处理空格的快捷函数：

- **`LTRIM(str)`**: 等同于 `TRIM(LEADING ' ' FROM str)`，仅去除左侧空格[[source_group_web_3]]。
- **`RTRIM(str)`**: 等同于 `TRIM(TRAILING ' ' FROM str)`，仅去除右侧空格[[source_group_web_4]]。

##### 🌍 实战场景

- 用户登录验证：

   防止用户因输入账号时多打了一个空格而导致登录失败。

  ```sql
  SELECT * FROM users WHERE TRIM(username) = 'admin';
  ```

- 数据清洗：

   批量更新表，去除字段两端的空格。

  ```sql
  UPDATE products SET name = TRIM(name);
  ```

- **去除特殊符号：** 比如导入数据时，CSV 文件中的字段可能被双引号包围，可以用 `TRIM('"' FROM str)` 去除。

##### ⚠️ 注意事项

1. **不影响中间内容：** `TRIM` 只作用于字符串的**头部**和**尾部**，字符串中间的空格或字符不会被移除[[source_group_web_5]]。
2. **性能考量：** 在 SQL 的 `WHERE` 子句中对字段使用 `TRIM`（如 `WHERE TRIM(name) = 'test'`）通常会导致**索引失效**，因为数据库需要对每一行都进行计算。最佳实践是在数据插入时就进行清洗，或者使用触发器自动处理。
3. **大小写敏感：** 在某些数据库配置下，`TRIM` 可能是大小写敏感的（例如 `TRIM('X' FROM 'xXx')` 可能不生效，取决于排序规则）。
4. **NULL 值处理：** 如果输入的字符串是 `NULL`，`TRIM` 函数通常返回 `NULL`[[source_group_web_6]]。



#### `SUBSTRING`

在 SQL 中，`SUBSTRING` 是一个用于提取字符串中部分字符的函数，不同数据库（如 MySQL、SQL Server、Oracle 等）的语法略有差异，但核心功能一致：从指定位置开始，截取指定长度的子字符串。

##### 基本语法（通用逻辑）

```sql
SUBSTRING(字符串, 起始位置, 截取长度)
```
- **字符串**：要处理的原始字符串（可以是字段名或直接字符串）。
- **起始位置**：截取的起始索引（注意：不同数据库的索引起点可能不同，如 MySQL 从 1 开始，Oracle 也从 1 开始，而有些语言可能从 0 开始，需特别注意）。
- **截取长度**：可选参数，指定要截取的字符数。若省略，默认截取到字符串末尾。
- 支持 `SUBSTRING(字符串, 起始位置)` 或 `SUBSTR(字符串, 起始位置)`，省略长度时默认截取到末尾。

##### 不同数据库中的具体用法

###### 1. MySQL 中的 `SUBSTRING`

MySQL 中支持两种写法：`SUBSTRING()` 或 `SUBSTR()`（两者完全等价），起始位置从 **1 开始**。

**示例**：
```sql
-- 截取从第 2 个字符开始，长度为 3 的子串
SELECT SUBSTRING('HelloWorld', 2, 3); -- 结果：'ell'

-- 省略长度，截取从第 6 个字符到末尾
SELECT SUBSTR('HelloWorld', 6); -- 结果：'World'

-- 起始位置为负数：从字符串末尾倒数第 N 个字符开始
SELECT SUBSTRING('HelloWorld', -3, 2); -- 结果：'rl'（从倒数第3个字符开始，取2个）
```

###### 2. SQL Server 中的 `SUBSTRING`

语法与 MySQL 类似，起始位置从 **1 开始**，且必须指定截取长度（若要取到末尾，长度可设为字符串总长度）。

**示例**：
```sql
-- 截取从第 1 个字符开始，长度为 5 的子串
SELECT SUBSTRING('HelloWorld', 1, 5); -- 结果：'Hello'

-- 截取从第 6 个字符到末尾（假设总长度为 10）
SELECT SUBSTRING('HelloWorld', 6, 10); -- 结果：'World'
```

###### 3. Oracle 中的 `SUBSTR`

Oracle 中通常使用 `SUBSTR()`（而非 `SUBSTRING`），起始位置从 **1 开始**，支持负数起始位置（从末尾倒数）。

**示例**：
```sql
-- 截取从第 3 个字符开始，长度为 4 的子串
SELECT SUBSTR('HelloWorld', 3, 4) FROM DUAL; -- 结果：'lloW'

-- 省略长度，截取从第 6 个字符到末尾
SELECT SUBSTR('HelloWorld', 6) FROM DUAL; -- 结果：'World'

-- 负数起始位置：从末尾倒数第 5 个字符开始
SELECT SUBSTR('HelloWorld', -5) FROM DUAL; -- 结果：'World'
```

##### 注意事项

1. **起始位置的差异**：几乎所有 SQL 数据库的 `SUBSTRING` 起始位置都是从 **1 开始**（而非编程语言中常见的 0 开始），这是最容易出错的点。
   
2. **截取长度的处理**：
   - 若指定的长度超过字符串剩余字符数，会自动截取到字符串末尾（不会报错）。
   - 若长度为 0 或负数，不同数据库可能返回空字符串或报错（如 MySQL 返回空，SQL Server 报错）。

3. **与 `LEFT`/`RIGHT` 函数的区别**：
   - `LEFT(字符串, 长度)`：从左侧开始截取指定长度（等价于 `SUBSTRING(字符串, 1, 长度)`）。
   - `RIGHT(字符串, 长度)`：从右侧开始截取指定长度（等价于 `SUBSTRING(字符串, LENGTH(字符串)-长度+1, 长度)`）。

##### 典型应用场景

- 提取身份证号中的出生日期（如从第 7 位开始截取 8 位）。
- 处理固定格式的编码（如截取前 3 位作为分类标识）。
- 模糊匹配时截取部分字符（如 `WHERE SUBSTRING(name, 1, 1) = 'A'` 筛选首字母为 A 的名字）。

根据使用的数据库类型，需对应调整 `SUBSTRING` 的语法细节。

### 2. **日期和时间函数**

这些函数用于处理日期和时间类型数据。

- `NOW()`: 返回当前的日期和时间。
- `CURDATE()`: 返回当前日期（不含时间）。
- `DATE_ADD(date, INTERVAL expr unit)`: 给日期加上指定的时间间隔。
- `DATE_SUB(date, INTERVAL expr unit)`: 从日期减去指定的时间间隔。
- `YEAR(date)`, `MONTH(date)`, `DAY(date)`: 提取日期中的年、月、日部分。
- `DATE_FORMAT(date, format)`: 格式化日期，返回自定义格式的日期。

MySQL 的日期时间类型和函数覆盖了从存储到计算的全场景：

- 选择类型时，根据是否需要时区转换（`TIMESTAMP`）、精度（日期/时间/完整时间）和范围（`DATETIME` 范围更大）决定；
- 函数可实现日期提取、计算、转换等操作，结合 `DATE_FORMAT` 可灵活处理各种时间格式；
- 实际开发中，常用于订单时间筛选、数据按时间分组统计、时间差计算等场景。

#### 一、MySQL 日期时间类型

MySQL 支持多种日期时间类型，适用于不同的时间精度需求：

| 类型        | 格式                  | 范围                                               | 存储空间 | 用途说明                                         |
| ----------- | --------------------- | -------------------------------------------------- | -------- | ------------------------------------------------ |
| `DATE`      | `YYYY-MM-DD`          | 1000-01-01 至 9999-12-31                           | 3字节    | 仅存储日期（年/月/日）                           |
| `TIME`      | `HH:MM:SS`            | -838:59:59 至 838:59:59                            | 3字节    | 仅存储时间（时/分/秒），可表示负数（如时间间隔） |
| `DATETIME`  | `YYYY-MM-DD HH:MM:SS` | 1000-01-01 00:00:00 至 9999-12-31 23:59:59         | 8字节    | 存储日期+时间，不受时区影响                      |
| `TIMESTAMP` | `YYYY-MM-DD HH:MM:SS` | 1970-01-01 00:00:01 UTC 至 2038-01-19 03:14:07 UTC | 4字节    | 存储日期+时间，受时区影响（会自动转换时区）      |
| `YEAR`      | `YYYY`                | 1901 至 2155                                       | 1字节    | 仅存储年份                                       |


#### 核心区别：

- **`DATETIME` vs `TIMESTAMP`**：  
  - `DATETIME` 存储原始时间，不随数据库时区变化；  
  - `TIMESTAMP` 存储UTC时间，查询时会根据数据库时区转换为本地时间（适合跨时区场景）。  
- **存储范围**：`DATETIME` 范围更大（支持到9999年），`TIMESTAMP` 受限于4字节（到2038年）。  

#### 二、常用日期时间函数

MySQL 提供了大量函数用于日期的获取、转换、计算等操作，以下是高频使用的函数：

##### 1. 获取当前日期时间

| 函数                            | 作用                               | 示例结果              |
| ------------------------------- | ---------------------------------- | --------------------- |
| `CURDATE()` / `CURRENT_DATE()`  | 获取当前日期（`DATE`类型）         | `2023-10-08`          |
| `CURTIME()` / `CURRENT_TIME()`  | 获取当前时间（`TIME`类型）         | `14:30:25`            |
| `NOW()` / `CURRENT_TIMESTAMP()` | 获取当前日期时间（`DATETIME`类型） | `2023-10-08 14:30:25` |
| `UNIX_TIMESTAMP()`              | 获取当前Unix时间戳（秒数）         | `1696761025`          |

##### 2. 日期时间提取（从日期中获取年/月/日等）

```
'2023-10-08 14:30:25'`
**名称：** 日期时间格式。
**结构：** `YYYY-MM-DD HH:MM:SS

'2023-10-08'
名称： 日历日期格式。
结构： YYYY-MM-DD
```

| 函数              | 作用                             | 示例（基于 `'2023-10-08 14:30:25'`）    |
| ----------------- | -------------------------------- | --------------------------------------- |
| `YEAR(date)`      | 提取年份                         | `YEAR('2023-10-08')` → `2023`           |
| `MONTH(date)`     | 提取月份（1-12）                 | `MONTH('2023-10-08')` → `10`            |
| `DAY(date)`       | 提取日（1-31）                   | `DAY('2023-10-08')` → `8`               |
| `HOUR(time)`      | 提取小时（0-23）                 | `HOUR('14:30:25')` → `14`               |
| `MINUTE(time)`    | 提取分钟（0-59）                 | `MINUTE('14:30:25')` → `30`             |
| `SECOND(time)`    | 提取秒（0-59）                   | `SECOND('14:30:25')` → `25`             |
| `DAYOFWEEK(date)` | 一周中的第几天（1=周日，7=周六） | `DAYOFWEEK('2023-10-08')` → `1`（周日） |
| `WEEK(date)`      | 一年中的第几周（1-53）           | `WEEK('2023-10-08')` → `40`             |


#### 3. 日期时间计算（加减、差值）

| 函数/语法                            | 作用                              | 示例                                                      |
| ------------------------------------ | --------------------------------- | --------------------------------------------------------- |
| `DATE_ADD(date, INTERVAL expr unit)` | 日期加间隔                        | `DATE_ADD('2023-10-08', INTERVAL 1 MONTH)` → `2023-11-08` |
| `DATE_SUB(date, INTERVAL expr unit)` | 日期减间隔                        | `DATE_SUB('2023-10-08', INTERVAL 7 DAY)` → `2023-10-01`   |
| `DATEDIFF(date1, date2)`             | 两个日期的天数差（date1 - date2） | `DATEDIFF('2023-10-08', '2023-10-01')` → `7`              |
| `TIMESTAMPDIFF(unit, start, end)`    | 两个时间的差值（指定单位）        | `TIMESTAMPDIFF(HOUR, '10:00', '14:30')` → `4`（小时差）   |


#### 4. 日期时间转换（格式转换、类型转换）

| 函数                        | 作用                           | 示例                                                         |
| --------------------------- | ------------------------------ | ------------------------------------------------------------ |
| `DATE_FORMAT(date, format)` | 日期转字符串（自定义格式）     | `DATE_FORMAT('2023-10-08', '%Y年%m月%d日')` → `2023年10月08日` |
| `STR_TO_DATE(str, format)`  | 字符串转日期（按指定格式解析） | `STR_TO_DATE('2023年10月08日', '%Y年%m月%d日')` → `2023-10-08` |
| `UNIX_TIMESTAMP(date)`      | 日期转Unix时间戳               | `UNIX_TIMESTAMP('2023-10-08 14:30:00')` → `1696761000`       |
| `FROM_UNIXTIME(timestamp)`  | Unix时间戳转日期               | `FROM_UNIXTIME(1696761000)` → `2023-10-08 14:30:00`          |


#### 5. 其他实用函数

| 函数                  | 作用                                  | 示例                                      |
| --------------------- | ------------------------------------- | ----------------------------------------- |
| `LAST_DAY(date)`      | 获取当月最后一天                      | `LAST_DAY('2023-10-08')` → `2023-10-31`   |
| `ADDDATE(date, days)` | 日期加N天（等价于 `DATE_ADD` 简化版） | `ADDDATE('2023-10-08', 3)` → `2023-10-11` |
| `SUBDATE(date, days)` | 日期减N天（等价于 `DATE_SUB` 简化版） | `SUBDATE('2023-10-08', 3)` → `2023-10-05` |

#### 三、格式符（`DATE_FORMAT` 和 `STR_TO_DATE` 中使用）

常用格式符用于自定义日期字符串的显示或解析格式：

| 格式符 | 说明                  | 示例（基于 `2023-10-08 14:30:25`） |
| ------ | --------------------- | ---------------------------------- |
| `%Y`   | 四位年份              | `%Y` → `2023`                      |
| `%m`   | 两位月份（01-12）     | `%m` → `10`                        |
| `%d`   | 两位日（01-31）       | `%d` → `08`                        |
| `%H`   | 24小时制小时（00-23） | `%H` → `14`                        |
| `%i`   | 两位分钟（00-59）     | `%i` → `30`                        |
| `%s`   | 两位秒（00-59）       | `%s` → `25`                        |
| `%W`   | 星期全称              | `%W` → `Sunday`                    |
| `%M`   | 月份全称              | `%M` → `October`                   |

#### 四、使用示例

1. **查询近7天的订单**：

   ```sql
   SELECT * FROM orders 
   WHERE create_time >= DATE_SUB(CURDATE(), INTERVAL 7 DAY);
   ```

2. **统计每月的订单量**：

   ```sql
   SELECT 
     DATE_FORMAT(create_time, '%Y-%m') AS month,
     COUNT(*) AS order_count 
   FROM orders 
   GROUP BY month 
   ORDER BY month;
   ```

3. **计算两个时间的小时差**：

   ```sql
   SELECT 
     TIMESTAMPDIFF(HOUR, '2023-10-08 10:00:00', '2023-10-08 14:30:00') 
     AS hour_diff; -- 结果：4
   ```

4. **将字符串转换为日期**：

   ```sql
   SELECT STR_TO_DATE('2023/10/08', '%Y/%m/%d') AS date; -- 结果：2023-10-08
   ```



### 3. **数学函数**

用于进行数学计算。

- `ABS(x)`: 返回 x 的绝对值。
- `ROUND(x, d)`: 对 x 进行四舍五入，保留 d 位小数。ROUND（x）：默认四舍五入且不保留小数
- `FLOOR(x)`: 返回小于或等于 x 的最大整数。
- `CEIL(x)`: 返回大于或等于 x 的最小整数。
- `POWER(x, y)`: 返回 x 的 y 次方。

### 4. **聚合函数**

聚合函数用于执行一组数据上的计算，常见于 `GROUP BY` 操作中。

- `COUNT()`: 统计行数。
- `SUM()`: 求和。
- `AVG()`: 求平均值。
- `MIN()`, `MAX()`: 返回最小值和最大值。



#### AVG

在 MySQL 中，`AVG()` 是一个**聚合函数（Aggregate Function）**，用于计算**某列数值的平均值（算术平均数）**。

---

##### ✅ 基本语法

```sql
SELECT AVG(column_name) 
FROM table_name
[WHERE condition]
[GROUP BY group_column];
```

---

##### 🔑 核心特性

| 特性 | 说明 |
|------|------|
| **忽略 NULL 值** | `AVG()` 会自动跳过 `NULL`，只对非 NULL 数值求平均 |
| **返回 DECIMAL 或 DOUBLE** | 结果通常是高精度浮点数 |
| **只能用于数值列** | 对字符串、日期等非数值类型使用会返回 `0` 或警告（不推荐） |
| **常与 GROUP BY 搭配** | 按分组计算平均值 |

---

##### 📌 基础示例

假设有一张 `scores` 表：

| id | student | score |
|----|---------|-------|
| 1  | Alice   | 90    |
| 2  | Bob     | 85    |
| 3  | Carol   | NULL  |
| 4  | Dave    | 95    |

###### 1. 计算所有学生的平均分

```sql
SELECT AVG(score) AS avg_score FROM scores;
-- 结果: 90.0000  （(90 + 85 + 95) / 3 = 90）
-- 注意：NULL 被忽略，分母是 3 而不是 4
```

###### 2. 保留小数位数（配合 `ROUND()`）

```sql
SELECT ROUND(AVG(score), 2) AS avg_score FROM scores;
-- 结果: 90.00
```

---

##### 🧩 高级用法

###### 1. **按分组求平均（GROUP BY）**

```sql
SELECT class, AVG(score) AS avg_score
FROM students
GROUP BY class;
```

###### 2. **条件平均（配合 WHERE）**

```sql
-- 只计算数学成绩的平均分
SELECT AVG(score) 
FROM scores 
WHERE subject = 'Math';
```

###### 3. **布尔表达式 + AVG() → 计算比例（技巧！）**

> 💡 在 MySQL 中，`TRUE = 1`，`FALSE = 0`，所以 `AVG(condition)` = 满足条件的比例！

```sql
-- 计算及格率（score >= 60 的比例）
SELECT AVG(score >= 60) AS pass_rate FROM scores;

-- 等价于：
SELECT COUNT(CASE WHEN score >= 60 THEN 1 END) * 1.0 / COUNT(*) 
FROM scores;
```

✅ 这是 LeetCode 和实际开发中非常常用的技巧！

###### 4. **去重平均（AVG(DISTINCT ...)）**

```sql
-- 只对不重复的分数求平均
SELECT AVG(DISTINCT score) FROM scores;
```
> 例如：分数为 [90, 90, 80] → `AVG(score) = 86.67`，`AVG(DISTINCT score) = 85`

---

##### ⚠️ 常见误区

###### ❌ 对非数值列使用 AVG

```sql
SELECT AVG(name) FROM students;  -- 返回 0 或警告，无意义！
```

###### ❌ 忘记 NULL 的影响

```sql
-- 如果所有值都是 NULL
SELECT AVG(score) FROM scores WHERE 1=0;  -- 返回 NULL，不是 0！
```

###### ❌ 在 SELECT 中混用非聚合列（违反 ONLY_FULL_GROUP_BY）

```sql
-- ❌ 错误（除非关闭 sql_mode）
SELECT student, AVG(score) FROM scores;
-- ✅ 正确：要么加 GROUP BY，要么只选 AVG
SELECT AVG(score) FROM scores;
```

---

##### ✅ 实战案例（LeetCode 风格）

**题目**：求每个部门的平均工资  
**表**：`employees (id, name, dept, salary)`

```sql
SELECT dept, ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY dept;
```

---

##### 📊 总结

| 场景 | 写法 |
|------|------|
| 全表平均 | `SELECT AVG(col) FROM t;` |
| 分组平均 | `SELECT group_col, AVG(col) FROM t GROUP BY group_col;` |
| 比例计算 | `SELECT AVG(condition) FROM t;` |
| 保留小数 | `SELECT ROUND(AVG(col), 2) FROM t;` |
| 去重平均 | `SELECT AVG(DISTINCT col) FROM t;` |

> 💡 **记住**：`AVG()` 是数据分析中最常用的聚合函数之一，配合 `GROUP BY`、`WHERE` 和 `ROUND()` 能解决绝大多数平均值问题。

#### GROUP_CONCAT

`GROUP_CONCAT` 是 MySQL 中一个非常实用的**聚合函数**，用于将同一分组内的多个字段值**拼接成一个字符串**，常用于按组合并数据的场景（如将某用户的所有订单号合并为一个字符串）。

##### 一、基本语法

```sql
GROUP_CONCAT([DISTINCT] 字段名 [ORDER BY 排序字段 ASC/DESC] [SEPARATOR '分隔符'])
```

###### 各参数说明：

- `DISTINCT`（可选）：去重，仅保留不重复的字段值。
- `ORDER BY`（可选）：对拼接的字段值按指定规则排序（如按字母/数字升序/降序）。
- `SEPARATOR`（可选）：指定拼接时的分隔符（默认是逗号 `,`）。

##### 二、使用场景与示例

假设存在表 `student_courses`，存储学生选修的课程：

| student_id | course      |
|------------|-------------|
| 1          | 数学        |
| 1          | 英语        |
| 1          | 数学        |  -- 重复选修
| 2          | 物理        |
| 2          | 化学        |

###### 1. 基础用法：拼接同一分组的字段值

按 `student_id` 分组，拼接每个学生的所有课程（默认逗号分隔，包含重复值）：
```sql
SELECT 
  student_id,
  GROUP_CONCAT(course) AS all_courses
FROM student_courses
GROUP BY student_id;
```
**结果**：
```
student_id | all_courses
-----------|-------------
1          | 数学,英语,数学  -- 包含重复的“数学”
2          | 物理,化学
```

###### 2. 去重拼接（`DISTINCT`）

去除重复的课程后再拼接：
```sql
SELECT 
  student_id,
  GROUP_CONCAT(DISTINCT course) AS unique_courses
FROM student_courses
GROUP BY student_id;
```
**结果**：
```
student_id | unique_courses
-----------|----------------
1          | 数学,英语       -- 去重后仅保留一个“数学”
2          | 物理,化学
```

###### 3. 排序后拼接（`ORDER BY`）

按课程名称字母顺序（升序）拼接：
```sql
SELECT 
  student_id,
  GROUP_CONCAT(DISTINCT course ORDER BY course ASC) AS sorted_courses
FROM student_courses
GROUP BY student_id;
```
**结果**（假设课程按拼音首字母排序：英语 < 数学 < 化学 < 物理）：
```
student_id | sorted_courses
-----------|----------------
1          | 英语,数学       -- 按字母升序排列
2          | 化学,物理
```

###### 4. 自定义分隔符（`SEPARATOR`）

用分号 `;` 作为分隔符：
```sql
SELECT 
  student_id,
  GROUP_CONCAT(DISTINCT course SEPARATOR ';') AS semicolon_courses
FROM student_courses
GROUP BY student_id;
```
**结果**：
```
student_id | semicolon_courses
-----------|-------------------
1          | 数学;英语
2          | 物理;化学
```

##### 三、注意事项

1. **长度限制**：  
   MySQL 对 `GROUP_CONCAT` 拼接的字符串长度有默认限制（由系统变量 `group_concat_max_len` 控制，默认值通常为 1024 字节）。若拼接结果过长，会被截断。  
   可通过以下方式修改限制（临时生效，重启 MySQL 后恢复默认）：
   ```sql
   SET SESSION group_concat_max_len = 102400;  -- 设置为 100KB
   ```

2. **必须与 `GROUP BY` 配合使用**：  
   `GROUP_CONCAT` 是聚合函数，只能用于 `GROUP BY` 分组后的查询中，否则会将整个表视为一个组，返回所有记录的拼接结果。

3. **NULL 值处理**：  
   若字段值为 `NULL`，`GROUP_CONCAT` 会忽略 `NULL`（不参与拼接）。

##### 四、典型应用场景

- 生成标签列表：如将某篇文章的所有标签（`tag1,tag2,tag3`）合并展示。
- 汇总关联数据：如将某订单的所有商品名称合并为一个字符串。
- 简化多表查询：避免通过多次关联查询获取同一分组的多条记录，直接一次性拼接返回。

通过灵活使用 `DISTINCT`、`ORDER BY` 和 `SEPARATOR`，`GROUP_CONCAT` 能高效地按组合并数据，简化报表生成或数据展示逻辑。

#### SQL面试：介绍聚合函数

在 SQL 面试中，聚合函数是高频考点，核心要围绕「定义、常用函数、使用规则、底层逻辑、典型场景」展开，既要讲清基础用法，也要体现对细节和坑点的理解。以下是结构化的回答思路，适合面试场景：

##### 一、聚合函数的核心定义

聚合函数（Aggregate Function）是 SQL 中用于对**一组数据（多行记录）** 进行计算，返回**单个结果值**的函数。其核心作用是「数据汇总统计」，而非单行列的计算（如 `+/-` 运算）。

关键特性：
1. 输入是「多行数据的某一列/多列」，输出是「单个值」；
2. 常与 `GROUP BY` 搭配使用（按分组统计），也可单独使用（对全表统计）；
3. 会自动忽略 `NULL` 值（除 `COUNT(*)` 外）；
4. 属于「列函数」，不能直接与普通列（非聚合列）在 `SELECT` 中混用（需 `GROUP BY` 约束）。

##### 二、常用聚合函数及场景（结合面试高频考点）

###### 1. `COUNT()`：统计行数/非空值数量（面试必考）

- 语法：`COUNT(参数)`，参数支持 3 种形式：
  - `COUNT(*)`：统计**所有行数**（包括 `NULL` 行、重复行），效率最高（InnoDB 优化，无需扫描字段值）；
  - `COUNT(列名)`：统计该列**非 NULL 的行数**（忽略 `NULL` 值）；
  - `COUNT(DISTINCT 列名)`：统计该列**非 NULL 且不重复的行数**。
- 示例：
  ```sql
  SELECT COUNT(*) AS total_rows,        -- 全表行数（含 NULL 行）
         COUNT(name) AS non_null_names, -- name 列非 NULL 行数
         COUNT(DISTINCT age) AS unique_ages -- age 列非 NULL 不重复行数
  FROM users;
  ```
- 面试坑点：`COUNT(1)` 与 `COUNT(*)` 的区别？  
  结论：**功能一致（都统计行数），效率几乎无差异**（InnoDB 中 `COUNT(*)` 已优化，无需扫描字段，`COUNT(1)` 是占位符，等价于 `COUNT(*)`）；避免用 `COUNT(列名)` 替代 `COUNT(*)`（除非明确需要过滤 NULL）。

###### 2. `SUM()`：求和（数值型字段专用）

- 语法：`SUM(列名)` / `SUM(DISTINCT 列名)`（去重后求和）；
- 特性：仅支持数值型字段（int、decimal 等），字符串/日期类型会报错；自动忽略 `NULL` 值；
- 示例：
  ```sql
  SELECT SUM(salary) AS total_salary,       -- 所有员工工资总和
         SUM(DISTINCT salary) AS distinct_salary_sum -- 去重后工资总和
  FROM employees;
  ```
- 面试注意：若列中全是 `NULL`，`SUM()` 返回 `NULL`（而非 0），需用 `IFNULL(SUM(列名), 0)` 处理。

###### 3. `AVG()`：求平均值（数值型字段专用）

- 语法：`AVG(列名)` / `AVG(DISTINCT 列名)`；
- 底层逻辑：`AVG(col) = SUM(col) / COUNT(col)`（分母是该列非 NULL 行数，而非全表行数）；
- 示例：
  ```sql
  SELECT AVG(score) AS avg_score FROM students; -- 学生平均分
  ```
- 面试坑点：若需“全表行数（含 NULL）作为分母”，需手动计算：`SUM(col) / COUNT(*)`。

###### 4. `MAX()` / `MIN()`：求最大值/最小值

- 语法：`MAX(列名)` / `MIN(列名)`；
- 特性：支持数值型、字符串型、日期型字段：
  - 字符串：按字符编码排序（如 `MAX('a') < MAX('b')`）；
  - 日期：按时间先后排序（如 `MAX(create_time)` 是最新时间）；
- 示例：
  ```sql
  SELECT MAX(price) AS max_price,  -- 商品最高价格
         MIN(create_time) AS earliest_time -- 最早创建时间
  FROM products;
  ```
- 注意：自动忽略 `NULL` 值，若列全是 `NULL`，返回 `NULL`。

###### 5. 其他常用聚合函数（面试次要，了解即可）

- `GROUP_CONCAT()`：将分组内的字段值拼接为字符串（MySQL 特有）；
  示例：`SELECT dept, GROUP_CONCAT(name SEPARATOR ',') AS emp_names FROM employees GROUP BY dept;`（按部门拼接员工姓名）；
- `STDDEV()` / `VAR()`：求标准差、方差（统计类场景用）。

##### 三、聚合函数的使用规则（面试高频考点）

###### 1. 必须遵循「GROUP BY 子句规则」（SQL 标准）

- 核心原则：`SELECT` 中出现的**非聚合列**（即未被聚合函数包裹的列），必须出现在 `GROUP BY` 中；
- 示例（正确）：
  ```sql
  SELECT dept, COUNT(*) AS emp_count -- dept 是非聚合列，出现在 GROUP BY 中
  FROM employees
  GROUP BY dept;
  ```
- 错误示例（会报错，除非数据库关闭严格模式）：
  ```sql
  SELECT dept, name, COUNT(*) AS emp_count -- name 是非聚合列，未在 GROUP BY 中
  FROM employees
  GROUP BY dept;
  ```
- 面试延伸：MySQL 为什么默认允许“非聚合列出现在 SELECT 但不在 GROUP BY”？  
  答：MySQL 有 `ONLY_FULL_GROUP_BY` 模式（默认开启），关闭后会随机返回分组内的一条非聚合列值，这是不符合 SQL 标准的，可能导致数据不一致，生产环境必须开启该模式。

###### 2. 与 `HAVING` 搭配过滤聚合结果

- 问题：`WHERE` 不能过滤聚合结果（如“筛选员工数 > 10 的部门”），因为 `WHERE` 是在聚合前过滤行；
- 解决方案：用 `HAVING` 子句（聚合后过滤分组）；
- 示例：
  ```sql
  SELECT dept, COUNT(*) AS emp_count
  FROM employees
  GROUP BY dept
  HAVING emp_count > 10; -- 过滤聚合后的结果（部门人数>10）
  ```

###### 3. 与 `DISTINCT` 结合（去重后聚合）

- 语法：`聚合函数(DISTINCT 列名)`，仅对该列去重后再聚合；
- 示例：`SELECT AVG(DISTINCT score) AS avg_distinct_score FROM students;`（去重后计算平均分）；
- 注意：`DISTINCT` 仅作用于括号内的列，而非整个聚合函数。

##### 四、聚合函数的底层逻辑（面试加分项）

1. **执行顺序**：SQL 执行时，聚合函数在「分组（GROUP BY）」之后、「排序（ORDER BY）」之前执行；
   完整执行顺序：`FROM → WHERE → GROUP BY → 聚合函数 → HAVING → SELECT → ORDER BY → LIMIT`；
2. **NULL 处理**：除 `COUNT(*)` 外，所有聚合函数都会忽略 `NULL` 值（不会把 `NULL` 当作 0 或空字符串）；
3. **性能优化**：
   - 聚合函数依赖索引（尤其是 `GROUP BY + 聚合` 场景），建议创建「分组列 + 聚合列」的联合索引（如 `idx_dept_salary (dept, salary)`），避免全表扫描；
   - `COUNT(*)` 是效率最高的聚合函数（InnoDB 直接读取表的行数统计，无需扫描数据），优先使用 `COUNT(*)` 而非 `COUNT(列名)`。

##### 五、典型面试场景题（学以致用）

1. 题目：查询每个部门的员工数、平均工资、最高工资，且只保留平均工资 > 8000 的部门，按平均工资降序排序。
   答案：
   ```sql
   SELECT dept,
          COUNT(*) AS emp_count,
          ROUND(AVG(salary), 2) AS avg_salary,
          MAX(salary) AS max_salary
   FROM employees
   GROUP BY dept
   HAVING avg_salary > 8000
   ORDER BY avg_salary DESC;
   ```

2. 题目：统计用户表中，邮箱非空且不重复的用户数，以及所有用户的注册时间范围（最早和最晚）。
   答案：
   ```sql
   SELECT COUNT(DISTINCT email) AS unique_valid_email,
          MIN(register_time) AS earliest_register,
          MAX(register_time) AS latest_register
   FROM users;
   ```

##### 总结（面试收尾核心）

聚合函数的核心是「多行汇总为单行结果」，常用函数有 `COUNT`（重点）、`SUM`、`AVG`、`MAX`、`MIN`；使用时需遵循 `GROUP BY` 规则，用 `HAVING` 过滤聚合结果；底层要注意 `NULL` 处理和索引优化，面试中尤其要掌握 `COUNT(*)` 与 `COUNT(列名)` 的区别、`GROUP BY` 语法规范等高频考点。

### 5. **条件函数**

用于根据条件执行不同的操作。

- `IF(condition, true_value, false_value)`: 如果条件为真，返回 `true_value`，否则返回 `false_value`。
- `CASE WHEN condition THEN result ELSE default END`: 类似于 `if` 语句，处理复杂的条件表达式。
- `IFNULL()` 是 MySQL 中的一个条件函数，用于检查一个表达式是否为 `NULL`，如果是，则返回指定的替代值；否则返回该表达式的值。这个函数通常用于在查询中替代 `NULL` 值，避免出现空值，确保结果中有有效的数据。
- `COALESCE` 是 SQL 中一个非常实用的函数，用于**返回参数列表中第一个非 NULL 的值**。

#### IFNULL

##### 语法

  ```sql
  IFNULL(expression, alt_value)
  ```

  - **expression**: 要检查是否为 `NULL` 的表达式或字段。
  - **alt_value**: 如果 `expression` 为 `NULL`，则返回此值；否则，返回 `expression` 的值。

##### 示例

  1. **处理 `NULL` 值**
      假设有一个 `employees` 表，包含 `salary` 字段，其中某些员工的薪水可能是 `NULL`，表示他们没有薪水信息。我们可以使用 `IFNULL()` 来将 `NULL` 转换为 0，方便进行统计。

     ```sql
     SELECT name, IFNULL(salary, 0) AS salary
     FROM employees;
     --------------------
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
     BEGIN
 SET N=N-1;
        RETURN (
            # Write your MySQL query statement below.
           ifnull(  # ifnull里面的expression要加括号
             (SELECT DISTINCT salary
              from employee   
              order by salary desc
              limit 1 offset N),null)
        );
      END
      
     ```
     
      在这个查询中，如果 `salary` 为 `NULL`，则返回 0。
     
  2. **计算总薪水**
      假设我们想计算所有员工的薪水总和，考虑到某些薪水值可能是 `NULL`，我们可以使用 `IFNULL()` 来避免计算错误：

     ```sql
     SELECT SUM(IFNULL(salary, 0)) AS total_salary
     FROM employees;
     ```

  3. **替代 `NULL` 值为自定义信息**
      如果你希望在查询结果中将 `NULL` 值替换为某个具体信息，比如 "未提供"：

     ```sql
     SELECT name, IFNULL(salary, '未提供') AS salary_info
     FROM employees;
     ```

##### `IFNULL()` vs `COALESCE()`

  `IFNULL()` 只接受两个参数，而 `COALESCE()` 可以接受多个参数，并返回第一个非 `NULL` 的值。

  例如：

  ```sql
  SELECT COALESCE(salary, bonus, 0) AS final_payment
  FROM employees;
  ```

  这个查询会检查 `salary` 和 `bonus`，返回第一个非 `NULL` 的值，如果两者都为 `NULL`，则返回 0。

##### 总结

  `IFNULL()` 是一个简单但非常有用的函数，可以帮助我们在 MySQL 查询中处理 `NULL` 值。通过适当使用 `IFNULL()`，可以确保查询结果中的数据更加清晰和一致。



#### `COALESCE` 

`COALESCE` 是 SQL 中一个非常实用的函数，用于**返回参数列表中第一个非 NULL 的值**。

---

##### 🔹 语法：

```sql
COALESCE(value1, value2, ..., valueN)
```

- 从左到右依次检查每个参数。
- 返回**第一个不为 `NULL` 的值**。
- 如果所有参数都是 `NULL`，则返回 `NULL`。
- 至少需要提供 **一个参数**（大多数数据库要求至少两个，但标准 SQL 允许一个；实践中通常用两个或更多）。

---

##### 🔹 示例：

###### 示例 1：基本用法

```sql
SELECT COALESCE(NULL, 'hello', 'world');  -- 返回 'hello'
SELECT COALESCE(NULL, NULL, 42);          -- 返回 42
SELECT COALESCE(NULL, NULL, NULL);        -- 返回 NULL
```

###### 示例 2：处理缺失数据

假设有一个用户表，可能 `phone` 或 `email` 为空，你想显示一个联系方式：

```sql
SELECT name,
       COALESCE(phone, email, 'No contact info') AS contact
FROM users;
```

- 如果 `phone` 不为 NULL，就用 `phone`；
- 否则用 `email`；
- 如果两者都为 NULL，就显示 `'No contact info'`。

###### 示例 3：替代 IFNULL 或 ISNULL

在 MySQL 中，`IFNULL(a, b)` 等价于 `COALESCE(a, b)`，但 `COALESCE` 是 **SQL 标准函数**，兼容性更好（适用于 PostgreSQL、SQL Server、Oracle、SQLite 等）。

```sql
-- MySQL
SELECT IFNULL(email, 'N/A');

-- 标准 SQL（推荐）
SELECT COALESCE(email, 'N/A');
```

> ✅ 优势：`COALESCE` 支持**多个参数**，而 `IFNULL` 只支持两个。

---

##### 🔹 与 `CASE` 的关系

`COALESCE(expr1, expr2, ..., exprN)` 在逻辑上等价于：

```sql
CASE
    WHEN expr1 IS NOT NULL THEN expr1
    WHEN expr2 IS NOT NULL THEN expr2
    ...
    ELSE exprN
END
```

---

##### 🔹 注意事项：

- 所有参数应兼容数据类型（例如，不能混合 `DATE` 和 `VARCHAR` without casting，否则可能报错或隐式转换）。
- 在性能上，`COALESCE` 是轻量级的，但避免在 `WHERE` 子句中对列使用它（可能影响索引使用）。

---

##### ✅ 总结：

| 场景 | 推荐写法 |
|------|--------|
| 替代多个嵌套 `IFNULL` | `COALESCE(a, b, c, 'default')` |
| 跨数据库兼容 | 优先用 `COALESCE` 而不是 `IFNULL`/`ISNULL` |
| 返回首个有效值 | `COALESCE` 是最简洁的方式 |

> 💡 记住口诀：**“COALESCE 找第一个不为 NULL 的”**。

在 MySQL 中，**条件表达式（Conditional Expressions）** 是用于根据特定条件返回不同结果的表达式，主要用于实现逻辑判断和分支选择，类似于编程语言中的条件语句（如 `if-else`）。它们广泛应用于 `SELECT`、`WHERE`、`ORDER BY`、`UPDATE` 等子句中，用于动态生成结果或筛选数据。

#### 一、MySQL 中常用的条件表达式

##### 1. `CASE` 表达式（最常用）

`CASE` 是最灵活的条件表达式，支持多分支判断，有两种语法形式：

`CASE`的`THEN`子句中是不允许的。`CASE`表达式应该直接返回值，而不是进行赋值操作。

###### （1）简单 `CASE` 表达式（匹配值）

```sql
CASE 表达式
    WHEN 值1 THEN 结果1
    WHEN 值2 THEN 结果2
    ...
    ELSE 默认结果
END
```

- **逻辑**：将 `CASE` 后的“表达式”与每个 `WHEN` 后的“值”比较，返回第一个匹配的 `THEN` 结果；无匹配则返回 `ELSE` 结果（若省略 `ELSE`，则返回 `NULL`）。

**示例**：根据分数评级

```sql
SELECT 
    score,
    CASE score
        WHEN 100 THEN '满分'
        WHEN 90 THEN '优秀'
        ELSE '其他'
    END AS rating
FROM exam;
```

###### （2）搜索 `CASE` 表达式（匹配条件）

```sql
CASE
    WHEN 条件1 THEN 结果1
    WHEN 条件2 THEN 结果2
    ...
    ELSE 默认结果
END
```

- **逻辑**：依次判断每个 `WHEN` 后的“条件”，返回第一个为 `TRUE` 的 `THEN` 结果；无匹配则返回 `ELSE` 结果。
- **适用场景**：需要复杂条件判断（如范围、多字段组合）时使用。

**示例**：根据年龄分组

```sql
SELECT 
    name,
    age,
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age BETWEEN 18 AND 60 THEN '成年'
        ELSE '老年'
    END AS age_group
FROM users;
```

##### 2. `IF()` 函数（简化的双分支判断）

`IF()` 是 MySQL 扩展的函数，用于简单的“二选一”判断，语法：

```sql
IF(条件, 结果1, 结果2)
```

- **逻辑**：若“条件”为 `TRUE`，返回 `结果1`；否则返回 `结果2`。

**示例**：判断学生是否及格

```sql
SELECT 
    name,
    score,
    IF(score >= 60, '及格', '不及格') AS result
FROM exam;
```

##### 3. `IFNULL()` 函数（处理 `NULL` 值）

`IFNULL()` 用于判断值是否为 `NULL`，语法：

```sql
IFNULL(表达式, 替代值)
```

- **逻辑**：若“表达式”的值为 `NULL`，返回“替代值”；否则返回表达式本身的值。

**示例**：用 `0` 替代 `NULL` 的成绩

```sql
SELECT 
    name,
    IFNULL(score, 0) AS score  -- 若score为NULL，显示0
FROM exam;
```

##### 4. `NULLIF()` 函数（值相等则返回 `NULL`）

`NULLIF()` 用于比较两个值，语法：

```sql
NULLIF(表达式1, 表达式2)
```

- **逻辑**：若“表达式1”等于“表达式2”，返回 `NULL`；否则返回“表达式1”。

**示例**：若分数为 `0` 则显示 `NULL`

```sql
SELECT 
    name,
    NULLIF(score, 0) AS score  -- 若score=0，显示NULL
FROM exam;
```

#### 二、条件表达式的使用场景

1. **在 `SELECT` 中生成动态列**  
   如前面的示例，根据字段值动态生成分类（如评级、分组）。

2. **在 `WHERE` 中筛选复杂条件**  
   结合逻辑运算符实现多条件筛选：

   ```sql
   -- 筛选“成年且分数及格”或“未成年但分数优秀”的学生
   SELECT * FROM students
   WHERE 
       CASE
           WHEN age >= 18 THEN score >= 60
           ELSE score >= 90
       END;
   ```

3. **在 `ORDER BY` 中自定义排序**  
   按自定义规则排序（如先显示“优秀”，再显示“及格”）：

   ```sql
   SELECT name, score FROM exam
   ORDER BY 
       CASE
           WHEN score >= 90 THEN 1
           WHEN score >= 60 THEN 2
           ELSE 3
       END;
   ```

4. **在 `UPDATE` 中动态更新值**  
   根据条件更新不同字段：

   ```sql
   -- 对不同分数段的学生更新不同的奖励
   UPDATE students
   SET reward = CASE
       WHEN score >= 90 THEN 100
       WHEN score >= 60 THEN 50
       ELSE 0
   END;
   ```

#### 三、注意事项

1. **返回值类型一致性**  
   `CASE` 或 `IF()` 的所有可能返回值应保持兼容的类型（如都为字符串或数值），否则可能导致隐式类型转换，影响结果或性能。

2. **短路求值**  
   条件表达式采用“短路逻辑”：一旦某个 `WHEN` 条件满足，后续分支将不再判断，因此需注意分支顺序（范围小的条件放前面）。

3. **与函数的区别**  
   - `CASE` 是**表达式**，不属于函数，无需括号包裹整体条件。
   - `IF()`、`IFNULL()`、`NULLIF()` 是**函数**，需严格按函数语法调用（参数用括号）。

#### 总结

MySQL 的条件表达式（尤其是 `CASE` 和 `IF()`）是实现动态逻辑判断的核心工具，灵活运用它们可以处理各种复杂的业务场景，如数据分类、动态筛选、自定义排序等。根据场景复杂度选择合适的表达式（简单场景用 `IF()`，多分支用 `CASE`）能让代码更简洁高效。

### 6. **转换函数**

用于转换数据类型或处理不同数据格式。

- `CAST(expression AS target_type)`: 将表达式转换为指定类型。
- `CONVERT(expression, target_type)`: 与 `CAST` 类似，用于类型转换。

### 7. **信息性函数**

提供数据库、表和字段的元数据信息。

- `DATABASE()`: 返回当前数据库的名称。
- `USER()`: 返回当前数据库用户。
- `VERSION()`: 返回 MySQL 版本。
- `LAST_INSERT_ID()`: 返回最后插入的自动增长字段的值。

### 8. **流控制函数**

用于在 SQL 查询中实现控制流。

- `LIMIT`: 限制返回结果的行数。
- `SLEEP(seconds)`: 暂停指定的秒数。

### 9. **加密函数**

用于加密和解密数据。

- `MD5(str)`: 返回字符串的 MD5 哈希值。
- `SHA1(str)`: 返回字符串的 SHA1 哈希值。
- `AES_ENCRYPT(str, key)`: 使用 AES 算法加密字符串。
- `AES_DECRYPT(str, key)`: 解密加密的字符串。

### 10. **正则表达式函数**

用于正则表达式匹配。

- `REGEXP`: 用于在查询条件中进行正则匹配。
- `REGEXP_REPLACE()`: 替换匹配正则表达式的内容。
- `REGEXP_LIKE()`: 检查字符串是否匹配正则表达式。

#### `REGEXP` 基本用法

在 MySQL 中，`REGEXP` 是用于正则表达式匹配的运算符，用于判断字符串是否包含符合指定正则模式的子串。它的功能与 `REGEXP_LIKE` 类似，但语法上略有不同（`REGEXP` 是运算符，`REGEXP_LIKE` 是函数）。

##### 基本语法

```sql
string REGEXP pattern
```

- **`string`**：要检查的字符串（通常是表中的字段）。
- **`pattern`**：正则表达式模式，用于定义匹配规则。

返回结果：`1`（匹配成功）、`0`（匹配失败）或 `NULL`（有参数为 `NULL` 时）。

##### 核心特点

1. **默认行为**：
   - 不区分大小写（与 `REGEXP_LIKE` 不同，后者默认区分大小写）。若需区分大小写，可使用 `REGEXP BINARY`。
   - 匹配子串（即只要字符串中包含符合模式的部分就会返回 `1`，除非用 `^` 和 `$` 限制开头和结尾）。

2. **与 `REGEXP_LIKE` 的区别**：
   - `REGEXP` 是运算符，`REGEXP_LIKE` 是函数。
   - `REGEXP` 默认不区分大小写，`REGEXP_LIKE` 默认区分大小写。
   - `REGEXP_LIKE` 支持 `match_type` 参数（如 `'i'`、`'c'`），`REGEXP` 需通过 `BINARY` 等关键字控制大小写。

##### 示例

假设有表 `Patients`，字段 `conditions` 存储患者病症，筛选包含 `DIAB1`（糖尿病相关）的记录：

```sql
-- 匹配包含 DIAB1 的记录（不区分大小写，默认）
SELECT *
FROM Patients
WHERE conditions REGEXP 'DIAB1';

-- 区分大小写匹配（仅匹配 DIAB1，不匹配 diab1 等）
SELECT *
FROM Patients
WHERE conditions REGEXP BINARY 'DIAB1';

-- 精确匹配：DIAB1 要么是开头，要么前面有空格（避免匹配类似 XDIAB1 的子串）
SELECT *
FROM Patients
WHERE conditions REGEXP '^DIAB1|\\sDIAB1';
```

##### 常用场景

1. **模糊搜索**：匹配包含特定子串的记录（如搜索包含 "itis" 的病症）。

   ```sql
   WHERE conditions REGEXP 'itis';
   ```

2. **格式验证**：检查字符串是否符合特定格式（如手机号、邮箱）。

   ```sql
   -- 匹配以 1 开头的 11 位手机号
   WHERE phone REGEXP '^1[0-9]{10}$';
   ```

3. **多条件匹配**：用 `|` 匹配多个模式中的任意一个。

   ```sql
   -- 匹配包含 DIAB1 或 HYP1 的记录
   WHERE conditions REGEXP 'DIAB1|HYP1';
   ```

##### 注意事项

1. 正则中的特殊字符（如 `.`、`*`、`+` 等）需要用 `\\` 转义（MySQL 中需双重转义）。

   ```sql
   -- 匹配包含 "a.b" 的字符串（点是普通字符）
   WHERE col REGEXP 'a\\.b';
   ```

2. `^` 表示字符串开头，`$` 表示结尾，若省略则匹配任意位置的子串。

   ```sql
   'abc' REGEXP '^a' → 1（以 a 开头）
   'abc' REGEXP 'c$' → 1（以 c 结尾）
   'abc' REGEXP 'b' → 1（包含 b）
   ```

3. 若需不区分大小写，`REGEXP` 默认支持；若需区分大小写，需加 `BINARY`。

`REGEXP` 是 MySQL 中处理灵活字符串匹配的常用工具，结合正则语法可实现复杂的筛选逻辑。

#### `REGEXP_LIKE` 基本用法

`REGEXP_LIKE` 是 MySQL 中用于正则表达式匹配的函数，用于判断字符串是否符合指定的正则表达式模式。它通常用于 `WHERE` 子句中，筛选出满足正则匹配条件的记录。

##### 基本语法

```sql
REGEXP_LIKE(string, pattern[, match_type])
```

- **`string`**：要检查的字符串（通常是表中的某个字段）。
- **`pattern`**：正则表达式模式，用于定义匹配规则。
- **`match_type`**（可选）：匹配模式参数，用于指定匹配方式，常见取值：
  - `'c'`：区分大小写（默认值）。
  - `'i'`：不区分大小写。
  - `'n'`：允许 `.` 匹配换行符。
  - `'m'`：将字符串视为多行（影响 `^` 和 `$` 的匹配范围，默认是匹配整个字符串开头/结尾，`'m'` 下匹配每行开头/结尾）。

##### 作用

判断 `string` 是否与 `pattern` 匹配，返回 `1`（匹配）或 `0`（不匹配）。

##### 示例

假设有表 `Users`，字段 `mail` 存储邮箱地址，筛选出以 `@leetcode.com` 结尾的邮箱：

```sql
SELECT *
FROM Users
WHERE REGEXP_LIKE(mail, '@leetcode\\.com$'); -- 注意 `.` 需要转义为 `\\.`

SELECT *
FROM Users
WHERE regexp_like(mail, "^[a-zA-Z][a-zA-Z0-9\\._-]*@leetcode\\.com$",'c') // 这里的c不能省略，否则可能遗漏某些特殊情况
```

再比如，筛选出以字母开头、仅包含字母和数字的用户名（长度至少1）：

```sql
SELECT *
FROM Users
WHERE REGEXP_LIKE(username, '^[a-zA-Z][a-zA-Z0-9]*$');
```

##### 注意事项

1. 正则表达式中的特殊字符（如 `.`、`*`、`+`、`?`、`^`、`$` 等）需要用 `\` 转义，但在 MySQL 中需写为 `\\`（因为 `\` 本身在字符串中是转义符）。

2. 默认区分大小写，如需忽略大小写，可添加 `'i'` 参数，例如：

   ```sql
   REGEXP_LIKE(mail, '@leetcode\\.com$', 'i') -- 不区分大小写匹配
   ```

3. `^` 表示字符串开头，`$` 表示字符串结尾，若省略则匹配任意包含该模式的子串。

`REGEXP_LIKE` 是处理复杂字符串匹配场景的强大工具，灵活运用正则表达式可实现精准的筛选逻辑。

正则表达式（Regular Expression）是一种用于匹配字符串模式的工具，广泛应用于文本搜索、验证、替换等场景。以下是常用的正则语法规则，适用于大多数编程语言（包括MySQL、Python、JavaScript等）：

#### **一、基础匹配符**

| 符号       | 含义                                   | 示例                      |
| ---------- | -------------------------------------- | ------------------------- |
| `普通字符` | 匹配自身（如字母、数字、标点）         | `abc` 匹配 "abc"          |
| `.`        | 匹配任意单个字符（除换行符，除非指定） | `a.c` 匹配 "abc"、"a1c"   |
| `\`        | 转义字符，匹配特殊符号本身             | `a\.c` 匹配 "a.c"（仅点） |

#### **二、量词（重复匹配）**

| 符号    | 含义                                | 示例                                |
| ------- | ----------------------------------- | ----------------------------------- |
| `*`     | 匹配前面的元素 **0次或多次**        | `ab*` 匹配 "a"、"ab"、"abb"         |
| `+`     | 匹配前面的元素 **1次或多次**        | `ab+` 匹配 "ab"、"abb"（不匹配"a"） |
| `?`     | 匹配前面的元素 **0次或1次**         | `ab?` 匹配 "a"、"ab"                |
| `{n}`   | 匹配前面的元素 **恰好n次**          | `a{2}` 匹配 "aa"                    |
| `{n,}`  | 匹配前面的元素 **至少n次**          | `a{2,}` 匹配 "aa"、"aaa"            |
| `{n,m}` | 匹配前面的元素 **n到m次**（含边界） | `a{1,3}` 匹配 "a"、"aa"、"aaa"      |

#### **三、字符集（范围匹配）**

| 符号          | 含义                                   | 示例                          |
| ------------- | -------------------------------------- | ----------------------------- |
| `[abc]`       | 匹配 `a`、`b` 或 `c` 中的任意一个      | `[abc]` 匹配 "a"、"b"         |
| `[^abc]`      | 匹配 **除了** `a`、`b`、`c` 的任意字符 | `[^abc]` 匹配 "d"、"1"        |
| `[a-z]`       | 匹配小写字母（a到z）                   | `[a-z]` 匹配 "x"、"y"         |
| `[A-Z]`       | 匹配大写字母（A到Z）                   | `[A-Z]` 匹配 "X"、"Y"         |
| `[0-9]`       | 匹配数字（0到9）                       | `[0-9]` 匹配 "3"、"7"         |
| `[a-zA-Z0-9]` | 匹配字母或数字                         | 等价于 `\w`（见下方预定义类） |

#### **四、预定义字符类（简化写法）**

| 符号 | 含义                                 | 等价写法         |
| ---- | ------------------------------------ | ---------------- |
| `\d` | 匹配任意数字                         | `[0-9]`          |
| `\D` | 匹配非数字                           | `[^0-9]`         |
| `\w` | 匹配字母、数字、下划线（单词字符）   | `[a-zA-Z0-9_]`   |
| `\W` | 匹配非单词字符（如标点、空格）       | `[^a-zA-Z0-9_]`  |
| `\s` | 匹配空白字符（空格、制表符、换行等） | `[ \t\n\r\f\v]`  |
| `\S` | 匹配非空白字符                       | `[^ \t\n\r\f\v]` |

#### **五、边界匹配**

| 符号 | 含义                                        | 示例                                    |
| ---- | ------------------------------------------- | --------------------------------------- |
| `^`  | 匹配字符串的 **开头**（多行模式下匹配行首） | `^abc` 匹配 "abc123"（以abc开头）       |
| `$`  | 匹配字符串的 **结尾**（多行模式下匹配行尾） | `xyz$` 匹配 "123xyz"（以xyz结尾）       |
| `\b` | 匹配单词边界（单词与非单词的分隔处）        | `\bcat\b` 匹配 "cat"，不匹配 "category" |
| `\B` | 匹配非单词边界                              | `\Bcat\B` 匹配 "category" 中的 "cat"    |

#### **六、逻辑与分组**

| 符号      | 含义                                 | 示例                               |
| --------- | ------------------------------------ | ---------------------------------- |
| `|`       | 逻辑或，匹配左边或右边的模式         | `abc|def` 匹配 "abc" 或 "def"      |
| `(...)`   | 分组，将多个元素视为一个整体         | `(ab)+` 匹配 "ab"、"abab"          |
| `(?:...)` | 非捕获分组（仅分组，不保存匹配结果） | `(?:ab)+` 功能同 `(ab)+`，但不捕获 |

#### **七、贪婪与非贪婪匹配**

- **贪婪匹配**（默认）：尽可能匹配最长的字符串。  
  例如：`a.*b` 匹配 "a123b456b" 中的 "a123b456b"。
- **非贪婪匹配**：尽可能匹配最短的字符串（量词后加 `?`）。  
  例如：`a.*?b` 匹配 "a123b456b" 中的 "a123b"。

##### **示例：常见正则场景**

1. **匹配邮箱**（简化版）：  
   `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`  
   解析：用户名（字母、数字、特殊符号）+ @ + 域名 + . + 后缀（至少2个字母）。

2. **匹配手机号**（中国大陆）：  
   `^1[3-9]\d{9}$`  
   解析：以1开头，第2位是3-9，后面跟9位数字，共11位。

3. **匹配URL**（简化版）：  
   `^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`  
   解析：http或https开头， followed by 域名和后缀。

##### 注意事项

- 不同语言/工具的正则细节可能略有差异（如MySQL中需用 `\\` 转义，而JavaScript用 `\`）。
- 复杂正则建议结合工具调试（如 [Regex101](https://regex101.com/)）。

掌握这些基础语法，就能应对大多数字符串匹配需求啦！

### 11. **JSON 函数**

用于操作 JSON 数据类型。

- `JSON_EXTRACT(json_doc, path)`: 从 JSON 文档中提取数据。
- `JSON_SET(json_doc, path, value)`: 修改 JSON 文档中的值。
- `JSON_ARRAYAGG(expr)`: 聚合结果为 JSON 数组。

### 12. **窗口函数**

用于在查询中进行复杂的分组和排序分析。

- `ROW_NUMBER()`: 返回结果集中的行号。
- `RANK()`: 为结果集中的行分配排名。
- `LEAD()`, `LAG()`: 获取当前行的后续或前续数据。

#### 窗口函数

窗口函数（Window Function）是 MySQL 8.0+ 引入的核心特性之一，它能够在不改变结果集行数的前提下，对数据进行**分组、排序、聚合或计算排名**，解决了传统 `GROUP BY` 分组后只能返回聚合结果（无法保留原始行细节）的痛点。

##### 一、窗口函数的核心概念

窗口函数的本质是“在一个**数据窗口**（即指定的数据集范围）内对每行数据进行计算”，关键特点：
1. **不减少行数**：与 `GROUP BY` 分组后只返回每组1行不同，窗口函数会为原始表的**每一行**都返回一个计算结果；
2. **自定义窗口范围**：可通过语法指定计算的“窗口”（如“当前行的前3行到后1行”“同分组内的所有行”）；
3. **支持复杂计算**：可同时实现排序、排名、聚合（如求和、平均值）等逻辑，无需嵌套子查询。

##### 二、窗口函数的基本语法

```sql
窗口函数名(参数) OVER (
    [PARTITION BY 分组字段1, 分组字段2, ...]  -- 可选：按字段分组，组内独立计算
    [ORDER BY 排序字段1 排序方向, 排序字段2 排序方向, ...]  -- 可选：组内数据排序
    [ROWS/RANGE BETWEEN 边界条件1 AND 边界条件2]  -- 可选：定义窗口的具体范围（默认是组内所有行）
) AS 结果别名
```

###### 各部分说明：

1. **窗口函数名**：MySQL 提供的内置窗口函数（如排名函数、聚合函数等）；
2. **`OVER ()`**：窗口函数的标志，所有窗口逻辑都在括号内定义；
3. **`PARTITION BY`（可选）**：类似“分组”，将原始数据按指定字段分成多个独立的“窗口”，每个窗口内单独计算（无此子句则整个表视为一个窗口）；
4. **`ORDER BY`（可选）**：对每个“窗口”内的数据按指定字段排序（常用于排名、累计计算场景）；
5. **`ROWS/RANGE`（可选）**：精确控制窗口的范围（如“当前行及前2行”“小于当前值的所有行”），默认是“当前窗口内的所有行”。

##### 三、窗口函数的分类

MySQL 中的窗口函数主要分为 **专用窗口函数** 和 **聚合窗口函数** 两类。

###### 1. 专用窗口函数（常用于排名、行定位）

这类函数是窗口函数特有的，专门处理排序和排名逻辑，常用的有3个：

| 函数名          | 作用说明                                                                 |
|-----------------|--------------------------------------------------------------------------|
| `RANK()`        | 并列排名，排名会“跳号”（例：两个第1名后，下一个是第3名）                 |
| `DENSE_RANK()`  | 并列排名，排名“不跳号”（例：两个第1名后，下一个是第2名）                 |
| `ROW_NUMBER()`  | 不考虑并列，为每行分配唯一的连续排名（例：即使分数相同，也按顺序排1、2） |
| `NTILE(n)`      | 将窗口内的数据平均分成 `n` 组，返回每行所在的组号（用于数据分桶）        |

**示例**：用3个排名函数对比效果  
假设 `Scores` 表数据：
| score |
|-------|
| 90    |
| 80    |
| 90    |
| 75    |

查询语句：
```sql
SELECT 
  score,
  RANK() OVER (ORDER BY score DESC) AS rank,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank,
  ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num
FROM Scores;
```

结果：
| score | rank | dense_rank | row_num |
|-------|------|------------|---------|
| 90    | 1    | 1          | 1       |
| 90    | 1    | 1          | 2       |
| 80    | 3    | 2          | 3       |
| 75    | 4    | 3          | 4       |

###### 2. 聚合窗口函数（用聚合函数做窗口计算）

将传统的聚合函数（如 `SUM()`、`AVG()`、`MAX()`）作为窗口函数使用，实现“每行都显示聚合结果”的效果（例如“显示每行数据及所在部门的平均工资”）。

常用的聚合窗口函数：`SUM()`、`AVG()`、`MAX()`、`MIN()`、`COUNT()`。

**示例**：计算每个分数及“当前分数及以上的总分数”  
查询语句：
```sql
SELECT 
  score,
  SUM(score) OVER (ORDER BY score DESC) AS total_above,  -- 累计求和（当前及以上分数的总和）
  AVG(score) OVER () AS avg_all  -- 所有分数的平均值（无ORDER BY，窗口是全表）
FROM Scores;
```

结果（基于上述 `Scores` 表）：
| score | total_above | avg_all |
|-------|-------------|---------|
| 90    | 90          | 83.75   |
| 90    | 180         | 83.75   |
| 80    | 260         | 83.75   |
| 75    | 335         | 83.75   |

##### 四、关键语法：`PARTITION BY` 分组

`PARTITION BY` 用于将数据按指定字段分组，每个分组内独立计算窗口函数，相当于“多个独立的小窗口”。

**示例**：按“部门”分组，计算每个员工的工资排名  
假设 `Employees` 表数据：
| dept  | name  | salary |
|-------|-------|--------|
| 技术部 | 张三  | 15000  |
| 技术部 | 李四  | 12000  |
| 销售部 | 王五  | 18000  |
| 销售部 | 赵六  | 16000  |

查询语句：
```sql
SELECT 
  dept,
  name,
  salary,
  DENSE_RANK() OVER (
    PARTITION BY dept  -- 按部门分组，每个部门内单独排名
    ORDER BY salary DESC
  ) AS dept_sal_rank
FROM Employees;
```

结果：
| dept  | name  | salary | dept_sal_rank |
|-------|-------|--------|---------------|
| 技术部 | 张三  | 15000  | 1             |
| 技术部 | 李四  | 12000  | 2             |
| 销售部 | 王五  | 18000  | 1             |
| 销售部 | 赵六  | 16000  | 2             |

##### 五、高级语法：`ROWS/RANGE` 定义窗口范围

默认情况下，窗口范围是“`PARTITION BY` 分组内、`ORDER BY` 排序后的所有行”，但可通过 `ROWS/RANGE` 精确控制范围（如“当前行的前N行到后M行”）。

###### 常用的边界条件：

- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`：从分组的第一行到当前行（默认的累计范围）；
- `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`：当前行及前2行（共3行）；
- `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`：当前行到分组的最后一行；
- `RANGE BETWEEN 10 PRECEDING AND 10 FOLLOWING`：基于值的范围（如“当前值±10”的所有行）。

**示例**：计算“当前行及前1行”的平均工资  
查询语句：
```sql
SELECT 
  name,
  salary,
  AVG(salary) OVER (
    ORDER BY salary DESC
    ROWS BETWEEN 1 PRECEDING AND CURRENT ROW  -- 窗口范围：前1行到当前行
  ) AS avg_2rows
FROM Employees;
```

结果（基于上述 `Employees` 表）：
| name  | salary | avg_2rows |
|-------|--------|-----------|
| 王五  | 18000  | 18000.00  | （只有当前行，无前行）
| 赵六  | 16000  | 17000.00  | （18000+16000）/2
| 张三  | 15000  | 15500.00  | （16000+15000）/2
| 李四  | 12000  | 13500.00  | （15000+12000）/2

##### 六、窗口函数的使用场景

1. **排名需求**：如“各部门工资Top3”“考试成绩排名”；
2. **累计计算**：如“月度销售额累计到当前月”“用户消费累计金额”；
3. **分组对比**：如“显示每个员工工资与部门平均工资的差值”；
4. **数据分桶**：如“将用户按消费金额分成5个等级”（用 `NTILE(5)`）；
5. **相邻行比较**：如“计算每个员工工资与上一个员工的工资差”（需结合 `LAG()` 函数，属于“偏移窗口函数”）。

##### 七、注意事项

1. **MySQL 版本限制**：窗口函数仅支持 MySQL 8.0 及以上版本，5.x 版本无法使用；
2. **与 `GROUP BY` 的区别**：
   - `GROUP BY`：分组后每行对应一个组，只返回聚合结果，**减少行数**；
   - 窗口函数：不减少行数，每行都保留原始数据，同时附加窗口计算结果；
3. **性能优化**：如果窗口范围大（如全表），建议对 `PARTITION BY` 和 `ORDER BY` 的字段建立索引，提升计算效率；
4. **NULL 值处理**：`ORDER BY` 时，NULL 值默认排在最后（升序）或最前（降序），需注意业务逻辑是否允许。

##### 总结

窗口函数是 MySQL 中处理复杂数据计算的强大工具，它通过“窗口”的概念，在保留原始数据的同时实现分组、排序、聚合等逻辑，比传统的子查询+聚合函数方案更简洁、高效。掌握 `RANK()`/`DENSE_RANK()`、`PARTITION BY`、聚合窗口函数这几个核心点，就能解决大多数数据分析场景的需求。

#### 窗口函数

窗口函数（Window Function）是 SQL 中用于数据分析的强大工具，它能在保留原始数据行结构的同时，对一组与当前行相关的行（称为“窗口”）进行计算。以下是窗口函数的详细介绍及核心关键字说明：

##### 一、窗口函数的基本结构

窗口函数的语法格式如下：
```sql
函数名(参数) OVER (
  [PARTITION BY 列名]  -- 可选：分组划分窗口
  [ORDER BY 列名 [ASC|DESC]]  -- 可选：窗口内排序
  [窗口范围定义]  -- 可选：指定窗口包含的行范围
)
```

- **函数名**：可以是聚合函数（如 `SUM()`、`AVG()`）或专用窗口函数（如 `RANK()`、`ROW_NUMBER()`）。
- **OVER()**：关键字，用于标识这是窗口函数，括号内定义“窗口”的规则。

##### 二、核心关键字及功能

###### 1. `OVER()`

- 窗口函数的标志，所有窗口函数都必须跟在 `OVER()` 后面。
- 括号内可定义窗口的分组、排序和范围，为空时表示对整个表的所有行计算（全局窗口）。

  示例：
  ```sql
  -- 计算所有行的平均值，每行都显示相同结果
  SELECT score, AVG(score) OVER() AS avg_all FROM students;
  ```

###### 2. `PARTITION BY`（分组）

- 功能：将数据按指定列划分为多个独立的“窗口”（类似 `GROUP BY` 的分组逻辑），每个分组内单独计算窗口函数。
- 与 `GROUP BY` 的区别：`PARTITION BY` 不会合并行，每个分组内的每行都会保留并显示计算结果。

  示例：
  ```sql
  -- 按班级分组，计算每个学生的成绩在本班的排名
  SELECT 
    class, 
    name, 
    score,
    RANK() OVER (PARTITION BY class ORDER BY score DESC) AS rank_in_class
  FROM students;
  ```

###### 3. `ORDER BY`（窗口内排序）

- 功能：指定当前窗口内数据的排序规则，影响窗口函数的计算逻辑（如累计求和、排名）。
- 注意：仅在窗口内生效，不影响整个查询结果的最终排序（最终排序需在外部用 `ORDER BY`）。

  示例：
  ```sql
  -- 按日期排序，计算每天的累计销售额
  SELECT 
    sale_date, 
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS cumulative_sum
  FROM sales;
  ```

###### 4. 窗口范围定义（`ROWS`/`RANGE` + 范围关键字）

用于精确指定窗口包含哪些行，需配合 `ORDER BY` 使用。核心关键字包括：

###### （1）`ROWS`：按物理行数定义范围

- 基于行的位置确定窗口，适用于有序的连续数据。
- 常用范围关键字：
  - `n PRECEDING`：当前行之前的第 `n` 行（`n` 为数字）（包含当前行）。
  - `n FOLLOWING`：当前行之后的第 `n` 行。
  - `CURRENT ROW`：当前行。
  - `UNBOUNDED PRECEDING`：窗口的第一行（从开始到当前行）。
  - `UNBOUNDED FOLLOWING`：窗口的最后一行（从当前行到结束）。

  示例：
  ```sql
  -- 计算当前行及前2行、后1行的总和（共4行）
  SELECT 
    value,
    SUM(value) OVER (
      ORDER BY id 
      ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING
    ) AS sum_range
  FROM numbers;
  ```

###### （2）`RANGE`：按逻辑值范围定义

- 基于列的数值或日期范围确定窗口，适用于非连续但有逻辑关联的数据（如日期、数值区间）。

- 常用场景：日期范围（如“近7天”）、数值区间（如“与当前值相差10以内”）。

- 

  示例：
  ```sql
  -- 计算当前日期及前6天的销售额总和（共7天）
  SELECT 
    sale_date,
    SUM(amount) OVER (
      ORDER BY sale_date 
      RANGE INTERVAL 6 DAY PRECEDING  -- 等价于 BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
    ) AS 7day_sum
  FROM sales;
  ```

###### 5. 专用窗口函数的关键字

除了聚合函数（`SUM`、`AVG` 等），SQL 还提供了专门用于窗口计算的函数，常见的有：

- `RANK()`：排名，存在并列时会跳过后续名次（如 1,1,3）。
- `DENSE_RANK()`：排名，存在并列时不跳过后续名次（如 1,1,2）。
- `ROW_NUMBER()`：为每行分配唯一序号，即使值相同也按顺序递增（如 1,2,3）。
- `FIRST_VALUE(列)`：返回窗口内第一行的指定列值。
- `LAST_VALUE(列)`：返回窗口内最后一行的指定列值。
- `LAG(列, n)`：返回当前行的前第 `n` 行的指定列值（默认 `n=1`）。
- `LEAD(列, n)`：返回当前行的后第 `n` 行的指定列值（默认 `n=1`）。

  示例：
  ```sql
  -- 获取每个学生的成绩及前一名学生的成绩
  SELECT 
    name, 
    score,
    LAG(score) OVER (ORDER BY score DESC) AS prev_score
  FROM students;
  ```



##### `INTERVAL` 

在 SQL 中，`INTERVAL` 是一个用于表示时间间隔的关键字，主要用于日期时间类型的计算，常与窗口函数的范围定义（`RANGE`）或日期函数（如 `DATE_ADD`、`DATEDIFF`）配合使用。

###### 作用

`INTERVAL` 用于指定一个时间跨度（如几天、几小时、几个月等），以便对日期或时间进行加减运算，或定义基于时间的窗口范围。

###### 语法格式

```sql
INTERVAL 数值 时间单位
```
其中，**时间单位**可以是：`DAY`（天）、`HOUR`（小时）、`MINUTE`（分钟）、`SECOND`（秒）、`MONTH`（月）、`YEAR`（年）等。

###### 常见用法

###### 1. 在窗口函数的 `RANGE` 中定义时间范围

与窗口函数结合时，`INTERVAL` 常用来定义基于时间的滑动窗口。例如：
```sql
-- 计算当前日期及前6天（共7天）的销售额总和
SUM(amount) OVER (
  ORDER BY sale_date
  RANGE INTERVAL 6 DAY PRECEDING  -- 范围：当前行日期及之前6天
) AS 7day_sum
```
这里 `INTERVAL 6 DAY` 表示“6天的时间间隔”，结合 `PRECEDING` 定义了窗口的时间范围。

###### 2. 日期加减运算

用于对日期进行偏移计算，例如：
```sql
-- 计算某个日期加3天的结果
SELECT DATE_ADD('2023-10-01', INTERVAL 3 DAY);  -- 结果：2023-10-04

-- 计算某个日期减1个月的结果
SELECT DATE_SUB('2023-10-01', INTERVAL 1 MONTH);  -- 结果：2023-09-01
```

###### 3. 过滤时间范围

在 `WHERE` 子句中筛选某个时间范围内的数据，例如：
```sql
-- 查询近30天内的订单
SELECT *
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```

###### 注意事项

- `INTERVAL` 必须与具体的时间单位搭配使用，不能单独使用。
- 不同数据库对 `INTERVAL` 的支持略有差异（如单位的写法、支持的时间类型等），但基本逻辑一致。
- 在窗口函数中，`INTERVAL` 通常只与 `RANGE` 配合（用于逻辑时间范围），而不与 `ROWS` 配合（`ROWS` 基于物理行数）。

总之，`INTERVAL` 是 SQL 中处理日期时间计算的核心关键字，尤其在需要按时间范围分析数据时（如滑动窗口统计）非常常用。

##### 三、窗口函数的典型应用场景

1. 滑动窗口计算（如近7天销售额、移动平均值）。
2. 累计指标（如年度累计销量、截至当前的用户总数）。
3. 分组内排名（如各部门工资排名、各班级成绩排名）。
4. 前后行数据对比（如与上月销售额对比、用户上次登录时间）。
5. 分组内的首尾值提取（如每个地区的首单/末单时间）。

##### 总结

窗口函数通过 `OVER()` 子句定义“窗口”，结合 `PARTITION BY` 分组、`ORDER BY` 排序以及 `ROWS`/`RANGE` 范围关键字，实现了灵活的数据分析能力。它的核心优势是**在保留原始数据行的同时进行聚合或排序计算**，避免了传统 `GROUP BY` 丢失细节数据的问题，是复杂报表和数据分析的必备工具。

#### `RANK()`、`DENSE_RANK()`和`ROW_NUMBER()`

在MySQL中，`RANK()`、`DENSE_RANK()`和`ROW_NUMBER()`都是用于对数据进行排序并分配序号的窗口函数，它们的核心区别在于**如何处理“并列排名”**的情况。以下通过具体案例说明三者的差异：

##### 一、基础数据准备

假设有一张`score`表，存储学生的成绩数据：

| student_id | subject | score |
|------------|---------|-------|
| 1          | 数学    | 90    |
| 2          | 数学    | 85    |
| 3          | 数学    | 90    |
| 4          | 数学    | 80    |
| 5          | 数学    | 85    |

##### 二、函数对比与结果分析

以“按数学成绩降序排名”为例，分别使用三个函数的结果如下：

###### 1. `ROW_NUMBER()`：不考虑并列，按顺序连续编号

**特点**：即使成绩相同，也会按顺序分配唯一的序号（无并列），编号连续且不重复。

```sql
SELECT 
  student_id, 
  score,
  ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num
FROM score;
```

**结果**：
| student_id | score | row_num |
|------------|-------|---------|
| 1          | 90    | 1       |
| 3          | 90    | 2       |  -- 与1号成绩相同，但序号+1
| 2          | 85    | 3       |
| 5          | 85    | 4       |  -- 与2号成绩相同，序号+1
| 4          | 80    | 5       |

###### 2. `RANK()`：允许并列，但会跳过重复序号

**特点**：成绩相同的行获得相同的排名，但下一个不同成绩的排名会“跳过中间的序号”（例如两个并列第1，下一个就是第3）。

```sql
SELECT 
  student_id, 
  score,
  RANK() OVER (ORDER BY score DESC) AS rnk
FROM score;
```

**结果**：
| student_id | score | rnk  |
|------------|-------|------|
| 1          | 90    | 1    |
| 3          | 90    | 1    |  -- 与1号并列第1
| 2          | 85    | 3    |  -- 跳过了第2（因前有2个并列第1）
| 5          | 85    | 3    |  -- 与2号并列第3
| 4          | 80    | 5    |  -- 跳过了第4（因前有2个并列第3）

###### 3. `DENSE_RANK()`：允许并列，且不跳过重复序号

**特点**：成绩相同的行获得相同的排名，下一个不同成绩的排名“连续递增”（例如两个并列第1，下一个就是第2）。

```sql
SELECT 
  student_id, 
  score,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rnk
FROM score;
```

**结果**：
| student_id | score | dense_rnk |
|------------|-------|-----------|
| 1          | 90    | 1         |
| 3          | 90    | 1         |  -- 与1号并列第1
| 2          | 85    | 2         |  -- 连续递增到第2（不跳过）
| 5          | 85    | 2         |  -- 与2号并列第2
| 4          | 80    | 3         |  -- 连续递增到第3

##### 三、核心区别总结

| 函数          | 处理并列的逻辑                     | 序号连续性       | 适用场景举例                     |
|---------------|------------------------------------|------------------|----------------------------------|
| `ROW_NUMBER()` | 不承认并列，强制分配唯一序号       | 连续（1,2,3,4...） | 需唯一序号的场景（如分页排序）   |
| `RANK()`       | 承认并列，但跳过重复序号           | 不连续（1,1,3,3...） | 竞技排名（如比赛成绩排名）       |
| `DENSE_RANK()` | 承认并列，且序号连续不跳过         | 连续（1,1,2,2...） | 等级划分（如按分数段划分ABC等级） |

##### 四、补充说明

- 三个函数均需与`OVER()`子句配合使用，`ORDER BY`指定排序规则（升序/降序）。
- 可通过`PARTITION BY`实现“分组排名”（例如：按科目分组后，在每个科目内单独排名）。

例如，按科目分组排名：
```sql
SELECT 
  student_id, 
  subject,
  score,
  RANK() OVER (PARTITION BY subject ORDER BY score DESC) AS subject_rank
FROM score;
```

####  `DENSE_RANK()`

这个 SQL 查询使用了 MySQL 中的 **窗口函数 `DENSE_RANK()`**，用于计算 `Scores` 表中每个分数的**密集排名**，最终按分数从高到低展示结果。它是实现“并列排名且排名连续”的简洁方案，属于 MySQL 8.0+ 支持的标准窗口函数语法。

##### 1. 核心组件解析

###### （1）窗口函数 `DENSE_RANK()`

`DENSE_RANK()` 是专门用于计算排名的窗口函数，核心规则：
- **并列排名**：相同的数值会获得相同的排名（例如两个 90 分都排第 1）；
- **连续排名**：排名不会跳过数字（即使有并列，下一个不同数值的排名仍连续，例如 90 分排 1，80 分直接排 2，不会出现“3”）。

###### （2）窗口子句 `OVER (ORDER BY S.score DESC)`

`OVER ()` 是窗口函数的标志，用于定义“窗口范围”（即函数作用的数据集合）：
- `ORDER BY S.score DESC`：指定窗口内的数据按 `score` 字段**降序排序**（从高到低），排名会基于这个排序结果计算；
- 这里没有额外的 `PARTITION BY`（分组子句），所以函数会对整个 `Scores` 表的所有数据统一计算排名（如果加 `PARTITION BY`，则会按分组分别排名，比如按“班级”分组后各班级内单独算排名）。

###### （3）主查询逻辑

```sql
SELECT S.score, ... FROM Scores S;
```
- 从 `Scores` 表（别名 `S`）中获取每个分数 `score`，并为每个分数附加对应的排名 `rank`。

##### 2. 示例验证

假设 `Scores` 表数据如下：

| score |
|-------|
| 90    |
| 80    |
| 90    |
| 75    |
| 80    |

###### 执行查询后的结果：

| score | rank |
|-------|------|
| 90    | 1    |
| 90    | 1    |
| 80    | 2    |
| 80    | 2    |
| 75    | 3    |

###### 结果分析：

1. 两个 90 分并列第 1（`DENSE_RANK()` 处理相同分数为同一排名）；
2. 接下来的两个 80 分并列第 2（排名连续，没有跳过“2”直接到“3”）；
3. 最后的 75 分排第 3（排名完全连续）。

##### 3. 关键特点与对比

###### （1）与你之前的“子查询计数排名”对比

| 维度                | `DENSE_RANK()` 窗口函数               | 子查询 `count(distinct s2.score)` 方式 |
|---------------------|---------------------------------------|---------------------------------------|
| 语法简洁性          | 简洁，一行完成排名逻辑                | 需嵌套子查询，逻辑较繁琐              |
| 性能                | 高效（MySQL 对窗口函数有优化）        | 数据量大时性能较差（子查询需逐行遍历）|
| 排名规则            | 天然支持密集排名，也可换 `RANK()`/`ROW_NUMBER()` | 仅能实现“密集排名”，需修改逻辑才能实现其他排名 |
| MySQL 版本要求      | 8.0+（仅支持窗口函数的版本）          | 所有版本兼容                          |

###### （2）与其他排名函数的区别

MySQL 中还有另外两个常用的排名窗口函数，需注意区分：
- `RANK()`：并列排名，但排名会“跳号”（例如两个 90 分排 1，下一个 80 分排 3）；
- `ROW_NUMBER()`：不考虑并列，即使分数相同也会分配不同的连续排名（例如两个 90 分分别排 1、2）；
- `DENSE_RANK()`：并列排名且排名连续（你当前查询使用的，最适合“相同分数同排名，排名不跳号”的场景）。

##### 4. 注意事项

1. **MySQL 版本限制**：`DENSE_RANK()` 是 MySQL 8.0 及以上版本才支持的窗口函数，如果你使用的是 5.x 版本，该查询会报错（需升级版本或改用子查询方式）；
2. **排名字段的 NULL 值**：如果 `score` 存在 NULL 值，`ORDER BY S.score DESC` 会将 NULL 排在最后（MySQL 中 NULL 视为“最小的值”）；
3. **别名 `rank` 的引号**：`rank` 是 MySQL 的**保留关键字**（虽然可以直接用，但建议加单引号 `'rank'` 或反引号 `` `rank` `` 避免语法冲突，你的查询中加了单引号是规范的）。

总结：这个查询通过 `DENSE_RANK()` 窗口函数，以最简洁高效的方式实现了“分数从高到低、并列连续排名”的需求，是 MySQL 8.0+ 中处理排名问题的推荐方案。

#### LEAD

这个 SQL 查询的作用是从 `Logs` 表中找出**连续出现至少三次**的数字，并返回这些数字（去重后）。它使用了窗口函数 `LEAD()` 来获取后续行的数据，从而判断是否存在连续出现的情况。

##### 1. 子查询：使用 `LEAD()` 窗口函数获取后续数据

子查询的核心是通过 `LEAD()` 函数获取当前行之后的数字，便于判断连续性：
```sql
SELECT
    id,
    num,
    LEAD(num, 1) OVER(ORDER BY id) AS next_num,  -- 获取下一行的num
    LEAD(num, 2) OVER(ORDER BY id) AS next_next_num  -- 获取下两行的num
FROM Logs
```

- **`LEAD(num, n)` 函数**：用于获取当前行之后第 `n` 行的 `num` 值（`n=1` 表示下一行，`n=2` 表示下两行）
- **`OVER(ORDER BY id)`**：指定按 `id` 排序，确保行的顺序是连续的（`id` 通常是自增主键，代表数据插入顺序）

假设 `Logs` 表数据如下：

| id  | num |
|-----|-----|
| 1   | 1   |
| 2   | 1   |
| 3   | 1   |
| 4   | 2   |
| 5   | 2   |
| 6   | 3   |

子查询的结果会是：

| id  | num | next_num | next_next_num |
|-----|-----|----------|---------------|
| 1   | 1   | 1        | 1             | （id=1的下两行都是1）
| 2   | 1   | 1        | 2             | （id=2的下一行是1，下两行是2）
| 3   | 1   | 2        | 2             | （id=3的下两行是2）
| 4   | 2   | 2        | 3             |
| 5   | 2   | 3        | NULL          | （没有下两行，返回NULL）
| 6   | 3   | NULL     | NULL          | （没有下两行，返回NULL）

##### 2. 外层查询：筛选连续出现三次的数字

外层查询通过 `WHERE` 条件判断当前行、下一行、下两行的数字是否相同：
```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (子查询) n
WHERE num = next_num AND next_num = next_next_num
```

- **条件 `num = next_num AND next_num = next_next_num`**：确保当前数字与后两个数字完全相同（即连续出现三次）
- **`DISTINCT`**：去重（如果同一数字连续出现超过3次，会产生重复结果，例如连续出现4次会匹配两次条件）

##### 3. 最终结果

基于上面的示例数据，满足条件的只有 `id=1` 的行（`num=1` 连续出现三次），因此最终结果：

| ConsecutiveNums |
|-----------------|
| 1               |

##### 关键逻辑总结

1. 用 `LEAD()` 函数“预览”后续行的数据，将分散的连续数据“拉到”同一行中比较
2. 通过简单的等式判断三次连续出现的条件
3. 用 `DISTINCT` 确保结果中每个数字只出现一次

这种方法比使用自连接（多次关联表）的方式更简洁高效，尤其适合判断“连续出现N次”的场景（只需调整 `LEAD()` 的参数即可扩展）。

#### `FIRST_VALUE` 和 `LAST_VALUE`

你的判断完全正确，`FIRST_VALUE` 和 `LAST_VALUE` 确实是 MySQL 8.0 及以上版本引入的**窗口函数**，核心作用是在指定的窗口范围内，获取第一个或最后一个数据行的指定列值。

要熟练使用这两个函数，需理解其“窗口范围”的核心逻辑，可从基础语法、功能差异、常见场景三个维度展开：

---

##### 1. 基础语法：固定结构与核心参数

两个函数的语法结构高度相似，关键是通过 `OVER()` 子句定义“窗口范围”，决定数据的筛选和排序规则：
```sql
-- FIRST_VALUE：获取窗口内的第一个值
FIRST_VALUE(列名) OVER (
    [PARTITION BY 分组列] -- 可选，按指定列分组（如按部门分组）
    [ORDER BY 排序列 [ASC/DESC]] -- 必选，窗口内数据的排序规则
    [ROWS BETWEEN 起始边界 AND 结束边界] -- 可选，限定窗口的物理行范围
) AS 别名

-- LAST_VALUE：获取窗口内的最后一个值
LAST_VALUE(列名) OVER (
    [PARTITION BY 分组列]
    [ORDER BY 排序列 [ASC/DESC]]
    [ROWS BETWEEN 起始边界 AND 结束边界] -- 此参数对LAST_VALUE尤为关键
) AS 别名
```
- **核心参数说明**：
  - `PARTITION BY`：类似“分组”，将数据按指定列拆分为多个独立窗口（如按“部门ID”分组，分别计算每个部门的首尾值）。
  - `ORDER BY`：决定每个窗口内数据的排序顺序（如按“薪资”降序，获取每个部门薪资最高/最低的员工姓名）。
  - `ROWS BETWEEN`：限定窗口的“物理行范围”（如 `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` 表示“从窗口第一行到当前行”），不指定时默认范围可能影响 `LAST_VALUE` 的结果（下文会说明）。

---

##### 2. 功能差异：FIRST_VALUE 与 LAST_VALUE 的核心区别

两者的核心差异在于“获取值的位置”，但需特别注意 `LAST_VALUE` 的默认窗口范围问题：

| 函数         | 核心功能                  | 关键注意点                                                                 |
|--------------|---------------------------|----------------------------------------------------------------------------|
| `FIRST_VALUE`| 获取窗口内**排序后第一行**的指定列值 | 默认范围（从窗口首行到当前行）下，结果通常符合预期（如每个分组的第一个值）。 |
| `LAST_VALUE` | 获取窗口内**排序后最后一行**的指定列值 | 默认范围是“从窗口首行到当前行”，而非“整个窗口”，可能导致结果不符合预期（需手动指定完整窗口范围）。 |

###### 示例：理解 LAST_VALUE 的默认范围问题

假设表 `employee` 数据如下（部门ID、员工姓名、薪资）：
| dept_id | emp_name | salary |
|---------|----------|--------|
| 10      | 张三     | 8000   |
| 10      | 李四     | 9500   |
| 10      | 王五     | 12000  |
| 20      | 赵六     | 7500   |
| 20      | 孙七     | 10000  |

执行以下查询（未指定 `ROWS BETWEEN`）：
```sql
SELECT 
    dept_id,
    emp_name,
    salary,
    -- 获取每个部门薪资最低的员工姓名（FIRST_VALUE，按薪资升序）
    FIRST_VALUE(emp_name) OVER (PARTITION BY dept_id ORDER BY salary ASC) AS dept_min_sal_emp,
    -- 尝试获取每个部门薪资最高的员工姓名（LAST_VALUE，默认范围）
    LAST_VALUE(emp_name) OVER (PARTITION BY dept_id ORDER BY salary ASC) AS dept_max_sal_emp
FROM employee;
```
**结果分析**：
- `dept_min_sal_emp` 结果正确（每个部门薪资最低的员工）。
- `dept_max_sal_emp` 结果错误（返回的是“当前行”而非“整个部门最后一行”），例如部门10的第1行返回“张三”，第2行返回“李四”。

**解决方法**：手动指定窗口范围为“整个分组”：
```sql
LAST_VALUE(emp_name) OVER (
    PARTITION BY dept_id 
    ORDER BY salary ASC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING -- 完整窗口范围
) AS dept_max_sal_emp
```
此时 `dept_max_sal_emp` 会正确返回每个部门薪资最高的员工（部门10返回“王五”，部门20返回“孙七”）。

---

##### 3. 常见使用场景

两个函数常用于“分组内首尾值统计”，典型场景包括：
- **场景1：分组内首尾数据提取**  
  如“按部门分组，获取每个部门入职最早（FIRST_VALUE）和最晚（LAST_VALUE）的员工姓名及入职时间”。
- **场景2：与当前行对比**  
  如“按日期排序，获取当前订单的金额与本月第一笔订单金额（FIRST_VALUE）、最后一笔订单金额（LAST_VALUE）的差值”。
- **场景3：排名关联**  
  如“按薪资降序排序，获取每个员工的薪资及本部门最高薪资（LAST_VALUE），计算薪资占比”。

#### SUM() OVER

------

##### 📝 完整语法结构

标准的语法结构应该是这样的：

```sql
SUM(求和列)
OVER (
    PARTITION BY 分组列        -- 可选：先分组（如果不写，全表视为一组）
    ORDER BY 排序列           -- 可选：组内排序（如果不写，窗口默认包含组内所有行）
    ROWS BETWEEN 
        [起始位置] AND [结束位置] -- 可选：指定窗口范围
)
```

**位置常量说明：**

- `[n] PRECEDING`：往前数 n 行。
- `CURRENT ROW`：当前行。
- `[n] FOLLOWING`：往后数 n 行。
- `UNBOUNDED PRECEDING`：从分区第一行开始。
- `UNBOUNDED FOLLOWING`：到分区最后一行结束。

------

##### 🏢 场景实战：三种常见模式

假设我们有一个按月份排序的数据集。

###### 模式一：累计求和（默认行为）

**需求：** 年初到现在的累计销售额。
**写法：** (省略 `ROWS`，利用默认特性)

```sql
SUM(amount) OVER (ORDER BY month)
-- 等价于
SUM(amount) OVER (ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

###### 模式二：滑动窗口（移动平均）

**需求：** 包含当前月在内，最近 3 个月的销售额总和。
**写法：** (必须写 `ROWS`)

```sql
SUM(amount) OVER (
    ORDER BY month 
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
-- 解释：从“往前数2行”开始，到“当前行”结束，共3行。
```

###### 模式三：组内总和（忽略排序影响）

**需求：** 每一行都显示该部门的全年总销售额。
**写法：** (用 `ROWS` 强制包含所有行，或者干脆不写 `ORDER BY`)

```sql
SUM(amount) OVER (
    PARTITION BY dept 
    ORDER BY month 
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
-- 或者更简单的写法（不推荐依赖默认，建议显式写出）：
SUM(amount) OVER (PARTITION BY dept)
```

##### 📌 总结

对 `SUM() OVER()` 的理解已经非常扎实了。简单概括就是：

- **`PARTITION BY`**：分组（把大池子隔成小池子）。
- **`ORDER BY`**：排序（决定数据处理的先后顺序）。
- **`ROWS BETWEEN`**：定框（决定拿多大的数据窗口来计算）。

只要掌握了这三者的组合，窗口函数你就掌握了一大半！

### 13. **全文索引函数**

适用于全文索引类型的字段。

- `MATCH()`: 用于全文索引的全文搜索。
- `AGAINST()`: 与 `MATCH()` 一起使用，指定要查询的内容。

这些函数在 MySQL 中提供了丰富的数据处理能力，可以帮助用户更高效地进行数据查询和管理。

## MYSQL函数

MySQL 函数是一种存储在数据库中的可重用代码块，用于执行特定操作并返回结果。它们可以简化复杂查询、提高代码复用性，并增强 SQL 语句的功能。


### **一、MySQL 函数的分类**
MySQL 函数主要分为两类：

#### 1. **内置函数**（系统提供）
MySQL 内置了大量函数，覆盖数据处理的各种场景：
- **字符串函数**：`CONCAT()`、`SUBSTRING()`、`LENGTH()` 等
- **数值函数**：`SUM()`、`AVG()`、`ROUND()`、`MOD()` 等
- **日期函数**：`NOW()`、`DATE()`、`DATEDIFF()` 等
- **聚合函数**：`COUNT()`、`MAX()`、`MIN()`、`GROUP_CONCAT()` 等
- **条件函数**：`IF()`、`IFNULL()`、`CASE` 等

示例：
```sql
-- 计算字符串长度
SELECT LENGTH('Hello'); -- 返回 5

-- 求平均值
SELECT AVG(salary) FROM employees;
```


#### 2. **自定义函数**（用户创建）
用户可根据需求创建自定义函数（`FUNCTION`），完成特定业务逻辑。  
**特点**：
- 必须有返回值（与存储过程的主要区别）
- 可被 SQL 语句调用（如 `SELECT`、`UPDATE` 中）
- 可重用，简化复杂查询


### **二、自定义函数的基本语法**

#### 1. **创建函数**
```sql
DELIMITER //  -- 修改结束符（避免函数内分号冲突）delimiter
CREATE FUNCTION 函数名(参数列表) 
RETURNS 返回值类型
DETERMINISTIC  -- 可选，声明函数结果是否确定
BEGIN
  -- 函数体（SQL 语句）
  RETURN 结果;
END //
DELIMITER ;  -- 恢复默认结束符
```

**参数格式**：`参数名 数据类型`（如 `N INT`、`name VARCHAR(50)`）  
**返回值类型**：如 `INT`、`VARCHAR(100)`、`DATE` 等


#### 2. **调用函数**
```sql
SELECT 函数名(参数);
```


#### 3. **查看函数**
```sql
-- 查看所有函数
SHOW FUNCTION STATUS;

-- 查看函数定义
SHOW CREATE FUNCTION 函数名;
```


#### 4. **删除函数**
```sql
DROP FUNCTION IF EXISTS 函数名;
```


### **三、自定义函数示例**

#### 示例 1：计算两个数的和
```sql
DELIMITER //
CREATE FUNCTION add_num(a INT, b INT) 
RETURNS INT
BEGIN
  RETURN a + b;
END //
DELIMITER ;

-- 调用
SELECT add_num(3, 5); -- 返回 8
```


#### 示例 2：获取第 N 高的薪水
```sql
DELIMITER //
CREATE FUNCTION getNthHighestSalary(N INT) 
RETURNS INT
BEGIN
  SET N = N - 1; -- 转换为 OFFSET 偏移量（从 0 开始）
  RETURN (
    SELECT DISTINCT salary 
    FROM employees 
    ORDER BY salary DESC 
    LIMIT 1 OFFSET N
  );
END //
DELIMITER ;

-- 调用
SELECT getNthHighestSalary(2); -- 返回第二高的薪水（若不存在则返回 NULL）



```


### **四、注意事项**
1. **权限**：创建函数需要 `CREATE ROUTINE` 权限，修改需要 `ALTER ROUTINE` 权限。
2. **结束符**：创建函数时需用 `DELIMITER` 修改结束符（默认 `;` 会与函数内分号冲突）。
3. **返回值**：函数必须有且仅有一个返回值（`RETURN` 语句）。
4. **确定性**：`DETERMINISTIC` 表示函数输入相同则输出一定相同（优化器可据此优化），否则用 `NOT DETERMINISTIC`。
5. **与存储过程的区别**：
   - 函数必须返回值，存储过程可以没有返回值。
   - 函数可在 `SELECT` 中调用，存储过程需用 `CALL` 单独调用。

通过自定义函数，可将常用逻辑封装，减少重复代码，让 SQL 语句更简洁易维护。

## 查询分类

在 MySQL 中，查询（Query）可以根据 **功能目标、数据处理方式、复杂程度** 等维度进行分类。不同分类的查询适用于不同场景，理解分类有助于更精准地编写和优化 SQL 语句。以下是常见的 MySQL 查询分类及详细说明：


### 一、按功能目标分类（最核心分类）
这是最常用的分类方式，根据查询的核心目的划分，涵盖日常开发中 90% 以上的场景。

#### 1. 数据查询类（SELECT）
**作用**：从表中提取指定数据，不修改数据，是最基础、最常用的查询类型。  
**核心语法**：`SELECT ... FROM ... [WHERE ...] [GROUP BY ...] [HAVING ...] [ORDER BY ...] [LIMIT ...]`  
**细分场景**：
- **简单查询**：查询单个表的指定字段，无复杂条件。  
  示例：`SELECT id, name FROM users WHERE age > 18;`
- **关联查询**：查询多个表（通过 `JOIN` 关联）的数据，解决“跨表取数”需求。  
  示例：`SELECT u.name, o.order_id FROM users u JOIN orders o ON u.id = o.user_id;`
- **聚合查询**：通过聚合函数（`SUM`/`AVG`/`COUNT` 等）对数据进行统计计算。  
  示例：`SELECT dept, COUNT(*) AS emp_count FROM employees GROUP BY dept;`
- **窗口查询**：使用窗口函数（`RANK`/`DENSE_RANK`/`SUM() OVER ()` 等）实现排名、累计计算等复杂逻辑（MySQL 8.0+ 支持）。  
  示例：`SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank FROM employees;`


#### 2. 数据插入类（INSERT）
**作用**：向表中添加新数据，是数据写入的核心方式。  
**核心语法**：`INSERT INTO ... [VALUES/SELECT] ...`  
**细分场景**：
- **单行插入**：一次插入一条数据。  
  示例：`INSERT INTO users (name, age) VALUES ('Alice', 25);`
- **多行插入**：一次插入多条数据，比单行插入更高效。  
  示例：`INSERT INTO users (name, age) VALUES ('Bob', 30), ('Charlie', 35);`
- **查询插入（INSERT ... SELECT）**：将一个查询的结果插入到另一个表（常用于数据迁移、批量同步）。  
  示例：`INSERT INTO users_backup (id, name) SELECT id, name FROM users WHERE age > 20;`


#### 3. 数据修改类（UPDATE）
**作用**：更新表中已有的数据，仅修改字段值，不改变表结构。  
**核心语法**：`UPDATE ... SET ... [WHERE ...] [LIMIT ...]`  
**注意**：必须加 `WHERE` 条件（除非需要更新全表），否则会误改所有数据！  
**示例**：`UPDATE users SET age = 26 WHERE name = 'Alice';`（仅更新 Alice 的年龄）


#### 4. 数据删除类（DELETE）
**作用**：删除表中不需要的数据，仅删除行，不改变表结构。  
**核心语法**：`DELETE FROM ... [WHERE ...] [LIMIT ...]`  
**注意**：同样必须加 `WHERE` 条件（除非需要清空表），否则会删除全表数据！  
**细分场景**：
- **条件删除**：删除符合条件的行。  
  示例：`DELETE FROM users WHERE age < 18;`（删除未成年人记录）
- **限制删除**：用 `LIMIT` 控制删除行数，避免一次性删除过多数据。  
  示例：`DELETE FROM orders WHERE create_time < '2023-01-01' LIMIT 1000;`（分批删除历史订单）


#### 5. 表结构操作类（DDL 查询）
**作用**：创建、修改、删除表结构（非数据操作，属于“数据定义语言”）。  
**核心语法**：`CREATE TABLE`/`ALTER TABLE`/`DROP TABLE` 等  
**细分场景**：
- **创建表**：`CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));`
- **修改表结构**：添加字段、修改字段类型等，示例：`ALTER TABLE users ADD COLUMN email VARCHAR(100);`
- **删除表**：`DROP TABLE IF EXISTS users_backup;`（删除备份表，需谨慎）


### 二、按数据处理范围分类
根据查询操作的数据量或范围划分，影响查询性能和执行效率。

#### 1. 单行查询/操作
- **特点**：仅处理表中的一行数据（如查询单个用户、更新一条订单）。  
- **示例**：`SELECT * FROM users WHERE id = 100;`（按主键查询单行，性能极高）


#### 2. 批量查询/操作
- **特点**：处理多行数据（如查询所有部门的统计数据、批量插入1000条用户记录）。  
- **示例**：`UPDATE orders SET status = 'paid' WHERE create_time BETWEEN '2023-10-01' AND '2023-10-07';`（批量更新一周内的订单状态）


#### 3. 全表查询/操作
- **特点**：处理表中的所有数据（如统计全表行数、清空表数据）。  
- **风险**：全表操作（尤其是 `UPDATE`/`DELETE`/`SELECT *`）会占用大量资源，大数据量表需避免。  
- **示例**：`SELECT COUNT(*) FROM users;`（全表计数，若表大建议用 `EXPLAIN` 优化）


### 三、按复杂程度分类
根据查询是否包含嵌套、关联、聚合等逻辑划分，影响可读性和维护成本。

#### 1. 简单查询
- **特点**：无嵌套、无关联，仅操作单个表，逻辑直观。  
- **示例**：`SELECT name, age FROM users WHERE gender = 'male';`


#### 2. 复杂查询
- **特点**：包含子查询、多表关联、多层聚合、窗口函数等，逻辑较复杂，需注意执行顺序。  
- **示例**：查询“每个部门工资最高的员工姓名及工资”（含子查询+关联）：
  ```sql
  SELECT e.dept, e.name, e.salary
  FROM employees e
  JOIN (
      SELECT dept, MAX(salary) AS max_sal FROM employees GROUP BY dept
  ) temp ON e.dept = temp.dept AND e.salary = temp.max_sal;
  ```


### 四、按执行结果分类
根据查询返回的结果形式划分，适用于不同的业务需求。

#### 1. 结果集查询（返回多行多列）
- **特点**：返回结构化的表格数据（如 `SELECT * FROM users`），是最常见的结果形式。  
- **用途**：前端展示列表、数据导出、批量处理等。


#### 2. 单行单列查询（返回单个值）
- **特点**：仅返回一个值（如聚合统计、单个字段查询），常用于判断条件或赋值。  
- **示例**：`SELECT MAX(salary) FROM employees;`（返回最高工资，仅一个值）


#### 3. 空结果查询
- **特点**：查询无匹配数据，返回空结果集（如 `SELECT * FROM users WHERE id = 999`，若 id=999 不存在）。  
- **注意**：需在业务代码中处理空结果，避免空指针异常。


#### 4. 无结果查询（不返回数据）
- **特点**：执行后不返回数据，仅返回“影响行数”（如 `INSERT`/`UPDATE`/`DELETE`）。  
- **示例**：`UPDATE users SET age = 26 WHERE name = 'Alice';`（执行后返回“1 row affected”，无数据结果）


### 五、特殊查询类型
除上述常规分类外，还有一些针对特定场景的特殊查询。

#### 1. 事务查询（含事务控制）
- **特点**：通过 `BEGIN`/`COMMIT`/`ROLLBACK` 控制一组查询的原子性（要么全成功，要么全失败）。  
- **示例**：转账操作（扣减A的余额 + 增加B的余额，需在一个事务中）：
  ```sql
  BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE user_id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE user_id = 'B';
  COMMIT; -- 若有错误则执行 ROLLBACK;
  ```


#### 2. 索引操作查询（DDL 子类）
- **特点**：创建、删除、查看索引（优化查询性能的核心手段）。  
- **示例**：`CREATE INDEX idx_users_age ON users(age);`（为 users 表的 age 字段创建索引）


#### 3. 视图查询（VIEW）
- **特点**：基于查询结果创建“虚拟表”（视图），简化复杂查询的复用。  
- **示例**：创建“部门工资统计视图”：
  ```sql
  CREATE VIEW dept_sal_view AS
  SELECT dept, AVG(salary) AS avg_sal, COUNT(*) AS emp_count FROM employees GROUP BY dept;
  -- 后续查询视图即可，无需重复写聚合逻辑
  SELECT * FROM dept_sal_view;
  ```


### 总结
MySQL 查询的分类本质是“按场景划分功能”，不同分类的查询在语法、性能、用途上有显著差异：
- 日常开发中，**数据查询类（SELECT）** 和 **数据操作类（INSERT/UPDATE/DELETE）** 是高频使用场景；
- 复杂业务（如排名、统计）需用 **窗口查询** 或 **子查询**；
- 性能优化需关注 **全表查询** 和 **批量操作**，避免资源浪费；
- 表结构变更需谨慎使用 **DDL 查询**（如 `ALTER TABLE`/`DROP TABLE`）。

理解这些分类，能帮助你在实际开发中快速定位需求对应的查询类型，编写更高效、更安全的 SQL 语句。

## 特殊情况

#### 不包含某些条件

在 SQL 的 `SELECT` 语句中，要获取“不包含某些条件”的结果，主要通过 **`WHERE` 子句配合否定逻辑** 实现，核心是否定特定条件或排除符合条件的记录。以下是常见方法：

##### 一、使用 `NOT` 关键字否定条件

`NOT` 用于否定一个条件，直接排除满足该条件的记录。

###### 示例：

假设有表 `students`：

| id  | name  | age | gender |
|-----|-------|-----|--------|
| 1   | 张三  | 18  | 男     |
| 2   | 李四  | 20  | 女     |
| 3   | 王五  | 17  | 男     |
| 4   | 赵六  | 22  | 女     |

**需求：查询年龄不大于 18 岁的学生（即排除年龄 > 18 的记录）**
```sql
SELECT * FROM students
WHERE NOT (age > 18);  -- 等价于 WHERE age <= 18
```
**结果**：
| id  | name  | age | gender |
|-----|-------|-----|--------|
| 1   | 张三  | 18  | 男     |
| 3   | 王五  | 17  | 男     |

##### 二、使用 `<>` 或 `!=` 排除等于特定值的记录

`<>` 和 `!=` 是等价的，用于排除字段值等于某个特定值的记录。

**需求：查询性别不是“男”的学生**
```sql
SELECT * FROM students
WHERE gender <> '男';  -- 等价于 WHERE gender != '男'
```
**结果**：
| id  | name  | age | gender |
|-----|-------|-----|--------|
| 2   | 李四  | 20  | 女     |
| 4   | 赵六  | 22  | 女     |

##### 三、使用 `NOT IN` 排除多个特定值

当需要排除字段值属于某个“值列表”中的记录时，用 `NOT IN`。

**需求：查询 id 不是 1、3 的学生**
```sql
SELECT * FROM students
WHERE id NOT IN (1, 3);
```
**结果**：
| id  | name  | age | gender |
|-----|-------|-----|--------|
| 2   | 李四  | 20  | 女     |
| 4   | 赵六  | 22  | 女     |

##### 四、使用 `NOT BETWEEN ... AND ...` 排除区间内的记录

`BETWEEN ... AND ...` 用于匹配某个区间，`NOT` 否定后则排除该区间内的记录。

**需求：查询年龄不在 18~20 岁之间的学生**
```sql
SELECT * FROM students
WHERE age NOT BETWEEN 18 AND 20;  -- 等价于 age < 18 OR age > 20
```
**结果**：
| id  | name  | age | gender |
|-----|-------|-----|--------|
| 3   | 王五  | 17  | 男     |
| 4   | 赵六  | 22  | 女     |

##### 五、使用 `NOT EXISTS` 排除关联表中存在匹配的记录

`NOT EXISTS` 用于排除“在关联表中存在满足条件的记录”的行，通常与子查询配合使用。

假设有表 `scores`（存储学生成绩）：

| id  | student_id | score |
|-----|------------|-------|
| 1   | 1          | 90    |
| 2   | 2          | 85    |

**需求：查询没有成绩记录的学生（即 `scores` 表中无对应 `student_id` 的学生）**
```sql
SELECT * FROM students s
WHERE NOT EXISTS (
  SELECT 1 FROM scores sc 
  WHERE sc.student_id = s.id  -- 关联条件：学生id匹配成绩表的student_id
);
```
**结果**：
| id  | name  | age | gender |
|-----|-------|-----|--------|
| 3   | 王五  | 17  | 男     |
| 4   | 赵六  | 22  | 女     |

##### 六、使用 `IS NOT NULL` 排除空值

`IS NULL` 用于匹配空值，`IS NOT NULL` 则排除空值记录。

**需求：查询姓名不为空的学生（假设表中可能有 `name` 为 `NULL` 的记录）**
```sql
SELECT * FROM students
WHERE name IS NOT NULL;
```

##### 七、名字的首字母不是’M‘

可以使用正则表达式

```
SELECT 
    employee_id,
    IF(employee_id % 2 = 1 AND name NOT REGEXP '^M', salary, 0) AS bonus 
FROM 
    employees 
ORDER BY 
    employee_id

```



##### 总结：常用否定逻辑对照表

| 目标                     | 语法示例                          |
|--------------------------|-----------------------------------|
| 排除满足某个条件的记录    | `WHERE NOT (条件)`                |
| 排除等于特定值的记录      | `WHERE 字段 <> 值` 或 `!= 值`     |
| 排除属于多个值的记录      | `WHERE 字段 NOT IN (值1, 值2...)` |
| 排除区间内的记录          | `WHERE 字段 NOT BETWEEN a AND b`  |
| 排除关联表中存在匹配的记录| `WHERE NOT EXISTS (子查询)`       |
| 排除空值记录              | `WHERE 字段 IS NOT NULL`          |

根据具体场景选择合适的方法，复杂条件可通过 `AND`/`OR` 组合使用（例如 `WHERE NOT (age > 20 AND gender = '男')`）。





#### SQL 语法规定

1.SQL 语法规定：当一个子查询出现在 FROM 子句中时（即作为 “派生表” 使用），必须给它指定一个别名。这个别名的作用是：
让 SQL 解析器能够识别这个子查询的结果集，将其视为一个临时表；
允许外层查询通过别名引用子查询中的字段（例如 n.num、n.next_num 等）。

```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (子查询)  n -- 缺失别名会报错
WHERE num = next_num AND next_num = next_next_num
```

2.子查询无结果时自动返回 `NULL`

#### **为结果集中的列指定固定值并命名**

`'store3' AS store` 的作用是**为结果集中的列指定固定值并命名**,  AS 的作用是起别名，不能起别名后再进行 round等操作

```sql
SELECT product_id, 'store1' AS store, store1 AS price 
FROM Products 
WHERE store1 IS NOT NULL

UNION 
SELECT product_id, 'store2' AS store, store2 AS price 
FROM Products 
WHERE store2 IS NOT NULL

UNION 
SELECT product_id, 'store3' AS store, store3 AS price 
FROM Products 
WHERE store3 IS NOT NULL

这里的'store3' AS store
```



## 面试题

下面给出一套**完整的演示数据库**（MySQL 8.x 语法），包括

1. **建表语句**（含主键 / 外键 / 典型约束与索引）
2. **示例数据插入**（足够覆盖练习）
3. **15 道中高难度 SQL 面试题**，尽量囊括所有知识点：
   - 连接 / 子查询 / 聚合
   - 窗口函数 / 递归 CTE
   - 分页优化 / 索引思考
   - 事务与锁 / 分组陷阱 等

> **复制即可执行**：先建库 `test_interview`，再逐段运行即可。

------

### 一、建表

```sql
CREATE DATABASE IF NOT EXISTS test_interview CHARACTER SET utf8mb4;
USE test_interview;

-- 部门
CREATE TABLE departments (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(50) NOT NULL UNIQUE
);

-- 员工
CREATE TABLE employees (
    id           INT PRIMARY KEY AUTO_INCREMENT,
    name         VARCHAR(50) NOT NULL,
    department_id INT,
    salary       DECIMAL(10,2) NOT NULL,
    manager_id   INT NULL,
    hire_date    DATE NOT NULL,
    CONSTRAINT fk_emp_dept FOREIGN KEY (department_id) REFERENCES departments(id),
    CONSTRAINT fk_emp_mgr  FOREIGN KEY (manager_id)   REFERENCES employees(id)
);
CREATE INDEX idx_emp_dept_salary ON employees(department_id, salary);

-- 客户
CREATE TABLE customers (
    id            INT PRIMARY KEY AUTO_INCREMENT,
    name          VARCHAR(50) NOT NULL,
    phone_model   VARCHAR(100),
    register_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 产品分类（树形）
CREATE TABLE categories (
    id        INT PRIMARY KEY AUTO_INCREMENT,
    name      VARCHAR(50) NOT NULL,
    parent_id INT NULL,
    CONSTRAINT fk_cat_parent FOREIGN KEY (parent_id) REFERENCES categories(id)
);

-- 产品
CREATE TABLE products (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(100) NOT NULL,
    category_id INT,
    price       DECIMAL(10,2) NOT NULL,
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_prod_cat FOREIGN KEY (category_id) REFERENCES categories(id)
);
CREATE INDEX idx_prod_price ON products(price);

-- 订单
CREATE TABLE orders (
    id           BIGINT PRIMARY KEY AUTO_INCREMENT,
    customer_id  INT NOT NULL,
    amount       DECIMAL(12,2) NOT NULL,
    status       ENUM('PENDING','PAID','CANCELLED','SHIPPED') DEFAULT 'PENDING',
    create_time  DATETIME DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_ord_cust FOREIGN KEY (customer_id) REFERENCES customers(id),
    INDEX idx_orders_time_status (create_time, status)
);

-- 订单明细
CREATE TABLE order_items (
    id         BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id   BIGINT,
    product_id INT,
    quantity   INT DEFAULT 1,
    price      DECIMAL(10,2) NOT NULL,   -- 下单时的单价
    CONSTRAINT fk_item_order  FOREIGN KEY (order_id)   REFERENCES orders(id),
    CONSTRAINT fk_item_product FOREIGN KEY (product_id) REFERENCES products(id)
);
```

------

### 二、示例数据

```sql
-- 部门
INSERT INTO departments (name) VALUES ('HR'), ('Engineering'), ('Finance');

-- 员工
INSERT INTO employees (name, department_id, salary, manager_id, hire_date) VALUES
('Alice', 1, 6000, NULL,        '2021-05-01'),
('Bob',   2, 12000, NULL,       '2019-08-01'),
('Carol', 2,  8500, 2 /*Bob*/,  '2022-03-15'),
('David', 3,  9000, NULL,       '2020-11-20');

-- 客户
INSERT INTO customers (name, phone_model, register_date) VALUES
('Tom',   'iPhone 14',      NOW() - INTERVAL 10 DAY),
('Jerry', 'Huawei P60',     NOW() - INTERVAL 40 DAY),
('Marry', 'iPhone 13 Pro',  NOW() - INTERVAL  5 DAY),
('Peter', 'Samsung S23',    NOW() - INTERVAL 15 DAY);

-- 分类
INSERT INTO categories (id, name, parent_id) VALUES
(1,'Electronics', NULL),
(2,'Phone',        1),
(3,'Laptop',       1);

-- 产品
INSERT INTO products (name, category_id, price) VALUES
('iPhone 14', 2, 7999),
('MacBook Pro', 3, 15999),
('Huawei P60', 2, 5999),
('Dell XPS 13', 3, 8999);

-- 订单 & 明细
INSERT INTO orders (customer_id, amount, status, create_time) VALUES
(1,  7999, 'PAID',      NOW() - INTERVAL 2 DAY),        -- Tom
(2, 11998, 'PAID',      NOW() - INTERVAL 31 DAY),       -- Jerry
(3, 13998, 'SHIPPED',   NOW() - INTERVAL 1 DAY),        -- Marry
(4,  5999, 'PENDING',   NOW() - INTERVAL 3 HOUR);       -- Peter

INSERT INTO order_items (order_id, product_id, quantity, price) VALUES
(1, 1, 1, 7999),
(2, 2, 1, 15999),
(2, 3, 1, 5999),
(3, 1, 1, 7999),
(3, 4, 1, 8999),
(4, 3, 1, 5999);
```

------

### 三、综合 SQL 面试题 (15 题)

> **说明**：所有题目均可在上述数据上直接运行验证。
>
>  根据难度由浅入深，括号中标明考点。

| #    | 题目 (自行作答)                                              | 考点                  |
| ---- | ------------------------------------------------------------ | --------------------- |
| 1    | 统计每个部门的员工人数与平均工资。                           | `GROUP BY` + 聚合     |
| 2    | 查询近 30 天内购买金额 > 8000 的订单（含客户名、订单金额）。 | 日期函数 + 连接       |
| 3    | 找出拥有重复手机号型号(`phone_model`)的客户及数量。          | `GROUP BY` + `HAVING` |
| 4    | 查询订单金额最大的订单及对应客户姓名。                       | 子查询 / `MAX()`      |
| 5    | 查询“Engineering”部门工资最高的三位员工及工资。              | 排名窗口函数          |
| 6    | 统计每月的订单总额（近 一年）。                              | `DATE_FORMAT` + 分组  |
| 7    | 写一条 **游标分页**：查询 `orders` 表中 `id < ?` 的下一页 5 条（按 id 降序）。 | Keyset 分页           |
| 8    | 查询已支付订单中，单价高于该订单平均单价的订单明细。         | 相关子查询            |
| 9    | 递归查询并列出 `Electronics` 分类下的所有子分类。            | `WITH RECURSIVE`      |
| 10   | 对每位客户统计：订单数、最后一次下单时间、总消费金额。       | 聚合+`MAX()`+连接     |
| 11   | 查询每个部门薪资总额占公司薪资总额的百分比（保留两位小数）。 | 窗口函数或子查询      |
| 12   | 删除 `customers` 表中注册时间早于一年且没有订单的客户（写 DELETE）。 | 多表删除              |
| 13   | 找出连续两天都有下单记录的客户 ID（提示：窗口函数或自连接+日期差）。 | 连续日期分析          |
| 14   | 统计近 30 天内每天的新增客户数与新增订单数并同列展示。       | 衍生表 + 左连接       |
| 15   | 对 `orders` 建议一个合适的复合索引，使下面查询（近 30 天、已支付）更优：`SELECT * FROM orders WHERE create_time >= NOW() - INTERVAL 30 DAY AND status = 'PAID';` 并说明原因。 | 索引设计              |

------

#### ✨ 你可以这样作答

- **基础题**（1–4）常考语法正确性。
- **中级题**（5–10）验证窗口函数、复杂连接、聚合 + 业务场景。
- **高级题**（11–15）侧重性能、递归、连续性分析、索引与删除策略。

可先尝试自行编写 SQL，再用 `EXPLAIN` 或结果集验证。
 需要官方参考答案或解析时，告诉我题号即可！

## 面试题2

好的，下面是一个包含多个 SQL 关键点的综合表设计、数据插入示例及高质量面试题（含官方参考答案与解析），涵盖连接、分组、聚合、分页、窗口函数、子查询、索引优化等常考内容。

------

### ✅ 基础表设计 + 初始化数据

#### 1. 用户表（users）

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    gender ENUM('M', 'F'),
    age INT,
    city VARCHAR(50),
    signup_date DATE
);

INSERT INTO users (name, gender, age, city, signup_date) VALUES
('Alice', 'F', 28, 'Beijing', '2023-06-01'),
('Bob', 'M', 35, 'Shanghai', '2023-07-15'),
('Charlie', 'M', 22, 'Beijing', '2024-01-10'),
('Diana', 'F', 30, 'Shenzhen', '2024-03-05');
```

#### 2. 订单表（orders）

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    amount DECIMAL(10, 2),
    status ENUM('PAID', 'PENDING', 'CANCELLED'),
    create_time DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

INSERT INTO orders (user_id, amount, status, create_time) VALUES
(1, 199.99, 'PAID', '2024-06-01 12:00:00'),
(1, 59.90, 'PENDING', '2024-06-08 15:20:00'),
(2, 299.50, 'PAID', '2024-06-02 08:15:00'),
(3, 88.00, 'CANCELLED', '2024-06-05 09:00:00'),
(3, 150.00, 'PAID', '2024-06-07 11:30:00'),
(4, 30.00, 'PAID', '2024-06-09 18:00:00');
```

------

### 🧠 面试题（带参考答案）

------

#### ❶ 查询每个用户的订单总数和总金额（包括没有订单的用户）

```sql
SELECT u.id, u.name, COUNT(o.id) AS order_count, COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

**解析**：考察 `LEFT JOIN` + 分组 + `COALESCE()` 防空值。



##### 用 `LEFT JOIN` 的原因，

通常是在查询中**需要包含主表（这里是用户表）所有记录**，即使关联表（订单表）没有匹配的记录，也要显示主表的那条数据。具体解释如下：

------

###### 1. **业务需求：显示所有用户及其订单信息**

- 目的：即使某些用户没有任何订单，也要在结果中显示他们（例如显示订单数为 0，总金额为 0）。
- 如果用 `INNER JOIN`，只会返回有订单的用户，没订单的用户会被过滤掉。

------

###### 2. **`LEFT JOIN` 是左连接的含义**

- 返回左表（users）的所有行。
- 右表（orders）有匹配时，显示匹配的数据；无匹配时，右表字段返回 `NULL`。

------

###### 3. **举例说明**

- `users` 表有用户 A、B；
- `orders` 表只有 A 的订单。

```sql
-- LEFT JOIN 包含所有用户
SELECT u.id, u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

结果：

| id   | name | order_id |
| ---- | ---- | -------- |
| 1    | A    | 10       |
| 2    | B    | NULL     |

------

###### 4. **如果用 `INNER JOIN`**

```sql
SELECT u.id, u.name, o.id AS order_id
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

结果：

| id   | name | order_id |
| ---- | ---- | -------- |
| 1    | A    | 10       |

- 用户 B 被过滤掉了，因为没有匹配订单。

------

###### 5. **总结**

| 用法         | 结果解释                                | 适用场景                         |
| ------------ | --------------------------------------- | -------------------------------- |
| `INNER JOIN` | 只显示两表都有匹配的记录                | 只关心有订单的用户               |
| `LEFT JOIN`  | 显示左表全部记录，无匹配右表字段为 NULL | 需要显示所有用户，包含无订单用户 |

------

所以题目中要查「每个用户的订单总数和总金额，包括没有订单的用户」，必须用 `LEFT JOIN`，保证没有订单的用户也能被查询出来，`SUM` 和 `COUNT` 结合 `COALESCE` 处理 NULL 值。

##### `COALESCE()` 防空值

这是一个非常经典的 SQL 面试考点，重点在于：**`COALESCE(SUM(...), 0)` 是为了解决当用户没有订单时，`SUM()` 返回 NULL 的问题**。

------

###### ✅ 一、为什么直接用 `SUM()` 可能出问题？

当一个用户 **没有任何订单** 时，`LEFT JOIN` 结果中该用户对应的 `orders` 字段全为 NULL。此时：

```sql
SUM(o.amount) → NULL
```

因为 SQL 规范中，`SUM(NULL)` 的结果是 **NULL**，而不是 0。

------

###### ✅ 二、为什么要加 `COALESCE(SUM(...), 0)`？

```sql
COALESCE(SUM(o.amount), 0)
```

表示：

- 如果 `SUM()` 是 NULL（即没有订单），就返回 `0`，
- 否则返回实际的总和。

这样可以防止结果中出现 NULL 值，更符合“**总金额为 0**”的语义。

------

###### ✅ 三、演示对比（如下场景）

假设 `users` 表有用户 A 和 B，只有 A 下了订单：

| id   | name |
| ---- | ---- |
| 1    | A    |
| 2    | B    |

`orders` 表只有 A 的记录：

| id   | user_id | amount |
| ---- | ------- | ------ |
| 10   | 1       | 100.00 |

查询对比如下：

 **正确写法：**

```sql
SELECT u.name, COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```

| name | total_spent |
| ---- | ----------- |
| A    | 100.00      |
| B    | 0.00        |

**错误写法：**

```sql
SELECT u.name, SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```

| name | total_spent    |
| ---- | -------------- |
| A    | 100.00         |
| B    | NULL         ❌ |

------

###### ✅ 四、面试延伸考点

- `COUNT(*)` 永远不为 NULL，但 `SUM()` 和 `AVG()` 在无数据时为 NULL；
- `COALESCE()` 常用于防止 NULL 干扰业务逻辑；
- 可替换写法：`IFNULL(SUM(o.amount), 0)`（MySQL专用）；
- `LEFT JOIN` + 聚合必须注意空值影响。

------

如需更多这类 SQL 面试陷阱总结，我可以继续整理。是否需要我生成一个“空值处理 + 聚合函数常见误区”小册子？

------

#### ❷ 查询近 7 天内下单金额最高的用户及其订单金额

```sql
SELECT u.id, u.name, SUM(o.amount) AS total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.create_time >= NOW() - INTERVAL 7 DAY
GROUP BY u.id
ORDER BY total_spent DESC
LIMIT 1;
```

**解析**：时间过滤 + 分组聚合 + 排序分页。

##### 为什么 `GROUP BY` 只写了 `u.id`，却没有写 `u.name`？

------

###### 1. 标准 SQL 要求

- `SELECT` 中出现的字段，要么是聚合函数（如 `SUM`、`COUNT`），
- 要么出现在 `GROUP BY` 子句里。
- 否则结果是 **不确定的** 或在某些数据库中会报错。

------

###### 2. MySQL 的宽松处理（非标准SQL）

- MySQL 默认（默认 `sql_mode` 不严格）允许你只 `GROUP BY` 主键字段（这里是 `u.id`），
- 然后 `SELECT` 里出现的非聚合字段（如 `u.name`）会被隐式地按照该分组字段确定唯一值返回。
- 这是因为 `u.id` 是 `users` 表的主键，`u.name` 对应 `u.id` 是一对一的关系，`u.name` 可以唯一确定。

------

###### 3. 具体原因总结：

- `u.id` 是 `users` 表的主键（唯一标识），
- 所以 `GROUP BY u.id` 相当于根据用户唯一分组，
- `u.name` 在分组中是一致的，没有歧义，
- 因此 MySQL 允许省略 `u.name`，默认按主键分组取对应的字段值。

------

###### 4. 不同数据库的行为：

- PostgreSQL、Oracle 等严格遵循 SQL 标准，执行此语句会报错，必须在 `GROUP BY` 中写上 `u.name`。
- 推荐写法（兼容所有数据库）：

```sql
GROUP BY u.id, u.name
```

------

###### 5. 总结：

| 情况                 | 说明                                             |
| -------------------- | ------------------------------------------------ |
| 只写 `GROUP BY u.id` | MySQL 宽松模式允许，`u.id` 是主键，`u.name` 唯一 |
| 标准 SQL 要求        | 必须写全所有非聚合字段                           |
| 建议                 | 明确写出所有非聚合字段，增强代码兼容性和可读性   |

------

如果你想保证 SQL 跨数据库兼容，最好加上 `u.name` 到 `GROUP BY`。

------

#### ❸ 使用窗口函数查询每个用户的最近一笔订单时间与金额

```sql
SELECT *
FROM (
    SELECT o.*, ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY create_time DESC) AS rn
    FROM orders o
) t
WHERE rn = 1;
```

**解析**：考察窗口函数 `ROW_NUMBER()` 提取每组最新记录。

------

#### ❹ 查询累计消费金额超过 200 的用户信息

```sql
SELECT u.*
FROM users u
JOIN (
    SELECT user_id, SUM(amount) AS total
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
    HAVING total > 200
) t ON u.id = t.user_id;
```

**解析**：子查询 + `HAVING` 条件控制。

------

#### ❺ 查询用户最新一条订单记录（不使用窗口函数）

```sql
SELECT o.*
FROM orders o
JOIN (
    SELECT user_id, MAX(create_time) AS latest_time
    FROM orders
    GROUP BY user_id
) t ON o.user_id = t.user_id AND o.create_time = t.latest_time;
```

**解析**：考察非窗口方式提取分组最新记录。

------

#### ❻ 查询注册时间早于 2024 年、消费总金额大于所有北京用户平均消费金额的用户

```sql
SELECT u.id, u.name
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.signup_date < '2024-01-01'
GROUP BY u.id, u.name
HAVING SUM(o.amount) > (
    SELECT AVG(amount)
    FROM orders o2
    JOIN users u2 ON o2.user_id = u2.id
    WHERE u2.city = 'Beijing'
);
```

**解析**：考察相关子查询 + `HAVING` + 日期筛选。

------

#### ❼ 查询每个城市每种性别的平均年龄（多字段分组）

```sql
SELECT city, gender, AVG(age) AS avg_age
FROM users
GROUP BY city, gender;
```

------

#### ❽ 查询订单状态为 PAID 且金额排名前 3 的订单（不分用户）

```sql
SELECT *
FROM (
    SELECT *, RANK() OVER(ORDER BY amount DESC) AS rnk
    FROM orders
    WHERE status = 'PAID'
) t
WHERE rnk <= 3;
```

------

#### ❾ 查询有多个订单的用户，并显示订单数量

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 1;
```

------

#### ❿ 实现分页查询：每页 2 条，第 2 页，按订单创建时间倒序

```sql
SELECT *
FROM orders
ORDER BY create_time DESC
LIMIT 2 OFFSET 2;
```

------

如需继续出题（如多表嵌套、触发器、函数、自定义分页优化等），或需要将这些内容转为面试文档或题库格式，我可以继续帮你整理。是否需要？

下面再给出 **7 道进阶面试题**，覆盖 *多表嵌套查询、`EXISTS/NOT EXISTS`、触发器、存储过程/函数、Key-Set 分页优化* 等较高级主题。依旧基于先前的 `users / orders / order_items / products / categories / employees / departments` 这些表，可直接在同一数据库里练习。

------

#### 题 11 多表嵌套（相关子查询 + 聚合 + JOIN）

> **要求**：找出每个城市 *消费金额最高* 的那位用户及其消费总额
>  **要点**：相关子查询、`JOIN`、聚合

```sql
SELECT u.id,
       u.name,
       u.city,
       t.total_spent
FROM users u
JOIN (
      SELECT user_id,
             SUM(amount) AS total_spent
      FROM orders
      WHERE status = 'PAID'
      GROUP BY user_id
) t ON u.id = t.user_id
WHERE t.total_spent = (
        SELECT MAX(SUM(amount))
        FROM orders o2
        JOIN users u2 ON o2.user_id = u2.id
        WHERE o2.status = 'PAID'
          AND u2.city = u.city
        GROUP BY u2.city
      );
```

**解析**

- 内层子查询按用户统计消费；
- 外层使用相关子查询取每座城市的最大 `SUM(amount)`；
- *考察点*：嵌套聚合 + 相关子查询性能。

------

#### 题 12 `EXISTS / NOT EXISTS` 典型用法

> **要求**：查询 *从未下过已支付订单* 的用户（即没有任何 `status='PAID'` 的订单）。

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
      SELECT 1
      FROM orders o
      WHERE o.user_id = u.id
        AND o.status = 'PAID'
);
```

**解析**

- `NOT EXISTS` 扫描短路，比 `LEFT JOIN … IS NULL` 在大数据下更高效；
- *考察点*：半连接 / 反连接的写法与优化。

------

#### 题 13 触发器 —— 订单状态变更日志

> **场景**：要求任何订单状态发生更新时，将变化记录到日志表 `order_status_log(order_id, old_status, new_status, change_time)`。
>  **任务**：编写 **`AFTER UPDATE`** 触发器。

```sql
CREATE TABLE order_status_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT,
    old_status ENUM('PAID','PENDING','CANCELLED','SHIPPED'),
    new_status ENUM('PAID','PENDING','CANCELLED','SHIPPED'),
    change_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //
CREATE TRIGGER trg_order_status_change
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
    IF OLD.status <> NEW.status THEN
        INSERT INTO order_status_log(order_id, old_status, new_status)
        VALUES (NEW.id, OLD.status, NEW.status);
    END IF;
END //
DELIMITER ;
```

**解析**

- 使用 `OLD` / `NEW` 对比；
- 写在 **`AFTER UPDATE`**，可确保主表更新成功后再写日志；
- *考察点*：触发器语法、业务审计。

------

#### 题 14 存储过程 + 事务演示：内部转账

> **需求**：在账户表 `accounts(id, balance)` 中，实现用户 A→B 转账的存储过程，失败则回滚。

```sql
DELIMITER //
CREATE PROCEDURE transfer(IN from_id INT, IN to_id INT, IN amt DECIMAL(10,2))
BEGIN
  DECLARE exit HANDLER FOR SQLEXCEPTION
  BEGIN
      ROLLBACK;
      RESIGNAL;
  END;

  START TRANSACTION;
      UPDATE accounts SET balance = balance - amt WHERE id = from_id;
      UPDATE accounts SET balance = balance + amt WHERE id = to_id;
  COMMIT;
END //
DELIMITER ;
```

**解析**

- 显式 `START TRANSACTION / COMMIT / ROLLBACK`；
- 异常处理器回滚；
- *考察点*：事务控制 + 存储过程流程。

------

#### 题 15 自定义函数：年龄段划分并统计

> **步骤**
>
> 1. 创建函数 `age_group(age)`：返回 `'青年'` `<30`、`'中年'` 30-49、`'老年'` ≥50。
> 2. 统计各年龄段人数。

```sql
-- 1) 函数
DELIMITER //
CREATE FUNCTION age_group(p_age INT) RETURNS VARCHAR(10)
DETERMINISTIC
BEGIN
  RETURN CASE
    WHEN p_age < 30 THEN '青年'
    WHEN p_age BETWEEN 30 AND 49 THEN '中年'
    ELSE '老年'
  END;
END //
DELIMITER ;

-- 2) 使用
SELECT age_group(age)   AS group_name,
       COUNT(*)         AS cnt
FROM users
GROUP BY group_name;
```

**解析**

- 自定义函数可在 `SELECT`/`WHERE` 中直接调用；
- *考察点*：函数创建、`DETERMINISTIC`、复用。

------

#### 题 16 Key-Set（游标）分页接口

> **需求**：在大表 `orders` 中实现“向下翻页”（最新→历史）：
>
> - 若首次进入，不传参，返回最新 10 条；
> - 前端下拉时传回 *上一页最后一条 id*，SQL 取 `id < ?` 的下一批。

```sql
-- 初次加载
SELECT * 
FROM orders
ORDER BY id DESC
LIMIT 10;

-- 向下加载：假设上一页最后 id = 12000
SELECT * 
FROM orders
WHERE id < 12000
ORDER BY id DESC
LIMIT 10;
```

**解析**

- `id` 必须为聚簇/唯一索引；
- 避免 `OFFSET` 扫描；
- *考察点*：高性能分页设计。

------

#### 题 17 多级递归 + 聚合：分类树商品计数

> **要求**：列出 *“Electronics” 下所有子分类* 及其商品数。

```sql
WITH RECURSIVE cat_tree AS (
  SELECT id, name, parent_id
  FROM categories
  WHERE name = 'Electronics'          -- 根节点
  UNION ALL
  SELECT c.id, c.name, c.parent_id
  FROM categories c
  JOIN cat_tree ct ON c.parent_id = ct.id
)
SELECT ct.id,
       ct.name,
       COUNT(p.id) AS product_cnt
FROM cat_tree ct
LEFT JOIN products p ON p.category_id = ct.id
GROUP BY ct.id, ct.name;
```

**解析**

- 递归 CTE 获取整棵树；
- 再 `LEFT JOIN` 产品、分组计数；
- *考察点*：多级递归、分组聚合组合。

------

### 📌 面试官可追问

| 追问方向     | 可能问题                       |
| ------------ | ------------------------------ |
| 触发器       | 如何避免触发器递归？如何调试？ |
| 存储过程     | 并发转账导致死锁怎么办？       |
| Key-Set 分页 | 如何实现“上一页”回滚？         |
| 递归         | 如何限制递归层数？防止死循环？ |

------

> 以上 17 道题（含此前 10 题）基本覆盖 **连接 / 分组 / 子查询 / 窗口函数 / 递归 / 事务 / 触发器 / 函数 / 分页 / 索引** 等主流面试考点。
>  如需更多方向（如 **全文索引、分区表、JSON 操作、GIS 类型** 等）或完整实现文档，请告诉我！

## 面试题3

### 树状层级

这是一个典型的“树状层级”数据查询问题，对应到 SQL 中，主要涉及**多表连接（JOIN）\**和\**递归查询**两种思路。

根据常见的学校管理系统设计，通常有以下两种情况。我为你整理了具体的表结构设计思路和对应的 SQL 查询语句。

#### 场景设定：表结构设计

通常我们会设计四张表，或者在用户表中通过字段关联。为了清晰，我们假设存在以下四张表：

1. **`schools` (学校表)**: `school_id`, `school_name`
2. **`grades` (年级表)**: `grade_id`, `school_id`, `grade_name` (如 高一)
3. **`classes` (班级表)**: `class_id`, `grade_id`, `class_name` (如 1班)
4. **`students` (学生表)**: `student_id`, `class_id`, `student_name`

------

#### 🌳 一、从学校查到个人 (自顶向下)

**业务逻辑：** 给定一个学校 ID，找出该学校下的**所有学生**。

**SQL 思路：** 使用 `INNER JOIN` 或 `LEFT JOIN` 逐级向下穿透。从学校关联年级，年级关联班级，班级关联学生。

```sql
SELECT 
    s.school_name,
    g.grade_name,
    c.class_name,
    st.student_name
FROM schools s
-- 1. 学校 -> 年级
INNER JOIN grades g ON s.school_id = g.school_id
-- 2. 年级 -> 班级
INNER JOIN classes c ON g.grade_id = c.grade_id
-- 3. 班级 -> 学生
INNER JOIN students st ON c.class_id = st.class_id
WHERE s.school_id = ?; -- 这里填具体的学校ID
```

**说明：**

- 如果你想查某个特定年级（如高一）的学生，可以在 `WHERE` 中加 `AND g.grade_name = '高一'`。
- 如果你想查某个特定班级的学生，可以在 `WHERE` 中加 `AND c.class_name = '1班'`。

------

#### 👤 二、从个人查到学校 (自底向上)

**业务逻辑：** 给定一个学生 ID，找出他所在的班级、年级以及学校。

**SQL 思路：** 依然是 `JOIN`，但是方向是从学生表向上反查。通常我们只需要查出学校名称，所以连接到 `schools` 表即可。

```sql
SELECT 
    st.student_name,
    c.class_name,
    g.grade_name,
    s.school_name
FROM students st
-- 1. 学生 -> 班级
LEFT JOIN classes c ON st.class_id = c.class_id
-- 2. 班级 -> 年级
LEFT JOIN grades g ON c.grade_id = g.grade_id
-- 3. 年级 -> 学校
LEFT JOIN schools s ON g.school_id = s.school_id
WHERE st.student_id = ?; -- 这里填具体的学生ID
```

**说明：**

- 这里使用 `LEFT JOIN` 是为了防止出现“学生有班级，但班级对应的年级被删除了”这种脏数据导致查不到人。如果数据完整性有强约束，用 `INNER JOIN` 也可以。

------

#### 🧩 特殊情况：如果数据都在一张大宽表里

有些简单的系统设计可能只有一张表，包含所有信息：

**表 `student_info`**: `student_id`, `student_name`, `school_id`, `school_name`, `grade_id`, `grade_name`, `class_id`, `class_name`

在这种情况下，查询极其简单：

- 从学校查个人：

  ```sql
  SELECT * FROM student_info 
  WHERE school_id = ?;
  ```

- 从个人查学校：

  ```sql
  SELECT school_name, grade_name, class_name 
  FROM student_info 
  WHERE student_id = ?;
  ```

#### 📌 总结

| 查询方向         | 核心方法           | 关键点                                                       |
| ---------------- | ------------------ | ------------------------------------------------------------ |
| **学校 -> 个人** | 多表 `JOIN` (向下) | 逐级关联：学校.id -> 年级.school_id -> 班级.grade_id -> 学生.class_id |
| **个人 -> 学校** | 多表 `JOIN` (向上) | 反向关联，通常只需要取最顶层的 `school_name` 字段            |

**面试/实战加分点：**
如果层级非常深（比如省-市-区-学校-年级-班级-个人），或者层级不固定，MySQL 8.0+ 可以使用 **CTE 递归查询**。但在学校-年级-班级这种固定三级结构中，直接用 `JOIN` 性能最好，也最清晰。

## 面试题4：选择题

### 空值相关

#### 1.SQL中,涉及空值的操作

AGE = NULL ❌ 不正确：

- SQL 中，NULL与任何值（包括自身）使用=比较时，结果均为 **UNKNOWN**（非真非假）。
- 例如，即使AGE为 NULL，AGE = NULL不会返回TRUE，而是返回UNKNOWN，导致查询结果不符合预期。
- 正确写法应为AGE IS NULL。

### 索引相关

1.如果WHERE子句中使用的索引包含了ORDER BY中的列，因为索引本身会对列进行排序，所以ORDER BY中不需要再次使用索引排序；如果WHERE子句中使用的索引没有包含ORDER BY中的列，则ORDER BY中的列无法使用索引，因为一个查询只能选择一个索引，这时候排序就进行了filesort，非常费时。综上所述，当WHERE中使用了索引，ORDER BY中不会使用索引。

### Explain相关

在MySQL的EXPLAIN输出中，

'Using temporary'表示查询需要创建临时表存储中间结果，可能涉及磁盘存储（当临时表超过内存限制）；

'Using filesort'表示排序操作无法在内存中完成，需使用文件系统进行外部排序。这通常导致额外磁盘I/O和性能下降。



### MySQL语法相关

WHERE 子句中不能直接使用聚合函数 AVG(amount)，会导致语法错误。



### 视图相关

视图是虚拟表，视图所引用的表称为视图的基表。

若基表的数据发生变化，则变化也会自动反映到视图中

数据库存储的是视图的定义，不存放视图对应的数据

视图可以解决检索数据时一个表中得不到一个实体所有信息的问题



视图不能对临时表或表变量进行引用；

sp_helptext用于获取自定义视图创建的T_SQL文本，更新视图数据可用sp_refreshview。

```
当SELECT语句的选择列表有TOP子句时，视图可以包含ORDER BY子句；
为了保证视图是可更新的，其定义中不能包含以下语法结构：
  集合操作符
  DISTINCT 操作符
  聚合函数或分析型函数
  GROUP BY，ORDER BY，CONNECT BY，或 START WITH 字句
  在 SELECT 之后的列表中使用集合表达式
  在 SELECT 之后的列表中使用子查询
  连接（join）（但是有例外情况）
  对于不可更新的视图，可以利用 INSTEAD OF 触发器对其数据进行修改
```

### 存储过程

存储过程包含用户定义的SQL语句集合，可以使用CALL语句调用存储过程，当然在存储过程中也可以使用CALL语句调用其他存储过程，但是不能使用DROP语句删除其他存储过程。

MySQL在定义存储过程中的参数列表时，参数不能与数据表中的字段名相同。

MySQL不能对已经存在的存储过程代码进行修改，如果必须要修改存储过程，必须使用DROP语句删除之后，重新编写。

如果在存储过程中传入中文参数，则需要在定义存储过程的时候在后面加上character set gbk。否则会在调用过程使用该参数的时候出错。