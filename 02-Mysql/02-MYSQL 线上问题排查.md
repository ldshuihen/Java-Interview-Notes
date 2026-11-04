### JDBC 批量插入越插越慢的常见原因

典型规律：**前期靠内存“吸收”写入，后期退化成磁盘同步刷盘**。按常见程度排序：

#### 数据库层（以 MySQL/InnoDB 为例）

**1. Redo log 太小，写满后被迫刷脏页（最常见）**
- redo log 是环形复用的，写满后必须把脏页刷盘、推进 checkpoint 才能继续写
- 症状：前期飞快，插入到某个量级后**断崖式变慢**
- 检查：`SHOW ENGINE INNODB STATUS\G` 的 LOG 段看 checkpoint age；`Innodb_log_waits` 是否增长
- 解决：调大 redo log（8.0.30+ 用 `innodb_redo_log_capacity=4G`，老版本 `innodb_log_file_size`）

**2. Buffer pool 被写满**
- 前期脏页全在内存里，后期需要淘汰脏页腾空间，每次插入伴随刷盘
- 症状：**逐渐变慢**（渐变而非断崖）
- 检查：`Innodb_buffer_pool_wait_free` 增长；`iostat -x 1` 磁盘 util 接近 100%
- 解决：调大 `innodb_buffer_pool_size`、`innodb_io_capacity`，或换更快的磁盘

**3. 单个事务太大，undo log 持续膨胀**

- 如果所有插入放在一个事务里不 commit，undo 越来越大，内存和刷盘压力递增
- 解决：分批提交，例如每 1000~5000 条 commit 一次

**4. 索引页分裂 / 随机键插入**
- 主键是 UUID 等随机值，或二级索引无序，B+ 树频繁页分裂、缓存命中率越来越低
- 解决：用自增主键按序插入；巨量导入可先删二级索引，导完再重建

**5. 其他**
- 磁盘空间不足、表空间自动扩容停顿
- `sync_binlog=1` + 慢磁盘（每次 commit 都 fsync binlog）
- 并发长事务阻塞 purge，history list 持续增长
- 表上有触发器/审计逻辑

#### 应用 / JDBC 层

**1. MySQL 没开 `rewriteBatchedStatements`**
- 不加此参数，`executeBatch` 实际是一条一条发 SQL，“批量”是假的
- URL 加：`jdbc:mysql://...?rewriteBatchedStatements=true`

**2. 批太大或数据在内存累积 → GC 压力**
- 一次 addBatch 几十万条、或把全部待插数据放在 List 里，GC 越来越频繁
- 检查 GC 日志（`jstat -gcutil`）

**3. 资源泄漏**
- 每批 `new PreparedStatement`/ResultSet 未关闭，游标和内存逐渐累积，越跑越慢

#### 快速定位方法

1. 给每批记录耗时日志，区分模式：
   - **渐变变慢** → buffer pool / undo / GC
   - **某点后断崖** → redo log 写满
2. 插入过程中执行：
```sql
SHOW ENGINE INNODB STATUS\G   -- 看 dirty pages、checkpoint age、history list
SHOW GLOBAL STATUS WHERE Variable_name IN
 ('Innodb_log_waits','Innodb_buffer_pool_wait_free','Innodb_data_pending_writes');
```
3. `iostat -x 1` 看磁盘 util/await 是否打满

#### 常用优化组合

- 每批 1000~5000 条，批后 commit（不要一个超大事务）
- URL 加 `rewriteBatchedStatements=true`
- 导入期间临时设 `innodb_flush_log_at_trx_commit=2`、`sync_binlog=0`，导完恢复
- 调大 redo log 和 buffer pool
- 自增主键、按序插入
- 数据量特别大（千万级以上）直接用 `LOAD DATA INFILE`，比 batch insert 快一个数量级

如果能说明具体数据库类型、单批条数、事务提交方式，我可以帮你更精确地定位。

# MySQL 线上问题排查

下面按照 **排查思路 → 常用命令 → 慢SQL、锁问题、CPU高、IO高、内存、死锁、连接爆满、卡顿、事务、索引、主从故障、常见坑、排查流程模板**，全部为线上实战流程，适配 InnoDB 生产环境。

> 前置说明：MySQL8.0 废弃 `show‑innodb‑status`，改用 performance_schema、sys 库；5.7 两种方式都支持。

## 一、通用排查顺序（标准线上排查流程，必背）

线上故障优先按照下面顺序定位，不要盲目执行 SQL

1. 确认业务现象：卡顿、超时、报错、慢、死锁、连接失败、CPU飙升
2. 查看服务器指标：CPU、磁盘IO、内存、网络、磁盘满没满
3. 查看 MySQL 整体状态：连接数、线程、运行负载
4. 抓取正在运行的实时 SQL、长事务
5. 排查慢查询日志 / performance_schema 历史慢SQL
6. 排查锁等待、行锁、表锁、MDL锁、死锁
7. 检查索引、全表扫描、隐式转换、索引失效
8. 事务问题：长事务、未提交事务、事务回滚堆积
9. 主从延迟、主从报错
10. 参数配置、缓冲区、临时表、磁盘满

---

## 二、基础状态查看命令

### 2.1 查看所有会话、正在执行SQL

`show processlist;` 是 MySQL 命令，**查看当前数据库所有正在执行/等待的客户端连接线程**。
> 相当于看 MySQL 当前正在干什么，排查慢查询、锁阻塞、sleep连接、卡死线程最常用命令。

```sql
show processlist;
-- 显示完整SQL（不加的话sql字段会截断，只显示前100字符）
show full processlist;
```

#### 字段详解（面试重点）

|字段|含义|
|---|---|
|Id|线程ID，杀掉线程用 `kill id;`|
|User|哪个数据库用户|
|Host|客户端IP+端口，定位来源机器|
|db|操作的数据库名|
|Command|线程状态：Sleep、Query、Connect等|
|Time|**持续时间，单位秒**|
|State|SQL当前执行状态，核心排查字段|
|Info|正在执行的SQL语句，`show full processlist` 显示完整SQL|

#### Command 常见值

1. `Query`：正在执行SQL（select/update/delete）
2. `Sleep`：线程空闲，连接保活，等待应用下发新SQL；大量sleep一般连接池配置问题
3. `Connect`：正在建立连接

#### State 常见状态（故障排查高频）

- `Sending data`：正在读取&处理数据，大概率慢查询，扫描大量数据
- `Copying to tmp table`：创建临时表，group by / order by 无索引
- `Locked`：被锁阻塞，等待行锁/表锁；发生锁等待
- `Sorting result`：文件排序，缺少索引
- `Waiting for table metadata lock`：元数据锁MDL锁，alter table卡住经典现象
- `Waiting for row lock`：等待行锁，事务没提交，锁住行

#### 常用实操

1. **杀掉卡住的线程**
```sql
kill 123; -- 123是Id，终止这个数据库线程
```
> ⚠️ kill只会终止数据库侧线程；**不会停止Java应用程序代码**，应用会收到异常。

2. 只看正在干活的线程，过滤Sleep空闲连接
```sql
show full processlist where command != 'Sleep';
```

#### 常见业务场景

1. 接口响应很慢，数据库怀疑有慢SQL：执行`show full processlist`，看`Time`大、State为`Sending data`的线程。
2. `alter table`一直卡住：找`Waiting for table metadata lock`，找到持有MDL锁的长事务线程kill掉。
3. 大量线程处于`Locked`，说明发生锁等待，有事务长时间不提交。
4. 大量Sleep连接：应用连接池最大连接数设置过大，或者事务不释放。

#### 补充

- `show processlist`只列出**100条**线程；想看全部：查询 `information_schema.processlist` 表
```sql
select * from information_schema.processlist where command != 'Sleep';
```
- 生产不要频繁执行，本身也是一条查询。

#### 面试简答版

> show processlist 查看MySQL当前所有客户端连接线程，show full processlist查看完整SQL；Id用来kill线程；Time代表执行秒数；State看执行状态；常用于排查慢SQL、MDL锁、行锁阻塞、长事务。Sleep代表空闲连接；Sending data代表慢查询扫描大量数据；Waiting for table metadata lock是MDL锁等待。

#### 小坑提醒

`kill id`，**不能kill自己当前会话id**；kill之后，客户端会断开，已经执行一半的事务会回滚。

### 2.2 查看全局状态、运行指标

```sql
-- 会话级状态
show status;
-- 全局状态，线上优先看这个
show global status like '%xxx%';
```

高频监控项

```sql
-- 当前连接数
show global status like 'Threads_connected';
-- 正在运行活跃线程
show global status like 'Threads_running';
-- 连接失败次数
show global status like 'Aborted_connects';
-- 慢查询数量
show global status like 'Slow_queries';
-- Innodb行锁等待次数
show global status like 'Innodb_row_lock_waits';
```

### 2.3 查看系统变量配置

```sql
show variables like '%long_query%'; --慢查询阈值
show variables like '%innodb_buffer_pool_size%';
show variables like '%max_connections%';
show variables like '%transaction_isolation%';
```

---

## 三、慢SQL排查（线上80%故障根源）

### 方式1：慢查询日志

1. 开启慢日志

```ini
slow_query_log=ON
slow_query_log_file=/usr/local/mysql/slow.log
long_query_time=1   --超过1秒即为慢SQL
log_queries_not_using_indexes=ON --记录没有走索引的SQL
```

2. 工具分析慢日志

- `mysqldumpslow` mysql自带分析工具

```bash
# 查看耗时最高的前20条
mysqldumpslow -s t -t 20 /usr/local/mysql/slow.log
```

### 方式2：performance_schema 实时抓取慢SQL（最推荐，无需开启慢日志）

#### MySQL5.7+ 查看最近历史SQL

```sql
-- 开启语句监控
update performance_schema.setup_instruments set ENABLED = 'YES', TIMED='YES' where NAME = 'statement/sql/%';
update performance_schema.setup_consumers set ENABLED = 'YES' where NAME like '%events_statements_history%';

-- 查询历史执行过的SQL、耗时、扫描行数、返回行数
SELECT
  sql_text,
  TIMER_WAIT/1000000000 as cost_sec,
  ROWS_EXAMINED 扫描行数,
  ROWS_SENT 返回行数
FROM performance_schema.events_statements_history
ORDER BY TIMER_WAIT DESC LIMIT 30;
```

> 核心判断标准：**扫描行数远大于返回行数 → 索引失效、全表扫描**

#### 方式3：sys库一键排查 Top 慢SQL（MySQL8.0开箱即用，5.7可导入sys库）

```sql
select * from sys.statement_analysis order by exec_time desc limit 20;
```

##### SQL优化排查清单

1. `explain` 分析执行计划

```sql
explain select * from user where phone='13800138000';
```

重点字段

- `type`：性能从优到劣 `system>const>eq_ref>ref>range>index>ALL`
- `ALL` = 全表扫描，必须优化索引
- `key`：实际命中索引，null=没走索引
- `rows`：预估扫描行数
- `Extra`：
  - `Using filesort` 需要额外排序，缺少排序索引
  - `Using temporary` 创建临时表，group by、多表连接缺陷
  - `Using index` 覆盖索引，最优

#### 索引失效常见原因

1. where条件索引字段做函数运算 `date(create_time)`
2. 隐式类型转换，字符串不加引号
3. like 左通配 `like '%xxx'`
4. or 连接无索引字段
5. 事务隔离级别、优化器选择放弃索引
6. 索引区分度太低（性别这类低基数索引）

---

## 四、锁问题线上排查（死锁、行锁等待、MDL锁、表锁）

### 4.1 查看 InnoDB 锁等待（5.7）

```sql
show engine innodb status;
```

输出内容重点查看

- `TRANSACTIONS`：事务列表、未提交长事务
- `LOCK WAIT`：锁等待SQL、等待哪个行锁
- `DEADLOCK`：最近死锁日志

### 4.2 MySQL8.0 锁查询（performance_schema，推荐）

#### 1. 查询正在发生的行锁等待

```sql
SELECT * FROM performance_schema.data_locks; --已经持有的锁
SELECT * FROM performance_schema.data_lock_waits; --正在等待锁的会话
```

字段说明

- `ENGINE_TRANSACTION_ID` 事务ID
- `LOCK_TYPE` RECORD 行锁、GAP间隙锁、NEXT‑KEY临键锁
- `LOCK_TABLE` 被锁数据表
- `LOCK_INDEX` 命中哪个索引
- `LOCK_MODE` X排他、S共享锁

#### 2. MDL元数据锁排查（alter 卡住经典故障）

```sql
select * from performance_schema.metadata_locks;
```

> 排查思路：找到持有 MDL 读锁长时间不释放的长事务会话 kill 掉，DDL立刻执行成功

### 4.3 死锁排查

#### 死锁产生条件

1. 多个事务互相持有对方需要的行锁
2. 更新行顺序不一致
3. 长事务占用锁时间太久

#### 死锁日志开启

```sql
show variables like 'innodb_print_all_deadlocks';
set global innodb_print_all_deadlocks=1;
```

死锁日志写入错误日志；8.0也可以查询 `performance_schema.data_locks`

#### 线上死锁解决方案

1. 统一所有业务更新行的主键顺序
2. 缩小事务，尽早commit，不要事务内做网络请求、业务耗时逻辑
3. 合理创建索引，减少锁范围（普通索引RR级别会触发间隙锁）
4. 避免select ... for update 大范围加锁

### 4.4 意向锁、表锁阻塞排查

当出现表级锁等待，检查是否有人手动执行 `lock tables`

---

## 五、长事务 & 未提交事务排查（线上头号杀手）

长事务会引发：锁长时间占用、死锁、MDL锁阻塞、undo日志膨胀、回滚耗时、数据库卡顿

```sql
-- 查询运行超过5秒未提交事务（5.7/8.0通用）
SELECT
  trx_id,
  trx_state,
  trx_mysql_thread_id 会话id,
  trx_started,
  now() - trx_started as 耗时秒,
  trx_query
FROM information_schema.INNODB_TRX
WHERE trx_state = 'RUNNING' AND (NOW() - trx_started) > 5
ORDER BY 耗时秒 DESC;
```

拿到会话ID直接 kill。

> 坑点：代码开启事务之后，异常没有回滚、忘记commit；前端接口超时断开但是数据库事务依旧存活。

---

## 六、连接数爆满、无法连接 MySQL

### 1.查看最大连接数与当前连接

```sql
show variables like 'max_connections';
show global status like 'Threads_connected';
```

Threads_connected 接近 max_connections → 连接打满

#### 连接爆满根因

1. 大量慢SQL阻塞会话，线程长时间不释放
2. 连接池配置错误、连接泄露（代码忘记关闭连接）
3. 死锁、行锁等待堆积大量休眠会话
4. 长事务霸占连接

#### 应急处理

1. kill掉阻塞的慢会话
2. 临时调高max_connections
3. 修复代码连接池泄露、优化慢SQL、缩短事务

查看各个客户端连接分布

```sql
select host,count(*) from information_schema.PROCESSLIST group by host;
```

---

## 七、CPU、磁盘IO、内存高负载排查

### 7.1 CPU 100%

诱因优先级：

1. 大量全表扫描、无索引SQL，MySQL线程疯狂扫描磁盘数据
2. 大批量排序 filesort、临时表
3. 大量并发热点行锁争抢、事务频繁回滚
4. 大批量计算类SQL、函数

排查步骤

1. top 查看 mysqld 线程CPU占用
2. 抓取 processlist，找出长时间运行SQL
3. explain 检查索引
4. 优化业务减少高并发热点更新（如同一行库存）

### 7.2 磁盘IO飙升

1. redo‑log、undo日志频繁刷盘
2. 大批量insert/update 写入压力
3. 缓冲池过小，缓存命中率低，频繁读取磁盘

```sql
-- 查看缓冲池命中率
show global status like 'Innodb_buffer_pool%';
```

> Innodb_buffer_pool_reads 物理读越高，IO压力越大，调大 innodb_buffer_pool_size（机器内存50%~70%）

4. 慢查询大量扫描磁盘

### 7.3 内存占用过高

- innodb_buffer_pool_size 设置过大，超过服务器内存
- 大量临时表、排序占用内存
- 未提交事务堆积undo日志占用内存

---

## 八、主从复制故障排查

```sql
--从库执行
show slave status\G
```

重点参数

1. `Slave_IO_Running`：IO线程是否正常，负责拉取主库binlog；No=网络、账号、binlog文件异常
2. `Slave_SQL_Running`：SQL线程，执行binlog；No=SQL报错、主键冲突、数据不一致
3. `Seconds_Behind_Master`：主从延迟秒数

### 常见主从问题

1. 主库大批量写入，从库单线程回放跟不上 → 开启并行复制
2. 主库大事务，binlog执行耗时久
3. 主键冲突、重复数据
4. 网络卡顿导致IO线程断开

---

## 九、MySQL各类高频线上故障速查手册

### 1. 业务查询超时

- 查看慢SQL、全表扫描
- 行锁 / MDL锁阻塞
- 磁盘IO打满

### 2. 更新时报死锁

- 查询死锁日志，统一更新顺序、缩短事务、优化索引缩小锁范围

### 3. alter table 卡住很久

- 存在长时间未提交事务持有该表MDL读锁，kill长事务即可

### 4. 接口偶尔超时，数据库时卡时好

- 间歇性慢SQL、热点行争抢、定时任务备份、统计SQL抢占IO

### 5. 插入很慢

- 索引太多、磁盘IO瓶颈、自增锁等待、事务太大

### 6. 数据库连接超时

- 连接池超时参数、wait_timeout 空闲连接被MySQL回收，代码连接池要配置保活

```sql
show variables like 'wait_timeout';
```

---

## 十、线上问题标准排查流程模板（直接照着操作）

1. 服务器层面：top、iostat、free、df‑h，确认CPU/IO/内存/磁盘是否爆满
2. `show full processlist;` 抓取所有正在执行会话，识别阻塞SQL、长事务
3. 查询 `information_schema.INNODB_TRX` 找出长时间运行未提交事务
4. 通过 performance_schema 抓取耗时最高的Top‑N慢SQL
5. explain 分析慢SQL执行计划，定位索引失效、全表扫描、filesort
6. 检查 data_locks、metadata_locks 判断行锁、间隙锁、MDL锁阻塞
7. 检查主从状态（从库实例）
8. 临时kill阻塞会话应急恢复业务；再优化索引、改造事务、优化业务SQL根治

## 十一、必备线上优化方向总结

1. 所有DML、查询必须命中合适索引，杜绝全表扫描
2. 事务尽量短小，禁止长事务，事务内部不要执行http调用、耗时逻辑
3. RR隔离级别理解间隙锁，普通索引等值更新会锁间隙，产生锁等待
4. 热点库存等高并发更新控制并发量，乐观锁减少行锁争抢
5. innodb_buffer_pool_size 设置合理大小，提升缓存命中率，降低磁盘IO
6. 定时巡检长事务、慢查询、连接数、锁等待指标，故障提前预警







# MySQL 线上慢 SQL 排查

下面按照 **排查思路 → 开启慢查询日志 → 内置工具 → performance_schema/实时抓取SQL → explain执行计划详解 → 索引失效清单 → 各类慢SQL诱因、应急处理、优化方案、巡检脚本**，全部都是生产环境实操流程，适配 MySQL5.7 / 8.0、InnoDB。

## 一、整体排查流程（标准步骤，线上照着走）

1. 服务器层面：top、iostat、free、df‑h，确认 CPU、磁盘IO、内存、磁盘是否打满。很多时候慢不是SQL问题，是机器资源瓶颈。
2. 查看 MySQL 当前会话 `show full processlist`，观察长时间处于 Sending‑data、Creating‑sort‑index、Waiting for lock 的会话。
3. 获取耗时较长的 SQL：慢查询日志、performance‑schema、sys 库。
4. `explain` 分析执行计划，定位全表扫描、索引失效、filesort、临时表。
5. 检查锁阻塞：行锁、MDL锁、长事务阻塞导致SQL超时。
6. 优化索引、改写SQL、拆分大事务、限制返回行数。
7. 上线之后持续监控慢查询。

---

## 二、三种方式抓取慢SQL

### 方式1：慢查询日志（最传统）

#### 1、查看慢日志参数

```sql
show variables like 'slow_query_log';
show variables like 'slow_query_log_file';
show variables like 'long_query_time';
show variables like 'log_queries_not_using_indexes';
```
参数含义
- slow_query_log：ON开启；OFF关闭
- long_query_time：慢SQL阈值(秒)，线上一般配置 1
- log_queries_not_using_indexes = ON，记录所有没有走索引的SQL

#### 2、临时开启（重启失效）

```sql
set global slow_query_log = ON;
set global long_query_time = 1;
set global log_queries_not_using_indexes = ON;
```

#### 3、永久配置 my.cnf / my.ini

```ini
slow_query_log=ON
slow_query_log_file=/data/mysql/slow.log
long_query_time=1
log_queries_not_using_indexes=ON
```

#### 4、mysqldumpslow 分析慢日志

```bash
# 按照执行耗时从高到低，输出前20条
mysqldumpslow -s t -t 20 /data/mysql/slow.log

# 按照扫描行数排序
mysqldumpslow -s r -t 20 /data/mysql/slow.log

# 筛选带select语句
mysqldumpslow -s t -t 20 -g "select" /data/mysql/slow.log
```
- -s t：query_time 耗时
- -s r：rows_examined 扫描行数

> 缺点：日志有延迟；新业务不想重启可以优先使用 performance_schema。

### 方式2：performance_schema 实时抓取最近执行SQL（推荐）

#### 开启监控

```sql
-- 开启SQL语句监控
update performance_schema.setup_instruments 
set ENABLED='YES',TIMED='YES' 
where NAME like 'statement/sql/%';

-- 开启历史事件存储
update performance_schema.setup_consumers 
set ENABLED='YES' 
where NAME like '%events_statements_history%';
```

#### 查询 Top 耗时SQL

```sql
SELECT
  sql_text SQL语句,
  round(TIMER_WAIT/1000000000,3) 执行耗时秒,
  ROWS_EXAMINED 扫描行数,
  ROWS_SENT 返回结果行数,
  ROWS_AFFECTED 影响行数,
  CREATE_TIME 执行开始时间
FROM performance_schema.events_statements_history
WHERE TIMER_WAIT > 1*1000000000
ORDER BY TIMER_WAIT DESC LIMIT 30;
```
**核心判断标准**
> `扫描行数 >> 返回行数` = 索引失效、大量无效扫描、全表扫描

### 方式3、sys 库一键查询Top慢SQL（MySQL8.0自带，5.7可导入sys库）

```sql
SELECT * FROM sys.statement_analysis 
ORDER BY exec_time DESC LIMIT 30;
```
关键字段
- query：SQL文本
- exec_time：平均执行耗时
- rows_examined：扫描行数
- rows_sent：返回行数
- full_scan 是否全表扫描

---

## 三、show full processlist 排查正在卡住的SQL

```sql
show full processlist;
```
重点看 `State` 状态，遇到下面状态代表SQL有问题
| State状态                            | 问题原因                                   |
| ------------------------------------ | ------------------------------------------ |
| Sending data                         | 正在大量扫描磁盘数据、全表扫描             |
| Creating sort index / Using filesort | 需要文件排序，缺少排序索引                 |
| Copying to tmp table                 | 创建临时表，group by、多表联查、无合适索引 |
| Waiting for row lock                 | 等待行锁阻塞                               |
| Waiting for table metadata lock      | MDL元数据锁阻塞，存在长事务                |
| Locked                               | 被表级锁阻塞                               |

> 长时间卡住的会话，可以使用 `kill thread_id` 应急释放连接。

---

## 四、Explain 执行计划深度分析（慢SQL排查核心）

使用方式
```sql
explain select xxx;
```
### 1、type（访问方式，性能优先级从好到差）

`system > const > eq_ref > ref > range > index > ALL`
- ALL：**全表扫描，必须优化**
- index：扫描整个索引树
- range：索引范围查询 in、>、between、like 前缀匹配
- ref：普通二级索引等值匹配
- eq_ref：唯一索引/主键等值关联
- const：主键等值查询，常量匹配

### 2、key

- null：**没有命中索引，全表扫描**
- 索引名称：实际使用到的索引

### 3、rows

预估需要扫描的数据行数，数值越大压力越高

### 4、Extra 额外信息（重中之重）

1. **Using filesort**
MySQL无法利用索引排序，需要额外内存/磁盘文件排序。
> 解决：建立联合索引 (order by字段,查询字段)
2. **Using temporary**
创建临时表，常见于 group by、多表连接、distinct。
> 解决：优化索引，尽量让group‑by字段落在索引最左侧
3. **Using index**
**覆盖索引，最优状态**，直接读取索引，不需要回表查询主键聚簇索引
4. Using where：存储引擎返回数据之后服务层再做过滤，索引命中不完全

---

## 五、索引失效 10 大常见原因（线上高频坑）

1. where 索引字段做函数运算
```sql
-- 索引失效
select * from t where date(create_time) = '2026-08-13';
-- 改写
select * from t where create_time >= '2026-08-13' and create_time < '2026-08-14';
```
2. **隐式类型转换**，字符串索引不带引号
```sql
-- phone为varchar索引，发生隐式转换索引失效
select * from t where phone=13800138000;
-- 正确写法
select * from t where phone='13800138000';
```
3. like 左通配符
```sql
like '%test' --失效
like 'test%' --可以命中前缀索引
```
4. or 两边存在无索引字段
> or后面字段没有索引，整条SQL索引失效；改用 union all
5. 联合索引不遵守最左匹配原则
索引(a,b,c)；where b=? 直接失效
6. MySQL优化器判定索引区分度太低，放弃索引走全表扫描
例如 sex 只有0和1，基数很低
7. 事务隔离级别造成锁、索引选择问题
8. 多表关联字段字符集不一致，触发隐式转换
9. 使用 !=、is not null、not in
> 会导致索引失效，可以改写为 left join
10. 优化器预估数据量过大，认为全表扫描更快主动放弃索引

---

## 六、慢SQL常见根因分类

### 1、索引问题（占80%）

全表扫描、索引失效、缺少联合索引、缺少覆盖索引、回表次数过多

### 2、SQL写法问题

1. select * 读取全部字段，产生大量回表IO
2. limit 深度分页 `limit 100000,10`；扫描十万行再丢弃
> 优化：主键书签分页 `where id>100000 limit 10`
3. in 集合数据量巨大，超过1000条
4. group by、distinct、order by 没有索引
5. 大表 join 小表，关联字段没有索引
6. 子查询嵌套，产生循环扫描

### 3、事务与锁阻塞造成SQL假性缓慢

SQL本身很快，但是被行锁、间隙锁、MDL锁、长事务阻塞，造成超时缓慢。
排查长事务SQL：
```sql
SELECT
trx_id,
trx_mysql_thread_id,
trx_started,
now()-trx_started cost,
trx_query
FROM information_schema.INNODB_TRX
WHERE trx_state='RUNNING' AND (NOW()-trx_started) > 3
ORDER BY cost DESC;
```

### 4、数据库参数配置问题

1. innodb_buffer_pool_size 太小，缓存命中率低、频繁读取磁盘
```sql
show global status like 'Innodb_buffer_pool%';
```
Innodb_buffer_pool_reads 物理读数值很高代表缓存不足；
服务器内存充足时配置为物理内存 50%~70%。
2. sort_buffer_size、join_buffer_size 过小导致磁盘排序
3. 磁盘IO打满，定时任务、备份、大批量写入抢占IO资源

### 5、业务层面问题

1. 定时统计报表、大表统计SQL白天高峰期执行
2. 一次性加载上万条数据到内存
3. 未做分页，一次性查询整张表

---

## 七、各类慢SQL优化实战方案

### 1. 深度分页优化

```sql
-- 低效
select * from user limit 100000,10;
-- 书签优化
select * from user where id > 100000 limit 10;
```

### 2. 避免回表，建立覆盖索引

```sql
-- 查询 name、phone；建立覆盖索引，不需要回主键表
create index idx_name_phone on user(name,phone);
```

### 3. 大in改写

in(1,2,3...)数量过大 → 拆分多次查询、或者临时表

### 4. order by + group‑by 联合索引

```sql
where user_id=? order by create_time
索引建立 (user_id,create_time)
```

### 5. 大表关联

小表驱动大表，关联字段添加索引

### 6. 大查询拆分

一次性查询几千行 → 分页、分批读取

---

## 八、线上应急处理流程

1. 检查机器CPU、IO负载，如果机器打满优先处理服务器资源
2. show full processlist 找到长时间阻塞会话 kill 掉，快速恢复业务
3. 通过慢查询日志/performance_schema定位问题SQL
4. explain分析执行计划，补充索引、改写SQL
5. 排查长事务、MDL锁、行锁等待造成的阻塞
6. 高峰期关闭耗时报表SQL，迁移至低峰或者从库执行统计

---

## 九、日常巡检SQL（定时监控慢查询）

```sql
-- 查看累计慢查询数量
show global status like 'Slow_queries';

-- 查看行锁等待次数，锁竞争造成SQL卡顿
show global status like 'Innodb_row_lock_waits';
```
搭建监控告警：当每秒新增慢查询超过阈值触发告警。

---

## 十、开发规范，从根源减少慢SQL

1. DML、查询语句必须命中索引
2. 禁止 select *
3. 联合索引遵守最左前缀原则
4. 索引字段避免函数、运算、隐式类型转换
5. 分页使用书签分页，杜绝深度limit
6. 事务短小，事务内部不要执行远程调用
7. 大表统计、报表查询放到从库执行
8. 大批量数据分批更新、分批查询

# 线上 MySQL 锁超时排查

> 报错：`Lock wait timeout exceeded; try restarting transaction`
> 默认超时参数 `innodb_lock_wait_timeout = 50`，代表事务最多等待行锁 50s，拿不到锁直接抛出锁超时异常。
> 下面梳理 **底层原理 → 全套排查SQL → 锁超时各类诱因 → 区分行锁/间隙锁/MDL锁/表锁 → 线上应急、定位步骤、根治方案、巡检脚本**，全部为生产实操流程。

## 一、先搞懂基础概念

1. 锁超时本质：当前事务需要某一行锁，但该锁长时间被别的事务占用，等待超时。
2. 和死锁区别
    - **死锁**：循环等待，MySQL主动回滚其中一个事务，速度很快
    - **锁超时**：单向等待别人的锁，等待超过阈值直接失败
3. 核心超时参数
```sql
-- 行锁等待超时，单位秒，默认50
show variables like 'innodb_lock_wait_timeout';
-- MDL锁超时
show variables like 'lock_wait_timeout';
```

## 二、标准化线上排查顺序（严格照着执行）

1. 查看当前运行会话，查看处于锁等待的线程 `show full processlist`
2. 查询正在等待锁、持有锁的事务，揪出**长事务**（头号元凶）
3. 查询 `performance_schema.data_locks`、`data_lock_waits` 查看锁类型、索引、锁区间
4. 判断是普通记录锁、间隙锁、临键锁、MDL元数据锁、表锁阻塞
5. explain 检查 SQL 是否索引失效、锁范围过大
6. 排查死锁、大批量更新、热点行争抢
7. kill 掉长时间持有锁的会话应急恢复业务
8. 代码、索引、事务结构优化根治问题

---

## 三、线上所有排查 SQL（可直接复制执行）

### 1. 查看所有正在等待锁的会话

```sql
show full processlist;
```
重点观察 State 字段
- `Waiting for row lock`：等待 InnoDB 行排他锁
- `Waiting for table metadata lock`：MDL元数据锁阻塞（alter 卡死经典场景）
- `Waiting for table level lock`：等待表级锁

### 2. 查询所有未提交、正在运行的事务（最关键）

绝大多数锁超时都是**长事务迟迟不提交，一直霸占行锁**
```sql
SELECT
    trx_id AS 事务ID,
    trx_mysql_thread_id AS 会话ID,
    trx_started AS 事务开启时间,
    NOW() - trx_started AS 事务耗时_秒,
    trx_state AS 事务状态,
    trx_query AS 当前执行SQL,
    trx_isolation_level AS 隔离级别
FROM information_schema.INNODB_TRX
WHERE trx_state = 'RUNNING'
ORDER BY 事务耗时_秒 DESC;
```
> 只要事务耗时 >3s 就要警惕；找到会话ID，`kill 会话ID` 释放锁。

### 3. MySQL8.0 锁详情查询（精准定位哪条索引、哪段区间被锁住）

```sql
-- 当前所有已经被事务持有的锁
SELECT * FROM performance_schema.data_locks;

-- 当前正在等待锁的请求
SELECT * FROM performance_schema.data_lock_waits;
```

#### 关键字段解读

1. `LOCK_TYPE`
    - RECORD：普通记录行锁
    - GAP：间隙锁，RR隔离级别专属
    - NEXT‑KEY：临键锁 = 记录锁+间隙锁
2. `LOCK_TABLE`：被锁住数据表
3. `LOCK_INDEX`：加锁依赖的索引，判断是不是索引失效
4. `LOCK_MODE`：X排他锁 / S共享锁

### 4. MDL元数据锁阻塞排查（alter、add‑index卡住）

```sql
SELECT * FROM performance_schema.metadata_locks
WHERE LOCK_STATUS = 'PENDING'; --处于等待状态的MDL锁
```
> 只要有事务没有提交，就一直持有该表MDL读锁；DDL需要MDL写锁，会被阻塞；后续所有CRUD全部阻塞超时。

### 5. 查看 InnoDB 行锁等待统计指标

```sql
show global status like 'Innodb_row_lock%';
```
字段释义
- `Innodb_row_lock_waits`：行锁等待总次数，数值持续上涨代表锁竞争激烈
- `Innodb_row_lock_time_avg`：平均锁等待时长，数值越高阻塞越严重

### 6. 5.7 查看锁、最近死锁信息

```sql
show engine innodb status;
```
查看 `TRANSACTIONS` 事务列表、`LOCK WAIT` 锁等待详情。

---

## 四、锁超时 6 大类线上常见原因

### 1. 长事务霸占行锁（最高频）

#### 问题场景

1. Java代码开启事务之后，执行HTTP远程调用、RPC、文件IO、sleep、复杂业务逻辑
2. 代码异常没有捕获，忘记 commit / rollback
3. 接口超时断开连接，但数据库事务仍然存活，一直持有行锁

> 铁律：**事务内禁止执行任何外部耗时请求；事务只存放DML增删改语句**

### 2. SQL索引失效，锁范围疯狂扩大

update、delete 没有命中合适索引，InnoDB只能全表扫描，给每一行加上排他行锁。
- 隐式类型转换
- where索引字段使用函数
- like '%xxx' 左通配
- or 条件索引失效
> 结果：一条SQL锁住整张表绝大多数行，后续操作全部锁等待超时

排查方式：`explain update xxx`，观察 key 是否为 null、type=ALL 全表扫描

### 3. RR（可重复读）隔离级别间隙锁 / 临键锁阻塞

> 坑点：**普通二级索引等值更新，RR隔离级别除了锁住命中行，还会锁住索引间隙**，锁区间很大，极易阻塞其他插入、更新语句。
- 主键唯一索引等值查询：临键锁自动降级成单行记录锁，不会锁间隙
- 普通非唯一索引：等值更新依旧存在间隙锁

#### 间隙锁典型超时场景

索引存在数值 10、20；事务A更新索引=10，锁住间隙 (10,20)；事务B想要插入15，被间隙锁阻塞，触发锁超时。

### 4. 高并发热点行争抢

大量请求同时更新同一行记录，例如商品库存、账户余额、订单状态；
MySQL同一时刻只允许一个事务持有该行X锁，其余请求排队等待，队列堆积之后触发锁超时。

### 5. MDL元数据锁阻塞

1. 一条长事务查询某张表一直不提交
2. DDL语句（alter table、add index、drop column）发起，等待MDL锁
3. DDL卡住之后，之后所有该表CRUD全部等待MDL锁，大批量锁超时

> 现象：之前查询正常，执行alter之后整表业务全部超时

### 6. 加锁顺序混乱、批量更新引发大面积锁竞争

1. 多个事务更新多行主键顺序不一致，不仅会死锁，也容易长时间锁等待
2. for循环逐条更新数据，循环之间不提交事务，一次性锁住大量行
3. 批量update一次性修改成千上万条记录，持有海量行锁

---

## 五、线上紧急应急处理步骤

1. 先通过 `INNODB_TRX` 查出耗时很久、未提交的长事务
2. kill 掉持有锁的阻塞会话，锁立刻释放，业务马上恢复
```sql
kill 会话ID;
```
3. 如果是MDL锁阻塞，杀掉持有MDL读锁的长事务，DDL即可执行成功
4. 热点行瞬间大量超时：临时限流、队列削峰，降低并发
5. 间隙锁造成频繁超时：业务可接受幻读时，会话切换RC隔离级别
```sql
set session transaction_isolation = 'READ-COMMITTED';
```
> RC隔离级别关闭间隙锁，锁范围只锁定命中记录

---

## 六、分场景根治锁超时（生产落地方案）

### 方案1：消灭长事务（首要优化点）

1. 所有远程调用、参数校验、查询逻辑全部放到事务外面
2. try‑finally 保证事务一定会 commit / rollback
3. 定时SQL巡检，告警运行超过3s未提交事务
4. 接口设置超时，框架自动回滚超时事务

### 方案2：缩小行锁范围，保证DML命中高效索引

1. update / delete 的where条件必须命中主键或者唯一索引，只锁住单行
2. 禁止在索引字段做运算、函数、隐式转换
3. 普通二级索引容易触发间隙锁，优先使用主键id作为更新条件

### 方案3：热点行从悲观锁改成乐观锁（库存、余额首选）

抛弃 `select for update` 悲观行锁争抢，version版本号实现无锁更新，彻底规避行锁等待
```sql
UPDATE stock 
SET stock = stock - 1, version = version + 1
WHERE goods_id = ? AND version = old_version AND stock > 0;
```
更新失败业务进行重试。

### 方案4：统一多行更新顺序

当事务需要更新多条记录，永远按照主键id升序顺序更新，减少锁竞争、死锁、锁等待。

### 方案5：控制事务粒度，大事务拆分为小事务

大批量循环更新数据，每处理若干条就commit一次，不要一次性锁住成千上万行。

### 方案6：高并发热点行串行化

- Redis分布式锁、消息队列，保证同一时间只有一个请求操作热点库存行
- 热点商品做库存分片，一行拆分为多行分散锁竞争压力

### 方案7：调整事务隔离级别

业务能够接受幻读，全局改为RC读已提交，消除间隙锁。

---

## 七、日常定时巡检脚本，提前规避锁超时

1. 监控运行超过3s的未提交长事务，配置告警
2. 监控 `Innodb_row_lock_waits` 行锁等待次数，持续上涨则预警锁竞争
3. 监控慢SQL，优化全表扫描、索引失效SQL，防止超大范围行锁
4. DDL变更选低峰期；执行alter之前排查所有运行中的长事务

---

## 八、锁超时、死锁、MDL锁超时快速区分

| 故障现象                       | 阻塞类型          | 核心排查点                           |
| ------------------------------ | ----------------- | ------------------------------------ |
| 报错Lock wait timeout exceeded | InnoDB行锁/间隙锁 | 长事务、索引失效、RR间隙锁、热点更新 |
| 瞬间抛出异常、事务被回滚       | 死锁              | 循环等待、更新顺序混乱               |
| alter卡住，整张表所有CRUD超时  | MDL元数据锁       | 长时间未提交的查询事务               |

如果你需要，我可以整理一份 **MySQL锁超时、死锁、长事务一体化定时巡检脚本**。

# MySQL 线上死锁完整排查、原因、排查SQL、复现、解决方案、避坑指南

> 环境默认 InnoDB、支持事务；死锁本质：**多个事务互相持有对方需要的行锁，循环等待，MySQL自动回滚代价最小的事务**。下面全部是生产实战流程，分为：死锁原理 → 开启日志 → 全套排查SQL → 解读死锁日志 → 常见诱因 → 复现案例 → 根治方案、应急处理、日常规避。

## 一、前置基础知识

### 1. InnoDB两种行锁

- S 共享锁：`select ... lock in share mode`
- X 排他锁：update、delete、insert、`select ... for update`
锁互斥规则：S与X互斥、X和X互斥

### 2. 产生死锁4个必要条件

1. **互斥条件**：行锁同一时刻只能被一个事务占有
2. **持有并等待**：事务A拿着锁，还请求事务B手里的锁
3. **不可剥夺**：别人持有的锁不能强行抢走
4. **循环等待**：A等B、B等C、C等A，形成环路等待

### 3. RR可重复读隔离级别额外坑点

RR 存在 **间隙锁、临键锁**，锁范围变大，非常容易触发死锁；RC隔离级别关闭间隙锁，死锁概率大幅下降。

---

## 二、第一步：开启死锁日志（最核心）

### 方式1：开启将全部死锁记录写入mysql错误日志

```sql
-- 查看是否开启
show variables like 'innodb_print_all_deadlocks';

-- 全局开启，重启失效；写到my.cnf永久配置
set global innodb_print_all_deadlocks = 1;
```
开启之后每次死锁详情都会写入错误日志 `error.log`，可以看到两条冲突事务完整 SQL、锁范围、索引。

> MySQL5.7 `show engine innodb status` 只会保存最近一次死锁；开启上面参数所有死锁都会落地日志。

### 方式2、performance‑schema 实时查询锁和死锁(MySQL8.0首选)

```sql
-- 1. 当前已经持有的所有锁
SELECT * FROM performance_schema.data_locks;

-- 2. 当前正在等待锁的会话
SELECT * FROM performance_schema.data_lock_waits;
```
#### 关键字段解释

1. `ENGINE_TRANSACTION_ID`：事务ID
2. `THREAD_ID`：会话线程id，对应 processlist
3. `LOCK_TYPE`：
   - RECORD：记录行锁
   - GAP：间隙锁
   - NEXT‑KEY：临键锁（记录锁+间隙锁）
4. `LOCK_TABLE`：被锁的表
5. `LOCK_INDEX`：加锁使用的索引
6. `LOCK_MODE` X=排他锁，S=共享锁

---

## 三、全套排查 SQL（线上直接复制执行）

### 1. 查询所有正在运行、未提交事务（找出长事务，死锁元凶之首）

```sql
SELECT
    trx_id 事务ID,
    trx_state 事务状态,
    trx_mysql_thread_id 会话ID,
    trx_started 事务开始时间,
    NOW() - trx_started AS 事务耗时_秒,
    trx_query 当前执行sql,
    trx_isolation_level 隔离级别
FROM information_schema.INNODB_TRX
ORDER BY 事务耗时_秒 DESC;
```
> 事务长时间不提交，锁一直持有，极易触发循环等待死锁；耗时高的长事务直接 `kill trx_mysql_thread_id`

### 2. 根据会话ID查看完整会话详情

```sql
show full processlist;
kill 会话id;
```

### 3. 5.7查看最近一次死锁信息

```sql
show engine innodb status;
```
往下滚动找到 `LATEST DETECTED DEADLOCK` 区块。

---

## 四、手把手解读死锁日志

### 示例死锁日志

```
LATEST DETECTED DEADLOCK
------------------------
2026-08-13 10:00:00 7fxxxx INNODB MONITOR OUTPUT
TRANSACTION 1, ACTIVE 2 sec
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 100, OS thread handle 140xxxx, query id 500 xxx update
update t set num=num-1 where id=2

TRANSACTION 2, ACTIVE 2 sec
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 101, OS thread handle 140xxxx, query id 600 xxx update
update t set num=num-1 where id=1
```
#### 解读

- 事务1：已经锁住 id=1，现在申请 id=2 的排他锁
- 事务2：已经锁住 id=2，现在申请 id=1 的排他锁
- 循环等待 → 触发死锁，MySQL回滚其中一个事务

> 根因：**更新主键顺序不统一**

#### 间隙锁引发死锁日志特征

日志里面会出现 `GAP`、`NEXT‑KEY`，普通二级索引等值更新产生间隙锁，锁区间冲突造成死锁。

---

## 五、线上死锁最常见8类原因

### 1. 事务更新主键顺序不一致（最高发）

事务A更新 1→2；事务B更新 2→1，循环等待
> 解决方案：所有业务代码更新多行时，**永远按照主键升序/降序统一顺序更新**

### 2. 长事务，锁迟迟不释放

事务开启之后执行耗时逻辑、HTTP请求、sleep、异常忘记commit/rollback，行锁长时间占用，并发场景极易死锁
> 铁律：事务尽可能短小；事务内部禁止调用外部接口

### 3. 索引失效，锁范围超大

update 没有走到索引，InnoDB会扫描全表、逐行加X行锁，海量行锁极易出现循环等待
> 检查explain，确保update、delete条件命中索引

### 4. RR隔离级别下普通二级索引带来间隙锁

- 主键唯一索引等值查询：临键锁降级成记录锁，只锁单行
- 普通非唯一二级索引等值更新：RR模式下依旧存在间隙锁，锁住索引间隙，锁范围变大，死锁概率暴涨

### 5. 共享锁 + 排他锁争抢

事务A执行 `select ... lock in share mode` 加S锁；事务B想要加X锁；多线程组合等待环路

### 6. 一张表多条SQL加锁顺序混乱

例如：先更新订单表、再更新库存；另一个事务先更新库存、再更新订单表

### 7. 批量循环更新

for循环循环更新多行，每次循环一次commit；循环顺序随机

### 8. Insert 插入意向锁冲突、间隙锁竞争

多个事务往同一个索引间隙插入数据，RR间隙锁阻塞引发死锁

---

## 六、应急处理（线上发生死锁故障操作步骤）

1. 查看error.log死锁日志，拿到冲突事务、SQL、索引
2. 执行 `INNODB_TRX` 排查长时间未提交长事务，kill掉阻塞会话
3. 高并发死锁紧急优化方案：
   - 业务层面增加重试机制（捕获死锁异常，重试2‑3次）
   - 热点行排队、分布式锁控制并发
4. 临时降低锁范围：确认业务可以接受幻读后，会话级别改成RC隔离级别
```sql
set session transaction_isolation = 'READ‑COMMITTED';
```
> RC关闭间隙锁，绝大多数间隙锁死锁直接消失

---

## 七、死锁完整解决方案（根治，分优先级）

### 🔹 方案1：统一资源更新顺序（最有效、零成本）

只要需要操作多条行记录，永远固定按照主键id升序更新，杜绝循环等待。
伪代码反面案例（会死锁）
```java
//事务A
update user set ... where id=2;
update user set ... where id=1;

//事务B
update user set ... where id=1;
update user set ... where id=2;
```
正确写法：所有事务永远先更新id小的数据

### 🔹 方案2：缩短事务生命周期

1. 开启事务之前完成所有查询、参数校验、远程调用
2. 事务只保留DML增删改代码
3. 代码异常必须强制回滚，finally提交/回滚事务
4. 禁止大事务批量一次性修改上万行，拆分小批次提交

### 🔹 方案3：优化索引，缩小加锁范围

1. update / delete 的where条件必须命中唯一索引，锁仅锁定单行记录
2. 尽量使用主键索引做更新条件，避开普通二级索引触发RR间隙锁

### 🔹 方案4、业务增加死锁重试

捕获数据库死锁异常（Deadlock found when trying to get lock），sleep一小段时间重试2~3次；适合低并发场景。

### 🔹 方案5、高并发热点行改用乐观锁

库存、余额等热点更新放弃悲观锁 `for‑update`，使用version版本号乐观锁，完全避开行锁争抢
```sql
update stock set num = num‑1,version=version+1 
where id=? and version = oldVersion and num>0;
```

### 🔹 方案6、调整事务隔离级别

业务允许幻读，改成RC读已提交，关闭间隙锁，锁范围大幅缩小，死锁概率大幅下降

### 🔹 方案7、热点行串行控制

- 热点商品库存：redis分布式锁、消息队列串行消费，同一时间只允许一个事务操作该行

---

## 八、日常巡检规避死锁（定时巡检SQL）

1. 定时监控运行超过3s的未提交长事务，告警
2. 监控 `Innodb_row_lock_waits` 行锁等待次数
```sql
show global status like 'Innodb_row_lock%';
```
- Innodb_row_lock_waits：锁等待次数，数值持续上涨代表锁竞争严重
3. 监控error.log新增死锁日志，触发告警

---

## 九、高频面试总结

1. 死锁必要条件：互斥、持有并等待、不可剥夺、循环等待
2. RR隔离普通索引存在间隙锁，扩大锁范围，死锁多于RC
3. 最常见原因：多行更新顺序混乱 + 长事务
4. 优先根治方案：固定主键更新顺序 + 短事务 + 合理索引
5. 应急手段：业务重试、RC隔离、乐观锁、分布式锁

如果你需要，我可以给你一份 **可直接部署的MySQL死锁、长事务定时巡检脚本**。