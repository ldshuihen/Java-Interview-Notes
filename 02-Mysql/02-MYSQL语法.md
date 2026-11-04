# MySQL 核心语法、内置函数与高阶 SQL 实战手册

> **文档定位与知识库联动说明**：
> - 本文档专注于 **MySQL 核心语法规范、DQL/DML/DDL 编写细节、内置函数速查、高阶查询特性（窗口函数 & 递归 CTE）以及笔试/面试高频手写 SQL 模板**。
> - 关于 InnoDB 存储引擎底层原理（B+ 树定量推算、MVCC、锁机制、Redo/Undo/Binlog、慢查询调优、主从复制与分库分表），请联动参阅同目录专章：[`02-Mysql.md`](./02-Mysql.md)、[`02-Mysql 表设计.md`](./02-Mysql%20表设计.md)、[`02-Mysql 集群.md`](./02-Mysql%20集群.md)。

---

## 一、SQL 语言五大分类与词法基石

### 1.1 SQL 语句五大标准分类（DDL / DML / DQL / DCL / TCL）

| 分类缩写 | 英文全称与中文名 | 核心职责 | 代表关键字 | 事务与隐式提交特性 |
| :--- | :--- | :--- | :--- | :--- |
| **DDL** | Data Definition Language<br>（数据定义语言） | 定义或修改库、表、索引、视图等**元数据对象结构** | `CREATE`、`ALTER`、`DROP`、`TRUNCATE`、`RENAME` | **执行即触发隐式提交（Implicit Commit）**，无法通过 `ROLLBACK` 回滚！ |
| **DML** | Data Manipulation Language<br>（数据操纵语言） | 对表中的**行数据**进行增、删、改 | `INSERT`、`UPDATE`、`DELETE`、`REPLACE` | 受事务控制，未提交前可通过 Undo Log 执行 `ROLLBACK` 回滚。 |
| **DQL** | Data Query Language<br>（数据查询语言） | 检索表或视图中的数据结果集（高频核心） | `SELECT`（配合 `FROM/JOIN/WHERE/GROUP BY/HAVING/ORDER BY/LIMIT`） | 普通 `SELECT` 为 MVCC 快照读不加锁；加 `FOR UPDATE / FOR SHARE` 为当前读。 |
| **DCL** | Data Control Language<br>（数据控制语言） | 管理数据库账号、角色与访问权限 | `GRANT`、`REVOKE`、`CREATE USER` | 执行权限变更同样会触发隐式提交。 |
| **TCL** | Transaction Control Language<br>（事务控制语言） | 管理事务边界、提交、回滚与保存点 | `BEGIN` / `START TRANSACTION`、`COMMIT`、`ROLLBACK`、`SAVEPOINT` | 控制 DML 操作的原子性与一致性边界。 |

> *(注：部分传统教材将 `DQL (SELECT)` 归入广义 `DML` 形成“四大分类”，面试答出五大分类并点明 **DDL 隐式提交不可回滚** 更显专业度。)*

---

### 1.2 MySQL 三种引号铁律与保留字避坑

核心口诀：**单引号包“数据值”，反引号包“标识符（库/表/列名）”，双引号一律弃用。**

1. **单引号 `' '` —— 字符串/日期/枚举字面量（标准 SQL）**
   - 凡是插入表中或出现在 `WHERE` 条件里的非数值常量（字符串、日期时间、`ENUM` 值），一律使用单引号包裹。
   - **内部单引号转义**：使用两个连续单引号 `''`（SQL 标准，推荐）或反斜杠 `\'`：
     ```sql
     SELECT * FROM users WHERE name = 'O''Neill';   -- SQL 标准转义（推荐）
     SELECT * FROM logs  WHERE create_time >= '2026-10-01 00:00:00';
     ```
2. **反引号 `` ` ` `` —— 数据库对象标识符（MySQL 专属防冲突符号）**
   - 用于包裹**库名、表名、字段名或别名**，消除与 MySQL 保留关键字的语法解析歧义。
   - **高频踩坑保留字（面试笔试极易报错）**：`order`、`group`、`desc`、`key`、`status`（非保留但常见）、以及 **MySQL 8.0.2+ 新增保留字 `rank`、`row_number`、`lead`、`lag`、`system`、`window`**：
     ```sql
     -- ❌ MySQL 8.0+ 直接报 Syntax Error（rank 是窗口函数保留字！）
     SELECT username, RANK() OVER (ORDER BY points DESC) AS rank FROM users;

     -- ✅ 正确做法 1：改用非保留字别名（最推荐）
     SELECT username, RANK() OVER (ORDER BY points DESC) AS rk FROM users;

     -- ✅ 正确做法 2：使用反引号包裹保留字标识符
     SELECT `order`, RANK() OVER (ORDER BY points DESC) AS `rank` FROM `orders`;
     ```
3. **双引号 `" "` —— 受 `sql_mode` 影响的“双面人”（生产严禁使用）**
   - MySQL 默认模式下 `"` 等同于单引号 `'`（表示字符串）；但一旦开启 **`SET sql_mode = 'ANSI_QUOTES';`**（或迁移至 PostgreSQL / Oracle），`"` 会瞬间变成标识符引号（等同于反引号），导致 `WHERE name = "John"` 被当成“比较 `name` 列与 `John` 列”而报错 `Unknown column 'John'`！

---

### 1.3 `NULL` 三值逻辑与空字符串 `''` 的本质区别

#### 一、SQL 三值逻辑（`TRUE` / `FALSE` / `UNKNOWN`）
在 SQL 中，`NULL` 代表**“未知（Unknown）或缺失状态”**，它不是任何具体的值（既不是 `0`，也不是 `''`）：
- **算术与拼接传染性**：任何值与 `NULL` 做四则运算（`10 + NULL`）或普通拼接（`CONCAT('a', NULL)`），结果一律为 **`NULL`**。
- **比较运算铁律**：任何值（**包括 `NULL` 自身**）用 `=`、`!=`、`<>`、`>`、`<` 与 `NULL` 比较，结果既不是 `TRUE` 也不是 `FALSE`，而是 **`UNKNOWN`**！
  - 在 `WHERE` 和 `ON` 中，只有条件为 `TRUE` 的行才会保留，`FALSE` 和 `UNKNOWN` 都会被过滤掉（因此 `WHERE age = NULL` 和 `WHERE age != NULL` 永远查不出任何数据！）。
  - **正确判空语法**：
    - `IS NULL` / `IS NOT NULL`；
    - MySQL 专属 **NULL 安全等于运算符 `<=>`**：`NULL <=> NULL` 返回 `1 (TRUE)`，`'a' <=> NULL` 返回 `0 (FALSE)`。
- **布尔短路特殊规则**：
  - `TRUE OR UNKNOWN` → **`TRUE`**；`FALSE OR UNKNOWN` → **`UNKNOWN`**；
  - `FALSE AND UNKNOWN` → **`FALSE`**；`TRUE AND UNKNOWN` → **`UNKNOWN`**；
  - `NOT UNKNOWN` → **`UNKNOWN`**（这也是为什么 `WHERE col NOT IN (1, 2, NULL)` 等价于 `col != 1 AND col != 2 AND col != NULL`，结果恒为 `FALSE` 或 `UNKNOWN`，导致**整条 SQL 永远返回 0 行空集**！）。

#### 二、空字符串 `''` vs `NULL` 四维对比（高频面试题）

| 对比维度 | 空字符串 `''` | 空值 `NULL` |
| :--- | :--- | :--- |
| **语义本质** | **已知确定的值**（长度为 0 的字符串） | **未知或不存在的状态** |
| **存储开销** | `VARCHAR` 中占 1~2 字节长度头（内容 0 字节）；`CHAR(N)` 空格填充 | InnoDB 行记录头的 **NULL 值位图（Bitmap）** 中占 `1 bit` 标记位，二级索引 `key_len` 多占 `1` 字节 |
| **过滤匹配方式** | 支持 `=` / `!=`（如 `WHERE phone = ''`） | 严禁用 `=`，必须用 **`IS NULL` / `IS NOT NULL`** 或 `<=>` |
| **聚合函数 `COUNT(col)`** | ✅ **计入统计行数** | ❌ **直接忽略，不计入行数**（`SUM/AVG/MAX/MIN` 同样忽略 `NULL`） |
| **`UNIQUE` 唯一索引约束** | 视为相同确定值，**最多只能插入 1 条 `''`**，第二条报唯一键冲突 | SQL 标准视每个 `NULL` 彼此不等，**MySQL 唯一索引列允许插入任意多个 `NULL`** |

---

## 二、数据类型、完整性约束与 DDL 表结构操作

### 2.1 建表字段类型选型全景

| 大类 | 常用类型 | 占用字节 | 范围 / 特性说明 | 生产建表选型规范 |
| :--- | :--- | :---: | :--- | :--- |
| **整数类型** | `TINYINT` | `1 B` | 有符号 `-128~127`，`UNSIGNED` `0~255` | 布尔标记（`is_deleted TINYINT UNSIGNED`）、状态枚举（**禁止写 `TINYINT(1)` 映射布尔以外的状态，防 MyBatis 误映射为 `Boolean`**） |
| *(注：`INT(11)` 括号内数字仅代表配合 `ZEROFILL` 时的显示宽度，与存储字节和数值上限毫无关系，MySQL 8.0.17+ 已废弃显示宽度)* | `SMALLINT` | `2 B` | `-32768~32767`（无符号 `0~65535`） | 排序权重、小范围计数 |
| | `MEDIUMINT` | `3 B` | 约 `±838 万` | 较少使用 |
| | `INT` (`INTEGER`) | `4 B` | 约 `±21.4 亿`（无符号约 `42.9 亿`） | 常规业务表自增 ID、计数字段 |
| | `BIGINT` | `8 B` | `±9.22 × 10^18` | **千万级核心表主键（`BIGINT UNSIGNED`）、雪花算法分布式 ID** |
| **小数与金额** | `DECIMAL(M, D)` | 变长 | **定点数精确存储**（`M` 总位数最大 65，`D` 小数位数最大 30） | **金额、汇率、财务结算唯一指定类型**（如 `DECIMAL(12, 2)`，对应 Java `BigDecimal`），或用 `BIGINT` 存“分” |
| | `FLOAT` / `DOUBLE` | `4 B` / `8 B` | IEEE 754 二进制近似浮点数，存在精度丢失（如 `0.1 + 0.2 != 0.3`） | 严禁存金额！仅限科学计算或非精确传感器数值 |
| **字符串类型** | `CHAR(N)` | 定长 `N` 字符 | `N <= 255`，不足右补空格，检索自动去除尾部空格 | 长度严格固定的字段：MD5（`CHAR(32)`）、BCrypt 密码哈希（`CHAR(60)`）、国家码 |
| *(注：`CHAR(N)` 与 `VARCHAR(N)` 的 `N` 在 MySQL 5.0+ 均代表**字符个数**而非字节数！)* | `VARCHAR(N)` | 变长 + `1~2 B` 头 | 单行总字节上限 `65535 B`（`utf8mb4` 下最多约 16380 字符）；长度字节 `<=255B` 用 1B 头，否则 2B 头 | 姓名、手机号、邮箱、短备注、头像 URL（**严禁把图片二进制直接存 `BLOB`**） |
| | `TEXT` / `LONGTEXT` | 最大 `64KB` / `4GB` | 溢出页存储，不能设常规默认值（8.0.13+ 支持表达式默认值），排序易落磁盘临时表 | 富文本文章详情、超长报文日志（建议垂直拆分至独立扩展表 `article_content`） |
| **日期与时间** | `DATE` / `TIME` / `YEAR` | `3 B` / `3 B` / `1 B` | `'YYYY-MM-DD'` / `'HH:MM:SS'` / `'YYYY'` | 生日、账期日、每日营业时段 |
| | `DATETIME` | `5 B`（5.6.4+，无小数秒） | `'1000-01-01'` ~ `'9999-12-31'`，**与时区无关**，原样存取 | **业务建表首选**（对应 Java 8+ `LocalDateTime`），无 2038 年溢出危机 |
| | `TIMESTAMP` | `4 B` | `'1970-01-01'` ~ **`'2038-01-19'`**，存时转 UTC、读时转会话时区 | 跨国多时区时间戳（受 **2038 问题**限制，中长期系统慎用） |
| **半结构与二进制** | `JSON`（5.7.8+） | 变长二进制格式 | 自动校验 JSON 语法，底层二进制树结构支持按路径快速定位（免全文解析） | 动态扩展属性配置（配合虚拟生成列可建 B+ 树索引） |
| | `VARBINARY` / `BLOB` | 二进制字节流 | 按原始字节比较（天然区分大小写，无字符集概念） | 加密密文、小体积证书密钥 |

---

### 2.2 `ENUM` 与 `SET` 底层整数/位索引机制（笔试高频细节）

1. **`ENUM('dog', 'cat', 'monkey')` —— 单选互斥枚举（底层整数索引从 `1` 开始！）**
   - **底层存储**：占用 `1~2` 字节整数（`<=255` 个枚举占 1B，最多 65535 个占 2B）。索引按**建表声明顺序从 `1` 开始编号**（`'dog'=1`, `'cat'=2`, `'monkey'=3`；`0` 保留给非法空串 `''` 错误值）。
   - **隐式数值运算陷阱**：当 `ENUM` 列参与算术运算或数值排序时，使用的是**底层整数编号**而非字符串字面量！
     ```sql
     SELECT animal + 0 FROM pet;         -- 返回底层整数索引 1, 2, 3
     INSERT INTO pet(animal) VALUES (2); -- 插入数字 2 等价于插入 'cat'！
     ORDER BY animal;                    -- 注意：默认按声明索引 1,2,3 排序，按字母序需写 ORDER BY CAST(animal AS CHAR)
     ```
   - **生产建议**：阿里开发手册**不推荐在业务大表使用 `ENUM`**，因为删除或中间插入枚举项需 `ALTER TABLE` 重建数据且会打乱底层整数映射，生产优先使用 `TINYINT` + 应用层 Java `Enum` 字典管理。
2. **`SET('read', 'write', 'exec')` —— 多选集合（底层 `2^n` 位掩码存储）**
   - 占 `1~8` 字节（最多 64 个成员），按位权分配 `1, 2, 4, 8...`（如 `'read,exec'` 底层存 `1 + 4 = 5`），支持 `FIND_IN_SET('read', perms) > 0` 或位与 `perms & 1` 检索。

---

### 2.3 七大表字段完整性约束

| 约束关键字 | 核心作用 | `NULL` 规则 | 数量限制与核心注意点 |
| :--- | :--- | :---: | :--- |
| **`PRIMARY KEY`** | 主键约束（唯一标识行，隐含 `NOT NULL`） | ❌ 严禁 | 每表最多 **1 个**（可为单列或 `(order_id, item_id)` 复合主键） |
| **`UNIQUE`** | 唯一性约束（自动创建唯一二级索引） | ✅ **允许多个 `NULL`** | 每表可有多个；复合唯一 `UNIQUE uk_a_b(a, b)` 仅当 `a, b` 同时相同时冲突 |
| **`NOT NULL`** | 非空约束 | ❌ 严禁 | 核心业务字段强烈建议加 `NOT NULL` 并配合 `DEFAULT` |
| **`DEFAULT`** | 未显式赋值时的默认值 | 视列定义 | 时间列常用 `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` |
| **`AUTO_INCREMENT`** | 整数列单调自增 | ❌ 配合主键/键使用 | MySQL 8.0+ 自增值持久化在 Redo Log 中，解决了 5.7 重启后自增值回退复用 Bug |
| **`CHECK`（8.0.16+）** | 取值表达式校验（如 `CHECK (age >= 0)`） | `UNKNOWN` 视为通过 | **注意**：MySQL 5.7 及更早版本虽解析 `CHECK` 语法但**完全忽略不生效**，8.0.16 起才真正强制校验！ |
| **`FOREIGN KEY`** | 跨表参照完整性（支持 `ON DELETE CASCADE`） | 允许 | **高并发生产规范严禁建物理外键**（每次更新需跨表加锁检查，极易死锁且无法分库分表），一律由业务服务层保证关联完整性 |

---

### 2.4 库表索引 DDL 运维语法与 `DELETE / TRUNCATE / DROP` 三维对比

#### 一、高频生产标准建表与 DDL 速查模板

```sql
-- 1. 生产级标准建表模板
CREATE TABLE `t_user` (
    `id`         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `username`   VARCHAR(64)     NOT NULL DEFAULT ''     COMMENT '登录账号',
    `mobile`     VARCHAR(20)     NOT NULL DEFAULT ''     COMMENT '手机号',
    `balance`    DECIMAL(12, 2)  NOT NULL DEFAULT 0.00   COMMENT '账户余额(元)',
    `status`     TINYINT         NOT NULL DEFAULT 1      COMMENT '状态: 1-正常 0-禁用',
    `created_at` DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_username` (`username`),
    KEY `idx_status_created` (`status`, `created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户基础信息表';

-- 2. 查看表结构与完整建表 DDL
DESC `t_user`;
SHOW CREATE TABLE `t_user`;
SHOW INDEX FROM `t_user`;

-- 3. 修改表结构（ALTER TABLE）
ALTER TABLE `t_user` ADD COLUMN `email` VARCHAR(100) NOT NULL DEFAULT '' COMMENT '邮箱' AFTER `mobile`;
ALTER TABLE `t_user` MODIFY COLUMN `email` VARCHAR(128) NOT NULL DEFAULT '';      -- 改类型/属性（不改列名）
ALTER TABLE `t_user` CHANGE COLUMN `email` `user_email` VARCHAR(128) NOT NULL DEFAULT ''; -- 改列名+属性
ALTER TABLE `t_user` DROP COLUMN `user_email`;
ALTER TABLE `t_user` ADD INDEX `idx_mobile` (`mobile`);
ALTER TABLE `t_user` DROP INDEX `idx_mobile`;
RENAME TABLE `t_user` TO `sys_user`;

-- 4. 快速复制表结构 / 复制表结构+数据
CREATE TABLE `user_empty_copy` LIKE `t_user`;                    -- 仅复制完整结构与索引（不含数据）
CREATE TABLE `user_data_backup` AS SELECT * FROM `t_user`;       -- 复制结构与查询数据（注意：会丢失原表主键与二级索引定义！）
```

#### 二、【必考对比】`DELETE` vs `TRUNCATE` vs `DROP`

| 对比维度 | `DELETE FROM t [WHERE ...]` | `TRUNCATE TABLE t` | `DROP TABLE t` |
| :--- | :--- | :--- | :--- |
| **语言分类** | **DML** | **DDL** | **DDL** |
| **删除范围** | 按 `WHERE` 逐行删除（保留表结构与索引） | 一次性清空全表数据（保留表结构与索引，**不可带 `WHERE`**） | **连表结构、索引、权限、数据彻底删除** |
| **事务回滚** | ✅ 写 Undo Log，未提交前**可 `ROLLBACK` 回滚** | ❌ 隐式提交事务（重建空 `.ibd` 表空间），**不可回滚** | ❌ 隐式提交事务，**不可回滚** |
| **自增 ID 重置** | ❌ **不重置** `AUTO_INCREMENT` 计数器 | ✅ **重置归 1** | 表已不存在 |
| **触发器与耗时** | 逐行激活 `DELETE` 触发器，逐行加锁写日志，**大表极慢** | 不激活触发器，直接释放数据页，**极快** | 直接删除表定义与物理文件，**最快** |

---

## 三、DQL 单表查询核心与 `SELECT` 十大物理执行顺序

### 3.1 `SELECT` 编写顺序 vs 10 步真实物理执行顺序（面试必背重中之重）

SQL 最反直觉的地方在于：**写在最开头的 `SELECT`，在底层其实快到最后才执行！** 引擎每执行一步都会在逻辑上生成一张中间虚拟表（`vt1 ~ vt9`）：

```text
【SQL 语法编写顺序】
SELECT [DISTINCT] 字段/表达式/聚合/窗口函数
FROM 左表 [INNER|LEFT|RIGHT] JOIN 右表 ON 连接条件
WHERE 原始行过滤
GROUP BY 分组列
HAVING 分组后聚合过滤
ORDER BY 排序列 [ASC|DESC]
LIMIT [offset,] row_count;

=======================================================================

【MySQL 真实逻辑/物理执行顺序（10 步虚拟表推演 —— 必背口诀）】
 ① FROM & JOIN ... ON  ──► 笛卡尔积/关联匹配连接条件，拼出初始虚拟表 vt1（LEFT JOIN 未匹配右表补 NULL）
         ↓
 ② WHERE               ──► 对 vt1 逐行筛选（此时尚未分组、尚未执行 SELECT，不可用聚合函数，不可用 SELECT 别名！）→ vt2
         ↓
 ③ GROUP BY            ──► 按指定列将 vt2 切分为多个分组槽 → vt3
         ↓
 ④ 聚合函数计算         ──► 对每个分组执行 COUNT / SUM / AVG / MAX / MIN 汇总 → vt4
         ↓
 ⑤ HAVING              ──► 对分组聚合后的 vt4 做组级过滤（可使用聚合函数！）→ vt5
         ↓
 ⑥ 窗口函数 OVER()      ──► 在分组过滤后的结果集上执行分区排名/滑动聚合计算（因此窗口函数严禁写在 WHERE / GROUP BY / HAVING 中！）→ vt6
         ↓
 ⑦ SELECT 投影         ──► 挑选输出列、计算算术/标量表达式、正式生成 AS 列别名 → vt7
         ↓
 ⑧ DISTINCT            ──► 对 SELECT 投影出的行做整行组合去重 → vt8
         ↓
 ⑨ ORDER BY            ──► 按指定列或 SELECT 别名排序（此时 SELECT 已完成，别名完全可用！）→ vt9
         ↓
 ⑩ LIMIT / OFFSET      ──► 截取指定偏移与条数，将最终结果集返回客户端
```

---

### 3.2 字段别名与表别名作用域铁律（厘清标准 SQL vs MySQL 方言特例）

1. **字段别名（`SELECT salary * 12 AS annual_sal`）各子句可用性真相表**：

| 子句位置 | 能否使用 `SELECT` 定义的列别名？ | 底层根因解析（彻底终结网传矛盾说法） |
| :--- | :---: | :--- |
| **`ON` / `WHERE`** | ❌ **绝对报错** | `ON`（第①步）和 `WHERE`（第②步）执行时，第⑦步 `SELECT` 尚未执行，解析器根本不知道别名的存在！ |
| **`GROUP BY` / `HAVING`** | ⚠️ **标准 SQL 禁止，但 MySQL 方言特许支持！** | 在 Oracle / SQL Server / PostgreSQL（HAVING）中按标准顺序会报错；**MySQL 解析器专门做了方言扩展，允许在 `GROUP BY` 和 `HAVING` 中直接引用 `SELECT` 别名**（跨库可移植代码仍建议写原始表达式）。 |
| **`ORDER BY`** | ✅ **所有数据库均支持** | `ORDER BY`（第⑨步）在 `SELECT`（第⑦步）之后执行，别名早已生成。 |
| **外层父查询** | ✅ **支持（且唯一访问方式）** | 当子查询作为 `FROM (SELECT ... AS x) AS tmp` 派生表时，外层查询可直接使用 `tmp.x`。 |

2. **表别名与派生表别名三大强制铁律**：
   - **`FROM` 子查询（派生表）必须显式起表别名**：`SELECT * FROM (SELECT id FROM users) AS t;` —— 漏写 `AS t` 在 MySQL 中必报 `Every derived table must have its own alias`！
   - **一旦在 `FROM / JOIN` 给表起了别名，该语句所有地方（含 `SELECT`、`WHERE`、`ON`）必须统一用别名限定列，严禁再混用原表名**（如 `FROM users u WHERE users.id = 1` 直接报错 `Unknown column 'users.id'`）。
   - **别名中含空格或特殊符号**：使用单引号/反引号包裹（建议反引号 `` AS `total count` ``），**且同一级 `SELECT` 中刚定义的别名不能立刻在同一级另一个表达式里二次嵌套计算**（如 `SELECT price * qty AS amt, ROUND(amt, 2)` ❌ 报错，需用子查询/CTE 或重写原表达式）。

---

### 3.3 条件过滤运算符全景、优先级与避坑指南

1. **逻辑运算符优先级铁律**：**`NOT` > `AND` > `OR`**！
   - ⚠️ **经典线上 Bug**：`WHERE status = 'ACTIVE' AND email IS NOT NULL OR phone IS NOT NULL`
     - 因为 `AND` 优先级高于 `OR`，等价于 `(status = 'ACTIVE' AND email IS NOT NULL) OR (phone IS NOT NULL)`，把已禁用但有手机号的用户全查出来了！**混用 `AND` 与 `OR` 时必须显式加圆括号 `(...)`**。
2. **区间与集合匹配要点**：
   - `BETWEEN a AND b`：**闭区间（包含左右边界 `>= a AND <= b`）**；注意日期时间列 `WHERE create_time BETWEEN '2026-10-01' AND '2026-10-02'` 等价于截至 `'2026-10-02 00:00:00'`，**会漏掉 10 月 2 日全天 `00:00:01` 之后的数据**！时间范围一律推荐半开区间 `>= '2026-10-01' AND < '2026-10-03'`。
   - `IN (v1, v2, ...)` vs `NOT IN (...)`：`IN` 列表建议不超过 `500~1000` 个元素；**`NOT IN` 列表或子查询中绝不能出现 `NULL`**（否则恒返回空集，改用 `NOT EXISTS`）。
3. **模糊匹配 `LIKE` 通配符与 `ESCAPE` 转义**：
   - `%` 匹配任意 **0 个或多个**字符；`_`（下划线）匹配**恰好 1 个**任意字符。
   - **坑点**：业务搜索文件名包含真实下划线 `_` 或百分号 `%`（如查含 `_test` 的记录）时，不转义会把 `_` 当成任意单字符通配符！
     ```sql
     -- 使用反斜杠转义（MySQL 默认）或自定义 ESCAPE 转义符
     SELECT * FROM files WHERE file_name LIKE '%\_test%';
     SELECT * FROM files WHERE file_name LIKE '%#_test%' ESCAPE '#';
     ```

---

### 3.4 `DISTINCT` 去重、`ORDER BY` 排序与 `LIMIT` 分页规则

1. **`DISTINCT` 整行去重 vs `GROUP BY` 分组**：
   - `DISTINCT` 必须紧跟在 `SELECT` 之后（如 `SELECT DISTINCT dept_id, job_title FROM emp`），它是对**后面列出的所有字段组合做整行去重**（不存在 `SELECT id, DISTINCT name` 这种局部去重语法，会直接报错！）。
   - `COUNT(DISTINCT col1, col2)` 可统计不重复的多列非空组合数（任意一列为 `NULL` 的组合不计入）。
   - 在无聚合函数时，`SELECT DISTINCT a FROM t` 与 `SELECT a FROM t GROUP BY a` 结果相同（MySQL 8.0 已取消 `GROUP BY` 隐式排序，两者底层均可走松散索引扫描 `Using index for group-by` 或临时表去重）。
2. **`ORDER BY` 排序误区澄清**：
   - 普通单表/联表查询中，**`ORDER BY` 的字段完全不需要出现在 `SELECT` 列中**（如 `SELECT name FROM emp ORDER BY salary DESC` 完全合法）；
   - **唯一例外**：当查询使用了 **`DISTINCT`**（或集合运算 `UNION`）时，因为去重后虚拟表 `vt8` 只保留了 `SELECT` 投影列，此时若 `ORDER BY` 一个不在 `SELECT` 中的列，由于一组合并后对应多个不同排序值会产生语义歧义，在 `ONLY_FULL_GROUP_BY` 下会报错 `Expression #1 of ORDER BY clause is not in SELECT list`。
   - **`NULL` 的排序位置**：MySQL 中 `NULL` 被视为最小值——`ORDER BY col ASC` 时 **`NULL` 排在最前面**，`DESC` 时排在最后面。若想升序但把 `NULL` 放最后：`ORDER BY col IS NULL ASC, col ASC`。
3. **`LIMIT` 两种标准分页语法**：
   - **标准推荐写法（语义最清晰）**：`LIMIT 页大小 OFFSET 跳过行数` → 查询第 `pageNo` 页（从 1 开始）、每页 `pageSize` 条：
     `LIMIT {pageSize} OFFSET {(pageNo - 1) * pageSize}`（如第 3 页每页 10 条：`LIMIT 10 OFFSET 20`）。
   - **MySQL 简写语法（注意参数顺序相反！）**：`LIMIT 跳过行数, 页大小` → `LIMIT 20, 10`。（千万大表深偏移优化见第十章母题与 [`02-Mysql.md` 2.6 节](./02-Mysql.md)）。

---

## 四、多表连接（`JOIN`）、子查询与集合运算

### 4.1 七种连接查询模式与 `ON` vs `USING`

假设左表 `A` 有 `M` 行，右表 `B` 有 `N` 行：

| 连接类型 | 标准语法 | 结果集保留规则 | 典型业务场景 |
| :--- | :--- | :--- | :--- |
| **内连接** | `A INNER JOIN B ON ...`<br>*(简写 `JOIN` 即 `INNER JOIN`)* | **仅保留两表均满足连接条件的交集行**（任一侧关联键为 `NULL` 均无法匹配） | 查询已有有效部门的员工及部门名 |
| **左外连接** | `A LEFT [OUTER] JOIN B ON ...` | **无条件保留左表 `A` 全部行**；右表 `B` 匹配不上时，右表所有列填充 `NULL` | 统计所有用户的订单数（含 0 单新用户） |
| **右外连接** | `A RIGHT [OUTER] JOIN B ON ...` | **无条件保留右表 `B` 全部行**；左表匹配不上补 `NULL`（生产规范建议统一改写为 `LEFT JOIN` 保持阅读方向一致） | — |
| **反连接（Anti-Join）** | `A LEFT JOIN B ON A.id = B.a_id WHERE B.a_id IS NULL` | **专门筛选左表 `A` 中存在、但右表 `B` 中不存在匹配的差集行**（注意 `WHERE` 判空列必须选右表主键或非空关联列！） | 查找从未下过单的用户、查找孤儿订单 |
| **全外连接** | **MySQL 无原生 `FULL OUTER JOIN`！**<br>用 `LEFT JOIN` + `UNION` + `RIGHT JOIN` 模拟 | 同时保留左、右两表所有匹配与未匹配行 | 两套异构系统全量对账差异核对 |
| **交叉连接** | `A CROSS JOIN B`（或不带 `ON` 的 `A, B`） | 返回两表**笛卡尔积（`M × N` 行）**，大表严禁误触！ | 日期维表 × 门店维表生成每日全门店底表骨架 |
| **自连接** | `emp e LEFT JOIN emp m ON e.manager_id = m.id` | 同一张物理表起两个不同别名当两张逻辑表连接 | 单层上下级员工与直属领导姓名同列展示 |

- **连接条件 `ON` vs `USING(同名列)` 细节差异**：
  - 当两表关联字段**名称完全相同**（如都有 `dept_id`）时，`JOIN dept USING (dept_id)` 等价于 `ON e.dept_id = d.dept_id`；
  - **唯一区别**：在 `SELECT *` 时，`ON` 的结果集会出现**两列重复的 `dept_id`**，而 `USING (dept_id)` 会**自动合并去重为 1 列 `dept_id` 并置于首列**。

```sql
-- MySQL 模拟 FULL OUTER JOIN 标准模板
SELECT a.id, a.name, b.amount FROM users a LEFT JOIN orders b ON a.id = b.user_id
UNION
SELECT a.id, a.name, b.amount FROM users a RIGHT JOIN orders b ON a.id = b.user_id;
```

---

### 4.2 【面试超高频陷阱】`LEFT JOIN` 与 `INNER JOIN` 中 `ON` 和 `WHERE` 的本质区别

1. **在 `LEFT JOIN` 中：右表过滤条件放 `ON` vs 放 `WHERE` 结果天差地别！**
   - **执行机制**：
     - **`ON` 是连接生成虚拟表 `vt1` 时的匹配门卫**：右表行不满足 `ON` 条件时，只是“不把这行右表数据拼上来”，**左表原行依然保留，右表列补 `NULL`**；
     - **`WHERE` 是连接完毕得到 `vt1` 后的事后裁员**：所有右表补了 `NULL` 的左表保留行，遇到 `WHERE 右表列 = 'xxx'`（`NULL = 'xxx'` 为 `UNKNOWN`）会被**整行直接杀掉**，导致 **`LEFT JOIN` 彻底退化为 `INNER JOIN`**！
   ```sql
   -- 需求：查询【所有部门】（包括 0 人的空部门），并仅展示其中薪资 > 10000 的员工
   -- ✅ 正确：右表业务过滤放进 ON，无高薪员工的部门依然保留（员工列显示 NULL）
   SELECT d.dept_name, e.emp_name, e.salary
   FROM departments d
   LEFT JOIN employees e ON d.id = e.dept_id AND e.salary > 10000;

   -- ❌ 踩坑：右表过滤放到 WHERE，没有高薪员工的部门因 e.salary 为 NULL 被过滤消失，退化成 INNER JOIN！
   SELECT d.dept_name, e.emp_name, e.salary
   FROM departments d
   LEFT JOIN employees e ON d.id = e.dept_id
   WHERE e.salary > 10000;
   ```
2. **在 `INNER JOIN` 中：`ON` 与 `WHERE` 结果完全等价**
   - 内连接不保留未匹配的 `NULL` 行，MySQL 优化器会将 `INNER JOIN` 中 `ON` 里的单表过滤条件自动等价下推/合并到 `WHERE`。**工业编码规范**：内连接中 **`ON` 只写主外键关联条件，业务过滤统一写进 `WHERE`**，职责最清晰。

---

### 4.3 子查询体系与逻辑量词（`EXISTS / IN / ANY / ALL` 空集与 `NULL` 陷阱）

#### 一、按返回维度与相关性分类
- **按返回形状**：
  - **标量子查询**（1 行 1 列单值）：`WHERE amount > (SELECT AVG(amount) FROM orders)`（若子查询返回 `>1` 行直接运行时报错 `Subquery returns more than 1 row`；返回 0 行则视为 `NULL`）；
  - **列子查询**（N 行 1 列）：配合 `IN` / `ANY` / `ALL`；
  - **行子查询**（1 行多列元组）：`WHERE (dept_id, salary) = (10, 50000)` 或 `IN (SELECT dept_id, MAX(salary) ...)`；
  - **表子查询（派生表）**：出现在 `FROM (...) AS tmp`，**必须起别名**。
- **非相关子查询 vs 相关子查询（Correlated Subquery）**：
  - **非相关子查询**：子查询独立于外部表，只执行 1 次算出常量/集合供外部复用；
  - **相关子查询**：子查询内部引用了外层表的列（如 `WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id)`），逻辑上外层每扫一行都要代入子查询校验。

#### 二、`IN` vs `EXISTS` 与存在/全称量词（`ANY/SOME` vs `ALL`）

1. **`IN` vs `EXISTS` 选型与空值安全性**：
   - **选型口诀**：**外表大、子查询表小用 `IN`**（先算小集合走外表索引）；**外表小、子查询表极大用 `EXISTS`**（外表小循环走大表索引，且 `EXISTS` 一旦命中 1 条立即短路返回 `TRUE`，无需扫完子表）。
   - **`NOT IN` vs `NOT EXISTS`**：永远优先用 **`NOT EXISTS`**！因为 `NOT IN` 子查询只要含 `NULL` 就全盘返回 0 行，而 `NOT EXISTS` 只判断是否有行返回，完全不受列值 `NULL` 污染。
2. **集合比较量词 `ANY`（同义词 `SOME`）与 `ALL` 速查表**：

| 量词表达式 | 离散数学本质 | 通俗语义与等价聚合改写（推荐用聚合更直观） | 当子查询返回 **0 行空集** 时的特殊行为（经典笔试坑！） |
| :--- | :--- | :--- | :--- |
| **`x = ANY (subq)`** | 存在量词（存在一个相等） | 完全等价于 **`x IN (subq)`** | 恒返回 **`FALSE`** |
| **`x > ANY (subq)`** | 存在量词（比任意一个大） | 只要大于子查询**最小值**即可 → 等价于 `x > (SELECT MIN(...) )` | 恒返回 **`FALSE`**（空集中不存在满足条件的元素） |
| **`x > ALL (subq)`** | 全称量词（比所有都大） | 必须大于子查询**最大值** → 等价于 `x > (SELECT MAX(...) )` | 恒返回 **`TRUE`（“空真 Vacuous Truth”！）**——因为空集中找不到任何反例！ |
| **`x != ALL (subq)`** | 全称量词（与所有都不等） | 完全等价于 **`x NOT IN (subq)`** | 子查询空集时返回 **`TRUE`**；子查询含 `NULL` 时恒不为真！ |

---

### 4.4 结果集集合操作符（`UNION` / `UNION ALL` / `INTERSECT` / `EXCEPT`）

- **硬性前提**：参与集合运算的各条 `SELECT` 语句，**列数必须完全相等**，且对应位置列的数据类型兼容；最终结果集的列名以**第一条 `SELECT`** 的列名/别名为准；全局 `ORDER BY` / `LIMIT` 只能写在最后一条语句末尾。

| 集合操作符 | 支持版本 | 是否自动去重 | 性能与底层行为 |
| :--- | :---: | :---: | :--- |
| **`UNION ALL`** | 全版本 | ❌ **不去重，保留全部重复行** | **极快（无临时表去重/排序开销）**；只要业务确认无重复或允许重复，**生产强制优先用 `UNION ALL`**！ |
| **`UNION`** | 全版本 | ✅ **自动整行去重** | 慢——需创建内部临时表对合并结果做哈希/排序去重（同时去除了单表内部原有的重复行）。 |
| **`INTERSECT [ALL]`** | **MySQL 8.0.31+** | 默认去重（加 `ALL` 保留重复计数） | 求两结果集**交集**（MySQL 5.7/早期 8.0 用 `INNER JOIN` 或 `EXISTS` 替代）。 |
| **`EXCEPT [ALL]`** | **MySQL 8.0.31+** | 默认去重 | 求**差集**（在第一条结果集中但不在第二条中；早期版本用 `LEFT JOIN ... WHERE b.id IS NULL` 替代）。 |

---

## 五、聚合统计与分组查询（`GROUP BY` & `HAVING`）

### 5.1 五大聚合函数、`AVG` 布尔求率技巧与 `GROUP_CONCAT`

#### 一、核心聚合函数空值行为速查
> **铁律**：除 **`COUNT(*)` / `COUNT(1)`** 统计包含 `NULL` 的所有可见行外，**`COUNT(列)`、`SUM(列)`、`AVG(列)`、`MAX(列)`、`MIN(列)`、`GROUP_CONCAT(列)` 全部自动忽略 `NULL` 值！**

- ⚠️ **`AVG(col)` 分母丢失 `NULL` 行大坑**：
  假设 4 名员工奖金分别为 `100, 200, 300, NULL`：
  - `SELECT AVG(bonus)` 计算的是 `(100 + 200 + 300) / 3 = 200`（分母是 **3** 而不是 4！）；
  - 若业务要求未发奖金的人按 `0` 拉平全员均值，必须写 **`AVG(IFNULL(bonus, 0))`**（结果为 `600 / 4 = 150`）！
- 💡 **【笔试神技】`AVG(布尔条件表达式)` 一步计算比率 / 留存率 / 及格率**：
  在 MySQL 中，布尔表达式成立返回整数 `1`，不成立返回 `0`。因此**对 `0/1` 布尔表达式求平均值，结果直接就是满足条件的行占比**，完全省去繁琐的 `SUM(CASE WHEN ...) / COUNT(*)`！
  ```sql
  -- 一行算出订单履约完成率（保留 2 位小数）与大额订单数
  SELECT ROUND(AVG(status = 'COMPLETED'), 2) AS completion_rate,
         SUM(amount >= 1000)                 AS big_order_cnt
  FROM orders;
  ```

#### 二、`GROUP_CONCAT()` 分组字符串行合并函数（MySQL 杀手锏）
用于将同一分组内的多行字段值拼接成单个字符串（如把用户的多个角色名拼成 `'admin,editor'`）：
```sql
SELECT dept_id,
       GROUP_CONCAT(DISTINCT emp_name ORDER BY salary DESC SEPARATOR ' | ') AS emp_list
FROM employees
GROUP BY dept_id;
```
- **生产截断大坑**：`GROUP_CONCAT` 结果默认受系统变量 **`group_concat_max_len = 1024`（仅 1KB！）** 限制，超出部分会被**静默截断且不报错**！拼接长列表前务必调大：`SET SESSION group_concat_max_len = 1048576;`。

---

### 5.2 `GROUP BY` 核心机制与 `ONLY_FULL_GROUP_BY` 报错原理解析

- **MySQL 5.7.5+ / 8.0 默认开启 `sql_mode = 'ONLY_FULL_GROUP_BY'`**：
  - **强制规则**：使用 `GROUP BY` 后，`SELECT`、`HAVING`、`ORDER BY` 中出现的非聚合列，**必须要么显式出现在 `GROUP BY` 列表中，要么在函数依赖上由 `GROUP BY` 列唯一决定（例如 `GROUP BY 主键/唯一非空列 u.id` 时，可以直接 `SELECT u.name`！）**。
  - **为什么禁止随便 `SELECT` 非分组列？** 假设 `GROUP BY dept_id`（研发部有张三、李四、王五 3 人），若写 `SELECT dept_id, emp_name, MAX(salary) FROM emp GROUP BY dept_id`：最高薪 `MAX(salary)` 是王五的 50000，但 `emp_name` 却可能随机取出第一行张三的名字，造成**“张冠李戴”的致命数据失真**！
  - **合规解决方案**：① 将需要展示的维度列一并加入 `GROUP BY u.id, u.name`；② 若确认任取一值即可，使用 `ANY_VALUE(emp_name)` 或聚合函数 `MAX(emp_name)`；③ 若要求“最高薪对应的那个人名”，使用**窗口函数 `DENSE_RANK()`** 或**派生表 `JOIN`**（绝不能靠关掉 `ONLY_FULL_GROUP_BY` 掩耳盗铃！）。

---

### 5.3 `WHERE` vs `HAVING` 深度对比与 `WITH ROLLUP` 多维小计

| 对比维度 | `WHERE` 子句 | `HAVING` 子句 |
| :--- | :--- | :--- |
| **过滤时机与对象** | **分组前（第②步）**对原始表/连接表的**单行数据**过滤 | **分组并完成聚合计算后（第⑤步）**对**分组汇总结果**过滤 |
| **能否使用聚合函数** | ❌ **严禁**（写 `WHERE COUNT(*) > 5` 直接语法报错） | ✅ **专门用于过滤聚合结果**（如 `HAVING COUNT(*) > 5 AND SUM(amt) > 1000`） |
| **能否走索引加速** | ✅ 直接利用索引在存储引擎层大幅裁剪参与分组的数据量 | ❌ 仅在内存中过滤分组中间结果，无法利用原始表索引减少扫描量 |
| **性能黄金铁律** | **凡是不涉及聚合函数的普通列条件，必须 100% 前置写在 `WHERE` 中！** | **仅保留必须依赖 `COUNT / SUM / AVG` 等聚合结果的筛选条件，严禁把普通列拖到 `HAVING` 过滤！** |

- **分组汇总总计 `WITH ROLLUP`（报表常用）**：
  - 在 `GROUP BY a, b WITH ROLLUP` 后，结果集会自动追加按 `a` 的小计行（此时 `b` 列显示为 `NULL`）以及全表总计行（`a, b` 均为 `NULL`），可配合 `GROUPING(col)` 函数（MySQL 8.0+，汇总生成的 `NULL` 返回 `1`，真实数据 `NULL` 返回 `0`）替换为 `'小计/合计'` 文案：
    ```sql
    SELECT IF(GROUPING(dept_id), '全公司合计', dept_id) AS dept,
           SUM(salary) AS total_salary
    FROM employees
    GROUP BY dept_id WITH ROLLUP;
    ```
  - *(方言避坑：Oracle / SQL Server / PostgreSQL 支持 `GROUP BY CUBE(a, b)` 全组合交叉小计，**但 MySQL 截至 8.0+ 仅支持 `WITH ROLLUP`，不支持 `CUBE` 关键字**！)*

---

## 六、数据操纵高阶语法与事务锁命令（DML & TCL）

### 6.1 高效写入与幂等 Upsert 三剑客对比

```sql
-- 1. 批量插入（生产大批量写入标配，每批建议 500~1000 行，减少网络 RTT 与事务刷盘次数）
INSERT INTO `t_order` (`order_sn`, `user_id`, `amount`)
VALUES ('SN001', 101, 99.00),
       ('SN002', 102, 188.50);

-- 2. 查询结果直接写入目标表（数据归档/报表落库）
INSERT INTO `order_archive` (`id`, `order_sn`, `amount`)
SELECT `id`, `order_sn`, `amount` FROM `t_order` WHERE `created_at` < '2025-01-01';
```

#### 【高频面试对比】唯一键冲突时的三种幂等写入语法：

| 语法形式 | 冲突时行为 | 底层物理执行机制与隐藏大坑 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **`INSERT IGNORE INTO t ...`** | 命中主键/唯一键冲突时**静默跳过忽略**（受影响行数返回 `0`），不抛异常 | 坑：不仅忽略唯一键冲突，还会把非空/类型截断等错误降级为 Warning 静默吞掉，需谨慎校验数据合法性。 | 批量日志去重导入，“有则跳过，无则新增” |
| **`INSERT INTO t ... ON DUPLICATE KEY UPDATE ...`** | 无冲突执行 `INSERT`（返回 `1`）；有冲突**原位执行 `UPDATE`**（有变更返回 `2`，值未变返回 `0`） | **原位更新（In-place Update）**，主键 ID 保持不变，二级索引仅按需增量维护，**高并发幂等 Upsert 首选**！（注意在 RC 隔离级别下比 RR 更不易发生 Gap Lock 死锁） | 计数器累加、库存/日统计表幂等写入或更新 |
| **`REPLACE INTO t ...`** | 无冲突直接 `INSERT`；有冲突**先物理 `DELETE` 旧行，再 `INSERT` 一条全新行**（返回 `2`） | ⚠️ **生产巨坑**：① 自增主键 `AUTO_INCREMENT` 会被不断消耗飙升产生空洞；② 所有二级索引被迫全量删除重建，I/O 极重；③ 若未写全所有列，旧行中未指定的列会被重置为默认值而丢失数据！ | **生产基本禁用**，一律改用 `ON DUPLICATE KEY UPDATE` |

```sql
-- ON DUPLICATE KEY UPDATE 标准写法（MySQL 8.0.20+ 推荐用 AS new 行别名替代已废弃的 VALUES(col)）
INSERT INTO `daily_stat` (`stat_date`, `pv`, `uv`)
VALUES ('2026-10-08', 1, 1) AS new_row
ON DUPLICATE KEY UPDATE
    `pv` = `daily_stat`.`pv` + new_row.`pv`,
    `uv` = new_row.`uv`;
```

---

### 6.2 多表关联 `UPDATE / DELETE`、`CASE WHEN` 批量差异化更新与 `Error 1093` 避坑

1. **单条 SQL 批量将不同主键更新为不同值（`CASE WHEN` 神技，避免 Java 循环发 N 条 `UPDATE`）**：
   ```sql
   -- 务必在末尾加 WHERE id IN (...) 锁死扫描范围，否则全表每一行都会被加排他锁更新一遍！
   UPDATE `products`
   SET `price` = CASE `id`
           WHEN 101 THEN 99.90
           WHEN 102 THEN 199.00
           WHEN 103 THEN 299.50
           ELSE `price`
       END,
       `stock` = CASE `id`
           WHEN 101 THEN `stock` - 1
           WHEN 102 THEN `stock` - 2
           ELSE `stock`
       END
   WHERE `id` IN (101, 102, 103);
   ```
2. **多表 `JOIN` 关联更新与关联删除**：
   ```sql
   -- 关联更新：将所有 VIP 等级用户的订单折扣率刷为 0.85
   UPDATE `orders` o
   INNER JOIN `users` u ON o.`user_id` = u.`id`
   SET o.`discount` = 0.85
   WHERE u.`vip_level` >= 3 AND o.`status` = 'UNPAID';

   -- 关联删除：删除无对应有效用户的孤儿订单（DELETE 后显式声明要删哪个表别名！）
   DELETE o
   FROM `orders` o
   LEFT JOIN `users` u ON o.`user_id` = u.`id`
   WHERE u.`id` IS NULL;
   ```
3. **⚠️ MySQL 经典报错 `ERROR 1093`（同表子查询更新/删除限制）**：
   - **铁律**：MySQL **严禁在 `UPDATE` 或 `DELETE` 的 `WHERE` 子查询中直接 `SELECT` 同一张目标表**（防止边遍历边修改引发迭代器不一致）：
     ```sql
     -- ❌ 直接报错 ERROR 1093: You can't specify target table 'users' for update in FROM clause
     DELETE FROM `users` WHERE `id` NOT IN (SELECT MIN(`id`) FROM `users` GROUP BY `email`);

     -- ✅ 破局标准写法：在子查询外再套一层 FROM (...) AS tmp 派生表！
     -- 原理：MySQL 会先将派生表 tmp 物化为独立的内部临时表，从而解除对原表 users 的直接引用冲突
     DELETE FROM `users`
     WHERE `id` NOT IN (
         SELECT `min_id` FROM (
             SELECT MIN(`id`) AS `min_id` FROM `users` GROUP BY `email`
         ) AS tmp
     );
     ```

---

### 6.3 事务控制（TCL）与悲观/乐观加锁纯语法速查

> *(关于 ACID、Redo/Undo/Binlog 协同、MVCC ReadView 与 Next-Key Lock 临键锁原理，详见 [`02-Mysql.md` 第三章](./02-Mysql.md)。此处汇总纯 SQL 控制语法)*

```sql
-- 1. 查看与切换会话事务隔离级别
SELECT @@transaction_isolation;  -- MySQL 8.0+（5.7 为 SELECT @@tx_isolation;）
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- 可选 READ UNCOMMITTED / READ COMMITTED / REPEATABLE READ / SERIALIZABLE

-- 2. 显式事务与保存点（SAVEPOINT 局部回滚）
START TRANSACTION;               -- 或 BEGIN;
UPDATE `account` SET `balance` = `balance` - 100 WHERE `id` = 1;
SAVEPOINT sp_after_debit;        -- 设置局部保存点
UPDATE `bonus` SET `points` = `points` + 10 WHERE `user_id` = 1;
ROLLBACK TO SAVEPOINT sp_after_debit; -- 仅回滚积分更新，保留扣款操作！
COMMIT;                          -- 全部失败则执行 ROLLBACK;

-- 3. 当前读加锁语法（必须在开启的事务内生效，且 WHERE 必须命中索引否则退化锁全表！）
SELECT * FROM `seat` WHERE `id` = 58 LOCK IN SHARE MODE;     -- 共享读锁（S 锁，5.7 通用）
SELECT * FROM `seat` WHERE `id` = 58 FOR SHARE;              -- 共享读锁（MySQL 8.0+ 标准语法）
SELECT * FROM `seat` WHERE `id` = 58 FOR UPDATE;             -- 排他写锁（X 锁，阻塞其他读写锁）
SELECT * FROM `seat` WHERE `id` = 58 FOR UPDATE NOWAIT;      -- 8.0+ 抢不到锁立即抛错返回，拒绝线程排队卡死
SELECT * FROM `seat` WHERE `status` = 0 LIMIT 1 FOR UPDATE SKIP LOCKED; -- 8.0+ 跳过已被锁住的行抢下一张空座（秒杀/任务队列神器！）

-- 4. 乐观锁 CAS 原子更新语法（防超卖 & 防丢失更新，无需先加 FOR UPDATE）
UPDATE `goods`
SET `stock` = `stock` - 1, `version` = `version` + 1
WHERE `id` = 1001 AND `stock` >= 1 AND `version` = 12;
```

---

## 七、MySQL 内置函数实战速查大全（含索引避坑指南）

### 7.1 字符串函数全景与高频考点

| 函数签名 | 核心功能与返回值 | 面试/笔试经典示例与避坑要点 |
| :--- | :--- | :--- |
| **`CHAR_LENGTH(s)`**<br>vs **`LENGTH(s)`** | `CHAR_LENGTH` 返回**字符个数**；<br>`LENGTH` 返回**底层字节数** | ⚠️ **高频笔试坑**：`utf8mb4` 下一个汉字占 `3`（常规汉字）或 `4` 字节！<br>查“姓名为 3 个字的员工”**必须写 `WHERE CHAR_LENGTH(name) = 3`**（写 `LENGTH(name)=3` 一个汉字都查不出来）！ |
| **`CONCAT(s1, s2, ...)`**<br>vs **`CONCAT_WS(sep, s1, s2)`** | `CONCAT` 直接拼接（**任一参数为 `NULL` 则整串变 `NULL`！**）；<br>`CONCAT_WS` 指定分隔符拼接（**自动忽略 `NULL` 参数**） | `CONCAT('A', NULL) → NULL`<br>`CONCAT_WS('-', '2026', NULL, '10') → '2026-10'`（地址/标签安全拼接首选） |
| **`SUBSTRING(s, pos, len)`**<br>*(同义词 `SUBSTR`)* | 从位置 `pos`（**从 `1` 开始计数**，负数表示倒数）截取 `len` 个字符 | `SUBSTRING('abcdef', 2, 3) → 'bcd'`<br>`SUBSTRING('abcdef', -2) → 'ef'` |
| **`LEFT(s, n)` / `RIGHT(s, n)`** | 截取最左侧 / 最右侧 `n` 个字符 | 首字母大写其余小写：`CONCAT(UPPER(LEFT(name,1)), LOWER(SUBSTRING(name,2)))` |
| **`SUBSTRING_INDEX(s, delim, count)`** | **按分隔符切割取子串神器**：<br>`count > 0` 返回第 `count` 个分隔符**左边全部**；<br>`count < 0` 返回倒数第 `|count|` 个分隔符**右边全部** | 拆邮箱域名：`SUBSTRING_INDEX('tom@gmail.com', '@', -1) → 'gmail.com'`<br>取 CSV 第 2 项（先左取前2再右取末1）：<br>`SUBSTRING_INDEX(SUBSTRING_INDEX('a,b,c', ',', 2), ',', -1) → 'b'` |
| **`INSERT(s, pos, len, newstr)`** | 从 `pos` 起删掉 `len` 个字符并替换为 `newstr` | **手机号中间 4 位脱敏最简写法**：<br>`INSERT('13812345678', 4, 4, '****') → '138****5678'` |
| **`REPLACE(s, from_s, to_s)`** | 全局替换所有匹配子串（区分大小写） | 统计字符串中逗号出现次数：`CHAR_LENGTH(s) - CHAR_LENGTH(REPLACE(s, ',', ''))` |
| **`TRIM([BOTH\|LEADING\|TRAILING] [rem] FROM s)`** | 默认去除首尾空格（`LTRIM` 去左空格，`RTRIM` 去右空格）；支持去除指定首尾字符 | `TRIM('  admin  ') → 'admin'`<br>`TRIM(BOTH '/' FROM '/api/v1/') → 'api/v1'` |
| **`LOWER(s)` / `UPPER(s)`** | 全转小写 / 全转大写 | `UPPER('abc') → 'ABC'` |
| **`LOCATE(sub, s)` / `INSTR(s, sub)`** | 返回子串首次出现的位置（从 1 起，未找到返 `0`，注意两函数参数顺序相反！） | `LOCATE('admin', email) > 0` |
| **`FIND_IN_SET(str, strlist)`** | 在逗号分隔列表 `'a,b,c'` 中精确查找元素位置（未找到返 `0`，避免 `LIKE '%1%'` 误命中 `'10'`） | `FIND_IN_SET('2', '1,2,12') → 2`（注意：无法走常规 B+ 树索引，大表慎用） |
| **`LPAD(s, len, pad)` / `RPAD`** | 左侧 / 右侧补齐至固定长度 `len` | 生成定长流水号：`LPAD(42, 6, '0') → '000042'` |

---

### 7.2 日期时间函数与【索引友好型】时间范围查询规范

#### 一、高频日期时间函数速查

| 类别 | 函数与语法 | 示例与关键坑点 |
| :--- | :--- | :--- |
| **获取当前时间** | **`NOW()`** / `CURRENT_TIMESTAMP` | 返回**当前 SQL 开始执行时刻**的 `DATETIME`（同一条 SQL 内多次调用值恒定）：`'2026-10-08 14:30:00'` |
| | **`SYSDATE()`** | 返回**函数被调用瞬间**的实时系统时间（若 SQL 内有 `SLEEP`，前后两次 `SYSDATE()` 不同，且主从 Statement 复制不安全，业务优先用 `NOW()`） |
| | **`CURDATE()`** / **`CURTIME()`** | `CURDATE()` 仅返日期 `'2026-10-08'`；`CURTIME()` 仅返时间 `'14:30:00'` |
| **提取与截断** | `DATE(dt)`、`YEAR(dt)`、`MONTH(dt)`、`DAY(dt)`、`HOUR(dt)`、`DAYOFWEEK(dt)`、`LAST_DAY(dt)` | `LAST_DAY('2026-02-10') → '2026-02-28'`（当月最后一天）<br>*(⚠️ 仅用于 `SELECT/GROUP BY` 投影，**严禁套在 `WHERE` 索引列外**！)* |
| **加减偏移** | `dt + INTERVAL n UNIT`<br>`DATE_ADD(dt, INTERVAL n UNIT)`<br>`DATE_SUB(dt, INTERVAL n UNIT)` | 单位支持 `SECOND / MINUTE / HOUR / DAY / WEEK / MONTH / QUARTER / YEAR`<br>例：`NOW() - INTERVAL 7 DAY`、`CURDATE() + INTERVAL 1 MONTH` |
| **时间差计算** | **`DATEDIFF(end_dt, start_dt)`** | 仅计算日期部分相差天数：**`end - start`**（如 `DATEDIFF('2026-10-08','2026-10-01') → 7`） |
| *(⚠️ 顺序相反巨坑！)* | **`TIMESTAMPDIFF(UNIT, start_dt, end_dt)`** | 按指定单位（`SECOND/MINUTE/HOUR/DAY/YEAR`）精确算差值：**`end - start`**（注意与 `DATEDIFF` 参数顺序刚好相反！算周岁年龄首选：`TIMESTAMPDIFF(YEAR, birth_date, CURDATE())`） |
| **格式化转换** | **`DATE_FORMAT(dt, fmt)`**<br>**`STR_TO_DATE(str, fmt)`** | 标准占位符：**`%Y` 四位年**、`%m` 两位月、`%d` 两位日、**`%H` 24小时制**（`%h` 为12小时制）、**`%i` 分钟**（切勿写成 `%m`！）、`%s` 秒。<br>按月统计报表：`DATE_FORMAT(create_time, '%Y-%m')` |
| **Unix 时间戳** | `UNIX_TIMESTAMP([dt])` / `FROM_UNIXTIME(ts)` | 日期与秒级整数时间戳互转 |

#### 二、【面试必考避坑】常见时间范围过滤：索引失效写法 vs 生产标准半开区间写法

假设 `orders` 表在 `create_time (DATETIME)` 列上建有索引 `idx_create_time`：

| 业务需求 | ❌ 错误示范（列套函数，导致 `idx_create_time` 100% 失效！） | ✅ 生产标准写法（半开区间 `[start, end)`，完美命中 `type=range` 索引！） |
| :--- | :--- | :--- |
| **查询今天全部订单** | `WHERE DATE(create_time) = CURDATE()` | `WHERE create_time >= CURDATE()`<br>`  AND create_time < CURDATE() + INTERVAL 1 DAY` |
| **查询昨天全部订单** | `WHERE DATE(create_time) = CURDATE() - INTERVAL 1 DAY` | `WHERE create_time >= CURDATE() - INTERVAL 1 DAY`<br>`  AND create_time < CURDATE()` |
| **查询 2026 年 10 月订单** | `WHERE YEAR(create_time)=2026 AND MONTH(create_time)=10`<br>或 `DATE_FORMAT(create_time,'%Y-%m')='2026-10'` | `WHERE create_time >= '2026-10-01 00:00:00'`<br>`  AND create_time < '2026-11-01 00:00:00'` |
| **查询年龄 > 30 岁用户** | `WHERE TIMESTAMPDIFF(YEAR, birthday, CURDATE()) > 30` | `WHERE birthday <= CURDATE() - INTERVAL 31 YEAR`（把函数算在右侧常量上！） |

---

### 7.3 数学函数与条件流控/空值处理函数

#### 一、核心数学函数
- **`ROUND(x, d)` vs `TRUNCATE(x, d)`**：
  - `ROUND(3.146, 2) → 3.15`（四舍五入保留 `d` 位小数；`d` 省略默认取整）；
  - `TRUNCATE(3.146, 2) → 3.14`（直接暴力截断保留 `d` 位小数，不做四舍五入，`d` 不可省略）。
- **`CEIL(x)` / `CEILING(x)`**（向上取整：`CEIL(3.1) → 4`） vs **`FLOOR(x)`**（向下取整：`FLOOR(3.9) → 3`）。
- **`ABS(x)`**（绝对值）、**`MOD(x, y)`**（取模 `x % y`）、**`POW(x, y)`**（幂运算）、**`SIGN(x)`**（正数返 `1`、零返 `0`、负数返 `-1`）。

#### 二、条件表达式与空值处理四剑客（笔试最高频工具箱）

1. **`CASE` 表达式的两种标准形态**（注意末尾必须以 **`END`** 收尾！）：
   ```sql
   -- 形态 A：简单值匹配（类似 Java switch-case，注意不能匹配 NULL！）
   CASE status
       WHEN 1 THEN '待支付'
       WHEN 2 THEN '已支付'
       ELSE '已关闭'
   END

   -- 形态 B：搜索条件匹配（类似 if - else if - else，支持范围、IS NULL、复杂布尔组合，自上而下短路命中即停！）
   CASE
       WHEN score >= 90 THEN '优秀'
       WHEN score >= 60 THEN '及格'
       WHEN score IS NULL THEN '缺考'
       ELSE '不及格'
   END
   ```
2. **`IF(cond, expr_true, expr_false)`**：MySQL 专属三元运算符简写（如 `IF(gender = 1, '男', '女')`）。
3. **`IFNULL(expr1, default_val)` vs `COALESCE(v1, v2, v3, ...)`**：
   - `IFNULL(a, 0)`：MySQL 专属，仅接受 **2 个**参数（`a` 不为 `NULL` 返 `a`，否则返 `0`）；
   - `COALESCE(mobile, work_phone, email, '未留联系方式')`：**ANSI SQL 跨库标准**，接受 **任意多个**参数，从左向右返回**第一个非 `NULL` 值**（多字段优先级兜底首选）。
4. **`NULLIF(expr1, expr2)` —— 防除零报错神技**：
   - 规则：若 `expr1 = expr2` 则返回 `NULL`，否则返回 `expr1`。
   - **经典场景（安全除法防 `Division by 0` 或优雅控制结果为 `0`）**：
     ```sql
     -- 当 total_views = 0 时，NULLIF(total_views, 0) 返回 NULL，clicks / NULL 得 NULL，再由 IFNULL 兜底为 0.00！
     SELECT IFNULL(ROUND(clicks / NULLIF(total_views, 0), 4), 0.0000) AS ctr FROM ad_stats;
     ```

---

### 7.4 类型转换、JSON 提取、正则表达式与系统函数

1. **显式类型转换 `CAST(expr AS type)` / `CONVERT(expr, type)`**：
   - 注意 MySQL `CAST` 目标类型关键字与建表类型略有不同：**不能写 `INT` 或 `VARCHAR`**，必须写：
     - **`SIGNED` / `UNSIGNED`**（有符号/无符号整数）、**`DECIMAL(M, D)`**、**`CHAR[(N)]`**（字符串）、**`DATE` / `DATETIME` / `TIME`**、**`JSON`**。
     - 例：`SELECT CAST('123' AS UNSIGNED);`、`SELECT CONVERT(name USING utf8mb4);`。
2. **MySQL 5.7+ / 8.0+ JSON 核心操作语法**：
   ```sql
   -- 假设 ext_info 列存储 JSON: {"city": "Shanghai", "tags": ["java", "mysql"]}
   -- ⚠️ 关键区分：-> 等价于 JSON_EXTRACT（返回带双引号的 "Shanghai"），->> 等价于 JSON_UNQUOTE(JSON_EXTRACT)（返回纯文本 Shanghai！）
   SELECT ext_info ->  '$.city'    AS raw_json_str,  -- 结果: "Shanghai"
          ext_info ->> '$.city'    AS clean_city,    -- 结果: Shanghai（WHERE 比较和前端展示必用 ->>！）
          ext_info ->> '$.tags[0]' AS first_tag      -- 结果: java
   FROM user_profile
   WHERE JSON_CONTAINS(ext_info, '"mysql"', '$.tags');
   ```
3. **正则表达式匹配：`REGEXP` 运算符 vs `REGEXP_LIKE()` 函数（8.0+）**：
   - **元字符速记**：`^` 开头、`$` 结尾（省略 `^$` 默认子串包含即匹配！）、`.` 任意单字符、`*` 0或多次、`+` 1或多次、`?` 0或1次、`{n,m}` 次数、`[a-z0-9]` 字符集、`[^0-9]` 取反、`|` 分支或。
   - ⚠️ **MySQL 字符串双重反斜杠转义坑**：在 MySQL SQL 字符串字面量中，`\` 本身会被词法解析器吃掉一层转义，因此正则里的 `\.`（匹配真实小数点）、`\d`、`\s` 在 SQL 中**必须写成双反斜杠 `\\.`、`\\d`、`\\s`**（或用字符类 `[.]`、`[0-9]` 规避转义）！
   - **大小写区分差异**：`col REGEXP ' pattern'` 默认受字段排序规则（Collation，通常 `_ci` 不区分大小写）控制；若需**严格区分大小写**，在 8.0+ 中推荐使用 **`REGEXP_LIKE(col, pattern, 'c')`**（`'c'` 区分大小写，`'i'` 忽略大小写）：
     ```sql
     -- LeetCode 高频真题：校验合法邮箱（字母开头，仅含字母/数字/下划线/点/横线，且严格小写 @leetcode.com 结尾）
     SELECT * FROM Users
     WHERE REGEXP_LIKE(mail, '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$', 'c');
     ```
4. **高频系统信息函数**：
   - `LAST_INSERT_ID()`：返回**当前数据库连接会话**最近一次 `INSERT` 生成的第一个 `AUTO_INCREMENT` 主键值（**基于会话连接隔离，高并发多线程互不干扰，绝对线程安全**！MyBatis `useGeneratedKeys` / `<selectKey>` 底层常用）；
   - `DATABASE()`（当前库名）、`USER()`（当前登录用户）、`VERSION()`（MySQL 版本号）。

---

## 八、MySQL 8.0+ 高阶查询专章（窗口函数 & 递归 CTE 树形方案）

### 8.1 窗口函数（Window Function）完全指南

> **一句话定位**：普通 `GROUP BY` 聚合会把多行**折叠合并成 1 行**（丢失明细行）；而 **窗口函数（`OVER(...)`，MySQL 8.0+ 支持）在不改变原表行数的前提下**，为每一行开辟一个逻辑滑动窗口计算排名、分区汇总或前后行对比，作为新列附加在每行后面。

#### 一、标准语法骨架与三大家族速查

```sql
窗口函数名([参数]) OVER (
    [PARTITION BY 分区列1, ...]          -- 类似分组但绝不合并行；省略则全表视为 1 个大窗口
    [ORDER BY 排序列1 ASC|DESC, ...]     -- 决定排名顺序，且会改变聚合窗口的默认帧边界！
    [ROWS | RANGE BETWEEN 帧起点 AND 帧终点] -- 精确控制滑动窗口边界（可选）
) AS col_alias
```

| 家族分类 | 核心函数 | 核心行为与面试必背区分 |
| :--- | :--- | :--- |
| **1. 排名窗口函数**<br>*(均无参数)* | **`ROW_NUMBER()`** | **连续唯一行号（`1, 2, 3, 4`）**——同分也强行分先后，不并列。适合：**去重保留第一条、严格定量取前 N 条**。 |
| | **`RANK()`** | **并列跳跃排名（`1, 1, 3, 4`）**——同分同名次，占用名额跳过下一名次（奥运奖牌榜模式）。 |
| | **`DENSE_RANK()`** | **并列连续密集排名（`1, 1, 2, 3`）**——同分同名次，且**不跳号**。适合：**每部门薪资前 3 高（含并列全保留）**！ |
| | **`NTILE(k)`** | 将分区内有序数据尽量均匀切分为 `k` 个桶，返回桶编号 `1 ~ k`（如 `NTILE(10)` 取前 10% 头部高净值用户）。 |
| **2. 聚合窗口函数** | `SUM / AVG / COUNT / MAX / MIN(col) OVER(...)` | - **不写 `ORDER BY`**：如 `SUM(sal) OVER(PARTITION BY dept)` → 计算整个部门总薪资（便于算个人占部门薪资比 `sal / SUM(sal) OVER(...)`）；<br>- **写了 `ORDER BY`**：如 `SUM(sal) OVER(PARTITION BY dept ORDER BY month)` → 自动变为**从首行累加到当前行的累计流水账（Running Total）**！ |
| **3. 偏移取值函数**<br>*(同比/环比/连续性神器)* | **`LAG(col, n, def)`** | 取分区排序后**当前行往上第 `n` 行（前 `n` 行）**的 `col` 值（省略 `n` 默认 1；无前驱行时返回默认值 `def`，省略返 `NULL`）。 |
| | **`LEAD(col, n, def)`** | 取分区排序后**当前行往下第 `n` 行（后 `n` 行）**的 `col` 值。 |
| | `FIRST_VALUE(col)` / **`LAST_VALUE(col)`** | 取当前窗口帧的第 1 行 / 最后 1 行的值（⚠️ **必踩大坑见下方窗口帧解析**！）。 |

#### 二、窗口帧（Window Frame）：`ROWS` vs `RANGE` 与默认边界巨坑

1. **默认窗口帧的隐蔽行为（为什么 `LAST_VALUE` 直接写永远等于当前行自己？）**：
   - 当 `OVER()` 中**没写 `ORDER BY`** 时，默认帧是整个分区：`RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`；
   - 一旦 `OVER()` 中**写了 `ORDER BY`**，默认帧会瞬间收缩为 **`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`（从分区第一行到当前行）**！
   - 因此，直接写 `LAST_VALUE(salary) OVER(PARTITION BY dept ORDER BY salary)` 取到的“窗口最后一行”永远是**当前行自身**！要取全分区真正的最后一行，必须显式声明全分区帧：
     `LAST_VALUE(salary) OVER(PARTITION BY dept ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)`。
2. **`ROWS`（按物理行偏移）vs `RANGE`（按逻辑列值/时间跨度偏移）**：
   - **`ROWS` 按物理行号数人头**：哪怕多行值相同或日期有断档，也严格按前后物理行数框定：
     ```sql
     -- 近 3 行移动平均（前 1 行 + 当前行 + 后 1 行）
     AVG(amount) OVER (ORDER BY id ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
     ```
   - **`RANGE` 按 `ORDER BY` 列的实际数值差或 `INTERVAL` 时间跨度框定**（天然处理日期缺失断档或并列同分！）：
     ```sql
     -- 即使中间某天没订单缺行，也能精准汇总【当前订单日期往前推 6 天（共 7 个自然日）】的滚动销售额！
     SUM(amount) OVER (
         ORDER BY sale_date
         RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
     ) AS rolling_7d_gmv
     ```
3. **执行顺序限制**：窗口函数在第⑥步执行（晚于 `WHERE` / `GROUP BY` / `HAVING`），**严禁把窗口函数直接写在 `WHERE` 中**（如 `WHERE ROW_NUMBER() OVER(...) = 1` 必报错），必须外包一层**派生表子查询**或 **`WITH` CTE** 再在外部 `WHERE rn = 1`！

---

### 8.2 公用表表达式（`WITH` CTE）与 `VIEW / 派生表 / 临时表` 四维对比

```sql
-- 支持在同一条 SQL 顶部通过逗号连续定义多个链式 CTE（后一个 CTE 可直接引用前一个 CTE，自顶向下极易阅读维护）
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id
),
high_paid_emp AS (
    SELECT e.id, e.name, e.salary, d.avg_sal
    FROM employees e
    INNER JOIN dept_avg d ON e.dept_id = d.dept_id
    WHERE e.salary > d.avg_sal
)
SELECT * FROM high_paid_emp ORDER BY salary DESC;
```

- **底层优化机制**：若某个 CTE 在主查询中只引用 1 次且无聚合阻塞，MySQL 8.0 优化器默认采用 **`Merge`（合并展开）** 不产生临时表开销；若同一 CTE 被主查询**多次复用**，优化器仅计算 1 次并 **`Materialize`（物化）** 到内部临时表供多处共享（比重复写两次相同派生表子查询性能高一倍！）。

| 对比维度 | `WITH` CTE（8.0+） | `FROM (...) t` 派生表子查询 | `VIEW` 视图 | `TEMPORARY TABLE` 临时表 |
| :--- | :--- | :--- | :--- | :--- |
| **生命周期与作用域** | **仅限当前单条 SQL**，执行完即销毁 | 仅限当前单条 SQL 所在 `FROM` 块 | **永久持久化在数据字典**（跨会话全局复用） | **当前数据库连接会话内**跨多条 SQL 有效，断开自动销毁 |
| **同语句内多次引用** | ✅ **定义 1 次，同语句可多次 `JOIN` 引用** | ❌ 只能用 1 次，复用必须复制粘贴多遍 SQL | ✅ 任意语句无限复用 | ⚠️ MySQL 同一条 SQL 中不能 `JOIN` 同一张临时表两次（报错 `Can't reopen table`） |
| **是否支持递归 / 索引** | ✅ **支持 `WITH RECURSIVE` 递归**；不可手动建索引 | ❌ 不支持递归；不可建索引 | ❌ 只存 SQL 定义不存数据，不可建索引 | ❌ 不支持语法递归；✅ **物理存数据，可手动建索引加速多次复杂大查询** |

---

### 8.3 递归 CTE（`WITH RECURSIVE`）与树形层级数据 5 大方案终极对比

#### 一、`WITH RECURSIVE` 标准模板与三大应用场景（MySQL 8.0+）
- **三条语法铁律**：① 必须写 **`RECURSIVE`** 关键字；② 锚点成员（初始起点）与递归成员之间必须用 **`UNION ALL`**（用 `UNION` 若无需去重防环时部分场景受限，生产标准用 `UNION ALL`，若有图环风险在 8.0+ 可用 `UNION DISTINCT` 去重防死循环）；③ 受系统变量 **`cte_max_recursion_depth = 1000`** 保护防死循环（超限报错，大深度可 `SET SESSION cte_max_recursion_depth = 10000;`）。

```sql
-- 场景 1：生成连续日期序列，配合 LEFT JOIN 补齐折线图报表中“0 订单缺失日期”（高频实战神技！）
WITH RECURSIVE date_seq AS (
    SELECT CAST('2026-10-01' AS DATE) AS dt
    UNION ALL
    SELECT dt + INTERVAL 1 DAY FROM date_seq WHERE dt < '2026-10-07'
)
SELECT s.dt, COUNT(o.id) AS order_cnt, IFNULL(SUM(o.amount), 0) AS daily_gmv
FROM date_seq s
LEFT JOIN orders o ON o.created_at >= s.dt AND o.created_at < s.dt + INTERVAL 1 DAY
GROUP BY s.dt
ORDER BY s.dt;

-- 场景 2：树形部门表 dept(id, name, pid) —— 自顶向下查指定节点（如 id=1 总公司）的所有子孙层级及完整路径
WITH RECURSIVE dept_tree AS (
    -- 锚点成员：查根节点
    SELECT id, name, pid, 1 AS level, CAST(name AS CHAR(500)) AS full_path
    FROM dept WHERE id = 1
    UNION ALL
    -- 递归成员：子节点的 pid = 上一层父节点的 id（若改为 d.id = t.pid 即为自底向上反查所有祖先！）
    SELECT d.id, d.name, d.pid, t.level + 1, CONCAT(t.full_path, ' -> ', d.name)
    FROM dept d
    INNER JOIN dept_tree t ON d.pid = t.id
)
SELECT * FROM dept_tree;
```

#### 二、【后端架构面试必问】树形层级查询 5 种方案选型决策总表

| 方案名称 | 核心实现方式 | 是否改表 | 版本要求 | 核心优缺点与生产选型边界 |
| :--- | :--- | :---: | :---: | :--- |
| **1. 一次查全表 + Java 内存 `Map` 组装树**<br>*(工业界最主流 ⭐⭐⭐⭐⭐)* | `SELECT * FROM menu;` 仅查 **1 次** DB，Java 用 `Collectors.groupingBy(Menu::getPid)` 内存 `O(N)` 组装树（绝非循环查库的 N+1！） | ❌ 否 | 全版本 | **几百~几千条以内的菜单、部门、后台权限树绝对首选**！不耗宝贵的 DB CPU，方便在 Java 层做复杂角色权限剪枝与 Caffeine/Redis 整树缓存。*(数据量达数万条以上严禁全表加载)* |
| **2. MySQL 8.0 递归 CTE**<br>*(按需查局部子树首选 ⭐⭐⭐⭐⭐)* | 单条 `WITH RECURSIVE` 在 DB 内迭代拉取目标分支扁平节点列表 | ❌ 否 | **8.0+** | **万级以上大分类/大组织树、且每次只需查其中某一棵局部子树（或向上溯源祖先链）时首选**！网络传输最小，无需改表。 |
| **3. 固定层级多表 `JOIN`** | 如 `学校 -> 年级 -> 班级 -> 学生` 明确拆为 4 张表逐级 `JOIN` | 独立表 | 全版本 | **层级深度固定且每一层实体属性完全不同**时最清晰、走索引最快；不适用于无限层级自引用树。 |
| **4. 冗余路径枚举法（`path` 字段）**<br>*(5.7 大树读多写少首选 ⭐⭐⭐⭐)* | 表新增 `path VARCHAR(255)` 存 `'1/2/3/'`；查 `id=2` 所有子孙：`WHERE path LIKE '1/2/%'`（右模糊完美走索引！） | ✅ 新增 `path` | 全版本 | 单条普通 SQL 秒查任意子树与全部祖先（直接切分 `path` 字符串）；缺点是**移动节点改父级时需批量级联更新所有子孙的 `path`**，适合改动极少的商品类目/行政区划。 |
| **5. 左右值嵌套集 / 存储过程循环** | 维护 `lft/rgt` 深度优先序号或写 `WHILE` 存储过程 | ✅ / ❌ | 全版本 | **生产基本淘汰**：左右值每次插入一个节点几乎要重刷半张表的编号，写锁灾难；复杂存储过程违背微服务可维护性规范。 |

---

## 九、数据库可编程对象（视图、用户会话变量、UDF 与存储过程）

### 9.1 视图（`VIEW`）核心语法、底层算法与可更新边界

> **核心本质**：视图是一张**只保存 `SELECT` 查询定义、不物理存储真实行数据的虚拟表**。每次查询视图时，底层都会实时读取基表（Base Table）最新数据；基表数据变更会立刻体现在视图结果中。

```sql
-- 1. 创建或替换脱敏视图（隐藏手机号中间四位与薪资敏感列，用于按角色开放只读数据权限）
CREATE OR REPLACE ALGORITHM = MERGE VIEW `v_public_emp` AS
SELECT `id`, `emp_name`, `dept_id`, INSERT(`mobile`, 4, 4, '****') AS `masked_mobile`
FROM `employees`
WHERE `status` = 1
WITH CHECK OPTION; -- 若通过该视图插入/修改数据，强制校验修改后仍必须满足 WHERE status = 1

-- 2. 查看视图定义与删除视图
SHOW CREATE VIEW `v_public_emp`;
DROP VIEW IF EXISTS `v_public_emp`;
```

- **两种视图执行算法（`ALGORITHM`）**：
  1. **`MERGE`（合并重写，默认最优）**：将外层查视图的 `WHERE` 条件直接合并展开进视图定义 SQL 中一起优化，**可正常命中基表索引**；
  2. **`TEMPTABLE`（物化临时表）**：当视图包含聚合、`DISTINCT`、`UNION` 等无法合并的结构时，被迫先执行视图生成内部临时表再外层过滤，**外层条件无法下推走基表索引，大表性能极差**。
- **不可更新视图的 6 类语法特征**（若视图定义包含以下任一结构，则**严禁对该视图执行 `INSERT / UPDATE / DELETE`**）：
  ① 包含聚合函数（`SUM/COUNT/AVG/MAX/MIN`）或窗口函数；② 包含 `DISTINCT`；③ 包含 `GROUP BY` 或 `HAVING`；④ 包含 `UNION` / `UNION ALL`；⑤ `SELECT` 投影列表中包含子查询或常量计算列；⑥ 引用了不可更新视图或复杂多表聚合连接。
  *(⚠️ 跨库方言勘误：SQL Server 支持 `INSTEAD OF` 触发器强行更新复杂视图、以及 `sp_helptext / sp_refreshview`，Oracle 支持 `CONNECT BY`，**MySQL 均无此类语法**，切勿混记！)*

---

### 9.2 MySQL 用户会话变量（`@var`）与三类变量作用域对比

```sql
-- 1. 赋值运算符死规矩：SET 中可用 = 或 :=，但在 SELECT 语句中赋值【必须用 :=】（因为 SELECT 中 = 是相等比较！）
SET @min_sal = 10000;
SELECT @max_sal := MAX(salary) FROM employees;

-- 2. 经典实战（MySQL 5.7 无窗口函数时代模拟 ROW_NUMBER() 行号）
SELECT (@rn := @rn + 1) AS row_num, e.id, e.emp_name, e.salary
FROM employees e, (SELECT @rn := 0) AS init_vars
ORDER BY e.salary DESC;
```

| 变量类别 | 前缀标志 | 声明方式与类型 | 作用域与生命周期 |
| :--- | :---: | :--- | :--- |
| **系统变量** | **`@@`** | MySQL 内核预定义（`@@global.max_connections` / `@@session.sql_mode`） | `GLOBAL` 全局跨连接生效（重启需配 `my.cnf`）；`SESSION` 仅当前连接生效 |
| **用户会话变量** | **`@`** | **无需 `DECLARE`**，弱类型即赋即用（未初始化默认值为 `NULL`） | **当前单个 TCP 数据库连接会话内**跨多条 SQL 有效，连接关闭自动销毁（连接池复用时注意残留污染！） |
| **局部变量** | **无前缀** | **必须在 `BEGIN...END` 首部用 `DECLARE v INT DEFAULT 0;` 强类型声明** | 仅在存储过程 / 自定义函数 / 触发器当前 `BEGIN...END` 块内有效 |

---

### 9.3 自定义函数（UDF）、存储过程与触发器语法速查

#### 一、自定义函数（`FUNCTION`）vs 存储过程（`PROCEDURE`）核心对比

| 对比维度 | 自定义函数（`CREATE FUNCTION`） | 存储过程（`CREATE PROCEDURE`） |
| :--- | :--- | :--- |
| **返回值与参数** | 仅支持输入参数；**必须声明 `RETURNS 类型` 且函数体必须 `RETURN` 单个值** | 支持 **`IN`（输入）、`OUT`（输出）、`INOUT`** 多参数，可返回 0 个或多个结果集 |
| **调用方式** | **可直接嵌在 `SELECT / WHERE` 表达式中像内置函数一样调用**：`SELECT fn_tax(sal)` | **只能用独立语句 `CALL proc_name(...)` 调用**，不可写在 `SELECT` 中 |
| **事务控制** | ❌ 函数内部**不允许**显式 `COMMIT` / `ROLLBACK` | ✅ 过程内部**允许**完整开启、提交与回滚事务 |

```sql
-- 自定义函数经典模板（LeetCode 177：获取第 N 高薪水）
DELIMITER $$
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE offset_val INT;
    SET offset_val = N - 1;
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET offset_val   -- 注意：LIMIT/OFFSET 后只能跟常量或局部变量，不能直接写表达式 N-1！
    );
END$$
DELIMITER ;
```

> **存储过程与触发器避坑指南**：
> 1. **存储过程参数名严禁与表字段同名**（如参数叫 `user_id`，写 `WHERE user_id = user_id` 会被解析为参数自己等于自己恒为 `TRUE`，酿成全表误删事故！）；修改逻辑需 `DROP PROCEDURE IF EXISTS xxx` 后重建。
> 2. **触发器（`TRIGGER`）**：MySQL 仅支持行级 `FOR EACH ROW` 下的 `{BEFORE | AFTER} × {INSERT | UPDATE | DELETE}` 共 6 种时机（`INSERT` 只有 `NEW`，`DELETE` 只有 `OLD`，`UPDATE` 既有 `OLD` 也有 `NEW`）。
> 3. **微服务工程规范**：阿里《Java 开发手册》**明确禁止将核心业务逻辑写进存储过程或触发器**（调试黑盒、无版本灰度、耗尽宝贵的 DB 主库 CPU、分库分表完全失效）。

---

## 十、笔试与面试高频手写 SQL 经典母题 20 道（去重纠错精编题库）

### 专题一：数据查重、安全删重与连续相同值（题 1 ~ 题 3）

#### 题 1：查找重复数据及重复次数
> **需求**：找出 `users` 表中重复的 `email` 及其出现次数。
```sql
SELECT email, COUNT(*) AS dup_cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

#### 题 2：删除重复记录，仅保留每组 `id` 最小的那一条（⚠️ 必避 `Error 1093` 坑！）
> **需求**：原地删除 `users` 表中 `email` 重复的多余行，只留 `id` 最小的一条。
```sql
-- ✅ 解法 A（最推荐，5.7/8.0 通杀，语法最简）：自连接多表 DELETE，找出“同邮箱且存在比自己 id 更小记录”的较大行删掉
DELETE u1
FROM users u1
INNER JOIN users u2 ON u1.email = u2.email AND u1.id > u2.id;

-- ✅ 解法 B（双层派生表包裹法，专门规避 MySQL Error 1093 同表子查询更新报错）
DELETE FROM users
WHERE id NOT IN (
    SELECT min_id FROM (
        SELECT MIN(id) AS min_id FROM users GROUP BY email
    ) AS keep_ids
);
```

#### 题 3：查找连续出现至少 3 次的数字（LeetCode 180）
> **需求**：表 `Logs(id, num)` 按 `id` 递增，找出所有连续出现 `>= 3` 次的 `num`。
```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT num,
           LAG(num, 1)  OVER (ORDER BY id) AS prev_num,
           LEAD(num, 1) OVER (ORDER BY id) AS next_num
    FROM Logs
) AS t
WHERE num = prev_num AND num = next_num;
```

---

### 专题二：极值对应整行、全局第 N 高与分组 Top-N（题 4 ~ 题 6）

#### 题 4：查询每个用户最新的一笔订单完整明细（分组极值对应整行）
```sql
SELECT id, user_id, order_sn, amount, created_at
FROM (
    SELECT o.*,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC, id DESC) AS rn
    FROM orders o
) AS t
WHERE rn = 1;
```

#### 题 5：查询第二高的薪水，若不存在必须返回 `NULL`（LeetCode 176 经典空集坑）
> **陷阱**：若表中只有 1 条记录，直接 `SELECT DISTINCT salary ... LIMIT 1 OFFSET 1` 返回的是 **0 行空结果集（Empty Set）** 而非包含 `NULL` 的 1 行！**必须在最外层套一层标量子查询 `SELECT (...)`** 将空集自动转化为标量 `NULL`（或配合 `MAX()` 聚合）：
```sql
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

#### 题 6：分组 Top-N —— 查询每个部门薪资排名前 3 的员工（含并列高薪）
> **陷阱**：① 并列前 3 名都需保留，必须用 **`DENSE_RANK()`** 而非 `ROW_NUMBER()`；② 别名切勿裸写 8.0 保留字 `rank`，统一用 `rk`！
```sql
SELECT d.name AS Department, t.name AS Employee, t.salary AS Salary
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (PARTITION BY e.departmentId ORDER BY e.salary DESC) AS rk
    FROM Employee e
) AS t
INNER JOIN Department d ON t.departmentId = d.id
WHERE t.rk <= 3;
```

---

### 专题三：连续登录、次日留存率、同比环比与沉默用户（题 7 ~ 题 10）

#### 题 7：【大厂压轴母题】找出连续登录天数 `>= 3` 天的用户（及最大连续登录天数）
> **满分核心算法（“日期减行号等差恒定法”）**：
> 1. **第一步必须先按 `(user_id, DATE(login_time))` 去重**！（否则同一天登录 3 次会被误算成连续 3 天！）
> 2. 用 `ROW_NUMBER()` 按用户分区、登录日期升序生成连续序号 `rn = 1, 2, 3...`；
> 3. 若日期也是连续递增 1 天的（如 `10-01, 10-02, 10-03`），则 **`DATE_SUB(login_date, INTERVAL rn DAY)` 得到的基准锚点日期必然完全相等**（全等于 `09-30`）！按 `(user_id, anchor_date)` 分组 `COUNT(*)` 即为每段连续登录的天数！
```sql
WITH distinct_logins AS (
    SELECT DISTINCT user_id, DATE(login_time) AS login_date
    FROM user_login_log
),
numbered_logins AS (
    SELECT user_id, login_date,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date ASC) AS rn
    FROM distinct_logins
),
streak_groups AS (
    SELECT user_id,
           DATE_SUB(login_date, INTERVAL rn DAY) AS anchor_date,
           COUNT(*) AS consecutive_days
    FROM numbered_logins
    GROUP BY user_id, DATE_SUB(login_date, INTERVAL rn DAY)
)
SELECT user_id, MAX(consecutive_days) AS max_streak
FROM streak_groups
GROUP BY user_id
HAVING MAX(consecutive_days) >= 3;
```

#### 题 8：计算新用户次日留存率（LeetCode 550）
> **定义**：在首次登录后的第二天再次登录的玩家数 / 总玩家数（保留 2 位小数）。
```sql
WITH first_login AS (
    SELECT player_id, MIN(event_date) AS first_date
    FROM Activity
    GROUP BY player_id
)
SELECT ROUND(AVG(a.player_id IS NOT NULL), 2) AS retention_rate
FROM first_login f
LEFT JOIN Activity a
       ON f.player_id = a.player_id
      AND a.event_date = DATE_ADD(f.first_date, INTERVAL 1 DAY);
```

#### 题 9：月度 GMV 累计流水与环比增长率（MoM）
```sql
WITH monthly_sales AS (
    SELECT DATE_FORMAT(created_at, '%Y-%m') AS ym,
           SUM(amount)                      AS gmv
    FROM orders
    WHERE status = 'PAID'
    GROUP BY DATE_FORMAT(created_at, '%Y-%m')
)
SELECT ym,
       gmv,
       SUM(gmv) OVER (ORDER BY ym)                                                     AS cumulative_gmv,
       LAG(gmv, 1) OVER (ORDER BY ym)                                                  AS prev_month_gmv,
       ROUND((gmv - LAG(gmv, 1) OVER (ORDER BY ym))
             / NULLIF(LAG(gmv, 1) OVER (ORDER BY ym), 0) * 100, 2)                     AS mom_growth_pct
FROM monthly_sales
ORDER BY ym;
```

#### 题 10：查找最近 30 天内从未登录过的沉默用户
> **陷阱**：刚注册从未登录过的用户 `last_login_time` 为 `NULL`，若只写 `last_login_time < NOW() - INTERVAL 30 DAY` 会把从未登录的 `NULL` 用户全部漏掉！
```sql
SELECT id, username, last_login_time
FROM users
WHERE last_login_time < NOW() - INTERVAL 30 DAY
   OR last_login_time IS NULL;
```

---

### 专题四：行列转换（Pivot & Unpivot）与条件翻转（题 11 ~ 题 13）

#### 题 11：行转列（Pivot）—— 学生各科成绩纵表转横表
> **需求**：原表 `scores(student_name, subject, score)`，转为一行展示 `姓名 | 语文 | 数学 | 英语 | 总分`。
```sql
SELECT student_name,
       MAX(CASE WHEN subject = '语文' THEN score ELSE 0 END) AS `语文`,
       MAX(CASE WHEN subject = '数学' THEN score ELSE 0 END) AS `数学`,
       MAX(CASE WHEN subject = '英语' THEN score ELSE 0 END) AS `英语`,
       SUM(score)                                            AS `总分`
FROM scores
GROUP BY student_name;
```

#### 题 12：列转行（Unpivot）—— 多门店价格宽表拆为窄表（LeetCode 1795）
> **需求**：原表 `Products(product_id, store1, store2, store3)`，拆为 `(product_id, store, price)` 且剔除未上架的 `NULL` 门店（使用常量投影列 + **`UNION ALL`** 避免无谓去重开销）：
```sql
SELECT product_id, 'store1' AS store, store1 AS price FROM Products WHERE store1 IS NOT NULL
UNION ALL
SELECT product_id, 'store2' AS store, store2 AS price FROM Products WHERE store2 IS NOT NULL
UNION ALL
SELECT product_id, 'store3' AS store, store3 AS price FROM Products WHERE store3 IS NOT NULL;
```

#### 题 13：单条 `UPDATE` 原地翻转性别字段（不可借助临时表）
```sql
UPDATE Salary
SET sex = CASE sex
        WHEN 'm' THEN 'f'
        WHEN 'f' THEN 'm'
        ELSE sex
    END;
```

---

### 专题五：反连接、聚合 `COUNT` 空值坑、关系除法与深分页（题 14 ~ 题 20）

#### 题 14：反连接查找从未下过单的客户（两种生产高性能写法）
```sql
-- 写法 1：LEFT JOIN 反连接（WHERE 必须判断右表非空主键 o.id IS NULL）
SELECT c.id, c.name
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

-- 写法 2：NOT EXISTS（完全免疫 NULL 值陷阱，避免 NOT IN 灾难）
SELECT c.id, c.name
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customer_id = c.id
);
```

#### 题 15：统计【所有用户】（含 0 单新用户）的订单数与消费总额（⚠️ `COUNT(*)` 经典陷阱！）
> **致命陷阱**：`LEFT JOIN` 后，0 单用户也会保留 1 行（右表 `o.*` 全为 `NULL`）。若写 `COUNT(*)` 会把这行 `NULL` 算成 **1 单**！**必须写 `COUNT(o.id)` 统计右表非空主键，并用 `IFNULL(SUM(...), 0)` 兜底金额**！
```sql
SELECT u.id,
       u.username,
       COUNT(o.id)               AS order_count,
       IFNULL(SUM(o.amount), 0)  AS total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'PAID' -- 注意：右表状态过滤必须放 ON 里，放 WHERE 会丢掉 0 单用户！
GROUP BY u.id, u.username;
```

#### 题 16：查找订单金额高于【该用户本人历史平均订单金额】的所有订单
```sql
-- 窗口函数单次扫描解法（比传统 WHERE amount > (SELECT AVG(...) WHERE user_id = o.user_id) 相关子查询快数倍）
SELECT id, user_id, amount, ROUND(user_avg_amt, 2) AS avg_amt
FROM (
    SELECT o.*,
           AVG(amount) OVER (PARTITION BY user_id) AS user_avg_amt
    FROM orders o
) AS t
WHERE amount > user_avg_amt;
```

#### 题 17：关系除法 —— 查询选修了“课程表全部课程”的学霸学生
```sql
SELECT student_id
FROM student_courses
GROUP BY student_id
HAVING COUNT(DISTINCT course_id) = (SELECT COUNT(*) FROM courses);
```

#### 题 18：查询两张用户表同时存在的交集用户 ID
```sql
-- 通用写法（确保双表 user_id 去重后内连接，或 8.0.31+ 直接用 SELECT user_id FROM tableA INTERSECT SELECT user_id FROM tableB）
SELECT DISTINCT a.user_id
FROM tableA a
INNER JOIN tableB b ON a.user_id = b.user_id;
```

#### 题 19：千万级订单表深分页查询（第 50,001 页，每页 20 条）标准双解
```sql
-- 方案 A：覆盖索引延迟关联法（支持任意跳页，子查询走 idx_status_id(status, id) 零回表，外层仅回表 20 次）
SELECT a.*
FROM orders a
INNER JOIN (
    SELECT id FROM orders WHERE status = 1 ORDER BY id ASC LIMIT 20 OFFSET 1000000
) AS b ON a.id = b.id;

-- 方案 B：主键游标连续翻页法（携带上一页末尾 last_id = 1054320，耗时恒定 1ms）
SELECT * FROM orders WHERE status = 1 AND id > 1054320 ORDER BY id ASC LIMIT 20;
```

#### 题 20：递归统计指定主管（如 `id = 1`）名下所有直接与间接下属列表及团队总薪资
```sql
WITH RECURSIVE sub_tree AS (
    SELECT id, name, manager_id, salary, 0 AS depth
    FROM employees
    WHERE id = 1
    UNION ALL
    SELECT e.id, e.name, e.manager_id, e.salary, s.depth + 1
    FROM employees e
    INNER JOIN sub_tree s ON e.manager_id = s.id
)
SELECT id, name, depth, salary,
       SUM(salary) OVER () AS team_total_salary
FROM sub_tree
WHERE depth > 0; -- 排除主管本人，仅展示所有下属
```
