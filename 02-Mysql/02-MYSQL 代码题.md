# SQL 代码题（手写实操题）

下面按照难度、业务场景划分，适用于 MySQL 后端面试手写 SQL，每类附带典型例题方向
## 一、基础单表查询类

1. 条件筛选、多条件 AND/OR 组合查询
2. NULL 值筛选、模糊匹配 like、区间 between‑and、in 集合查询
3. 字段别名、表别名、去重 distinct
4. order by 升降序、多字段排序、null 排序处理
5. limit 普通分页、大偏移分页优化（主键书签分页）

## 二、聚合与 group‑by 分组统计
1. count(*)、count(字段)、sum、avg、max、min 基础聚合
2. having 分组后过滤
3. group_concat 分组拼接字段
4. 按日/周/月分组统计报表
5. 查询分组下最大值、最新一条数据

## 三、多表联查代码题
1. 内连接、左连接、右连接基础写法
2. LEFT JOIN ON 和 WHERE 区分考题
3. 自连接（员工上级、商品分类层级、相邻数据对比）
4. 三表及以上关联查询
5. 联表更新 update join、联表删除 delete join

## 四、子查询 & Exists 题型
1. 标量子查询充当查询字段
2. IN 子查询、NOT‑IN（null坑考点）
3. 相关子查询
4. EXISTS / NOT EXISTS 代替 IN
5. ANY、ALL 比较子查询

## 五、结果集合并
1. union、union all 合并多条查询
2. 合并之后全局排序

## 六、条件函数（case‑when、if）
1. case when 实现行转列（男女数量、多状态统计）
2. 简单case、搜索case语法
3. if、ifnull 处理查询结果null兜底

## 七、字符串、日期处理类代码题
1. concat、concat_ws 字符串拼接
2. substring 截取字符串、trim去除空格
3. now、date_format、date_add、datediff 日期运算
4. 根据出生日期计算年龄、获取本周/本月第一天

## 八、窗口函数（中级高频手写题）
1. 排名：row_number() / rank() / dense_rank() 成绩排名
2. ntile 数据均分分组
3. lag、lead 获取上一行、下一行数据
4. 滑动窗口聚合、分组求和、求占比
5. 计算用户连续签到天数

## 九、重复数据处理经典考题
1. 查询表里面重复的数据
2. 删除重复数据，保留 id 最小/最大记录
3. 根据多字段判断重复并清理

## 十、高级业务场景SQL题
1. 查询第 N 高薪资（去重后的排名）
2. 树形递归查询部门、菜单层级（MySQL8.0 with 递归CTE）
3. 订单超时统计、间隔时间筛选
4. 行列互转、列转行
5. 查找日期断点、缺失的日期
6. 悲观锁 for update、共享锁书写
7. 乐观锁版本号更新语句

## 十一、增删改 DML 实操代码
1. 批量插入多条数据
2. insert on duplicate key update 冲突更新
3. replace into 替换插入
4. 条件更新、批量修改字段值
5. 带条件的数据删除

## 十二、DDL、索引、存储对象编写题
1. 建表、设置主键、唯一键、非空、默认约束
2. alter 添加、删除、修改字段
3. 创建普通索引、联合索引、唯一索引
4. 视图、存储过程、触发器简单编写

## 十三、易错陷阱专项代码题
1. 隐式类型转换案例
2. or 查询陷阱
3. not‑in 包含 null 返回空集合
4. group by 严格模式书写规范

需要我可以给每一类提供**示例SQL代码**。

# 15道MySQL面试题

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

## 一、建表

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

## 二、示例数据

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

## 三、综合 SQL 面试题 (15 题)

> **说明**：所有题目均可在上述数据上直接运行验证。
>
> 根据难度由浅入深，括号中标明考点。

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



环境：MySQL8.0+（支持递归CTE、窗口函数），所有SQL基于你提供的表结构与测试数据

### 1、统计每个部门的员工人数与平均工资（GROUP‑BY、聚合函数）

```
SELECT
    d.name AS dept_name,
    COUNT(e.id) AS emp_count,
    ROUND(AVG(e.salary),2) AS avg_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name;
```

### 2、查询近 30 天内购买金额 > 8000 的订单（含客户名、订单金额）

```
SELECT
    c.name AS customer_name,
    o.amount,
    o.create_time
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
WHERE o.create_time >= NOW() - INTERVAL 30 DAY
  AND o.amount > 8000;
```

### 3、找出拥有重复手机号型号(phone_model)的客户及数量（GROUP BY + HAVING）

```
SELECT
    phone_model,
    COUNT(id) AS repeat_num
FROM customers
GROUP BY phone_model
HAVING COUNT(id) > 1;
```

### 4、查询订单金额最大的订单及对应客户姓名

写法1：子查询

```
SELECT
    o.*,
    c.name AS customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.amount = (SELECT MAX(amount) FROM orders);
```

写法2：排序取第一条

```
SELECT o.*,c.name
FROM orders o
JOIN customers c ON o.customer_id=c.id
ORDER BY o.amount DESC
LIMIT 1;
```

### 5、查询“Engineering”部门工资最高的三位员工及工资，窗口函数排名

```
WITH eng_emp AS (
    SELECT
        e.name,
        e.salary,
        RANK() OVER(ORDER BY e.salary DESC) AS rk
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    WHERE d.name = 'Engineering'
)
SELECT name,salary
FROM eng_emp
WHERE rk <= 3;
```

> 如需即使同分也强制分出先后改为 `row_number()`；dense_rank() 为连续排名

### 6、统计近一年每月的订单总额，DATE_FORMAT 日期分组

```
SELECT
    DATE_FORMAT(create_time,'%Y-%m') AS order_month,
    SUM(amount) AS month_total
FROM orders
WHERE create_time >= NOW() - INTERVAL 1 YEAR
GROUP BY order_month
ORDER BY order_month;
```

### 7、游标分页：id < ?，每页5条、id降序（Keyset‑分页）

> ? 代表上一页最后一条订单id，示例假设上一页最后id=3

```
SELECT *
FROM orders
WHERE id < 3
ORDER BY id DESC
LIMIT 5;
```

### 8、已支付订单中，单价高于该订单平均单价的订单明细、相关子查询

```
SELECT oi.*
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'PAID'
AND oi.price > (
    SELECT AVG(price)
    FROM order_items oi2
    WHERE oi2.order_id = oi.order_id
);
```

### 9、递归查询 Electronics 分类下所有子分类 WITH RECURSIVE

```
WITH RECURSIVE cat_tree AS (
    SELECT * FROM categories WHERE name = 'Electronics'
    UNION ALL
    SELECT c.*
    FROM categories c
    INNER JOIN cat_tree ct ON c.parent_id = ct.id
)
SELECT * FROM cat_tree;
```

### 10、每位客户：订单数量、最后下单时间、总消费金额

```
SELECT
    c.id,
    c.name,
    COUNT(o.id) AS order_cnt,
    MAX(o.create_time) AS last_order_time,
    COALESCE(SUM(o.amount),0) AS total_consume
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id,c.name;
```

> COALESCE处理无订单客户总消费返回0，而不是null

### 11、每个部门薪资总额占公司薪资总额百分比、保留两位小数

#### 方案一：窗口函数（最简）

```
SELECT
    d.name AS dept_name,
    SUM(e.salary) AS dept_sum_salary,
    ROUND(SUM(e.salary)/SUM(SUM(e.salary)) OVER() *100,2) AS rate
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id,d.name;
```

#### 方案二：子查询

```
SELECT
    d.name,
    SUM(e.salary) dept_sum,
    ROUND(SUM(e.salary)/(SELECT SUM(salary) FROM employees)*100,2) AS percent
FROM departments d
LEFT JOIN employees e ON d.id=e.department_id
GROUP BY d.id,d.name;
```

### 12、删除注册超过一年、不存在任何订单的客户，多表删除

```
DELETE c
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE c.register_date < NOW() - INTERVAL 1 YEAR
AND o.id IS NULL;
```

### 13、找出连续两天下单的客户ID（窗口函数lag实现）

```
WITH cust_order_date AS (
    SELECT DISTINCT
        customer_id,
        DATE(create_time) AS order_day
    FROM orders
),
lag_data AS (
    SELECT
        customer_id,
        order_day,
        LAG(order_day) OVER(PARTITION BY customer_id ORDER BY order_day) AS prev_day
    FROM cust_order_date
)
SELECT DISTINCT customer_id
FROM lag_data
WHERE DATEDIFF(order_day,prev_day) = 1;
```

### 14、近30天每日新增客户数、新增订单数，同一张表展示

```
WITH day_customer AS (
    SELECT
        DATE(register_date) AS day,
        COUNT(id) AS new_customer
    FROM customers
    WHERE register_date >= NOW()-INTERVAL 30 DAY
    GROUP BY day
),
day_order AS (
    SELECT
        DATE(create_time) AS day,
        COUNT(id) AS new_order
    FROM orders
    WHERE create_time >= NOW()-INTERVAL 30 DAY
    GROUP BY day
)
SELECT
    COALESCE(dc.day,do.day) AS stat_day,
    COALESCE(dc.new_customer,0) AS new_customer,
    COALESCE(do.new_order,0) AS new_order
FROM day_customer dc
FULL JOIN day_order do ON dc.day = do.day
ORDER BY stat_day;
```

> MySQL8.0支持FULL JOIN；低版本可使用union all + group‑by 实现日期全集

### 15、复合索引设计

#### 需要优化SQL

```
SELECT * FROM orders WHERE create_time >= NOW() - INTERVAL 30 DAY AND status = 'PAID';
```

#### 建立索引

```
CREATE INDEX idx_orders_status_time ON orders(status,create_time);
```

#### 原理说明

1. 联合索引遵守**最左前缀原则**；
2. status为等值查询放在索引最前面，先快速筛选 `status='PAID'`；
3. 后续create_time做范围筛选；
4. 如果你把create_time放在前列，范围字段之后status索引失效，无法利用status等值过滤。

> 补充：原有索引 `idx_orders_time_status (create_time, status)` 不合适，因为create_time是范围条件，后面status无法命中索引。

## 基于当前15题缺失考点 

先梳理原有15套试卷**没有覆盖到的全部SQL语法、实战、索引、DML、坑点、高级考点**，之后按照考点分类，每一项附带「考点说明 + 面试练习题 + 参考答案SQL」，全部复用你上面的 `test_interview` 数据表，可直接运行。

### 一、DML增删改高级语法（原题几乎只考查询）

1. `insert … on duplicate key update` 冲突写入
2. replace into
3. 多表关联update更新
4. update 悲观锁 for update、共享锁
5. 防止全表更新、limit更新
6. 软删除语句

### 二、常见查询陷阱、null、运算符优先级、隐式转换

1. NULL 判断、`is null`，不能使用 = null
2. AND / OR 优先级
3. not in 包含null返回空结果坑
4. 模糊查询 like 通配符、索引失效
5. 隐式类型转换索引失效场景

### 三、高级条件语法

1. case‑when 行转列、条件统计
2. if、ifnull、coalesce
3. 分组拼接 group_concat、长度陷阱

### 四、多类窗口函数原题覆盖不全

原题只用了rank、lag；缺少：

1. dense_rank / row_number 三者对比考题
2. lead 获取后一行数据
3. ntile 均分分组
4. 滑动窗口 rows between 滚动求和
5. 分组内第一条、最后一条数据

### 五、联表查询经典坑

1. left‑join on 和 where 的区别
2. 自连接（查询员工直属上级名字）
3. cross join 笛卡尔积

### 六、索引专项考点（第15题仅一题，考点单薄）

1. 最左前缀失效场景
2. 覆盖索引、回表查询
3. explain执行计划
4. 删除无用索引
5. 普通索引、唯一索引创建

### 七、TCL事务、锁语法

1. 手动开启事务
2. 行锁、表锁触发场景SQL演示

### 八、视图、存储过程、触发器、自定义函数

原题完全没有 DDL存储对象考题

### 九、CTE递归之外的难题

1. 列转行
2. 查询第N高薪资
3. 查找日期空缺
4. exists 和 in 性能对比考题

### 十、DCL权限语法

用户创建、授权、回收权限，原题零涉及

## 补充练习题

整体新增考点总结

现在整套题库就完整覆盖：

1. DDL（表、索引、视图、存储过程、触发器）
2. DML（增删改、关联更新、冲突插入、锁语句）
3. DQL全部：基础查询、多表连接、自连接、分组聚合、子查询、in/exists、全套窗口函数、case‑when行转列、游标分页、递归CTE、日期运算
4. TCL事务、行锁共享锁排他锁
5. DCL 用户权限管理
6. MySQL高频坑点：NULL、and/or优先级、not‑in陷阱、索引失效、隐式类型转换
7. 索引原理、最左前缀、覆盖索引、联合索引设计
8. 各类经典手写SQL面试真题（第N高、分组最大值、连续日期、行列转换）

### 模块1 DML 增删改高级考点

#### 考点1：insert on duplicate key update

> 题目：员工姓名设置唯一，存在则更新薪资，不存在新增员工

```
-- 需要先给name添加唯一索引
CREATE UNIQUE INDEX uk_emp_name ON employees(name);

INSERT INTO employees(name,department_id,salary,manager_id,hire_date)
VALUES('Alice',1,6500,null,'2021‑05‑01')
ON DUPLICATE KEY UPDATE salary = 6500;
```

#### 考点2：多表关联update

> 题目：Engineering部门所有员工薪资上调10%

```
UPDATE employees e
INNER JOIN departments d ON e.department_id = d.id
SET e.salary = e.salary * 1.1
WHERE d.name = 'Engineering';
```

#### 考点3、悲观排他锁、共享锁

> 考题：锁住id=1订单，禁止其他事务修改订单

```
--排他锁
SELECT * FROM orders WHERE id = 1 FOR UPDATE;
--共享锁
SELECT * FROM orders WHERE id = 1 LOCK IN SHARE MODE;
```

#### 考点4、软删除

新增员工表is_deleted字段，删除改为更新标记

```
ALTER TABLE employees ADD COLUMN is_deleted TINYINT DEFAULT 0;
UPDATE employees SET is_deleted=1 WHERE id = 5;
```

### 模块2 SQL易错坑点专项题

#### 考点1 null 判断陷阱

> 题目：查询没有直属上级的员工（manager_id为null）

```
SELECT * FROM employees WHERE manager_id IS NULL;
```

> 考点：`manager_id = NULL` 永远查不到数据

#### 考点2 AND OR优先级

> 题目：查询部门2薪资大于10000 或者部门3薪资大于8000员工

```
SELECT * FROM employees
WHERE (department_id=2 AND salary>10000)
OR (department_id=3 AND salary>8000);
```

> 不加括号会因为and优先级高于or造成查询出错

#### 考点3 NOT IN null陷阱

考题：找出不是Bob下属的员工

```
SELECT * FROM employees
WHERE manager_id NOT IN (SELECT id FROM employees WHERE name='Bob');
```

> 子查询结果含有null，not‑in查询结果为空

#### 考点4 like模糊查询

考题：查询手机型号以iPhone开头的客户

```
SELECT * FROM customers WHERE phone_model LIKE 'iPhone%';
```

> `%iPhone`、`%iPhone%` 无法命中索引

### 模块3 条件函数、行转列、group_concat

#### 考点 case‑when 行转列

> 题目：统计四种订单状态各自订单数量

```
SELECT
SUM(CASE status WHEN 'PENDING' THEN 1 ELSE 0 END) AS pending_cnt,
SUM(CASE status WHEN 'PAID' THEN 1 ELSE 0 END) AS paid_cnt,
SUM(CASE status WHEN 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled_cnt,
SUM(CASE status WHEN 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_cnt
FROM orders;
```

#### 考点 group_concat

> 考题：查询每个部门下面所有员工姓名，逗号拼接

```
SELECT d.name, GROUP_CONCAT(e.name SEPARATOR ',') AS emp_names
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id,d.name;
```

#### 考点 coalesce空值兜底

```
SELECT c.name,COALESCE(SUM(o.amount),0) AS total FROM customers c
LEFT JOIN orders o ON c.id=o.customer_id GROUP BY c.id;
```

### 模块4 全套窗口函数练习题

#### 题1 区分 row_number / rank / dense_rank

> 获取Engineering部门员工薪资排名

```
SELECT
name,salary,
ROW_NUMBER() OVER(ORDER BY salary DESC) rn,
RANK() OVER(ORDER BY salary DESC) rk,
DENSE_RANK() OVER(ORDER BY salary DESC) drk
FROM employees e
JOIN departments d ON e.department_id=d.id
WHERE d.name='Engineering';
```

#### 题2 lead 获取下一条记录

> 获取每位客户相邻两次下单间隔天数

```
WITH t AS (
SELECT DISTINCT customer_id,DATE(create_time) day
FROM orders
)
SELECT
customer_id,day,
LEAD(day) OVER(PARTITION BY customer_id ORDER BY day) next_day,
DATEDIFF(LEAD(day) OVER(PARTITION BY customer_id ORDER BY day),day) diff
FROM t;
```

#### 题3 ntile均分分组

> 将员工薪资由高至低，平均分为2组

```
SELECT name,salary,NTILE(2) OVER(ORDER BY salary DESC) group_id
FROM employees;
```

#### 题4 滑动窗口滚动求和

> 按日期，计算每日以及近7天订单总金额

```
WITH day_amt AS (
SELECT DATE(create_time) day,SUM(amount) sum_amt
FROM orders GROUP BY day
)
SELECT
day,sum_amt,
SUM(sum_amt) OVER(ORDER BY day ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) seven_day_total
FROM day_amt;
```

### 模块5 联表查询、自连接经典考题

#### 考点 left‑join on 和 where区别

> 查询所有部门以及部门内薪资大于10000员工

```
--on 中过滤，部门全部保留
SELECT * FROM departments d
LEFT JOIN employees e ON d.id=e.department_id AND e.salary>10000;

--where过滤，会把null数据过滤变成内连接
SELECT * FROM departments d
LEFT JOIN employees e ON d.id=e.department_id
WHERE e.salary>10000;
```

#### 自连接：查询每个员工以及他直属上级名字

```
SELECT e.name emp_name,m.name manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### 模块6 索引专项题目

1. 创建产品价格普通索引、分类‑价格联合索引

```
CREATE INDEX idx_prod_category_price ON products(category_id,price);
```

1. 考题：说明查询 

   ```
   where category_id=2 and price>5000
   ```

    为什么适合 (category_id,price)联合索引

   > category_id等值匹配、price范围查询，满足最左前缀

2. 覆盖索引考题：只需要查询订单创建时间和状态，建立覆盖索引避免回表

```
CREATE INDEX idx_cover_status_time ON orders(status,create_time);
```

1. 删除索引

```
DROP INDEX idx_prod_price ON products;
```

### 模块7 子查询、in / exists对比

> 考题：查询存在下单记录的客户，分别使用IN、EXISTS，说明性能差异

```
--in
SELECT * FROM customers WHERE id IN (SELECT DISTINCT customer_id FROM orders);
--exists
SELECT * FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id=c.id);
```

> 子查询订单量大推荐exists；订单量小in可读性更好

### 模块8 高级手写经典面试原题

#### 题1 查询薪资第2高的员工（去重）

```
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

#### 题2 查询每个部门薪资最高的员工

```
WITH t AS (
SELECT *,RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) rk
FROM employees
)
SELECT name,salary,department_id FROM t WHERE rk=1;
```

#### 题3 查找没有任何订单的客户（not exists）

```
SELECT * FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id=c.id);
```

### 模块9 DDL 存储程序（视图、存储过程、触发器）

#### 1. 创建视图：已支付订单视图

```
CREATE VIEW view_paid_order AS
SELECT * FROM orders WHERE status='PAID';
```

#### 2. 创建存储过程：传入部门id，返回部门下员工数量

```
DELIMITER //
CREATE PROCEDURE getEmpCount(IN deptId INT,OUT total INT)
BEGIN
SELECT COUNT(id) INTO total FROM employees WHERE department_id=deptId;
END //
DELIMITER ;

--调用
CALL getEmpCount(2,@res);
SELECT @res;
```

### 模块10 DCL权限考题

```
--创建账号
CREATE USER 'interview_user'@'localhost' IDENTIFIED BY '123456';
--授予查询权限
GRANT SELECT ON test_interview.* TO 'interview_user'@'localhost';
--回收权限
REVOKE SELECT ON test_interview.* FROM 'interview_user'@'localhost';
FLUSH PRIVILEGES;
```

### 模块11 事务TCL语法

```
START TRANSACTION;
UPDATE employees SET salary=10000 WHERE id=1;
--提交
COMMIT;
--回滚
ROLLBACK;
```

