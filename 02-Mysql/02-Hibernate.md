# Hibernate

### ORM

**MyBatis‑Plus 绝对不是全自动 ORM，依旧属于半自动 ORM**
我给你用最直白的分层讲清楚三者等级：
1. Hibernate / Spring‑Data‑JPA → **全自动ORM**
2. MyBatis‑Plus → **增强版半自动ORM**
3. 原生MyBatis → 基础半自动ORM

#### 一、一句话区分本质

1. **全自动(JPA‑Hibernate)**
你只操作实体类，**所有SQL全部框架自动生成**，你看不见SQL。
复杂查询也要用 JPQL、方法名、代码构造器，不能直接写原生SQL为主。

2. **MyBatis‑Plus**
- 普通CRUD（insert、updateById、selectPage、list）由封装好的BaseMapper自动实现，不用写SQL；
- **一旦遇到复杂多表联查、子查询、复杂统计，你必须手写SQL（XML / 注解SQL）**

> MP只是补齐了重复CRUD模板代码，底层根基仍然是原生MyBatis，SQL的最终控制权握在开发者手上。

#### 二、通俗比喻

- JPA(Hibernate)：自动挡汽车，油门一踩，变速箱、离合全部内部处理，你干预不了
- Mybatis‑Plus：**自带自动启停、自动手刹的手动挡**
基础起步、停车自动化；山路、复杂路况你依旧需要自己挂挡（手写SQL）

#### 三、对比表格

|框架类型|CRUD|复杂联表SQL|SQL控制权|ORM类型|
|---|---|---|---|---|
|Spring‑Data‑JPA(Hibernate)|自动生成|框架生成JPQL，很难原生SQL|框架掌控|全自动|
|MyBatis‑Plus|BaseMapper自带|自己手写xml/sql|程序员掌控|半自动|
|原生MyBatis|全部手写SQL|手写xml/sql|程序员掌控|半自动|

#### 四、MP有哪些自动功能（很多人误以为全自动就是因为这些）

1. BaseMapper 内置：新增、根据id修改、删除、单条查询、列表查询
2. LambdaQueryWrapper 条件构造器，可以代码拼装简单条件查询，不用写XML
3. 分页插件、逻辑删除、自动填充、乐观锁
4. 代码生成器一键生成 mapper、service、entity、controller

**但是重点来了：Wrapper只适合单表查询！**
只要是多表join、多表统计、复杂子查询，Wrapper基本无力，你必须手写SQL。

#### 五、JPA 和 MP 最核心分水岭

1. JPA 从根源设计就是面向对象，尽量杜绝手写原生SQL；多表关联依靠实体注解 `@OneToMany`
2. MP 底层根基就是MyBatis，**原生SQL是第一优先级**，单表快捷API只是附加福利

#### 六、结合你电信ERP项目给你选型提醒

- 字典、配置、单表参数：直接 MP‑Wrapper 快速查询，享受自动CRUD
- 账单、计费、流水、多表账务统计：手写XML SQL，精细控制SQL、索引、性能

#### 七、简短总结

> MyBatis‑Plus = 单表CRUD自动化，多表查询仍然需要手写SQL，属于半自动ORM；
> Hibernate(JPA) 才是真正的全自动ORM。

#### 额外补充一个容易混淆知识点

如果你追求「全自动单表 + 可手写复杂SQL」，MP就是最佳选择；
如果你连单表、关联表都不想碰SQL，那就选JPA‑Hibernate。

### Hibernate(JPA‑Hibernate) vs MyBatis 完整选型指南

先核心结论：
> 传统互联网、复杂SQL、大数据、需要精细调优 → **MyBatis（MyBatis‑Plus）**
> ERP、后台管理、表单CRUD、数据表量大、SQL简单、快速开发 → **Spring‑Data‑JPA（底层Hibernate）**

下面分清楚原理、优缺点、适用场景、对比清单、踩坑点、行业现状、落地建议，结合你日常 PG、MySQL、Redis 架构来讲。

#### 一、本质区别（最关键）

##### 1、Hibernate / Spring‑Data‑JPA

**全自动 ORM**
你操作 Java 实体对象，框架自动生成 SQL；
你几乎不用手写 SQL。
查询语言：HQL / JPQL、方法名查询、Specification。

优点：面向对象，数据库无关，切换 MySQL → PostgreSQL 业务代码几乎不动。
缺点：SQL黑盒，不好控制，容易 N+1、批量更新性能差。

##### 2、MyBatis

**半自动 ORM**
SQL 由你自己写在 XML / 注解里面；Mybatis 只负责 SQL 参数注入、结果集封装实体。
优点：SQL完全可控，方便索引优化、复杂联表、存储过程、原生数据库函数。

> 通俗比喻
> JPA(Hibernate)=自动挡汽车，只管踩油门，内部传动你看不到
> MyBatis=手动挡，离合档位全部自己掌控

---

#### 二、详细优缺点对比

##### ✅ JPA(Hibernate) 优势

1. CRUD 零模板代码，内置分页、排序、条件构造器
2. 一级缓存、二级缓存、懒加载、级联关系一键配置
3. 面向对象关联：OneToOne、OneToMany、ManyToMany，处理主外键很省心
4. 跨数据库兼容，MySQL、PostgreSQL、Oracle切换不需要改动SQL
5. 支持 JSONB，适配 PostgreSQL 非常舒服
6. 适合快速开发管理后台、表单业务

##### ❌ JPA‑Hibernate 短板

1. 自动生成SQL不可控，关联查询极易触发 **N+1 陷阱**
2. 多表复杂Join、子查询、存储过程、大批量更新很难写
3. 批量操作性能差；大批量数据容易产生大量SQL
4. 调试麻烦，需要打印、分析生成出来的SQL
5. 部分PG独有的函数、索引方言适配需要配置

##### ✅ MyBatis 优势

1. SQL完全手写，每一条语句自己把控，方便调优、建索引
2. 复杂多表查询、统计报表、批量更新、原生数据库函数适配简单
3. 学习成本低，SQL就是日常你写的SQL
4. Mybatis‑Plus 补齐了自动CRUD、分页，兼顾手写SQL和快捷开发
5. 项目排查慢SQL极其方便

##### ❌ MyBatis 短板

1. 主外键、对象关联需要自己手动处理
2. 切换数据库时，SQL语法差异需要修改xml
3. 重复CRUD模板代码多（MP可以解决）

---

#### 三、硬性选型标准（直接照着选）

##### 选 Spring‑Data‑JPA（Hibernate底层），满足任意多条

1. 项目是后台管理系统、ERP、工单、配置类业务，以简单CRUD为主
2. 数据表数量非常多，不想写重复 insert/update/select
3. 经常需要切换 MySQL / PostgreSQL
4. 表与表之间外键关联多（一对多、多对多）
5. 需要大量使用 PG 的 JSONB 字段
6. 团队不想投入精力维护大量XML‑SQL文件

##### 选 MyBatis / MyBatis‑Plus，满足任意多条

1. 存在大量复杂联表、统计查询、报表、分组、子查询
2. 大数据量、高并发业务，SQL性能优化是重点
3. 存储过程、数据库自定义函数、原生数据库特性使用频繁
4. 电商、支付、订单、流水表、日志表等高频业务
5. 习惯手写SQL，不想被框架自动SQL束缚
6. 后期需要做SQL审计、慢SQL排查

> 你现在做电信ERP、支付系统（PMS/FAC/CMI）
> **电信业务大量复杂关联查询、账务统计、批量数据，优先 MyBatis‑Plus**
> 如果内部配置模块、基础档案表可以局部使用JPA。

---

#### 四、一张对比总表

| 对比维度     | Spring‑Data‑JPA(Hibernate) | MyBatis(‑Plus)       |
| ------------ | -------------------------- | -------------------- |
| SQL控制权    | 框架自动生成               | 开发者手写           |
| ORM等级      | 全自动                     | 半自动               |
| 复杂多表查询 | 困难，容易N+1              | 得心应手             |
| 主外键关联   | 注解配置、懒加载           | 手动处理关联         |
| 数据库迁移   | 适配极佳                   | SQL需要改动          |
| CRUD效率     | 极高                       | MP之后很高           |
| 性能调优难度 | 高，黑盒SQL                | 低，SQL可见          |
| 适合PG‑JSONB | 优秀                       | 原生需要手动处理json |

---

#### 五、常见踩坑提醒

##### JPA(Hibernate)必避坑

1. 一对多关联默认懒加载，遍历集合触发 N+1，要用 fetch join
2. 不要循环save批量数据，使用批量持久化配置
3. 实体equals、hashCode、事务、脏检查容易引发意料之外的update语句

##### MyBatis 坑

1. 多表关联需要手动封装结果
2. 不同数据库分页SQL语法不一样
3. 注意 #{} 和 ${} 防止SQL注入

---

#### 六、现在行业主流现状

1. 互联网大厂、支付、订单、流量业务：**MyBatis‑Plus** 占绝大多数
2. 传统ERP、政务、OA、中小型管理系统、外包项目：JPA(Hibernate)居多
3. 新项目极少裸写原生Hibernate API，都是 Spring‑Data‑JPA

---

#### 七、终极选型口诀

> **简单CRUD、多外键、PG‑JSONB → JPA(Hibernate)**
> **复杂SQL、账务报表、高并发调优 → MyBatis‑Plus**

需要我再给你整理一份 JPA N+1问题全套解决方案吗？