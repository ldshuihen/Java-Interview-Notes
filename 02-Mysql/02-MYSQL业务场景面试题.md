

## MySQL EXPLAIN

 explain 检查优先级

`type > key > possible_keys > key_len > ref > rows > filtered > Extra`

### Extra 完整标识

> 基于 InnoDB，MySQL8.0 版本，附带含义、触发原因、优化手段，结合你三表联查案例讲解

#### 一、正常无害标识

##### 1. `Using index`（最优，覆盖索引）

- 含义：**覆盖索引命中**，查询所需要全部字段都在二级索引里，InnoDB 不需要根据主键回表读取聚簇索引行数据
- 优化：建立联合索引 `(查询条件字段,查询返回字段)`
  例：`idx_proj_cover(user_id,title,category,stars)`

##### 2. `Using where`

- 含义：存储引擎读出数据之后，**MySQL Server‑执行器层再做 where 条件过滤**

1. 全表扫描时一定会出现，属于需要优化
2. 索引只能命中一部分条件、剩余字段过滤，属于正常

- 优化：给 where 查询字段添加合适索引

##### 3. `Using join buffer (Block Nested‑Loop)`

- 含义：被驱动表没有索引，MySQL 使用连接缓冲池缓存驱动表数据做嵌套循环匹配
- 问题：性能差
- 优化：JOIN 关联字段建立索引

##### 4. `Using MRR`

- Multi‑Range‑Read，范围查询优化；把索引读取到的主键排序、批量回表，减少随机IO，属于优化机制，无害

##### 5. `Using Batch Access`

批量索引访问优化，多表联查批量读取索引

---

#### 二、需要进行优化（常见危险标记）

##### 1. `Using filesort`（最常见坑）

- 释义：**无法利用已有索引完成排序**，MySQL 需要额外内存/磁盘进行文件排序

> filesort 不是磁盘文件，优先使用 sort_buffer 内存空间，超出上限才落到磁盘

- 触发：`order by`、`group by` 字段没有建立合适联合索引
- 优化：遵循 `where条件 + order by` 创建复合索引

> 示例：`where user_id=1 order by stars desc` → 索引 `(user_id,stars)`

##### 2. `Using temporary`（高危）

- 释义：MySQL 在内存（磁盘）创建临时数据表存放中间结果
- 触发场景：
  - distinct、group by、多表联查、union、多字段排序
- 性能很差
- 优化：建立联合索引，避免生成临时表；优化 group‑by 字段索引

##### 3. `Using index condition`（ICP‑索引条件下推）

索引条件下推，把索引可以筛选的条件下放至存储引擎层过滤，减少回表次数；
属于优化开关开启后的标识，一般不用处理。

---

#### 三、其他冷门 Extra 参数

##### 1. `Distinct`

MySQL 正在执行去重操作

##### 2. `Not exists`

左连接优化，找到一条匹配数据之后立刻停止继续扫描被驱动表

##### 3. `Range used for each record on join`

联表的时候使用范围索引匹配

##### 4. `Zero limit`

`limit 0`，不需要读取任何行

##### 5. `Backward index scan`

索引反向扫描，用于 `order by xxx desc`

##### 6. `Full scan on joined table`

联表时被驱动表全表扫描

##### 7. `Skip_open_table`、`Open_frm_only`

表缓存相关标识

##### 8. `Using derived table`

使用派生表（子查询生成临时表）

##### 9. `Using sort_union` / `Using union` / `Using intersect`

多个索引合并扫描

---

#### 四、快速调优优先级口诀

1. 优先保证 type ≠ ALL（杜绝全表扫描，达到 ref/range）
2. 目标出现 `Using index` 覆盖索引
3. **坚决消除 `Using filesort`、`Using temporary`**
4. 避免 `Using join buffer`（嵌套循环无索引）

### explain‑type 访问类型

`system > const > eq_ref > ref > range > index > ALL` type 代表 MySQL 查找数据表行的扫描方式，是执行计划最核心指标，下面逐个详解、触发场景、示例、优化目标，结合你三表联查案例说明。

#### 1、system（性能最高）

数据表只有 **1 条数据**，是 const 的特殊情况。

```
select * from t_user; -- 表仅一行记录
```

#### 2、const

常量查找；根据**主键 / 唯一索引**精准匹配一条记录。

> where 条件传入常量

```
select * from t_project where id = 1;
```

- 一次查询最多命中一行
- ref 列展示 const

#### 3、eq_ref

等值引用，联表查询专用 使用**主键、唯一非空索引**去关联匹配，每次连接只能匹配一条数据。

```
from a join b on a.b_id = b.id
-- b.id 为主键
```

> 一对一关联最佳状态。

#### 4、ref（你优化之后需要达到的标准）

普通二级索引等值匹配，可以命中多条符合条件的数据。 也是日常业务最理想的常态级别。

```
where user_id = 1;
-- user_id 普通二级索引，可查出多条该用户的数据
```

你 t_project 添加 user_id 索引之后 type 就会由 `ALL` → `ref`

> s、t 两张表现在已经是 ref，属于正常。

#### 5、range

索引范围扫描 使用索引做区间查询，常见 >、<、>=、between、in、like 前缀匹配

```
select * from t_project where id > 100;
```

> 性能次于 ref；范围之后索引字段失效，遵循最左前缀原则。

#### 6、index

扫描**全部二级索引树**（索引全扫描） 优点：索引文件一般小于聚簇索引数据文件，比全表扫描快； 触发场景：覆盖索引但是需要遍历全部索引、order‑by 索引排序。

> Extra 带有 Using‑index 经常对应 type=index

##### 1、基础定义

`type: index` = **二级索引全扫描**
MySQL 遍历整棵二级索引树完成查询，不会扫描主键聚簇索引的全部数据行。

> 注意区分两个极易混淆概念

- `ALL`：聚簇索引全表扫描，读取完整数据表行，IO开销最大
- `index`：只遍历二级索引，索引体积远小于整表，速度优于 ALL

##### 2、触发 index 的两类场景

###### 场景1：使用覆盖索引，读取索引即可拿到所有查询字段（最优）

Extra 伴随 `Using index`

```sql
-- 联合覆盖索引
create index idx_proj_cover on t_project(user_id,title,category,stars);

select title,category,stars from t_project;
```

查询字段全部存放在二级索引，InnoDB 只扫描索引树，**不需要回表**，性能很好。

###### 场景2：利用索引进行排序，避免 filesort

order‑by 的字段存在索引，于是遍历索引树天然有序

```sql
select * from t_project order by user_id;
```

此时只顺着索引顺序读取；因为需要 * 全部字段，索引只存 user_id，后续要回表拿主键行数据。

##### 3、性能层级回顾（从快→慢）

`system > const > eq_ref > ref > range > index > ALL`

1. `ref`：索引等值查找，直接定位少数几条索引记录（首选目标）
2. `range`：索引区间扫描
3. `index`：完整遍历二级索引树
4. `ALL`：遍历整张数据表聚簇索引

##### 4、ref、index、ALL 通俗对比

1. `ref`：根据索引条件，直接跳到符合条件的索引节点，只读少量索引行（等值查询理想结果）
2. `index`：从头到尾过完一整棵二级索引树
3. `ALL`：从头到尾扫描整张数据表

##### 5、结合你之前的 SQL 举例

```sql
SELECT title,category,stars FROM t_project WHERE user_id = 1;
```

1. 普通索引 `idx_proj_uid(user_id)`
   type=ref，精准定位 user_id=1 的索引条目，最优
2. 覆盖索引 `idx_proj_cover(user_id,title,category,stars)`、不带 where 条件
   type=index，Extra=Using‑index，遍历全部二级索引

##### 6、什么时候 type=index 需要优化？

- 不带查询条件、强制遍历整颗二级索引；表量大时开销上升
- 解决方案：加上 where 等值条件，让 type 降级为 `ref`

##### 7、key‑len、Using‑index 和 type(index) 关联

只要 Extra 出现 `Using index`，代表触发覆盖索引；
type 可以为 ref / range / index。

- ref+Using‑index：等值+覆盖索引（最佳）
- index+Using‑index：完整扫描二级索引树

##### 8、总结优化方案

1. 等值筛选查询，添加索引，目标 type = ref
2. 查询字段全部放入复合索引，开启覆盖索引 `Using index`，省去回表
3. 尽量避免无筛选条件造成 type=index 全索引遍历

#### 7、ALL（最差：全表扫描，你当前 t_project 的状态）

遍历整棵主键聚簇索引，扫描整张数据表所有行。

- 现在你的 `t_project` 没有 user_id 索引 type=ALL
- Extra 附带 `Using where`：读取全部行之后服务层过滤

------

#### 速记优先级口诀

```
系(system)常(const)等(eq_ref)参(ref)，范(range)索(index)全(ALL)
```

#### 业务优化硬性标准

1. 普通等值查询：**至少达到 ref**，绝不允许长期 ALL
2. 主键精准查询：尽量 const / eq_ref
3. 范围查询允许 range
4. 杜绝 ALL 全表扫描

#### 常见踩坑

1. 索引字段做函数运算，索引失效 type=ALL

```
where date(create_time)='2026‑08‑13'
```

1. 隐式类型转换，字符串索引传入数字，索引失效
2. like `%xxx` 前后通配符无法命中普通索引，触发全表扫描

#### type 优化总结

1. 等值查询条件字段建立普通二级索引 → type=ref
2. 主键精准查询 → const
3. join 被驱动表关联字段加索引，联表达到 ref / eq_ref
4. 范围查询尽量放在复合索引最右侧字段

### explain 字段 key 

#### 1、字段含义

`key`：**执行计划里 MySQL 实际最终选用的索引名称**
- `NULL`：没有选中任何索引，触发全表扫描
- 索引名：成功命中该索引

拿你当前执行计划举例
```
p‑key = NULL
t‑key = idx_time_uid
s‑key = idx_skill_uid
```
解读：
`t_project` 没有用上索引；t_timeline、t_skill 成功走 user_id 索引。

#### 2、区分 possible_keys 和 key

1. **possible_keys**：理论上可供本次SQL使用的全部候选索引，只是备选清单
2. **key**：优化器经过成本计算，真正选定执行的索引

常见情况：
- possible_keys 有索引、key 等于 NULL
> MySQL 判断走索引开销大于全表扫描，放弃索引直接全表扫描
> 常见诱因：
1. 数据表行数极少
2. 索引选择性很差，索引筛选之后仍然大部分数据都需要读取
3. 索引字段类型不匹配、索引失效

#### 3、key_len 和 key 的配套查看

`key_len` 代表当前选中的索引实际用到多少字节
- bigint 类型 user_id 占8字节，`key_len=8`，仅使用 user_id 单列

复合索引 `(user_id,stars,title)`
- where user_id=1：key_len=8，只用索引第一列
- where user_id=1 and stars=200：key‑len 变成12，命中前两列

依靠 key‑len 判断复合索引有没有充分利用，检查是否遵守最左前缀原则。

#### 4、ref 和 key 配合解读

ref 代表索引的匹配来源
1. `const`：常量匹配（`where user_id=1`），性能最优
2. 别的数据表字段：联表嵌套循环，依靠另一张表字段驱动索引查找

你的 s、t 表 `ref=const`，代表索引直接匹配常量1，非常理想。

#### 5、key 的几种优化目标

1. 等值查询：key **不能为 NULL**，type 达到 ref
2. 主键精准查询：key为主键索引，type=const
3. 开启覆盖索引，key为复合索引、Extra出现`Using index`，省去回表

#### 6、针对你这条SQL的解决办法

p表 key=NULL，缺少索引
```sql
CREATE INDEX idx_proj_uid ON t_project(user_id);
```
优化之后：
- key：idx_proj_uid
- type：ALL → ref
- ref：const
- Extra 的 `Using where` 消失

#### 7、进阶覆盖索引优化

```sql
DROP INDEX idx_proj_uid ON t_project;
CREATE INDEX idx_proj_cover ON t_project(user_id,title,category,stars);
```
key 命中复合覆盖索引，Extra = `Using index`，不用回表读取聚簇索引。

#### 简短总结

1. key=null → 全表扫描，优先解决
2. 观察key_len，判断复合索引利用程度
3. 结合ref看索引是常量查询还是联表字段查询
4. 尽量建立覆盖索引，索引一次性包含查询字段



### possible_keys 

#### 定义

`possible_keys`：**MySQL优化器评估之后，理论上能够适用于这条SQL的所有候选索引列表**
只是候选池，并不代表最终一定会选用该索引；真正生效索引看 `key` 字段。

结合你的执行计划：
```
p(possible_keys)=NULL
t(possible_keys)=idx_time_uid
s(possible_keys)=idx_skill_uid
```
含义：
1. `t_project` 不存在任何可以用于该SQL条件的索引；
2. t_timeline、t_skill 的 user_id 索引具备被选用的资格。

#### possible_keys 和 key 的4种常见组合

##### 情况1：possible_keys=NULL、key=NULL

没有任何适配索引 → 只能全表扫描，就是你现在 p‑表现状，需要新建索引。

##### 情况2：possible_keys 存在索引，key 等于该索引

优化器经过成本计算，决定启用这条索引，属于理想状态。

##### 情况3：possible_keys 有多个索引名称

多条索引都可以适配条件，优化器对比IO代价之后挑选一个作为最终 key。
> 一张表不要建立大量冗余索引，会加重写入开销、干扰优化器判断。

##### 情况4：possible_keys 有索引，但 key=NULL（很常见）

明明有可用索引，MySQL 却放弃索引、选择全表扫描。
#### 触发原因

1. 数据表数据量很小，遍历全表比索引搜索、回表更快；
2. 索引选择性极低，例如字段只有0和1两个值，索引筛选后大半数据仍需要读取；
3. like 左通配符、索引字段运算、隐式类型转换造成索引失效；
4. 优化器认为索引IO成本高于全表扫描。

#### key‑len、possible_keys、key 三者查看顺序

1. possible_keys：查看有没有备选索引；没有就去新建合适索引
2. key：查看最终选中哪一条索引
3. key_len：查看复合索引被利用了多少字节，检验最左前缀原则

#### 套入你SQL的优化流程

1. p表 possible_keys=NULL → 缺少 user_id 索引
```sql
CREATE INDEX idx_proj_uid ON t_project(user_id);
```
2. 再次 explain
```
possible_keys：idx_proj_uid
key：idx_proj_uid
type：ref
```

#### 通俗大白话比喻

- possible_keys＝所有适合本次任务的工具清单
- key＝优化器最后挑出来上手干活的那一件工具

#### 优化小结

1. 等值查询表尽量不要让 possible_keys 为空；
2. 候选索引存在但没有被选中，排查索引选择性、SQL写法、隐式转换；
3. 按需创建复合覆盖索引，兼顾 where 查询条件 + select 返回字段。

#### 顺带回顾整套 explain 检查优先级

`type > key > possible_keys > key_len > ref > rows > filtered > Extra`

### key‑len 

#### 1、含义

`key_len` 代表**索引在当前SQL里实际使用到的字节长度**，用来判断复合索引用到了前边几列、是否遵守最左前缀匹配原则。

> 只统计二级索引存储字段占用字节，不包含主键、额外存储开销。

#### 2、你案例中的数值

```
key_len=8
```
你的索引字段是 `user_id bigint`
- bigint 固定占用 **8 byte**
所以只用到 user_id 这一列索引。

#### 3、常见MySQL字段字节大小

```
tinyint   1
smallint  2
int       4
bigint    8
char(n)   n*字符集字节(utf8mb4一个字符4字节)
varchar(n) 可变长度
```

> 允许为 NULL 的索引字段会额外占用1字节标记位；
> char/varchar 变长字段会多出2字节存储长度。

#### 举例‑单列索引

```sql
create index idx_uid on t_project(user_id);
-- where user_id=1
key_len = 8
```

#### 举例‑复合索引

```sql
CREATE INDEX idx_com ON t_project(user_id,stars,category);
```
1. `where user_id=1`
只命中第一列，key_len=8
2. `where user_id=1 and stars=342`
命中前两列，bigint+int，8+4=12，key_len=12

通过 key‑len 就能一眼看出复合索引被利用了几列。

#### 4、核心作用

1. 判断复合索引有没有充分利用；
2. 检查有没有违反最左前缀法则，导致索引截断；
3. 判断字段是否允许null带来额外开销。

#### 5、结合你整条执行计划复盘

- t_skill、t_timeline 索引都是 user_id，key_len=8，仅使用 user_id
- t_project 当前没有索引，key=NULL、key_len=NULL

创建普通索引：
```sql
CREATE INDEX idx_proj_uid ON t_project(user_id);
```
此时 p表 key_len=8。

创建覆盖复合索引：
```sql
CREATE INDEX idx_cover ON t_project(user_id,title,category,stars);
```
查询条件只有 user_id=1，key_len依旧等于8，后面title、category、stars只用来覆盖索引、避免回表，不会参与索引条件筛选。

#### 6、调优总结

1. 等值条件字段放在复合索引最前面；
2. 通过 key‑len 判断命中索引列数量；
3. 尽量让 key‑len 尽可能大，充分利用复合索引；
4. 索引字段设置为 NOT NULL，可以省去NULL标识字节开销。

#### 快速回顾 explain 查看优先级

`type > key > possible_keys > key_len > ref > rows > filtered > Extra`



### explain‑rows 

#### 含义

`rows` 是 **MySQL优化器预估需要扫描多少行数据，才可以找到符合查询条件的数据**，属于预估数值，并不是真实检索行数。
数值越小，索引效率越好。

拿你当前执行计划举例：
```
p(t_project): rows=2
t(t_timeline): rows=2
s(t_skill): rows=3
```
解读：
1. t_project 需要预估扫描2行（全表扫描）
2. 根据索引查找 t_timeline 的 user_id=1，预估扫描2条索引行
3. t_skill 索引检索预估扫描3行

#### 原理

InnoDB依靠索引统计信息（页采样）估算行数，所以是近似值；数据表频繁增删之后统计信息会有偏差，可以执行
```sql
ANALYZE TABLE t_project;
```
刷新表索引统计，让 rows 预估更加精准。

#### 不同 type 之下 rows 的区别

1. **type=ALL（全表扫描）**
rows = 整张数据表总行数，需要遍历全部记录。
你现在 p 表type=ALL、rows=2；现在数据少速度尚可，后续上千上万条之后性能会急剧下滑。

2. **type=ref（普通二级索引等值查询）**
rows=预估该索引键值下的数据条数，理想情况下数值很小。

3. **type=range（范围索引扫描 > < between）**
rows代表范围区间预估命中行数，范围越大rows越大、性能越差。

#### rows 在联表 SQL 的作用（嵌套循环连接）

MySQL 嵌套循环 join：驱动表读取一条记录，再去被驱动表索引匹配数据
> **优先选择 rows 最小的表充当驱动表，可以减少循环次数**
> inner‑join 的优化器会自动挑选行数最少表作为驱动表。

#### 调优思路

1. 添加合适索引，压低 rows 的预估扫描行数；
2. 复合索引遵守最左前缀原则，精准过滤，减少索引扫描行数；
3. 范围条件放在复合索引末尾，防止索引截断，造成rows飙升；
4. 定期 `ANALYZE TABLE` 更新索引统计，避免优化器做出错误行数预估、选错索引。

#### 和你整套执行计划串联总结

1. p表 type=ALL、key=NULL、rows=2，全表扫描；建立 user_id 索引后 type=ref，rows变成匹配user_id=1的数据行数；
2. s、t 表已经成功命中索引、rows数值很小，状态正常。

#### 完整检查优先级再回顾

`type > key > possible_keys > key_len > ref > rows > filtered > Extra`



### explain‑filtered 

#### 基础释义

`filtered` 为百分比（0‑100），含义：
> 存储引擎读取出来的数据行里，有多少比例能够满足查询条件，会被保留下来。
> 公式：
> `有效行数 = rows × filtered / 100`

#### 结合你的执行计划

```
p：rows=2，filtered=50.00 → 2 × 50% = 1条符合 user_id=1
t：rows=2，filtered=100.00
s：rows=3，filtered=100.00
```
解读：
1. t_project 做全表扫描，引擎读出全部2行，之后由Server层筛选，只剩1条满足条件；因此 Extra 出现 `Using where`。
2. t_timeline、t_skill 通过索引直接定位到符合条件的数据，读取出来的行全部可用，过滤比例100%。

#### 数值好坏标准

1. **filtered=100%（理想）**
索引已经在引擎层筛选完毕，读到的数据全部符合where条件，服务层不需要再大量过滤，一般不会出现 `Using where`。

2. filtered 数值越低越差
代表引擎读取大量无用数据，之后交给MySQL服务层进行筛选，CPU开销上升。
典型场景：全表扫描、索引选择性很差，索引筛选之后残留大量数据。

#### 和索引之间的关系

- 没有索引、type=ALL：往往 filtered 偏低
- 等值索引命中 type=ref：大多 filtered=100

#### 针对你这条SQL优化之后的变化

```sql
CREATE INDEX idx_proj_uid ON t_project(user_id);
```
索引直接定位 user_id=1 的记录：
- type 变为 ref
- filtered=100
- Extra 的 `Using where` 消失

#### 调优总结

1. 等值查询尽量让 filtered 趋近100%；
2. filtered很低，代表索引失效或者缺少合适索引，引擎读取很多无效行；
3. 复合索引把where等值条件放在前边，提前过滤数据，减少扫描行数、拉高filtered。

#### explain全套检查顺序复盘

`type > key > possible_keys > key_len > ref > rows > filtered > Extra`

## MySQL的SQL调优

MySQL的SQL调优是一个系统性工程，涉及从硬件、操作系统、数据库配置到SQL语句本身及索引设计的多个层面。为了便于理解和实操，我们可以将其分为**六大详细分类**来进行介绍。

---

### 一、 诊断与分析工具

调优的第一步是找出问题所在，MySQL提供了丰富的工具来帮助我们诊断SQL性能瓶颈。

1. **慢查询日志**
   * **作用**：记录执行时间超过设定阈值的SQL。
   * **配置**：`slow_query_log = ON`，`long_query_time = 1`（超过1秒记录）。
   * **分析工具**：`mysqldumpslow` 或 `pt-query-digest`（Percona Toolkit，按耗时/次数聚合分析慢SQL）。
2. **EXPLAIN 执行计划**
   * **作用**：查看SQL的执行路径，是日常调优最核心的工具。
   * **核心字段**：
     * `type`：访问类型，从好到坏依次是 `system > const > eq_ref > ref > range > index > ALL`。通常要求达到 `range` 或 `ref` 级别，杜绝 `ALL`（全表扫描）。
     * `possible_keys`：可能用到的索引。
     * `key`：实际用到的索引。
     * `rows`：预估扫描行数，越小越好。
     * `Extra`：额外信息。`Using index`（覆盖索引，极佳）、`Using where`（过滤）、`Using temporary`（用临时表，需优化）、`Using filesort`（需额外排序，需优化）。
3. **SHOW PROFILE**
   * **作用**：查看SQL执行的具体时间消耗在哪个阶段（如 Sending data、Sorting result）。
   * **用法**：`SET profiling = 1;` 执行SQL后 `SHOW PROFILE FOR QUERY [Query_ID];`
4. **Performance Schema & Sys Schema**
   * **作用**：MySQL内置的性能监控引擎，可以查看锁等待、IO消耗、内存使用等底层详细数据。

---

### 二、 索引调优

索引是SQL调优最直接、最有效的手段。

1. **索引失效场景（坑点排查）**
   * **计算/函数**：`WHERE age + 1 = 20` 或 `WHERE YEAR(create_time) = 2023` 会导致失效，应改为 `WHERE age = 19` 或 `WHERE create_time >= '2023-01-01' AND create_time < '2024-01-01'`。
   * **隐式类型转换**：字符串列 `WHERE phone = 13800000000`（无引号）会导致全表扫描。
   * **最左前缀法则**：联合索引 `(a,b,c)`，查询必须包含 `a`，才能走索引；如果跳过 `b` 直接查 `c`，`c` 也无法走索引。
   * **LIKE查询**：`LIKE '%abc'`（左边带%）无法走索引；`LIKE 'abc%'` 可以走范围索引。
   * **OR条件**：如果OR前后两列没有同时建索引，会导致全表扫描。
   * **不等于操作**：`!=`、`<>`、`NOT IN` 通常会导致索引失效（取决于数据分布，优化器可能认为全表扫描更快）。
2. **高级索引应用**
   * **覆盖索引**：查询的列被索引完全覆盖，避免回表查询，`Extra` 会显示 `Using index`。
   * **索引下推**：MySQL 5.6+ 引入。在联合索引 `(a,b)` 查询 `WHERE a=? AND b LIKE '%xx'` 时，存储引擎层在利用 `a` 取出数据后，直接在内部判断 `b` 是否满足条件，减少回表次数。
   * **前缀索引**：对长字符串（如URL、长文本），只索引前N个字符，节省空间。`ALTER TABLE t ADD INDEX idx_str(str(10))`。

---

### 三、 SQL语句编写调优

即使有了索引，糟糕的SQL写法依然会拖垮数据库。

1. **SELECT * 的避免**
   * 强制写明需要的列，这不仅减少网络和内存消耗，还能增加使用**覆盖索引**的概率。
2. **大分页优化**
   * **问题**：`LIMIT 1000000, 10` 需要扫描100万行后丢弃，再取10行，极慢。
   * **优化方案1（延迟关联/子查询）**：先通过覆盖索引查出主键，再关联原表取数据。
     `SELECT * FROM table t1 INNER JOIN (SELECT id FROM table ORDER BY id LIMIT 1000000, 10) t2 ON t1.id = t2.id;`
   * **优化方案2（游标/游标标记）**：记住上一页的最大ID，下一页查询时加上 `WHERE id > max_id LIMIT 10`。
3. **JOIN 优化**
   * **小表驱动大表**：MySQL的 Nested-Loop Join 机制下，驱动表结果集越小，循环次数越少。如果优化器选错驱动表，可使用 `STRAIGHT_JOIN` 强制指定。
   * **被驱动表的关联字段必须有索引**，否则每次循环都会全表扫描。
   * 减少 JOIN 的表数量，一般不超过3张。
4. **聚合与排序优化**
   * **ORDER BY**：尽量利用索引的天然顺序，避免出现 `Using filesort`。如果必须 filesort，尽量使用单路排序（缓冲区够大）。
   * **GROUP BY**：默认会进行隐式排序，如果不需要排序，加上 `ORDER BY NULL`（MySQL 8.0 已废弃该隐式行为）。
5. **批量操作优化**
   * 大批量插入数据时，使用批量 `INSERT INTO ... VALUES (),(),()` 代替单条循环插入。
   * 如果数据量极大（百万级），先禁用索引 `ALTER TABLE ... DISABLE KEYS`，导入数据后再启用 `ENABLE KEYS`，或通过先删索引、建表后加索引的方式。
6. **IN 与 EXISTS 的选择**
   * **IN** 适合于外表大、内表小的情况（以外表为驱动表）。
   * **EXISTS** 适合于外表小、内表大的情况（以外表作循环，每次去内表验证）。

---

### 四、 表结构与数据类型调优

好的表结构是高性能的基础。

1. **数据类型选择**
   * **越小越好**：能用 `TINYINT` 就不用 `INT`，能存时间戳就不用 `VARCHAR` 存日期。
   * **整型 vs 字符串**：尽量使用整型。如果可以选择 `BIGINT` 作为主键，就不要用 `UUID`（随机生成导致B+树页分裂频繁，插入慢，占用空间大）。
   * **时间类型**：`DATETIME`（8字节，与时区无关） vs `TIMESTAMP`（4字节，到2038年受限，带时区转换）。根据业务选择。
   * **VARCHAR vs CHAR**：长度固定用 `CHAR`（如手机号定长11位），长度可变用 `VARCHAR`。
2. **避免NULL值**
   * 尽量将字段设置为 `NOT NULL` 并赋予默认值（如 0 或 ''）。NULL值不仅占用额外字节，还会导致索引统计复杂，且在拼接/比较时容易产生意外结果。
3. **适度反范式设计**
   * 为了追求绝对的三范式，可能需要大量的JOIN。为了查询性能，可以适当在表中冗余一些字段（空间换时间）。
4. **大表垂直拆分**
   * 将不常用的大字段（如TEXT、BLOB）单独拆分到扩展表中，减少主表的体积，提高缓存命中率。

---

### 五、 数据库架构与配置调优

当单机SQL调优到极限，需要借助架构和参数提升性能。

1. **核心参数调优**
   * **`innodb_buffer_pool_size`**：InnoDB缓存池大小，存放数据和索引。这是MySQL最重要的参数，通常设置为物理内存的 60%~80%。
   * **`innodb_flush_log_at_trx_commit`**：控制日志刷盘策略。设为1（最高安全，每次提交刷盘），设为2（高性能，每次提交写到OS Cache，每秒刷盘，宕机会丢1秒数据）。
   * **`sync_binlog`**：控制Binlog刷盘。设为1最安全，设为0或N性能好但有丢失风险。
   * **`innodb_io_capacity`**：根据磁盘IO性能调整，SSD通常可设为 2000~5000。
2. **读写分离**
   * 主库负责写，从库负责读，缓解单库读压力。
3. **分库分表**
   * 当单表数据量超过千万级，B+树高度增加，性能下降。通过水平拆分（分表）将数据分散到多个节点。

---

### 六、 MySQL 8.0+ 新特性调优

利用新版本的特性简化调优过程。

1. **隐藏索引**
   * 测试删除索引的影响时，可以先将索引设为不可见（`ALTER TABLE t ALTER INDEX idx INVISIBLE`），观察无影响后再物理删除，降低线上风险。
2. **Hash Join (哈希连接)**
   * MySQL 8.0.18 引入，替代了部分嵌套循环连接。当被驱动表没有索引时，Hash Join 能大幅提升等值JOIN的性能。
3. **并行查询**
   * MySQL 8.0.14+ 支持并行聚集索引扫描等操作，加快大表的全表扫描或大范围查询。

### 七、优化思路

多表JOIN联合查询：

思路1：先通过多分支 UNION‑ALL 做权限过滤，精简出主键 ID 集合，再用小 ID 集合驱动联表查询。

思路2：单次扫描主表 + OR 合并全部权限条件，优先做数据集裁剪，INNER‑JOIN 驱动顺序按需选择

我认为：1.提前缩小结果集，尽早过滤晚做关联，同时尽可能减少全表扫描次数。2.UNION ALL + 主键ID 集合。3.复合索引 / or，建立合适的索引

#### 一、第一层：业务与SQL架构优化（优先级最高，收益最大）

> SQL写法再完美，查询大批量无用数据也一定会慢

1. **提前缩小结果集，尽早过滤，晚做关联**
   能在where过滤就不要等到join之后再过滤；先筛条件、数据集变小之后再JOIN多表。
   反面案例：你原来SQL先查全部权限ID，最后才过滤发票模糊条件。
2. 分页、时间区间强制裁剪数据
   所有业务查询**必带时间范围**，禁止不带时间条件全表扫描。
3. 拆分大SQL，拒绝单条SQL承担过多逻辑
   超复杂多表、多权限条件SQL，可以拆成临时表、中间表、缓存；
   大查询拆成多条小SQL远比一条巨型SQL性能好。
4. 冷热数据分离
   历史归档数据迁移至归档表，业务表只存放近期热点数据。
5. 缓存兜底
   高频查询结果放入 Redis，避开数据库查询。

#### 二、第二层：表连接 JOIN 优化（Nested‑Loop‑Join核心准则）

1. **小表驱动大表**
   Nested‑Loop Join：外层循环（驱动表）结果集越小，循环次数越少。
   - INNER JOIN：优化器自动选择驱动表；选错用 `STRAIGHT_JOIN` 强制驱动表
   - LEFT JOIN：**左表固定是驱动表，小表放左边**
2. 被驱动表关联字段必须建立索引
   内层循环依靠索引快速命中，否则每次全表扫描。
3. 优先 Inner‑Join，谨慎使用 LEFT JOIN
   Inner‑Join可以双向选择最优驱动表；Left‑Join强制左表驱动。
4. 多表Join顺序：**由小到大依次关联**
   数据集A(100条) → join B → join C，每一步之后数据集持续缩小
5. 杜绝笛卡尔积，关联条件不能缺失

#### 三、第三层：子查询、条件、集合写法优化

##### 1. UNION ALL / UNION

- `UNION ALL`：合并结果、不去重，速度更快；
- `UNION` = UNION‑ALL + 自动distinct去重，开销更高；

> 像你原先7个权限分支用 UNION‑ALL，会多次扫描主表；优先改用 **OR 多条件合并，单次查表**

##### 2. EXISTS > IN

1. `EXISTS`：外层循环，子查询一旦命中立刻返回true，适合大外表、小内层表；
2. IN：先一次性查出子查询全部集合；子查询数据量大性能很差；
3. 禁止 `select *` 做子查询，只查主键字段

##### 3. OR 条件优化

- 同一张表多个权限判断，使用 OR 合并条件，避免多次查表
- OR 字段都需要建立复合索引或者函数索引；
- 复杂多OR权限条件，也可以使用 `CASE WHEN`

##### 4. 避免 SELECT *

只查询业务需要字段，减少IO、网络传输、回表开销

#### 四、第四层：索引优化（最核心落地手段）

##### 1. 复合索引最左前缀原则

索引(a,b,c) 支持 a / a+b / a+b+c 查询；不能跳过a直接使用b、c。

> 建立索引顺序：等值字段 > 范围字段 > 排序字段
> `delete_flag(等值)、business_type(等值)、request_date(时间范围)`

##### 2. 规避索引失效的坑

1. 索引字段做运算、函数：`upper(name)`、date(字段)、+‑*/运算
   解决方案：**创建函数索引**
2. 前导通配符 `like '%xxx'`，普通索引失效，函数索引适配
3. 隐式类型转换：字符串索引传入数字
4. or 两边字段一方无索引，整体索引失效
5. order by、group by、sort 字段没有索引，触发filesort临时排序

##### 3. 常见索引选型

- 等值查询：B‑Tree普通索引
- 模糊检索、文本：全文索引
- 重复极低、唯一字段：唯一索引
- 不要建立过多索引，索引会拖慢insert/update/delete

#### 五、第五层、函数、排序、去重优化

1. COUNT优化
   - `count(主键id)` > count(1) > count(*)
   - `COUNT(DISTINCT col)` 开销很高，尽量提前去重
2. 排序
   能依靠索引有序就不要手动order by；大结果集排序会产生临时表
3. 避免大表distinct去重
   先过滤缩小数据集之后再去重
4. 不要在SQL中做复杂计算，计算下沉至业务代码

#### 六、第六层、SQL执行计划与优化器调优

1. 必看执行计划 explain / explain‑plan
   观察：全表扫描TABLE ACCESS FULL、索引命中、驱动表、嵌套循环、哈希连接、临时表
2. 强制指定驱动表：STRAIGHT_JOIN
3. 统计信息更新（Oracle非常关键）
   统计信息老旧会造成优化器选错执行路径，导致SQL突然变慢

#### 七、第七层 数据库配置层面优化

1. 调整连接池(HikariCP/Druid)参数，合理设置连接数
2. 缓冲池、内存区配置调高，热点数据常驻内存，减少磁盘IO
3. 慢SQL监控，定期抓取慢查询进行优化

#### 八、第八层 业务层面优化（治本方案）

1. 分页深分页优化（limit 10000,10 → 主键书签分页）
2. 大事务拆分，事务尽量短小，减少行锁、表锁时长
3. 大批量数据：批量插入、分批提交
4. 合理分表：水平分表、垂直分表，单表数据量控制在千万以内

```
先过滤，后关联；
小表驱大表；
等值索引最在前；
少用union多次查，or合并一次查；
exists优于in；
禁止索引字段运算；
select不要带星号；
晚排序、晚去重；
尽早裁剪数据集
```

#### 九、代码示例

##### 一、先说清楚这条SQL现在**采用的写法思路**

###### 1、当前使用的优化思想：**先查全集ID、外层再做二次过滤**

执行逻辑拆解：

1. 内层7条`UNION ALL` 从 `RPC_PAYMENT_REQUEST` 查询出**所有满足7种权限条件的 COST_REQUEST_ID**
2. 内层嵌套一层 `DISTINCT` 先去重，拿到不重复申请单号集合
3. 外层再把这批ID回表关联主表、发票表，最后叠加**发票号模糊条件 `INVOICE_NUM like '%INV001%'`**

> 设计初衷：
> 权限判断逻辑很重（递归部门、工作流联表、合同组员关联、子查询EXISTS）；
> 先经过7分支权限筛选拿到最小数据集（仅主键ID），**避免带着多表权限关联去联表查询发票表**，减少大表Join开销。

```sql
SQL优化前
SELECT COUNT(DISTINCT t.COUNT_NO) COUNT_NO
FROM (
    SELECT t_id.COST_REQUEST_ID COUNT_NO
    FROM (
        SELECT DISTINCT COST_REQUEST_ID
        FROM (
            -- ====== 分支1: 创建人 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND scpr.CREATED_BY = '117510'

            UNION ALL

            -- ====== 分支2: OU权限 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND scpr.OU_ORG_CODE IN ('ORG001', 'ORG002')

            UNION ALL

            -- ====== 分支3: 草稿人 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND EXISTS (
                    SELECT 1 FROM RPC.SEC_STAFF_INNER heu
                    WHERE heu.staff_no = '117510'
                      AND heu.staff_no = scpr.DRAFT_EMPLOYEE_ID
                )

            UNION ALL

            -- ====== 分支4: 申请人 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND EXISTS (
                    SELECT 1 FROM RPC.SEC_STAFF_INNER heu
                    WHERE heu.staff_no = '117510'
                      AND heu.staff_no = scpr.REQUEST_EMPLOYEE_ID
                )

            UNION ALL

            -- ====== 分支5: 流程处理人 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND EXISTS (
                    SELECT 1
                    FROM RPC.RPC_WORKFLOW_INS rwi_sub
                    JOIN RPC.RPC_WORKFLOW_LOG rwl_sub
                        ON rwi_sub.FLOW_INS_ID = rwl_sub.FLOW_INS_ID
                    WHERE rwi_sub.PEER_FLOW_ID = scpr.BPM_CASE_ID
                      AND rwl_sub.OPERATOR = '117510'
                )

            UNION ALL

            -- ====== 分支6: 所属部门 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND scpr.REQUEST_DEPARTMENT IN (
                    SELECT ORGANIZE_ID
                    FROM RPC.SEC_ORGANIZE
                    START WITH ORGANIZE_ID IN ('DEPT001', 'DEPT002')
                    CONNECT BY PRIOR ORGANIZE_ID = parent_ORGANIZE_ID
                )

            UNION ALL

            -- ====== 分支7: 合同组员 ======
            SELECT scpr.COST_REQUEST_ID
            FROM RPC.RPC_PAYMENT_REQUEST scpr
            WHERE scpr.DELETE_FLAG = 'N'
              AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
              AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
              AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
              AND EXISTS (
                    SELECT 1
                    FROM RPC.RPC_PAY_EMS_INVO_LINE eil
                    JOIN RPC.RPC_CONTRACT_GROUP_MEMBER cgm
                        ON cgm.CONTRACT_ID = eil.CONTRACT_ID
                    WHERE eil.COST_REQUEST_ID = scpr.COST_REQUEST_ID
                      AND eil.DELETE_FLAG = 'N'
                      AND cgm.GROUP_PERSON_ID = '117510'
                )
        ) t_id_inner
    ) t_id
    LEFT JOIN RPC.RPC_PAYMENT_REQUEST scpr
        ON scpr.COST_REQUEST_ID = t_id.COST_REQUEST_ID
    LEFT JOIN RPC.RPC_PAYMENT_INVOICE scdi
        ON scdi.COST_REQUEST_ID = scpr.COST_REQUEST_ID
        AND scdi.DELETE_FLAG = 'N'
        AND scdi.INVOICE_TYPE = 'STANDARD'
    WHERE scpr.DELETE_FLAG = 'N'
      AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
      AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
      AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
      AND upper(scdi.INVOICE_NUM) LIKE upper('%INV001%')
) t
```



---

###### 二、致命缺点（这套写法本质是「伪优化」）

1. **主查询重复书写全部公共筛选条件**
   内层7个子查询、外层查询重复复制：

```sql
DELETE_FLAG = 'N'
BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
upper(REQUEST_TITLE) LIKE upper('%测试%')
REQUEST_DATE >= 2024‑01‑01
```

- 维护麻烦、容易漏改条件；
- Oracle优化器无法合并谓词，多次扫描同一张大表。

2. **UNION‑ALL + 七层子查询，7次全表扫描主业务表**
   `UNION ALL` 会依次执行7条select，**7次扫描 RPC_PAYMENT_REQUEST 大表**，IO成本极高。

> `UNION ALL`不会自动去重，内层再套一层`DISTINCT`做ID去重。

3. 外层再次回表关联主表，等于**主表至少访问8次**。

4. 后置过滤发票条件：先查全部权限ID，关联发票之后再筛发票编号；
   没有提前缩小数据集。

---

##### 三、正确的优化思路，分层次讲解

###### 优化方向1：把7次表扫描 → **单次扫描主表 + OR多条件权限判断（最核心）**

不要再使用 `UNION ALL` 拆分七个分支。
在一次访问主表之后，`WHERE` 使用 `OR` 整合全部7个权限条件，**主表只查询1次**

```sql
WHERE scpr.DELETE_FLAG = 'N'
AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
AND (
    --条件1 创建人
    scpr.CREATED_BY = '117510'
    OR
    --条件2 OU权限
    scpr.OU_ORG_CODE IN ('ORG001', 'ORG002')
    OR
    --条件3 草稿人
    EXISTS(...)
    OR
    --条件4 申请人
    EXISTS(...)
    OR
    --条件5 流程处理人
    EXISTS(...)
    OR
    --条件6 所属部门递归
    scpr.REQUEST_DEPARTMENT IN (子查询)
    OR
    --条件7 合同组员
    EXISTS(...)
)
```

> 对比：原先7次全表扫描 → 现在仅**一次扫描主表**，性能提升巨大。

###### 优化方向2：驱动表策略（Nested‑Loop Join 小表驱动大表）

1. 优先筛选发票条件：发票表 `RPC_PAYMENT_INVOICE` 先模糊匹配 `INVOICE_NUM LIKE '%INV001%'`，拿到少量的COST_REQUEST_ID
2. **小结果集（发票筛选后的申请ID）驱动主业务表**

> 也就是 **先查发票 → 再关联付款申请单 → 再校验7种权限**
> 如果发票过滤后数据量极少，是最优路线。

###### 优化方向3：拆分重型EXISTS子查询、建立复合索引

建立联合索引，避免全表扫描：

```
RPC_PAYMENT_REQUEST
索引1：(DELETE_FLAG,BUSINESS_TYPE,REQUEST_DATE)
```

各个exists关联表，关联字段建立索引：

- RPC_WORKFLOW_INS (PEER_FLOW_ID)
- RPC_WORKFLOW_LOG(FLOW_INS_ID,OPERATOR)
- RPC_CONTRACT_GROUP_MEMBER(GROUP_PERSON_ID,CONTRACT_ID)

###### 优化方向4：避免函数索引陷阱

`UPPER(REQUEST_TITLE) LIKE UPPER('%测试%')`

- 前导通配符`%xxx`无法命中普通索引；
  需要创建**函数索引**：

```sql
CREATE INDEX idx_req_title_upper ON RPC_PAYMENT_REQUEST (UPPER(REQUEST_TITLE));
```

###### 优化方向5：抛弃多层嵌套子查询、不要先查ID再回表

原SQL模式：`查ID → distinct → join回原表`
属于典型的**回表二次查询**，属于低效写法。
单层查询一次取出所有字段即可。

###### 优化方向6：区分 UNION ALL / UNION

- `UNION ALL`：直接合并结果、不去重，速度快；
- `UNION` = UNION‑ALL + 自动distinct去重
  原SQL：`UNION ALL`之后外层手动DISTINCT，多余开销。

---

##### 四、总结：原SQL属于什么优化思路？

1. **思路名称：ID先行过滤法（主键预筛选）**
   原理：先筛选权限条件，仅查出主键ID，缩减数据集，之后再关联多表；
   适合权限判断非常沉重、关联表极多的场景。

2. **但是该写法在当前案例属于得不偿失**
   因为它触发7次重复扫描主大表，开销 > “提前拿到ID”带来的收益。

##### 五、最终优化后的标准SQL骨架（Oracle）

这个优化后的SQL 使用OR可能索引无法命中，最终还是要比较一下两者的速度。

```sql
SELECT COUNT(DISTINCT scpr.COST_REQUEST_ID) COUNT_NO
FROM RPC.RPC_PAYMENT_REQUEST scpr
INNER JOIN RPC.RPC_PAYMENT_INVOICE scdi
    ON scpr.COST_REQUEST_ID = scdi.COST_REQUEST_ID
WHERE scpr.DELETE_FLAG = 'N'
  AND scdi.DELETE_FLAG = 'N'
  AND scdi.INVOICE_TYPE = 'STANDARD'
  AND scpr.BUSINESS_TYPE IN ('ERP_PO', 'ERP_XC')
  AND upper(scpr.REQUEST_TITLE) LIKE upper('%测试%')
  AND scpr.REQUEST_DATE >= TO_DATE('2024-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS')
  AND upper(scdi.INVOICE_NUM) LIKE upper('%INV001%')
  AND (
        scpr.CREATED_BY = '117510'
        OR scpr.OU_ORG_CODE IN ('ORG001', 'ORG002')
        OR EXISTS (SELECT 1 FROM RPC.SEC_STAFF_INNER heu WHERE heu.staff_no='117510' AND heu.staff_no=scpr.DRAFT_EMPLOYEE_ID)
        OR EXISTS (SELECT 1 FROM RPC.SEC_STAFF_INNER heu WHERE heu.staff_no='117510' AND heu.staff_no=scpr.REQUEST_EMPLOYEE_ID)
        OR EXISTS (SELECT 1 FROM RPC.RPC_WORKFLOW_INS rwi_sub JOIN RPC.RPC_WORKFLOW_LOG rwl_sub ON rwi_sub.FLOW_INS_ID=rwl_sub.FLOW_INS_ID WHERE rwi_sub.PEER_FLOW_ID = scpr.BPM_CASE_ID AND rwl_sub.OPERATOR = '117510')
        OR scpr.REQUEST_DEPARTMENT IN (SELECT ORGANIZE_ID FROM RPC.SEC_ORGANIZE START WITH ORGANIZE_ID IN ('DEPT001','DEPT002') CONNECT BY PRIOR ORGANIZE_ID=parent_ORGANIZE_ID)
        OR EXISTS (SELECT 1 FROM RPC.RPC_PAY_EMS_INVO_LINE eil JOIN RPC.RPC_CONTRACT_GROUP_MEMBER cgm ON cgm.CONTRACT_ID=eil.CONTRACT_ID WHERE eil.COST_REQUEST_ID=scpr.COST_REQUEST_ID AND eil.DELETE_FLAG='N' AND cgm.GROUP_PERSON_ID='117510')
  );
```

---

### 六、可选二种执行策略（根据数据量选择）

1. **发票筛选后数据很少**：先查发票表 → Nested‑Loop Join驱动付款申请表，再校验7项权限
2. **权限过滤之后申请单数量很少、发票数量庞大**：先查有权限的付款申请，再关联发票表

### 总结

MySQL SQL调优的黄金法则是：**“先定位，后优化；索引优先，结构为辅；架构兜底”**。
实际操作中，应先用慢日志和`EXPLAIN`发现问题，然后通过修改SQL写法和增加索引解决80%的问题。如果仍不满足，再考虑表结构重构和分库分表等重型架构操作。

## MySQL 核心优化机制

### MySQL 索引跳跃扫描（Index Skip Scan）

MySQL 8.0.13 引入的**索引跳跃扫描（Index Skip Scan）**，是优化器在**不满足联合索引最左前缀**时，仍能利用联合索引的关键机制，核心是**跳过前导列、按非前导列过滤**，大幅减少扫描范围。

---

#### 一、核心原理

- 场景：联合索引 `(A,B,C)`，查询只带 `B=? AND C=?`（无 A 条件）。
- 做法：优化器枚举 A 的**不同取值**，对每个 A 构造 `(A=val AND B=? AND C=?)` 的范围条件，依次做范围扫描，最后合并结果。
- 效果：把全索引扫描转为多次小范围索引扫描，显著减少 IO 与过滤开销。

一句话：**不满足最左前缀？跳过前导列，按后续列分段查**。

---

#### 二、关键条件（必须满足）

1. 单表查询（不支持 JOIN）
2. 无 `GROUP BY` / `DISTINCT`（8.0.41 对部分 IN/EXISTS 场景放宽）
3. 联合索引至少包含查询列（只扫索引列）
4. 前导列 A 为**低基数**（取值少，如性别、部门），否则成本过高
5. 查询含**范围条件**（如 `>`、`<`、`BETWEEN`）或 `IN()`
6. 优化器开关 `skip_scan=on`（默认开启）

---

#### 三、示例与 EXPLAIN 识别

##### 1. 表与索引

```sql
CREATE TABLE employee (
  dept    CHAR(2),     -- 联合索引前导列，低基数
  age     INT,
  name    VARCHAR(20),
  INDEX idx_dept_age (dept, age)
);
```

##### 2. 触发 ISS 的查询

```sql
-- 无 dept 条件，只查 age，触发 ISS
SELECT name FROM employee WHERE age > 30;
```

##### 3. EXPLAIN 识别

- `type`：`range`
- `key`：`idx_dept_age`
- `Extra`：`Using index for skip scan`（关键标志）

---

#### 四、性能对比（压测数据）

| 优化方式     | QPS  | 平均延迟(ms) | 扫描行数 | 提升倍数 |
| ------------ | ---- | ------------ | -------- | -------- |
| 全表扫描     | 128  | 251          | 100万    | 1x       |
| 强制索引     | 2450 | 13           | 10万     | ~19x     |
| 索引跳跃扫描 | 3120 | 10           | 10万     | ~24x     |

> 注：低基数前导列时，ISS 比强制索引更优，因减少了索引树跳转。

---

#### 五、限制与最佳实践

##### 1. 主要限制

- 前导列**高基数**（如用户 ID）：ISS 成本接近全表扫描，应避免
- 不支持多表 JOIN / `GROUP BY` / `DISTINCT`（部分版本放宽）
- 只对 BTREE 索引有效

##### 2. 最佳实践

1. **合理设计联合索引**：将**低基数列**放前导，高基数列放后续，提升 ISS 命中率
2. **优先覆盖索引**：查询列包含在索引中，避免回表，与 ISS 叠加效果更佳
3. **监控与调优**：
   - 用 `EXPLAIN` 确认 `Extra` 包含 `Using index for skip scan`
   - 定期 `ANALYZE TABLE` 保证统计信息准确，优化器成本估算正确
   - 必要时用提示：`/*+ SKIP_SCAN(employee idx_dept_age) */`

---

##### 六、终极背诵版（面试用）

1. **定义**：MySQL 8.0.13 引入，跳过联合索引前导列，按非前导列分段扫描，解决不满足最左前缀的索引利用问题。
2. **适用**：单表、无分组去重、低基数前导列、含范围条件。
3. **标志**：`EXPLAIN` 中 `Extra` 显示 `Using index for skip scan`。
4. **价值**：将全索引扫描转为多次小范围扫描，性能提升显著。

---

### MySQL‑条件下推（ICP索引条件下推）

> 全称：**Index Condition Pushdown，索引条件下推，俗称索引下推**，MySQL5.6 之后推出、InnoDB默认开启的核心优化，**核心目的：减少回表次数、降低磁盘IO**。
> 下面分为原理、有无ICP对比、触发条件、执行计划标识、多种MySQL条件下推分类、ICP和覆盖索引区别、实战案例、踩坑、调优。

#### 一、底层架构先搞懂

MySQL分为两层

1. **Server层**：查询优化器、处理where条件、结果过滤、返回数据
2. **Storage‑Engine存储引擎层(InnoDB)**：读取索引、读取聚簇索引完整行（回表）

##### 没有开启ICP的旧版执行流程

> 联合索引 `idx(name,age)`

```sql
SELECT * FROM user WHERE name LIKE '张%' AND age=20;
```

1. InnoDB存储引擎，根据索引前缀 `name LIKE '张%'`，扫描所有匹配索引条目
2. **全部拿着主键id，挨个回表读取整行数据**
3. 将整行数据交给Server层，再判断 `age=20`，丢弃不符合年龄的数据

痛点：大量索引age不满足条件，仍然执行昂贵的回表IO。

##### 开启ICP（索引条件下推）流程

1. 存储引擎拿到索引 `name、age` 字段
2. **在引擎层直接完成条件过滤 name like '张%' and age=20**
3. 只对过滤之后剩下的少量主键进行回表读取完整行
4. 再交给Server层处理剩余非索引字段的where条件

> 一句话概括：**能够依靠索引字段判断的where条件，从Server层下推至存储引擎层提前过滤，减少回表次数**

#### 二、执行计划如何判断是否命中ICP

```sql
explain select * from user where name like '张%' and age=20;
```

Extra字段标识

- `Using index condition` → 成功触发索引条件下推ICP
- `Using index` → 命中**覆盖索引**，不需要回表

#### 三、ICP触发的硬性条件

1. 查询访问类型为：`range、ref、eq_ref、ref_or_null`；全表扫描type=ALL不会触发ICP
2. 被下推的过滤字段**必须存在当前使用的二级索引当中**
   - 例：索引(a,b,c)，where a=? and b=?，a、b可以下推；where d=?（d不在索引）无法下推
3. 只支持二级索引；主键聚簇索引不会触发ICP
4. 优化器评估下推收益足够，才会启用ICP；少量数据时优化器可能放弃

#### 四、MySQL一共有4种「条件下推」，面试经常混淆

##### 1. ICP 索引条件下推（最常用，InnoDB支持）

只下推**索引内字段**的where条件，引擎层过滤、减少回表。

##### 2. 派生表条件下推 derived_condition_pushdown

外层查询where条件，下压到子查询/派生表内部过滤

```sql
--原SQL
select * from (select id,name from user) t where id>100;
--优化后，条件下推
select * from (select id,name from user where id>100) t;
```

> 避免查询全部数据之后再在外层过滤，提前裁剪数据量，8.0.29之后支持union子查询下压

##### 3. 引擎条件下推 Engine Condition Pushdown

仅NDB集群引擎可用，普通InnoDB无效，可以下压非索引字段条件到存储节点过滤，减少网络传输

##### 4. 谓词下推（通用优化）

多表关联的时候，把单表专属where条件下压到对应的表，先过滤再join，减少join的数据量。

#### 五、ICP 和 覆盖索引(Using index)区别（高频面试题）

| 优化项           | 触发标识              | 原理                                           | 是否回表                   |
| ---------------- | --------------------- | ---------------------------------------------- | -------------------------- |
| 索引条件下推 ICP | Using index condition | 索引字段提前过滤，减少回表次数                 | **仍然需要回表读取完整行** |
| 覆盖索引         | Using index           | 查询所有字段全部包含在二级索引，直接从索引返回 | **不需要回表**             |

举例子
索引 `(name,age)`

1. `select name,age from user where name like '张%' and age=20`
   Extra=Using‑index，命中覆盖索引，直接读取索引返回，无需回表，ICP不会生效（已经不需要回表）
2. `select * from user where name like '张%' and age=20`
   需要查询全部字段，必须回表；触发 `Using index condition`，ICP先在索引层过滤age条件，减少回表。

> 优先级：**覆盖索引 > ICP索引下推**，能做覆盖索引优先覆盖索引。

#### 六、开启、关闭ICP

##### 查看开关

```sql
show variables like 'optimizer_switch';
```

找到参数 `index_condition_pushdown=on`，MySQL5.6+默认开启

##### 全局关闭（极少场景需要关闭做对比测试）

```sql
set global optimizer_switch='index_condition_pushdown=off';
```

#### 七、实战演示

##### 表结构&索引

```sql
create table user(
id int primary key auto_increment,
name varchar(32),
age int,
email varchar(64)
);
create index idx_name_age on user(name,age);
```

##### 案例1：触发ICP

```sql
select * from user where name like '李%' and age between 18 and 30;
```

name、age都在联合索引，引擎层先过滤，只对符合条件id回表读取email，Extra：`Using index condition`

##### 案例2：age不在索引无法下推

```sql
create index idx_name on user(name);
select * from user where name like '李%' and age=20;
```

索引只有name；age无法下推。引擎取出所有name=李开头主键、全部回表，server层过滤age。ICP不会启用。

##### 案例3：命中覆盖索引、ICP失效

```sql
select name,age from user where name like '李%' and age=20;
```

查询字段全部位于索引；Extra=Using index，直接读取索引，不存在回表，ICP没有意义。

#### 八、ICP常见坑点

1. **最左前缀原则失效时ICP收益极低**
   索引(a,b,c) where b=xxx，没有命中a，索引失效全表扫描，ICP不会触发
2. 等值查询唯一主键索引不会触发ICP，主键索引没有回表开销
3. or条件拆分后的字段不在索引内，该部分条件无法下压
4. 索引选择性很差的时候，优化器会放弃索引直接全表扫描，ICP不会启用

#### 九、开发优化建议

1. 建立合适的联合索引，把where过滤字段放进索引，开启ICP提前过滤
2. 查询尽量只查需要字段，优先做成覆盖索引，直接跳过回表
3. like前缀匹配 `xxx%` 适合ICP优化；`%xxx` 索引失效，ICP无法生效
4. 范围查询（between、>、<）非常适合ICP，大量过滤无效索引条目

#### 十、一句话总结

ICP（索引条件下推）就是**把索引字段的where过滤逻辑下沉到InnoDB存储引擎层，在二级索引树提前筛掉无效条目，减少昂贵的回表磁盘IO**。

需要我整理一份 MySQL 查询优化器全部优化清单（MRR、BKA、BNL、ICP、覆盖索引）吗？

### MySQL 覆盖索引（Covering‑Index）

#### 一、通俗定义

> 一条 SQL 需要用到的 **所有查询字段，全部存在二级索引里面**，InnoDB 不需要回到聚簇索引（主键索引）读取完整行数据，直接读取二级索引就可以返回全部数据，该索引就叫**覆盖索引**。

回表：通过二级索引拿到主键 id，再去主键聚簇索引读取整行数据，属于磁盘 IO 开销，是查询性能大头；
**覆盖索引本质就是消灭回表。**

##### InnoDB 索引底层常识

1. 聚簇索引（主键索引）：叶子节点存放一整行全部字段
2. 普通二级索引：叶子节点只存「索引列 + 主键id」

举个例子

```sql
CREATE TABLE `user` (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(32),
  age INT,
  email VARCHAR(64)
);
-- 创建联合二级索引
CREATE INDEX idx_name_age ON user(name,age);
```

二级索引叶子节点存储结构：`(name,age,id)`

---

#### 二、区分普通索引查询 vs 覆盖索引查询

##### 1、需要回表（非覆盖索引）

```sql
SELECT * FROM user WHERE name = '张三';
```

需要 name、age、email；email 不在二级索引 `idx_name_age`
执行流程：

1. 在二级索引找到 name='张三'，拿到主键 id
2. **回表**，拿着 id 查询主键索引，读取整行所有字段

##### 2、命中覆盖索引，无回表

```sql
SELECT name,age FROM user WHERE name = '张三';
```

查询字段 name、age 全部位于二级索引；
引擎直接读取二级索引的数据直接返回，**不需要访问主键索引、不需要回表**。

#### 三、如何确认命中覆盖索引 —— explain Extra

执行 explain，看 Extra 字段

- `Using index` → 成功命中覆盖索引
- `Using index condition` → ICP索引条件下推，依旧需要回表

> 重点区分面试必背

1. Using‑index：覆盖索引、**无需回表**
2. Using index condition：ICP索引下推，只是提前过滤索引数据，**仍然需要回表**

---

#### 四、覆盖索引的优势

1. **省去回表IO，速度大幅提升**
   二级索引树体积远小于聚簇索引，内存可以缓存更多索引页，磁盘访问变少。
2. 减少磁盘IO、缓冲池内存占用
3. 适用于分页、排序、分组，避免大量回表开销
4. 可以配合 ICP，性能收益叠加

#### 五、联合索引建立覆盖索引实战

##### 索引设计口诀：最左前缀 + where条件 + order by + 查询返回字段

```sql
-- 查询语句
select name,age from user where name='xxx' order by age;
-- 建立联合索引
create index idx_name_age on user(name,age);
```

where 过滤字段 name，排序字段 age，返回字段 name、age，索引全部包含，命中覆盖索引。

##### 场景：查询条件、排序、返回字段不一样

```sql
select name,email from user where name='xxx' and age=20;
```

现在返回字段 name、email。email 不在索引，无法覆盖索引。
改造索引：

```sql
create index idx_name_age_email on user(name,age,email);
```

索引结构 `(name,age,email,id)`
where(name,age)、返回(name,email) 全部存在索引，触发覆盖索引。

#### 六、常见业务案例

##### 案例1：深分页优化，搭配覆盖索引（延迟关联）

低效SQL

```sql
select * from user order by id limit 100000,10;
```

优化：先用覆盖索引快速拿到分页id，再关联回表

```sql
SELECT u.* FROM user u
INNER JOIN (
    SELECT id FROM user ORDER BY id LIMIT 100000,10
) t ON u.id = t.id;
```

子查询 `select id` 命中主键覆盖索引，不需要回表。

##### 案例2：group by、distinct 利用覆盖索引

```sql
select distinct name from user;
```

建立索引 idx_name(name)，使用覆盖索引完成去重，不需要读取整行。

##### 案例3：count 计数超级优化

```sql
-- 会回表
select count(*) from user;
-- 选用短小二级索引做覆盖索引扫描，性能更快
select count(id) from user;
```

---

#### 七、覆盖索引容易踩坑

##### 坑1：select * 永远无法触发覆盖索引

除非整张表所有字段全部加到二级索引，现实不可能。

> 开发规范：禁止 select *，只查询业务所需字段，更容易命中覆盖索引

##### 坑2、联合索引顺序错误，导致无法覆盖

索引 `(age,name)`
sql：`select name,age from user where name='张三'`
违反最左前缀，索引失效，自然无法触发覆盖索引。

##### 坑3、主键id默认附带在二级索引末尾

InnoDB二级索引叶子节点自带主键，所以：
索引(a,b)，可以覆盖 `select a,b,id`

##### 坑4、覆盖索引不会存储大字段

text、blob 不会存入二级索引，带有大字段查询永远需要回表，无法使用覆盖索引

##### 坑5、ICP 和覆盖索引互斥

当 SQL 命中覆盖索引，不需要回表，ICP（索引条件下推）就不会触发。

> 优先级：覆盖索引 > ICP索引条件下推

---

#### 八、ICP、覆盖索引、回表三者关系总结

1. 普通查询：二级索引查找 → 回表取整行 → Server层过滤条件
2. ICP索引条件下推：索引层先过滤无效数据，**减少回表次数，仍然需要回表**
3. 覆盖索引：所有返回字段在索引内，**彻底省去回表动作**

#### 九、开发最佳实践

1. 查询只拿需要的字段，拒绝 select *
2. 联合索引设计：`where等值字段 → order by排序字段 → select返回字段`
3. 分页、count、distinct、group‑by优先使用覆盖索引
4. 优先覆盖索引，其次依靠ICP减少回表

如果你需要，我可以整理一份：**ICP、MRR、BKA、BNL、覆盖索引 全套查询优化器机制**。

### MySQL MRR(Multi‑Range Read) 多范围读取 

#### 一、基础定义

MRR 全称 **Multi‑Range Read，多范围读取**，属于 MySQL 查询优化器、InnoDB 的 IO 优化策略，5.6版本正式推出，默认开启。

> 核心目标：**把离散、随机IO，整理成顺序IO，降低磁盘随机读取开销，提升 range 范围查询性能**

#### 二、先搞懂没有 MRR 的缺点

##### 准备表与索引

```sql
create index idx_age on user(age);
```

执行范围SQL：

```sql
select * from user where age in (10,20,30,40);
```

InnoDB‑二级索引存储结构：`age + 主键id`

**关闭MRR时执行流程：**

1. 在二级索引依次查到 age=10，拿到一批主键ID；
2. 拿着id去聚簇索引回表读取行数据；（随机IO）
3. 再查 age=20 → 回表；
4. age=30 → 回表；
5. age=40 → 回表。

> 问题：每次range拿到的主键id是无序的，回表访问主键索引就是**大量随机磁盘IO**，机械硬盘随机IO速度极慢。

#### 三、开启 MRR 的完整执行流程

1. 先读取二级索引，一次性获取所有满足多个range条件对应的主键 id；
2. 将全部主键 ID **放入缓冲块(read‑rnd_buffer)，对主键进行排序**；
3. 根据排序之后有序的主键，访问聚簇索引回表；
4. 有序主键访问聚簇索引 = **顺序磁盘IO**

简单一句话：

> 先收集全部主键、排序、有序回表，随机IO → 顺序IO。

#### 四、MRR 触发条件

1. SQL 使用 range 访问类型：`in、between、>、<、or、like 前缀匹配`
2. 使用二级索引进行范围查询，需要回表查询聚簇索引
3. 优化器评估数据量达到阈值，开启MRR；少量数据不会启用
4. 支持 InnoDB、MyISAM；Memory引擎不会触发

> 等值 ref 查询不会触发 MRR，只有多区间 range 查询生效

#### 五、查看是否开启MRR

```sql
SHOW VARIABLES LIKE 'optimizer_switch';
```

参数：`mrr=on,mrr_cost_based=on`

- mrr=on：开启功能
- mrr_cost_based=on：优化器基于成本自动判断要不要启用MRR

手动开关：

```sql
-- 关闭MRR测试
set global optimizer_switch='mrr=off';
```

缓冲区大小参数，用来存放待排序主键

```sql
show variables like 'read_rnd_buffer_size';
```

缓冲区越大，可以一次性缓存更多主键id、排序；超出缓冲区则分段处理。

#### 六、explain 如何识别 MRR

MRR 不会直接在 Extra 展示标签；
可以通过 `show engine innodb status`、性能观测IO模式判断。

> 区别 ICP、覆盖索引、MRR

1. `Using index`：覆盖索引，无回表
2. `Using index condition`：ICP索引条件下推，引擎层过滤索引字段，减少回表数量
3. MRR：已经确定需要回表，主键排序，优化回表IO模式

#### 七、MRR 适用场景

1. where age in (1,3,5,7,9) 离散多个等值区间
2. where age between 10 and 20 or age between 30 and 50
3. 分页 + 范围查询
4. 二级索引range查询，查询字段包含非索引字段，必须回表

#### 八、MRR 和 ICP 可以叠加生效

执行顺序：

1. ICP（索引条件下推）：在二级索引提前过滤掉无效索引条目，减少主键数量
2. MRR：剩下的主键ID放入缓冲区排序，有序回表读取聚簇索引

> ICP负责**减少回表条数**；MRR负责**优化回表时的IO模式**，二者是先后两道优化。

#### 九、MRR衍生优化‑BKA 批量键访问（Batch Key Access）

顺带区分高频面试容易混淆的两个机制

1. **MRR**：单表range查询，主键排序优化回表IO
2. **BKA**：多表join关联，把驱动表拿到的关联id缓存排序，批量访问被驱动表索引，优化join的随机IO

#### 十、优缺点

##### 优点

1. 将无序主键的随机磁盘IO，变成顺序IO，机械硬盘收益巨大；SSD随机IO较快，收益下降
2. 批量回表，减少存储引擎与server层交互次数

##### 缺点

1. 需要开辟 read_rnd_buffer 内存缓冲区存放主键、消耗内存
2. 数据集极小的时候，排序操作带来额外CPU开销，优化器会自动放弃MRR

#### 十一、开发优化建议

1. 优先建立覆盖索引直接消除回表，不需要MRR；
2. 无法做成覆盖索引、大量range查询，依靠MRR优化回表IO；
3. 不要设置过大 read_rnd_buffer_size，避免占用大量内存；
4. in集合数值较多时MRR收益非常明显。

#### 十二、ICP / MRR / 覆盖索引极简总结

1. 覆盖索引：**不需要回表**，直接索引返回全部字段，最优；
2. ICP索引条件下推：索引层提前过滤，**减少需要回表的行数**；
3. MRR多范围读取：回表前主键排序，**优化回表IO，随机IO转为顺序IO**。

需要我接着讲解 BKA、BNL 两种 Join 优化原理吗。



### MySQL BKA(Batched Key Access) 批量键访问 

#### 一、定义定位

BKA‑批量键访问，是**Join连接专用优化策略**，从 MySQL5.6 推出；底层基于 MRR（多范围读取）机制。

> MRR 负责优化**单表 range 查询回表IO**；BKA 负责优化 **多表关联时，被驱动表的随机索引IO**。

#### 二、先搞懂传统嵌套循环连接 Nested‑Loop‑Join 痛点

##### 测试两张表

- t1 驱动表
- t2 被驱动表，关联字段 `t2.age` 建有二级索引 `idx_age(age)`

```sql
SELECT * FROM t1
INNER JOIN t2 ON t1.age = t2.age;
```

普通嵌套循环 NL‑Join 执行流程（关闭BKA）：

1. 从驱动表 t1 读取一条记录，拿到 age 值
2. 使用 age 去 t2 的索引查找匹配行，回表读取完整数据
3. 取下一条 t1 的age，再次访问 t2 索引

**致命缺陷**
t1 的 age 是乱序的，每次访问 t2 二级索引属于离散随机IO。
驱动表返回上万条数据，就会产生上万次随机磁盘IO，性能很差。

#### 三、开启 BKA 之后完整执行流程

BKA 的本质：**收集驱动表所有关联key → 排序 → 批量有序访问被驱动表索引**

1. 读取驱动表 t1 的一批数据，把所有用于关联的 age‑key 存入 join 缓冲区 `join_buffer`
2. 对缓冲区里面的 age 关键字进行排序
3. 使用有序的key批量查询被驱动表 t2 的二级索引
4. 有序访问索引，随机IO转换成顺序IO
5. 缓冲区装满之后就执行一轮批量查找，循环直至驱动表遍历完毕

> 简单大白话：不要拿一条key就查一次被驱动表；攒一批、排好序、批量查询。

#### 四、BKA 底层依赖 MRR

BKA 就是把 MRR 的排序缓冲区思想，复用在 join 场景：

- MRR：单表range收集主键id排序、有序回表
- BKA：join收集关联键排序、有序访问被驱动表索引

#### 五、开启参数

```sql
show variables like optimizer_switch;
```

关键配置项：`bka=on`，MySQL 默认开启

```sql
# 关闭BKA用于对比测试
set global optimizer_switch='bka=off';
```

join‑buffer 缓冲区大小，决定一次可以缓存多少驱动表关联键

```sql
show variables like join_buffer_size;
```

> 注意：每个连接的每一次join都会分配独立 join_buffer，不要设置过大，防止内存耗尽。

#### 六、BKA 触发条件

1. 使用嵌套循环连接 Nested‑Loop‑Join
2. **被驱动表关联字段存在索引**（没有索引只能使用BNL块嵌套循环）
3. 驱动表结果集足够大；数据集很小优化器评估成本，会跳过BKA
4. inner‑join / left‑join 都支持，right join可以等价改写为left‑join

#### 七、BNL 和 BKA 一定要分清（面试高频）

##### 1. BNL Block Nested‑Loop 块嵌套循环

> **被驱动表没有索引**
> 把驱动表数据放入 join_buffer，循环扫描整张大的被驱动表，在内存里面匹配。
> 缺点：需要全表扫描被驱动表多次。

##### 2. BKA Batched Key‑Access 批量键访问

> **被驱动表有索引**
> 缓存驱动表关联键、排序、批量有序访问被驱动表索引，优化随机IO。

对比总结：

- 无索引 → BNL
- 有索引 → BKA

#### 八、ICP、MRR、BKA 执行先后顺序

一条 range + join SQL完整优化链路

1. ICP：二级索引层过滤无效索引记录，减少主键数量
2. MRR：单表查询主键id排序，有序回表
3. BKA：Join场景缓存关联键排序，批量访问被驱动表索引

#### 九、优缺点

##### 优点

1. 将被驱动表离散随机IO，改成顺序IO，机械硬盘收益巨大
2. 减少存储引擎与Server层的交互次数，批量读取
3. 适合驱动表返回大量关联键的关联查询

##### 缺点

1. 需要占用 join_buffer_size 内存缓冲区
2. 小数据集下，key排序带来额外CPU开销，优化器会自动放弃BKA

#### 十、开发优化建议

1. join 的被驱动表关联字段务必建立索引，优先走BKA，避开BNL全表扫描；
2. 不要盲目调大 join_buffer_size；
3. 驱动表优先是小表；
4. 优先使用覆盖索引消除回表，从根源减少IO开销。

#### 十一、全套优化器机制速记版

1. **ICP索引条件下推**：索引层提前过滤，减少回表行数
2. **MRR多范围读取**：单表回表主键排序，随机IO转顺序IO
3. **BKA批量键访问**：Join场景，缓存排序关联键，批量访问被驱动表索引
4. **BNL块嵌套循环**：无索引时缓冲区缓存驱动表，内存匹配
5. **覆盖索引Using index**：不需要回表

接下来我可以给你完整讲解 BNL、Hash‑Join、Nested‑Loop‑Join 三种连接算法。

### MySQL BNL（Block Nested‑Loop，块嵌套循环连接）

#### 一、基础概述

BNL 是 MySQL 多表关联算法之一，专门应对**被驱动表关联字段没有索引**的场景。

> NL‑Join（普通嵌套循环）：读取驱动表一行、就去检索一次被驱动表；
> BNL：把驱动表一批数据放到 `join_buffer` 缓冲区，一次性批量比对，减少被驱动表全表扫描次数。

#### 二、先看懂普通嵌套循环 Nested‑Loop‑Join 的缺陷

举例：`t1`驱动表（1000条）、`t2`被驱动表（10万条），关联字段 t2.id 无索引

```sql
select * from t1 join t2 on t1.t2_id = t2.id;
```

原生 NL‑Join 流程：

1. 取出 t1 第1条记录
2. 全表扫描 t2 进行匹配
3. 取出 t1 第2条记录
4. 再次全表扫描 t2

> 最坏需要扫描 t2 1000 次，IO开销爆炸。

#### 三、BNL 块嵌套循环执行流程

1. 读取驱动表 t1 的多条记录，存入内存 `join_buffer` 连接缓冲区；
2. 完整扫描一次被驱动表 t2；
3. t2 的每一行，在内存 join_buffer 和缓冲区里所有 t1 记录做匹配；
4. 缓冲区装满之后清空，继续加载下一批驱动表数据，重复上面流程。

举个数值例子

- join_buffer 可以存放 200 条 t1 数据；t1 一共1000行
- 只需要扫描 t2：1000 ÷ 200 = **5次全表扫描**

> 核心收益：大幅降低被驱动表全表扫描次数。

#### 四、BNL、BKA、NL‑Join、Hash‑Join 区分（面试必背）

1. **普通 NL‑Join**
   驱动表一行一次查询被驱动表；被驱动表有索引、驱动表数据集很小的时候合适。

2. **BKA 批量键访问**
   被驱动表**存在索引**；缓冲关联键、排序、有序批量访问索引，优化随机IO。

3. **BNL 块嵌套循环**
   被驱动表**没有索引**；依靠 join_buffer 缓存驱动表数据，内存匹配，减少全表扫描次数。

4. **Hash‑Join（MySQL8.0+）**
   8.0.18之后支持；驱动表构建内存hash表，扫描被驱动表做hash匹配；大数据量关联性能优于BNL。

#### 五、BNL 触发条件

1. 关联字段**缺少索引**，无法走 BKA；
2. 使用 inner‑join / left‑join；
3. join_buffer 有可用内存存放驱动行；

> 优化器优先顺序：`BKA > NL‑Join > Hash‑Join > BNL`

#### 六、关键参数

```sql
-- 查看连接缓冲区大小，单条连接每个join独立分配一份buffer
show variables like join_buffer_size;
```

- 默认一般 256k；
- **禁止设置超大**，多个并发连接会耗尽服务器内存；
- 缓冲区放不下全部驱动表数据，BNL 就会分多批次处理。

查看优化器开关

```sql
show variables like optimizer_switch;
```

`block_nested_loop=on` 默认开启

#### 七、BNL优缺点

##### 优点

1. 相比基础嵌套循环，大幅度减少被驱动表全表扫描次数；
2. 不需要依赖索引即可完成表连接。

##### 缺点

1. 依旧需要多次全表扫描被驱动大表，IO压力很高；
2. 消耗 join_buffer 内存；
3. 内存里面循环比对消耗CPU。

#### 八、开发优化方案，尽量规避BNL

只要出现 BNL，就代表被驱动表关联键缺少索引，属于慢SQL信号：

1. **给被驱动表的 join 字段建立索引，升级为 BKA**，从全表扫描改成索引检索；
2. 小表作为驱动表，减少加载进 join_buffer 的行数；
3. MySQL8.0 依靠 Hash‑Join 替代笨重的BNL；
4. 拆分SQL、先过滤驱动表，减小驱动表结果集。

#### 九、全套 Join 算法一句话总结

- 被驱动表有索引 → BKA（排序批量访问索引，优化随机IO）
- 被驱动表无索引、版本低于8.0 → BNL 块嵌套循环，缓冲区缓存驱动表减少扫描次数
- 8.0.18以上大数据无索引关联 → Hash‑Join
- 驱动表数据极少 → 普通 NL‑Join

#### 十、InnoDB五大查询优化器机制整体梳理

1. ICP：索引层提前过滤，**减少回表行数**
2. MRR：单表range查询，主键排序，随机IO转为顺序IO
3. 覆盖索引Using‑index：**直接消除回表**
4. BKA：join有索引，批量有序访问被驱动表索引
5. BNL：join无索引，内存缓冲区缓存驱动表降低全表扫描次数

如果你需要，我接下来详细讲解 MySQL8.0 Hash‑Join 哈希连接原理。

### MySQL Hash‑Join（哈希连接）

#### 一、基础介绍

Hash‑Join 在 **MySQL 8.0.18** 正式推出，专门用来优化**关联字段没有索引**的多表连接，用来替代老旧低效的 BNL 块嵌套循环连接。

> BNL：反复多次全表扫描被驱动表；
> Hash‑Join：内存构建哈希表，只需要扫描两张表各一次，大数据量性能碾压 BNL。

#### 二、Hash‑Join 执行流程

##### 准备两张表

- build 构建表（优先选用结果集更小的表）
- probe 探测表（大表）

```sql
select * from t1 inner join t2 on t1.key = t2.key;
```

1. 优化器选择行数更少的 t1 当作 build‑表；
2. 遍历 t1，读取关联键 key 和行数据，在内存 `hash join buffer` 建立哈希表，key 作为哈希键；
3. 逐行扫描大表 t2（探测表），拿 t2 的关联键计算哈希值；
4. 和内存哈希表匹配，key 相等就拼接两行作为结果；

##### 内存不够触发磁盘溢出

当 build 表数据太大，内存缓冲区放不下哈希表：

1. 将 build‑表、probe‑表按照哈希值分成多个分片写入临时文件；
2. 逐个加载分片到内存做哈希匹配，也就是分批磁盘哈希连接。

#### 三、触发条件

1. MySQL ≥ 8.0.18；
2. 等值连接 `on a.col = b.col`，**不支持非等值、大于小于、like**；
3. 被驱动连接字段缺少索引，无法走 BKA；
4. inner‑join、left‑join、semi‑join、anti‑join 均支持哈希连接；
5. 优化器成本判断，大数据集优先选用 Hash‑Join，小表会选择 NL‑Join。

> 优先级：BKA > NL‑Join > Hash‑Join > BNL
> 只要关联字段建有索引，永远优先走 BKA，不会启用 Hash‑Join。

#### 四、相关系统参数

```sql
# 哈希连接内存缓冲区大小
show variables like 'hash_join_buffer_size';
```

- 该缓冲区为每个连接独立分配；
- 数据超出内存缓冲区，自动落地磁盘临时文件执行分批哈希连接，性能下降；

优化器开关：

```sql
show variables like optimizer_switch;
hash_join=on
```

#### 五、四种 Join 算法横向对比

| 连接算法             | 适用场景                         | 核心原理                                              |
| -------------------- | -------------------------------- | ----------------------------------------------------- |
| NL‑Join 普通嵌套循环 | 驱动表数据集很小、被驱动表带索引 | 读取一行，索引查找一次被驱动表                        |
| BKA 批量键访问       | 被驱动表存在索引                 | 缓存一批关联键、排序，有序批量访问索引，优化随机IO    |
| BNL 块嵌套循环       | 无索引、8.0之前版本              | join_buffer存放驱动表，多次扫描被驱动表全表做内存比对 |
| Hash‑Join哈希连接    | 无索引、等值关联、MySQL8.0+      | 小表构建内存哈希表，大表一次扫描探测匹配              |

#### 六、Hash‑Join 的优缺点

##### 优点

1. 正常内存充足情况下，两张表只扫描一遍，时间复杂度 O(N+M)；BNL 需要多次扫描大表；
2. 超大结果集关联，性能远超 BNL；
3. 不需要依赖索引即可完成表连接。

##### 缺点

1. **只支持等值连接**，`> < >= <=` 无法使用；
2. build‑表过大超出内存缓冲区，需要使用磁盘临时文件，速度暴跌；
3. 哈希计算占用 CPU；小数据量时普通嵌套循环更快。

#### 七、开发优化建议

1. join 优先给被驱动表关联字段建立索引，走 BKA 是最优解；
2. 如果确实没法建立索引，尽量让**小表充当 build 构建表**，减少哈希表内存占用；
3. 等值关联优先 Hash‑Join，范围关联只能依赖 BKA / BNL；
4. 避免查询 `select *`，减少哈希缓冲区存放的数据体积，可以容纳更多记录。

#### 八、全套 InnoDB 查询优化器机制完整复盘

##### 单表查询优化

1. **ICP 索引条件下推**：存储引擎层利用索引字段提前过滤，减少回表次数
2. **MRR 多范围读取**：range 查询收集主键排序，随机IO变为顺序IO
3. **覆盖索引 Using index**：查询字段全部存在二级索引，彻底消除回表

##### 多表 Join 优化

1. **BKA 批量键访问**：被驱动表有索引；批量排序关联键，有序访问索引
2. **BNL 块嵌套循环**：无索引，缓冲区缓存驱动表，减少大表扫描次数（旧版）
3. **Hash‑Join哈希连接**：8.0+、等值无索引连接，小表构建哈希表匹配大表

如果你需要，我整理一份 MySQL 慢 SQL 排查完整流程。

### Nested‑Loop‑Join 普通嵌套循环连接

#### 一、概念简介

Nested‑Loop‑Join（NLJ、嵌套循环连接）是 MySQL 最基础、最原始的表连接算法。 核心思路：**循环读取驱动表每一行，拿着关联条件去被驱动表索引里面查找匹配的数据**。

> 适合前提：**被驱动表关联字段有索引**，驱动表结果集很小。

#### 二、执行流程

两张表：

- 驱动表 t1（外层循环）
- 被驱动表 t2（内层循环，关联字段建有索引）

```
SELECT * FROM t1 INNER JOIN t2 ON t1.t2_id = t2.id;
```

1. 取出 t1 的第一条记录，拿到关联值 `t2_id`
2. 根据该值去 t2 的索引快速检索匹配行
3. 拼接结果
4. 循环读取 t1 下一行，重复内层索引查找

通俗来讲：**外层遍历驱动表，内层索引查询被驱动表**。

#### 三、NLJ、BKA 的关系

很多人容易混淆二者：

1. **原生 NLJ**：读取一条驱动表数据，立刻发起一次索引查找； 驱动表主键无序，访问被驱动表索引属于离散随机IO。
2. **BKA 批量键访问，属于 NLJ 的优化版** 积攒一批驱动表关联键 → 排序 → 有序批量访问索引，**随机IO转为顺序IO**。

> NL‑Join 是基础算法；BKA 是嵌套循环的 IO 优化策略。

#### 四、触发条件

1. 被驱动表连接字段存在索引；
2. 驱动表数据集很小的时候优化器优先选择原生 NLJ； 驱动表行数大，优化器会启用 BKA。

#### 五、四种 Join‑算法完整对比

##### 1. Nested‑Loop‑Join（普通嵌套循环）

- 使用场景：驱动表数据量很小、被驱动表带索引
- 原理：一条一条循环，索引查询被驱动表

##### 2. BKA Batched‑Key‑Access 批量键访问

- 使用场景：驱动表结果量大、被驱动表带索引
- 原理：批量缓存关联键、排序、有序访问索引，优化随机IO

##### 3. BNL Block‑Nested‑Loop 块嵌套循环

- 使用场景：关联字段**没有索引**，8.0‑旧版本
- 原理：join_buffer 存放驱动表，多次扫描被驱动表全表内存匹配

##### 4. Hash‑Join 哈希连接（MySQL8.0.18+）

- 使用场景：无索引、等值连接、大批量数据
- 原理：小表构建哈希表，遍历大表做哈希探测匹配

#### 六、NLJ 优缺点

##### 优点

1. 算法简单，CPU开销低；
2. 驱动表行数极少时速度极快；
3. 支持等值连接、范围连接、模糊匹配等非等值关联。

##### 缺点

1. 驱动表行数很多，就会产生大量随机IO，性能暴跌；

   > 这时优化器自动升级为 BKA。

#### 七、开发最佳实践

1. 小表做驱动表，减少外层循环次数；
2. 被驱动表 join 字段必须建立索引，优先走 NLJ / BKA；
3. 禁止大表驱动、无索引关联（会触发BNL多次全表扫描）；
4. 驱动表提前 where 过滤，缩小循环数据集。

#### 八、全套优化机制总结（一次性梳理完毕）

##### 单表查询优化

1. ICP 索引条件下推：索引层提前过滤，减少回表次数
2. MRR 多范围读取：range 查询主键排序，随机IO→顺序IO
3. 覆盖索引：字段全部位于索引，直接消除回表

##### Join 连接算法

1. NL‑Join：基础嵌套循环，小数据集+被驱动表带索引
2. BKA：嵌套循环的IO优化，批量有序访问索引
3. BNL：无索引旧版块嵌套循环
4. Hash‑Join：8.0+无索引等值连接最优方案

需要我整理一份 MySQL 所有 join 算法面试速记清单吗。

### MySQL 慢SQL完整排查+优化全流程

下面分为：开启慢查询日志 → 抓取慢SQL → SQL基础信息分析 → explain执行计划剖析 → 定位问题类型 → 针对性优化 → 优化后验证，整套标准化流程，附带常见故障、索引优化、join优化、优化器机制。

#### 一、第一步：开启慢查询，捕获慢SQL

##### 1、查看慢日志配置

```sql
show variables like '%slow_query%';
show variables like 'long_query_time';
```

核心参数

1. `slow_query_log=ON`：开启慢查询日志
2. `slow_query_log_file`：慢日志文件路径
3. `long_query_time`：SQL执行超过该阈值就记录，一般线上设置 1s
4. `log_queries_not_using_indexes = on`，记录没有走索引的SQL

##### 2、线上抓取慢SQL工具

1. 慢日志文件文本分析
2. mysqldumpslow 工具分析日志

```bash
# 查看耗时最高的10条慢SQL
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
```

3. 企业工具：pt‑query‑digest、MySQL‑slow‑log‑parser
4. 临时抓现场：performance_schema、show processlist 抓实时阻塞SQL

> 优先拿到完整慢SQL、执行耗时、扫描行数、返回行数、发生频次

#### 二、第二步：基础信息评估SQL

拿到SQL先确认基础指标

1. 执行耗时、锁等待耗时
2. **扫描行数 rows** 和返回行数，扫描远大于返回 = 索引失效
3. SQL类型：单表查询、深分页、多表Join、update/delete、group by、order by
4. 业务场景：高频SQL、大批量数据、事务长时间持有锁

#### 三、第三步 explain 分析执行计划（最核心）

```sql
explain 你的慢SQL;
```

逐字段解读

##### 1、type（访问级别，从优到劣）

`system > const > eq_ref > ref > range > index > ALL`

- ALL：全表扫描，必须优化
- index：扫描全部二级索引树
- range：范围查询 in、between、> <，可以触发 MRR、ICP
- ref：普通索引等值匹配
- eq_ref：主键/唯一索引等值关联(join最优)

> 生产最低标准：日常SQL至少达到 ref/range

##### 2、key：实际命中的索引；null=没有用到索引

##### 3、rows：预估扫描行数，数值越大越危险

##### 4、Extra 重点标记，判断优化器行为

1. `Using index`：命中**覆盖索引**，无需回表（最优）
2. `Using index condition`：ICP索引条件下推，索引层过滤减少回表
3. `Using temporary`：创建临时表，常见group by、多字段排序，性能很差
4. `Using filesort`：文件排序，MySQL无法利用索引排序，额外内存/磁盘排序
5. `Using join buffer (Block Nested Loop)`：触发BNL，关联字段缺少索引
6. `Range checked for each record`：索引失效

#### 四、第四步 归类慢SQL问题根源

##### 类别1：索引问题（80%慢SQL元凶）

1. 没有建立合适索引
2. 违反最左前缀原则
3. 索引失效：索引字段做函数运算、隐式类型转换、like %前缀、or两侧无索引
4. 索引选择性太差（性别、状态只有少数枚举值，索引失效）
5. select * 无法触发覆盖索引，大量回表IO

##### 类别2、分页问题：深分页 limit offset,size

offset过大，MySQL扫描并丢弃前面大量数据

> 优化：书签分页(search‑after)、延迟关联、后台限制超大页码

##### 类别3、Join多表关联问题

四种Join触发问题排查顺序

1. Extra出现BNL → **被驱动表关联字段缺少索引**，优先建立索引升级成BKA
2. 驱动表是大表 → 调整小表作为驱动表
3. 多表join过多，超过3~5张表建议业务拆分SQL
4. 等值关联无索引，8.0依靠Hash‑Join

##### 类别4、排序、分组、去重

- Using filesort：排序字段没有索引；建立联合索引 where + order by
- Using temporary：group‑by字段不在索引；利用覆盖索引避免临时表

##### 类别5、锁与事务慢（不是索引问题）

1. 长事务迟迟不commit，持有行锁，造成阻塞等待
2. 大量更新、行锁升级表锁
3. MVCC读取大量undo‑log快照
   优化：缩短事务、事务内操作顺序、避免大事务

##### 类别6、优化器机制层面

ICP、MRR、BKA是否正常触发；
range批量查询开启MRR将随机IO转为顺序IO

#### 五、第五步 分场景标准化优化方案

##### 1、索引优化规范

联合索引设计顺序口诀：
`等值查询字段 → 排序字段 → group by字段 → 查询返回字段`

- 禁止select *；只查询需要字段，尽量命中覆盖索引
- 索引字段不要做运算、函数、隐式转换
- like 使用前缀匹配 `xxx%`，不要 `%xxx`
- or两边字段都需要建立索引

##### 2、深分页优化

1. 书签分页：where id > 上一页最后id limit pageSize
2. 后台管理系统：延迟关联，先覆盖索引查id再回表
3. 业务层面禁止跳转超大页码

##### 3、Join优化

1. 小表驱动大表
2. 被驱动表on关联字段添加索引，走BKA
3. 减少join表数量
4. 超大关联拆分为多条单表查询，代码层做关联

##### 4、order by、group by

1. 排序字段放进联合索引末尾，消除Using filesort
2. group‑by 使用覆盖索引，避免临时表
3. 超大分组增加where条件提前过滤数据量

##### 5、DML update / delete

1. where条件带上索引，避免全表扫描更新，防止锁整张表
2. 大批量更新分批循环提交，防止长事务

#### 六、第六步：优化之后验证结果

1. 再次 explain，确认type、key、Extra已经优化
   - 消除ALL全表扫描、Using filesort、Using temporary、BNL块嵌套循环
2. 真实执行SQL对比耗时
3. 压测观察QPS、IO、慢日志是否不再出现
4. 观察锁等待、事务耗时

#### 七、高频优化器机制快速复盘（排查时对照）

1. ICP索引条件下推：二级索引提前过滤，减少回表行数
2. MRR多范围读取：range查询主键排序，随机IO→顺序IO
3. 覆盖索引：杜绝回表
4. NL‑Join：小表循环+索引查找
5. BKA：join批量有序访问索引
6. BNL：无索引全表扫描，需要新建索引
7. Hash‑Join：8.0+无索引等值连接方案

#### 八、线上避坑清单

1. 索引不是越多越好；索引会拖慢insert/update/delete
2. 不要在字符串索引不加引号，触发隐式转换索引失效
3. 避免大事务、长连接休眠
4. 分页、in查询数据集过大
5. 区分CPU慢（排序、哈希、函数）和IO慢（回表、全表扫描）

#### 九、整体排查流程一句话总结

**慢日志捕获SQL → 查看扫描行数与耗时 → explain解析执行计划 → 判断索引/分页/join/排序/事务锁问题 → 根据场景调整索引、改写SQL、业务层优化 → 复测验证性能**

需要我给你一份 explain 执行计划超详细手册 + 常见索引失效清单吗？

### 执行计划超详细手册 + 常见索引失效清

#### 一、Explain 执行计划超详细完整手册

##### 使用方式

```
EXPLAIN SELECT ……;
EXPLAIN ANALYZE SELECT ……; -- MySQL8.0.18+，真实执行SQL，输出耗时
```

输出字段一共：`id、select_type、table、type、possible_keys、key、key_len、ref、rows、filtered、Extra`，下面逐个拆解。

##### 1. id —— 查询执行顺序标识

1. id 相同：从上往下顺序执行
2. id 越大越优先执行；id 值越大，越先执行
3. id 为 NULL：派生表、结果集，最后执行

>
> 多表join id一般全部一致；子查询会产生多个id。

##### 2. select_type 查询类型

| 参数         | 说明                           |
| ------------ | ------------------------------ |
| SIMPLE       | 简单查询，无子查询、union      |
| PRIMARY      | 最外层主查询                   |
| SUBQUERY     | where后的子查询                |
| DERIVED      | from里面的派生表(子查询临时表) |
| UNION        | union之后第二条往后SQL         |
| UNION RESULT | union合并结果                  |

##### 3. table

当前操作数据表名称；`<derivedN>` 代表临时派生表。

##### 4. type 访问类型（**优化最重要指标，优先级从优到最差**）

>
> 线上SQL底线：普通业务 SQL 至少达到 ref / range

```
system > const > eq_ref > ref > range > index > ALL
```

1. **system**：表只有一行数据，系统表，可遇不可求
2. **const**：主键 / 唯一索引等值匹配，表最多命中一行；常出现在主键where id=1
3. **eq_ref**：Join连接，被驱动表使用主键、唯一索引关联，每次只命中一行，Join最优级别
4. **ref**：普通二级索引等值查询，可以命中多条行；日常最常见
5. **range**：索引范围扫描：`in、between、>、<、>=、like 'xxx%'`；可触发 ICP、MRR
6. **index**：遍历完整二级索引树；比全表扫描ALL快（索引体积小于聚簇索引），常见覆盖索引全索引扫描
7. **ALL**：**全表扫描聚簇索引，最差，必须优化**

##### 5. possible_keys

理论上可以使用到的索引，只是候选索引，不一定真正使用。

##### 6. key

**最终实际命中使用的索引**

- null = 没有走索引，发生索引失效

##### 7. key_len

索引使用字节长度，可以判断复合索引用到了几列

>
> 计算规则：

- char(n) = n * 字符集字节(utf8=3，utf8mb4=4)
- varchar(n) = n*字节 +2（长度标识）
- int =4，bigint=8，null标记占用1字节

示例：联合索引(a,b,c)，key‑len只等于a字段长度 → 只用到最左第一列。

##### 8. ref

索引匹配的对象：常量、别的表字段、函数

- const：常量匹配；
- 字段名：关联表字段join匹配

##### 9. rows

优化器预估需要扫描的数据行数；数值越大开销越高。

##### 10. filtered

返回数据 / 扫描数据的百分比。数值越低代表扫描大量无用行，需要ICP索引下推或者优化索引。

##### 11. Extra 扩展信息（最核心，判断各类问题）

###### 优秀标识

1. `Using index`
命中**覆盖索引**，查询字段全部存在二级索引，**不需要回表**，单表查询最优。
2. `Using index condition`
触发 **ICP索引条件下推**；存储引擎层利用索引字段提前过滤，减少回表次数。

###### 需要优化、慢SQL元凶

3. `Using filesort`
**文件排序**：MySQL无法借助索引完成order‑by，需要额外内存/磁盘排序。

>
> 解决：建立联合索引 where条件 + order by
4. `Using temporary`
创建临时表，多见于 group by、distinct、多字段排序；IO开销很高

>
> 解决：使用覆盖索引，group‑by字段放在索引当中
5. `Using join buffer (Block Nested Loop)`
触发 **BNL块嵌套循环连接**
含义：被驱动表关联字段**没有索引**。
优化方案：给on关联字段建立索引，升级为BKA批量键访问。
6. `Range checked for each record`
索引失效，每一行动态判断索引是否可用，性能极差。
7. `Not exists`
优化器做了not‑exists优化
8. `Using MRR`
开启MRR多范围读取优化，range查询主键排序，随机IO转为顺序IO
9. `Using join buffer (Batched Key Access)`
命中BKA批量键访问优化，join批量有序访问索引

---

#### 二、最全索引失效清单（开发高频踩坑点）

##### 1. 索引字段上做运算、函数、类型运算

```
-- 失效
select * from user where date(create_time)='2026-08-13';
select * from user where id+100=500;
```

>
> 原理：索引存储原始字段值，运算之后无法匹配索引树
> 改写：把运算迁移到常量一侧

```
select * from user where create_time >= '2026-08-13' and create_time < '2026-08-14';
```

##### 2. 隐式类型转换（最高发线上坑）

user_phone 为字符串varchar索引字段

```
-- 失效，字符串和数字对比，MySQL强制把字符串转为数字
select * from user where user_phone=13800138000;

-- 正确，带上引号
select * from user where user_phone='13800138000';
```

##### 3. like 左通配符 %xxx

```
-- 索引失效
select * from user where name like '%张三';
select * from user where name like '%张三%';

-- 有效，前缀匹配
select * from user where name like '张三%';
```

>
> 需要前后模糊检索：使用 Elasticsearch

##### 4. or 连接条件，一侧字段没有索引

```
-- age有索引，status无索引 → 整条SQL索引失效全表扫描
select * from user where age=18 or status=1;
```

解决：status添加索引；或者拆分为两条SQL union‑all

##### 5. 违背联合索引最左前缀原则

联合索引 idx(a,b,c)

```
-- 失效，跳过最左a
where b=10
where b=10 and c=20
```

>
> 索引必须从最左侧开始依次使用，中间不能跳过字段；
> 范围字段之后索引失效：`where a=? and b>? and c=?` c字段不会用到索引

##### 6. MySQL优化器成本判断，主动放弃索引走全表扫描

1. 索引选择性极差，例如sex字段只有0、1两个枚举，索引区分度极低
2. 查询条件命中表超过30%~50%数据，优化器判定全表扫描比索引回表更快，放弃索引

##### 7. 优化器优先选择别的索引

一张表存在多个索引，优化器评估成本选择其它索引，你预想的索引没有启用。
可以使用 force index 强制索引测试

```
select * from user force index(idx_age) where age=20;
```

##### 8. is null / is not null 索引失效

- innodb索引可以存放null；但是大量null数据会降低选择性
- `is not null` 在数据量大的时候优化器可能放弃索引

##### 9. 字符串字符集不匹配引发隐式转换

关联两张表，关联字段一个utf8、一个utf8mb4，字符集不一致触发隐式转换，索引失效。

##### 10. order by、group by 字段不在索引末尾

联合索引(a,b)

```
-- 失效产生Using filesort
where a=1 order by c
```

##### 11. 分区表没有带上分区键条件，扫描全部分区

##### 12. 存储引擎不支持索引，例如memory哈希索引等值生效、范围失效

##### 13. 单条SQL使用过多索引，优化器选择合并索引开销更高，放弃索引

##### 14. 字段长度过大，text、blob不能建立普通索引

---

#### 三、配套优化口诀

1. 等值字段放联合索引最前面，范围字段后置
2. where、order by、group‑by、返回字段依次排布在复合索引
3. 禁止索引字段函数运算、隐式转换
4. 禁止 select *，尽量命中覆盖索引消除回表
5. range范围查询利用ICP减少回表，MRR优化随机IO
6. join被驱动表关联字段必建索引，优先BKA，杜绝BNL块嵌套循环

#### 四、快速排查顺序

1. key==null → 索引失效，对照失效清单排查原因
2. type达到ALL全表扫描 → 新建合适索引
3. Extra出现Using filesort / Using temporary / BNL → 针对性改写索引
4. rows预估扫描行数巨大 → 增加where条件裁剪数据集

如果你需要，我可以整理一份 **MySQL索引设计规范 + 慢SQL优化面试题合集**。





## MySQL InnoDB 索引设计完整开发规范

### 一、基础硬性规范

1. 主键优先选用无符号 bigint 自增主键；业务主键、字符串UUID不推荐做主键，会造成索引页分裂、碎片。
2. 禁止在 text、blob、超大长度字段创建普通二级索引；如需模糊检索使用搜索引擎。
3. 一张表索引数量控制在 3‑5 个以内；索引加速查询，但会拖累 insert、update、delete，每次写入需要维护全部索引。
4. 索引字段 NOT NULL，尽量避免大量 NULL 值；NULL会降低索引选择性、增加判断逻辑。
5. 字符串索引必须带上引号，规避隐式类型转换造成索引失效。

### 二、联合索引黄金排序法则（重中之重）

> **等值查询字段 → 范围查询字段 → order by → group by → select返回字段**
#### 原理

1. 等值条件放在索引最左侧，满足最左前缀匹配；
2. 一旦碰到范围条件（> < between in），索引后面的列无法再使用；
3. 排序字段紧跟等值字段，避免 Using filesort 文件排序；
4. 查询返回字段放在索引末尾，构造**覆盖索引，消灭回表IO**。

示例SQL
```sql
select name,email from t where status=1 and create_time>'2026‑01‑01' order by age;
```
错误索引：`(create_time,status,age)`
正确联合索引：`(status,create_time,age,name,email)`

### 三、最左前缀原则使用规范

1. 复合索引必须从最左首列开始匹配，不能跳过中间字段；
索引(a,b,c)，where b=xx 无法触发索引。
2. 范围字段之后索引失效：`where a=1 and b>10 and c=20`，c不会走索引。

### 四、覆盖索引规范

1. 业务查询杜绝 `select *`；只查询需要的字段。
2. 分页、count、distinct、group‑by 优先设计覆盖索引，消除回表。
3. 二级索引默认携带主键id，索引(a,b)天然可以覆盖 a、b、id。

### 五、索引选择性规范

1. 区分度极低字段不要建普通索引：例如性别、状态仅有 0/1，索引扫描之后绝大多数数据需要回表，优化器会直接放弃索引走全表扫描。
2. 区分度计算公式：`select count(distinct col)/count(*)`，越靠近 1 索引质量越高。

### 六、like、or、in 索引规范

1. 模糊查询只允许前缀匹配 `like 'xxx%'`；`%xxx`、`%xxx%` 索引失效，使用 ES。
2. or 两侧字段均需要建立索引，否则整条SQL索引失效；也可以拆分为多条 SQL 使用 union‑all。
3. in 集合数值过多（上千），优化器容易放弃索引；大数据集合拆分多次查询。

### 七、Join 关联索引规范

1. 永远**小表驱动大表**。
2. 被驱动表 on 关联字段必须建立索引，走 BKA 批量键访问；禁止出现 BNL 块嵌套循环。
3. 多表Join不建议超过 3‑5 张表，复杂关联拆分为单表查询，业务代码进行数据组装。

### 八、排序分组索引规范

1. order‑by、group‑by、distinct 的字段紧跟等值索引字段，防止 Using filesort、Using temporary。
2. 排序方向需要统一，索引升序，不要混合 asc、desc 排序造成索引失效。

### 九、线上避坑清单

1. 索引字段禁止进行函数、算术运算；`date(create_time)`、`id+1` 索引失效，条件改写至常量端。
2. 关联两张表字段字符集统一，utf8 和 utf8mb4 混用触发隐式转换。
3. 长事务不要频繁更新索引字段，频繁更新会造成索引页分裂、大量碎片。
4. 定时清理碎片，optimize table；大批量数据删除之后索引碎片严重，性能下降。

---

## 慢SQL优化高频面试题

### 基础篇

#### Q1：explain 里面 type 访问类型优先级，最差和最优分别是什么？

> 顺序：system > const > eq_ref > ref > range > index > ALL
- const：主键唯一索引等值匹配
- eq_ref：join被驱动表主键索引，join最优级别
- ALL：聚簇索引全表扫描，最差，必须优化

#### Q2：Extra 中 Using index、Using index‑condition、Using filesort、Using temporary 含义？

1. Using‑index：命中覆盖索引，无需回表，最优；
2. Using index condition：ICP索引条件下推，引擎层过滤索引字段，减少回表次数，仍然需要回表；
3. Using filesort：无法利用索引排序，MySQL额外进行内存/磁盘排序，性能较差；
4. Using temporary：创建临时表，多出现在group by、distinct，需要优化索引。

#### Q3：ICP、MRR、BKA分别是什么？

1. ICP索引条件下推：二级索引提前过滤无效数据，减少回表行数；
2. MRR多范围读取：range查询收集主键id排序，随机磁盘IO转换成顺序IO；
3. BKA批量键访问：Join场景积攒一批关联键排序，有序访问被驱动表索引优化随机IO。

#### Q4：四种Join连接算法以及适用场景

1. NL‑Join普通嵌套循环：驱动表数据集很小，被驱动表带索引；
2. BKA批量键访问：驱动表数据量大、被驱动表存在索引；
3. BNL块嵌套循环：关联字段无索引，多次扫描被驱动表；
4. Hash‑Join哈希连接：MySQL8.0.18+，无索引等值连接，小表构建哈希表匹配大表。

### 索引失效高频面试题

#### Q5：列举常见索引失效场景

1. 索引字段进行函数、算术运算；
2. 字符串索引查询不带引号，触发隐式类型转换；
3. like 左通配符 %xxx；
4. or 一侧字段没有索引；
5. 复合索引违背最左前缀原则，跳过前置等值字段；
6. 查询条件命中表超过30%~50%数据量，优化器放弃索引；
7. 索引字段区分度过低；
8. 关联字段两边字符集不一致引发隐式转换；
9. range范围条件后面的复合索引字段失效。

#### Q6：联合索引(a,b,c)，where a=? and c=?，会用到几个索引字段？

只会命中 a；跳过了b，违反最左前缀，c无法使用索引过滤。

#### Q7：为什么不推荐UUID作为主键？

1. UUID无序，插入的时候随机写入索引页，频繁造成索引页分裂、磁盘碎片；
2. UUID占用存储空间更大，二级索引都会携带主键，索引体积膨胀，IO开销上涨。

### 分页优化面试题

#### Q8：limit offset,size 深分页为什么慢？

MySQL需要扫描 offset+pageSize 条记录，丢弃前面offset条，offset越大扫描行数越多，IO开销越高。

#### Q9：深分页有哪些优化方案？

1. 书签分页 search‑after：记录上一页最后一条主键id，where id > lastId limit size，最推荐；
2. 延迟关联：覆盖索引查询分页id，再inner‑join回表查询完整字段，适合后台页码跳转；
3. 业务层面限制超大页码，只提供上一页下一页滚动加载；
4. 大表使用 Elasticsearch search‑after 分页。

### SQL优化面试题

#### Q10：order by 慢、出现 Using filesort如何优化？

建立复合索引，顺序：等值查询字段 + 排序字段；让索引有序性直接完成排序，避免额外文件排序。

#### Q11：group by 优化方式？

group‑by字段放入联合索引，使用覆盖索引，避免生成临时表 Using temporary；优先where过滤减少数据集。

#### Q12：大表update慢、锁表怎么办？

1. update where条件带上索引，避免全表扫描更新；
2. 大批量更新拆分分批提交，缩短事务生命周期，减少行锁持有时长；
3. 更新尽量避开索引字段，防止索引维护开销。

#### Q13：为什么尽量使用覆盖索引？

InnoDB二级索引叶子节点存放索引列+主键；覆盖索引可以直接从二级索引拿到所有查询字段，省去回表访问聚簇索引的磁盘IO开销。

#### Q14：ICP 和覆盖索引的区别

1. ICP依旧需要回表，只是在索引层提前筛除无效条目，减少回表次数；
2. 覆盖索引直接获取所有查询字段，彻底不需要回表；
命中覆盖索引的时候ICP不会触发。

### 线上故障、事务锁面试题

#### Q15：SQL有索引但是执行很慢，除索引失效之外还有什么原因？

1. 长事务持有行锁，当前SQL陷入锁等待；
2. 索引碎片过多；
3. 返回数据集太大，大量回表IO；
4. order‑by、group‑by产生文件排序、临时表；
5. 并发较高，缓冲池命中率低，频繁读取磁盘。

#### Q16：长事务带来哪些隐患？

1. 长时间持有行锁，引发阻塞、死锁；
2. MVCC需要保存大量undo log快照，占用存储空间；
3. 阻碍purge线程回收undo日志，表空间不断膨胀；
4. 数据库事务快照过多，查询性能下降。

#### Q17：慢SQL完整排查流程

1. 通过慢查询日志、performance_schema捕获耗时较长SQL；
2. 查看扫描行数、返回行数、执行耗时、锁等待时长；
3. explain分析执行计划，检查type、key、Extra；
4. 判断问题根源：索引失效、深分页、join无索引、排序临时表、长事务锁等待；
5. 优化索引、改写SQL、业务层面改造；
6. 复测耗时、监控慢日志。

### 开放性面试题

#### Q18：一张表千万级数据如何优化？

1. 合理设计联合索引、覆盖索引，规避索引失效；
2. 优化limit深分页，书签分页；
3. 分库分表；冷热数据分离；
4. 热点查询增加Redis缓存；
5. 复杂搜索交由Elasticsearch；
6. 大批量DML分批处理，杜绝长事务。

#### Q19：索引是不是越多越好？

不是。索引可以加速查询，但是每次 insert、update、delete 都需要维护整张表全部索引；索引过多会拖慢写入性能。常规单表索引控制3‑5个。

需要我整理一份 MySQL 事务、MVCC、锁机制面试题库吗。

## MYSQL问题

### Mysql深分页问题

#### 一、什么是深分页问题？

```sql
select * from user limit 100000, 20;
```

这种就是**深分页**。

##### 发生了什么？

MySQL 会：

1. 扫描 **前 100020 条数据**
2. **扔掉前 100000 条**
3. 返回最后 20 条

**越往后翻页，效率越爆炸慢。**

---

#### 二、为什么慢？（核心原理）

- `limit M,N` 必须**先读取 M+N 条**，再丢弃 M 条
- M 越大，IO、CPU、内存开销越大
- **回表**会加剧灾难：
  二级索引查到大量行，再一条条回主键索引取数据 → 极慢

一句话：
**limit 越深，扫描的数据越多，性能越差。**

---

#### 三、最优解决方案（4 种，按推荐顺序）

##### 1. **主键连续 + 有序：推荐方案（最快）**

**记住：用 id 定位，而不是 offset**

```sql
select * from user 
where id > 100000
limit 20;
```

✅ 只扫描 20 条  
✅ 无回表、无浪费  
✅ **O(1) 速度**

##### 2. **主键不连续、但有序：延迟关联（经典）**

```sql
select u.* from user u
join (select id from user order by id limit 100000,20) t
on u.id = t.id;
```

原理：

- 子查询**只查 id（索引覆盖）**，极快
- 外层只回表 20 条
  ✅ 通用、稳定、面试最爱问

##### 3. **二级索引深分页：索引覆盖 + 延迟关联**

比如按 `age` 分页：

```sql
select u.* from user u
join (select id from user order by age limit 100000,20) t
on u.id = t.id;
```

✅ 避免大量回表

##### 4. **业务层禁止深翻页（产品方案）**

- 不允许跳到第 1000 页
- 只提供**上一页/下一页**
- APP 主流做法：**滚动加载 + 游标分页**

**分页游标**实现方式就是：当我们查询第二页的时候，把第一页的查询结果放到第二页的查询条件中。 例如：首先查询第一页

```sql
select * from user 
where create_time>'2022-07-03' 
limit 10;
```

然后查询第二页，把第一页的查询结果放到第二页查询条件中：

```sql
select * from user 
where create_time>'2022-07-03' and id>10 
limit 10;
```

这样相当于每次都是查询第一页，也就不存在深分页的问题了

---

#### 四、面试标准答案（背诵版）

##### 什么是深分页？

`limit M,N` 中 **M 非常大**，MySQL 必须读取 M+N 条数据并丢弃前 M 条，导致性能极差。

##### 为什么慢？

1. 必须扫描大量无用数据
2. 大量**随机IO回表**
3. 索引无法直接跳到目标位置

##### 最优解决方案？

1. **id 连续：where id > #{lastId} limit 20**
2. **通用：延迟关联（先查id，再关联）**
3. **业务：禁止跳页，使用滚动加载**

---

#### 五、一句话终极口诀

**深分页慢，是因为 offset 太大；
解决核心：不用 offset，用 id 定位。**

### MySQL 深分页全套解决方案、原理、优缺点、实战 SQL、踩坑点

#### 一、什么是深分页问题

```sql
SELECT * FROM `order` LIMIT 100000, 10;
```
`LIMIT offset,size` 底层逻辑：
MySQL 需要**先扫描并丢弃前 offset 条数据**，再读取后面10条；
offset 越大，扫描行数越多、IO越高、速度越慢，这就是深分页。

> 例：limit 100000,10 → 引擎扫描100010行，抛弃前面10w行，只返回10条。
> 扫描行数 = offset + pageSize

#### 二、全部优化方案（由优先顺序排序）

##### 方案1：主键书签分页（最推荐，性能最优）

###### 原理

不再偏移跳过数据，记录上一页最后一条主键ID，直接从该位置往后查找，**不需要扫描前面的数据**

```sql
-- 第一页
select id,name,create_time from t order by id limit 10;

-- 第二页，上一页最后一条id=10
select id,name,create_time from t where id > 10 order by id limit 10;
```

###### 通用模板

```sql
select 字段列表 
from table_name 
where id > last_page_max_id 
order by id asc 
limit page_size;
```

###### 支持倒序分页

```sql
where id < last_page_min_id order by id desc limit 10
```

###### 优点

1. 每次只扫描 pageSize 行，offset再大速度稳定
2. 性能极高，百万级分页毫无压力

###### 缺点 & 解决办法

1. **不能跳页**：无法直接跳到第1000页，只能上一页、下一页
> 产品层面限制，只提供「上一页 / 下一页」，禁止页码跳转，移动端、信息流最常用
2. 中间有数据被删除，会跳过条目
> 业务可接受；或者使用自增唯一有序字段，尽量少物理删除、使用逻辑删除

###### 如果排序字段不是主键

> 例如需要按 create_time 分页，create_time 存在重复值
> 联合索引书签分页，防止重复、漏数据：
```sql
-- 索引 (create_time,id)
select id,create_time from t 
where create_time < '2026‑08‑13 12:00:00' 
or (create_time = '2026‑08‑13 12:00:00' and id < 100000)
order by create_time desc,id desc limit 10;
```

---

##### 方案2：延迟关联（适合前端必须输入页码跳转）

原理：先通过覆盖索引查出分页主键id，再关联回表查询完整字段；
减少回表开销，避免大量无用数据回表。

###### 慢的原版写法

```sql
select * from t order by id limit 100000,10;
```

###### 优化后延迟关联

```sql
SELECT t.* 
FROM t
INNER JOIN (
    SELECT id FROM t ORDER BY id LIMIT 100000,10
) AS temp ON t.id = temp.id;
```
> 子查询走主键覆盖索引，不需要读取行数据，速度很快；之后只对10条id回表查询完整字段

###### 优点

支持页码跳转，可直接跳转到第1000页
###### 缺点

offset 超大时依然需要扫描10w条索引，性能还是会下降，只适合中等偏移量

---

##### 方案3：主键IN分页（延迟关联变体）

```sql
select * from t where id in (
    select id from t order by id limit 100000,10
);
```

---

##### 方案4、预估定位法（大数据量表）

适用于自增id连续、很少删除数据
```sql
-- 第10000页，每页10条，预估起始id
select * from t where id > 10000*10 order by id limit 10;
```
id存在断层时，返回条目不足，业务做兼容向后多取几条。

---

##### 方案5、业务层面限制，从根源杜绝深分页

1. 禁止用户直接跳转超大页码；只允许上一页、下一页、滚动加载
2. 超过指定页码（例如100页）不再提供分页查询
3. 大数据列表改用搜索条件过滤，缩小查询范围

> 阿里、抖音、淘宝列表都不支持跳转到几千页，就是因为深分页性能灾难

---

##### 方案6、覆盖索引优化普通limit分页

如果必须使用 limit offset，建立排序字段的覆盖索引，避免回表
```sql
-- 查询字段 a,b；排序字段 create_time
create index idx_create_ab on t(create_time,a,b);
```
`Extra` 出现 Using‑index，索引直接拿到全部字段，不需要回聚簇索引。

---

##### 方案7、搜索引擎

千万级以上大表、复杂排序、筛选条件多
使用 Elasticsearch 分页；ES同样不适合深分页，使用 search‑after（书签分页）

---

#### 三、不同场景选型对照表

| 业务场景                       | 最优方案                          |
| ------------------------------ | --------------------------------- |
| 信息流、滚动加载、只需要上下页 | 主键书签分页(search‑after)        |
| 后台管理系统、需要页码跳转     | 延迟关联                          |
| 百万级以上大表                 | 书签分页 / ES search‑after        |
| 排序字段重复（创建时间）       | 联合索引 (sort_field,id) 书签分页 |

---

#### 四、高频踩坑清单

1. **order by 字段没有索引**
limit 分页带排序，排序字段不加索引会触发 Using filesort，深度分页直接卡死
> 建立联合索引：(sort_column,id,需要查询字段)

2. 使用 `select *`
字段过多造成回表开销巨大；只查询业务需要字段，启用覆盖索引

3. 有 where 查询条件，索引顺序错误
where条件字段放在索引最左侧，排序字段在后
```sql
-- where status=1 order by create_time
index (status,create_time,id)
```

4. 逻辑删除字段没加入索引
`where is_deleted=0`，索引必须带上is_deleted，否则索引失效全表扫描

5. offset 很大还坚持使用 LIMIT offset,size
offset>5w之后，普通limit性能断崖式下跌，必须改用书签分页

---

#### 五、后台管理系统折中实战示例

需求：订单列表，按创建时间倒序分页，支持页码跳转
1. 页码小于100：延迟关联分页
2. 页码大于等于100：强制使用时间范围筛选，禁止超大偏移

---

#### 六、一句话总结

- 能书签分页优先书签分页，性能最稳；
- 必须跳页就延迟关联；
- 业务上限制超大页码，从源头规避深分页。

如果你需要，我可以整理一份 MySQL索引优化、慢SQL、分页、锁问题全套实战笔记。

## SQL 锁问题

### 这条SQL会对哪些数据加锁？

**知识点总结：**

1. MySQL锁是加在索引记录上面的。

2. 如果是非唯一性索引，不论表中是否存在该记录，除了会对该记录所在范围加锁，还会向右遍历到不满足条件的范围进行加锁。

3. 如果是唯一索引，如果表中存在该记录，只对该行记录加锁。如果表中不存在该记录，除了会对该记录所在范围加锁，还会向右遍历到不满足条件的范围进行加锁。

4. InnoDB 临键锁（Next-Key Lock）的区间，永远是：左开右闭 (prev, curr]。

5. 临键锁：锁住的是「当前值」，不包含「前一个值」。

   格式永远：(前值，当前值]

```sql
update user set name='一灯' where age=5;
```

表结构是这样的：

```sql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(255) DEFAULT NULL COMMENT '姓名',
  `age` int DEFAULT NULL COMMENT '年龄',
  PRIMARY KEY (`id`),
  KEY `idx_age` (`age`)
) ENGINE=InnoDB COMMENT='用户表';
```

假如表中只有这样两条数据的话：

| id   | name | age  |
| ---- | ---- | ---- |
| 1    | 张三 | 1    |
| 10   | 李四 | 10   |

针对age索引，很产生这样三个索引范围：

> (-∞,1]，(1,10]，(10,+∞)

刚才的这条SQL：

```sql
update user set name='一灯' where age=5;
```

由于表中不存在age=5的记录，并且age=5刚好落在 **(1,10]** 的区间范围内，所以会对 **(1,10]** 的范围加锁。

##### 如果已经存在age=5的数据，刚才的那条update语句会对哪些数据加锁？

**我：** 假如表中数据是这样的。

| id   | name     | age  |
| ---- | -------- | ---- |
| 1    | 张三     | 1    |
| 5    | 一灯架构 | 5    |
| 10   | 李四     | 10   |

针对age索引，很产生这样四个索引范围：

> (-∞,1]，(1,5]，(5,10]，(10,+∞)

刚才的这条SQL：

```sql
update user set name='一灯' where age=5;
```

age=5的数据落在 **(1,5]** 的区间范围内，所以会对 **(1,5]** 的范围加锁。

你以为这就完了吗？MySQL锁为了保证数据的安全性，还会向右遍历到不满足条件为止，还会再加一个间隙锁，也就是 **(5,10]** 的范围。

所以，这条SQL的加锁返回是  **(1,5]** 和 **(5,10]** 。

##### 如果我把SQL中where条件换成主键ID，加锁范围是什么样的？

```sql
update user set name='一灯' where id=5;
```

**我：** 由于锁是加在索引上面的。

如果不存在id=5的数据，加锁范围跟上条SQL是一样的， **(1,10]** 。

如果存在id=5的数据，MySQL的 **Next-Key Locks** 会退化成 **Record Locks** ，也就是只在id=5的这一行记录上加锁。

### MySQL怎么加锁的？了解

当查询的记录是存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「记录锁」。

当查询的记录是不存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「间隙锁」。

当查询的记录存在时，用非唯一索引进行等值查询除了会加 next-key lock 外，还额外加间隙锁。

当查询的记录不存在时，用非唯一索引进行等值查询只会加 next-key lock，然后会退化为间隙锁。

非唯一索引范围查，next-key lock 不会退化为间隙锁和记录锁。

### **MySQL 单表不要超过 2000 万行，靠谱吗？**

**不靠谱，这是经典误区！不是绝对限制！**
**MySQL 单表上限不是 2000W，而是由**数据行大小 + 索引结构 + B+树高度 **决定的。**

---

#### 底层原理（面试必讲，非常加分）

InnoDB 是 **B+树 索引结构**，**B+树高度决定了单表最大容量**。

#### 1. 计算过程（面试官最爱）

默认：

- 一个**数据页 16KB**
- 主键 BIGINT 8B + 指针 6B ≈ **14B/条索引**

一页能存：
16KB / 14B ≈ **1168 条索引**

##### B+树高度：

- **高度 2**：1168 条数据
- **高度 3**：1168 × 1168 ≈ **136 万**
- **高度 4**：1168 × 1168 × 1168 ≈ **15.9 亿**

#### 重点：

**B+树高度 3 就能存 1300W+，高度 4 能存 15 亿！**
所以 **2000W 根本不是 MySQL 的极限**。

---

#### 那为什么大家都说 2000W？

因为：

1. **行业经验值**：单表在 **2000W 内性能最稳**，简单、运维成本低。
2. 超过后：
   - 数据页缓存命中率下降
   - 磁盘 IO 变多
   - 索引变大、SQL 变慢
   - 加索引、DDL 极慢
3. **不是存不下，是性能开始明显下降**。

---

#### 最终面试满分回答（背这个）

**MySQL 单表不超过 2000 万行并不是官方限制，而是业界经验值。
InnoDB 使用 B+ 树存储，B+ 树高度为 3 时可存储约 1300 万行，高度 4 可支持上亿行。
2000 万是性能安全线：超过后索引变大、缓存命中率下降、IO 增加、DDL 困难，性能会明显下滑。
所以实际开发中，单表尽量控制在 2000W 以内，超过就考虑分表。**

---

#### 超精简记忆版

- **2000W 不是上限，是性能安全线**
- **B+树高度决定容量，不是行数**
- **B+树3层≈1300W，4层≈15亿**
- 超过 2000W **不是不能用，是性能变差**



# MySQL SQL 面试题

下面按**基础语法、查询优化、索引、事务、锁机制、引擎、高级SQL、运维、实战场景、原理底层**十大模块整理，覆盖初级‑中级‑高级、社招/校招全部面试考点，附带核心考题。

## 一、SQL 基础语法（入门必问，校招必考）

### 1.DDL 数据定义语言

1. create/drop/alter/truncate 区别
2. 修改表结构、添加删除字段、修改字段类型
3. 主键、唯一索引、外键、非空、默认值约束
4. truncate、delete、drop 三者区别、回滚、速度、自增主键重置

### 2.DML 数据操作语言

1. insert、update、delete 语法，批量插入
2. replace into 和 insert‑on‑duplicate‑key‑update
3. 多表更新、关联删除

### 3.DQL 查询语句（最高频）

1. select 完整执行顺序：`from→join→on→where→group by→聚合函数→having→order by→limit`
2. where 和 having 区别
3. 子查询：相关子查询、非相关子查询、exists、in
4. 联表查询：inner/left/right/full join，on 和 where 在左连接区别
5. 聚合函数：sum、count、avg、max/min；`count(*)` / count(1) / count(字段)区别
6. group by、分组报错、only_full_group_by 参数
7. distinct 去重，多字段去重
8. limit 分页、大分页问题
9. union 和 union‑all 区别、去重、排序

### 4.DCL 权限控制

1. grant、revoke 用户授权、创建账号、远程访问
2. 视图、存储过程、函数、触发器、事件使用场景

## 二、MySQL 索引（面试重中之重，必问）

### 基础概念

1. 索引是什么、优缺点
2. B+树为什么适合MySQL索引，对比B树、哈希索引、二叉树
3. 聚簇索引、非聚簇索引区别（InnoDB）
4. MyISAM 索引结构
5. 覆盖索引、最左前缀原则、联合索引创建顺序
6. 普通索引、唯一索引、主键索引、全文索引、空间索引

### 索引失效全套考题

1. 索引失效所有场景
   - like %xxx 左模糊
   - 索引字段做运算、函数调用、隐式类型转换
   - or 连接无索引字段
   - not in、!=、is not null
   - 优化器选择全表扫描
2. explain 详解：id、select_type、type、key、rows、Extra
3. type 性能顺序：system>const>eq_ref>ref>range>index>all
4. 回表查询是什么
5. 索引下推ICP、MRR、BKA
6. 索引选择、联合索引怎么建
7. 百万级表建索引注意事项、索引过多坏处

## 三、事务 Transaction（高频核心）

1. 事务四大特性 ACID，分别底层实现原理
2. 四种事务隔离级别，从低到高
   - 读未提交
   - 读已提交(RC‑MySQL默认)
   - 可重复读(RR)
   - 串行化
3. 隔离级别对应问题：脏读、不可重复读、幻读
4. InnoDB RR级别怎么解决幻读：MVCC+间隙锁
5. MVCC原理：undo log、read view、隐藏字段DB_TRX_ID、roll_pointer
6. 当前读、快照读区别
7. 事务开启、提交、回滚；autocommit
8. 长事务危害、如何排查长事务

## 四、MySQL 锁机制（中高级必问）

1. 共享锁S、排他锁X，手动加锁语法
2. 行锁、表锁、意向锁（意向共享IS、意向排他IX）
3. InnoDB 什么时候加表锁、什么时候行锁
4. 记录锁、间隙锁、临键锁、插入意向锁
5. 死锁：产生条件、排查、解决方案、避免死锁
6. MyISAM表锁问题，并发差原因

## 五、存储引擎

1. InnoDB 和 MyISAM 完整对比（事务、锁、索引、崩溃恢复、外键、缓存）
2. Memory、Archive、CSV引擎适用场景
3. InnoDB 内存结构：Buffer Pool、Undo Log Buffer、Redo Log Buffer
4. redo‑log 重做日志、undo‑log回滚日志作用、刷盘机制
5. binlog二进制日志，三种格式 statement / row / mixed
6. redo log 和 binlog 区别、二阶段提交2PC



## 六、高级SQL & 经典业务题（手写SQL）

1. 行列转换
2. 排名问题：row_number() / rank() / dense_rank() 窗口函数
3. 递归查询、树形结构
4. 连续签到、连续日期
5. 慢分页优化：limit offset 大偏移量优化
6. 重复数据查找、删除重复数据
7. exists 和 in 性能对比
8. case when 条件判断
9. 时间函数、日期处理
10. 悲观锁、乐观锁实现（version版本号）

## 七、SQL优化（大厂高频）

1. 慢查询排查：slow‑query‑log、show‑processlist、performance_schema
2. SQL优化通用思路
3. 避免索引失效
4. limit、join、子查询优化
5. 大表优化：分库分表、水平/垂直拆分
6. 大表加索引注意事项、online‑ddl
7. 字段设计优化：char/varchar区别、禁用null、合理主键、避免大字段
8. 连接池、预编译、避免select *

## 八、MySQL 架构、底层原理

1. MySQL整体架构：连接层、服务层、引擎层、存储层
2. Buffer‑Pool缓存淘汰机制LRU
3. 页Page、区Extent、段Segment
4. 自适应哈希索引
5. 刷新脏页机制、checkpoint
6. 一条SQL完整执行全过程

## 九、主从复制、高可用、运维

1. binlog 主从复制原理三线程：master dump、slave IO、slave SQL
2. 主从延迟原因、解决方案
3. 半同步复制、GTID
4. MGR、InnoDB‑Cluster、Keepalived+双主、Redis对比
5. 备份：mysqldump、xtrabackup物理备份、增量备份
6. 数据库崩溃恢复流程
7. 定时清理日志、磁盘爆满处理

## 十、经典场景面试题（业务实战）

1. 千万级大表优化方案
2. 超短链接、订单表海量数据
3. 并发扣减库存、超卖问题
4. 订单超时自动关闭
5. 数据库死锁排查流程
6. 线上慢SQL排查完整步骤
7. 数据库CPU高、IO高、连接数爆满排查思路

## 十一、容易被忽略冷门考点

1. sql_mode参数
2. 缓冲池、redo log 刷盘策略 innodb_flush_log_at_trx_commit
3. 事务日志刷盘三种配置
4. 字符集、排序规则、utf8与utf8mb4区别
5. 表分区：range、list、hash、key分区、优缺点
6. 读写分离

需要我可以给你输出**每一类对应的面试题答案精简版**，或者整理成可直接背诵的文档。

# MySQL‑SQL语法专项面试题

## 一、基础DQL 查询语句（最高频）

### 1.基础执行顺序

1. 一条完整select 的执行顺序？

> FROM → JOIN → ON → WHERE → GROUP BY → 聚合函数 → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT

2. `where` 和 `having` 的区别

- where：**分组前过滤**，不能使用聚合函数
- having：**分组之后过滤**，可以使用聚合函数

### 2.联表查询

1. inner join、left join、right join 区别

- inner join：只返回两边匹配成功的数据
- left join：以左表为主，左表全部数据，右表匹配不到返回null
- right join：以右表为主

2. left‑join 中 on 和 where 的区别

- on：连接条件，先关联表；左表数据不会丢失
- where：关联完成之后再做筛选，左连接写where有可能把null数据过滤掉，变成内连接

### 3.子查询、in / exists

1. in 和 exists 区别

- in：先查子查询全部结果集，外层循环匹配；子查询量大性能差
- exists：外层驱动内层，只要子查询存在一条命中就返回true；适合子表数据量大

2. 相关子查询、非相关子查询？

- 非相关：子查询独立运行一次
- 相关子查询：子查询依赖外层查询字段，外层每一行都会执行一次子查询

### 4.聚合函数

1. `count(*)`、count(1)、count(列名)区别

- count(*)：统计所有行，包含null
- count(1)：常数计数，统计行数
- count(字段)：忽略该字段为null 的行

> InnoDB下性能：count(*) ≈ count(1) > count(普通字段)

2. sum、avg遇到null怎么处理？

> 聚合函数会自动忽略null值

### 5.group by

1. only_full_group_by 报错原因

> select 后的字段必须出现在group by之后或者被聚合函数包裹

### 6.去重 distinct

1. distinct 放在多字段前面含义

> 多个字段组合整体去重

### 7.union、union all

1. 二者区别

- union：合并结果集、自动去重、支持排序
- union‑all：直接拼接，不去重，速度更快

### 8.limit 分页语法&坑

1. limit offset,size；大偏移量为什么慢

> offset偏移很大时需要扫描前面大量废弃数据；优化：主键书签分页

## 二、DML 数据操作语法面试题

1. `delete`、`truncate`、`drop`区别

- delete：DML，可以被事务回滚，不会重置自增主键，逐行删除、速度慢
- truncate：DDL，不可回滚，清空整张表、重置自增id，速度很快
- drop：删除整张表、表结构+数据全部销毁

2. replace into 语法原理

> 主键/唯一键冲突先删除旧记录，再插入新数据

3. `insert ... on duplicate key update`

> 唯一索引冲突时执行更新操作，推荐用于业务幂等，比replace性能更好

4. 联表update、联表delete语法

```sql
-- 多表更新
update a join b on a.b_id=b.id set a.name=b.name;
```

## 三、DDL 表结构语法

1. alter 修改表常用操作

```sql
-- 添加字段
alter table t add column age int;
-- 删除字段
alter table t drop column age;
-- 修改字段类型
alter table t modify column age tinyint;
-- 修改字段名
alter table t change old new int;
```

2. 五种约束语法：主键、唯一、非空、默认、外键
3. 什么是online‑ddl

> 在线修改表结构，期间尽量不锁表、不阻塞业务DML

## 四、TCL 事务控制语言

1. 事务开启、提交、回滚语法

```sql
start transaction;
commit;
rollback;
```

2. set autocommit=0; 关闭自动提交含义

> 每条sql不会自动提交，需要手动commit

## 五、DCL权限控制语法

1. 创建用户、授权、回收权限

```sql
create user 'test'@'%' identified by '123456';
grant select,insert on testdb.* to 'test'@'%';
revoke insert on testdb.* from 'test'@'%';
flush privileges;
```

## 六、高级语法

### 1.窗口函数（高频手写题）

1. row_number()、rank()、dense_rank()区别

- row_number()：序号连续，即使分数相同编号也不一样
- rank()：同分序号相同，后续序号跳过
- dense_rank()：同分序号相同，后续序号连续

示例：

```sql
select name,score,rank()over(order by score desc) r from student;
```

常用窗口：partition by 分组、order by排序

### 2.case when 条件语法

```sql
case sex
when 1 then '男'
when 2 then '女'
else '未知' end
```

常用于行转列

### 3.字符串、时间函数考题

1. now()、curdate()、curtime()区别

- now() 日期+时间
- curdate() 只返回年月日
- curtime() 返回时分秒

2. date_format() 日期格式化
3. concat、concat_ws字符串拼接，null坑：concat有null返回null

### 4.if、ifnull、nullif

- ifnull(a,b) a为null返回b
- if(条件,值1,值2)

## 七、关键字坑点、语法易错面试题

1. null 判断为什么不能用 = null

> null代表未知，=无法匹配；必须使用 is null / is not null

2. or 和 and 优先级

> and优先级高于or，多条件记得加括号

3. like模糊查询

- like '%abc'、'%abc%' 不走索引；like 'abc%' 可以命中索引

4. exists 语法格式

```sql
select * from a where exists (select 1 from b where a.id=b.a_id)
```

5. 视图、存储过程、触发器、自定义函数语法考点

- view：虚拟表
- procedure存储过程，可以带参数；函数必须有return返回值

## 八、手写SQL真题（语法实操）

1. 查询数据表重复name的数据

```sql
select name,count(*) cnt from user group by name having cnt>1;
```

2. 删除重复数据，保留id最小一条

```sql
delete u1 from user u1 join user u2 on u1.name=u2.name and u1.id>u2.id;
```

3. 行转列：统计男女人数

```sql
select
sum(case when sex=1 then 1 else 0 end) man,
sum(case when sex=2 then 1 else 0 end) woman
from student
```

## 九、索引相关语法题

1. 创建普通索引、联合索引、唯一索引

```sql
create index idx_age on user(age);
create index idx_age_name on user(age,name);
create unique index idx_phone on user(phone);
```

## 十、事务锁SQL语法

1. 手动添加共享锁、排他锁

```sql
-- 共享锁S
select * from t where id=1 lock in share mode;
-- 排他锁X
select * from t where id=1 for update;
```

需要我整理一份 **SQL语法面试题+答案精简背诵版** 或者 分基础/中级手写题清单吗。

# SQL语法面试题

整体分为 **DDL、DML、DQL、TCL、DCL、内置函数、窗口函数、存储对象、语法易错坑、手写实操题** 10大模块，适配MySQL面试。

## 一、DDL｜数据定义语言（库、表、索引、结构）

负责定义、修改、删除数据库对象

1. 数据库：创建、删除、指定字符集
2. 数据表创建、字段类型选择（char/varchar/text、数值、日期）
3. 表约束语法：主键、唯一键、非空、默认值、外键
4. alter 修改表：新增/删除字段、修改字段类型、重命名字段、调整字段顺序
5. drop 删除表、索引、视图
6. truncate 清空表语法、特性
7. 索引语法：普通索引、联合索引、唯一索引、全文索引的创建与删除
8. online‑ddl、表分区语法

**高频考题**

1. drop、delete、truncate 的区别
2. 主键和唯一索引区别
3. 大表加索引有什么隐患

## 二、DML｜数据操纵语言（增、删、改）

1. insert：单条插入、批量插入
2. `insert … on duplicate key update` 唯一键冲突更新
3. replace‑into 替换插入，底层执行流程
4. 单表update更新；多表关联update
5. 单表delete删除、联表删除
6. update 执行时 null 的赋值问题

**高频考题**

1. replace 和 insert on‑duplicate 区别
2. 多表更新语法
3. update 忘记加where造成全表更新如何规避

## 三、DQL｜数据查询语言（SELECT 核心，面试重中之重）

### 3.1 基础查询

1. select 完整执行顺序
2. select * 的缺点
3. 字段别名、表别名，别名不能用于where的原因
4. distinct 单字段、多字段组合去重

### 3.2 多表连接

1. inner、left、right、cross join 使用
2. left join 中 ON 和 WHERE 的区别
3. 自连接适用场景与写法

### 3.3 条件过滤

1. where 和 having 区别
2. and、or 的优先级
3. null 判断，is null / is not null
4. between、in、not in、like、正则匹配
5. 通配符 _、%，下划线转义

### 3.4 分组聚合

1. group by 语法、only_full_group_by报错
2. 聚合函数：count(*) / count(1) / count(字段)、sum、avg、max、min
3. group_concat 分组拼接字符串
4. 聚合函数自动忽略null

### 3.5 子查询

1. 相关子查询、非相关子查询
2. in、exists、not exists、any、all
3. 标量子查询、表子查询

### 3.6 结果集处理

1. union、union‑all 差异
2. order by 排序、NULL排序规则、多字段排序
3. limit 分页、大偏移量分页缺陷、书签分页

## 四、TCL｜事务控制语言

1. start‑transaction、commit、rollback
2. autocommit 自动提交开启关闭
3. savepoint 事务保存点
4. 手动锁语法：`lock in share mode` 共享锁、`for update`排他锁

## 五、DCL｜数据控制语言、权限语法

1. 创建用户、设置密码、指定可访问主机
2. grant 分配权限
3. revoke 回收权限
4. flush‑privileges 刷新权限
5. 视图权限、远程账号配置

## 六、内置函数语法类

### 6.1 条件函数

if()、ifnull()、nullif、case‑when（简单case、搜索case）

### 6.2 日期函数

now()、curdate()、curtime()、date_format、datediff、date_add

### 6.3 字符串函数

concat、concat_ws、substring、trim、replace、char_length、length

### 6.4 数学函数

rand、round、ceil、floor、mod

**考题**

1. concat存在null返回null怎么解决
2. char_length 和 length区别

## 七、窗口函数（高级语法）

1. 排序函数：row_number()、rank()、dense_rank()、ntile()
2. 偏移函数 lag()、lead()
3. 聚合窗口 sum/avg/max over()
4. partition by 分区、order by、rows滑动窗口
5. 排名、连续天数、分组占比

## 八、数据库存储对象语法

1. view 视图：创建、修改、删除、更新视图限制
2. procedure 存储过程：in / out / inout 参数、调用方式
3. function 自定义函数，必须带有return返回值，存储过程和函数区别
4. trigger 触发器、event定时事件

## 九、语法陷阱 & 易错考题

1. 不能使用 = NULL 判断空值
2. and优先级高于or，复杂条件需要括号包裹
3. like 'xxx%' 可以走索引；'%xxx'、'%xxx%' 无法命中索引
4. not in 子查询包含null，查询结果为空
5. where中索引字段运算、函数、隐式类型转换造成索引失效
6. group by 之后select只能放分组字段、聚合字段

## 十、业务手写SQL语法题

1. 查询并删除重复数据
2. case‑when 行转列
3. 查询每组最大/最小记录
4. 连续签到、连续日期查询
5. 递归查询树形结构
6. 查询第N高的数值
7. 多表联查报表、时间维度统计
8. 悲观锁、乐观锁SQL实现

如果你需要，我可以再整理**每类问题 + 标准答案**或者区分入门、中级、高级题库。