# PostgreSQL

## PostgreSQL 详细介绍

PostgreSQL 是目前最强大的开源关系型数据库之一，通常简称 **Postgres**。
它不仅是传统的 SQL 数据库，还具备：

- 高并发
- 高可靠
- 强事务
- JSON 文档能力
- GIS 地理能力
- 向量检索
- 全文搜索
- 流复制
- 分布式扩展

因此现在大量：

- 企业核心系统
- 金融系统
- SaaS 平台
- AI Agent 系统
- 向量数据库
- 数据分析平台

都在使用 PostgreSQL。

------

### 一、PostgreSQL 的历史

PostgreSQL 起源于：

- 加州大学伯克利分校
- Ingres 数据库项目

后续演化：

```
Ingres
   ↓
POSTGRES
   ↓
PostgreSQL
```

特点：

- 诞生非常早（1986）
- 学术背景极强
- SQL 标准支持非常完整
- 强调 extensible（可扩展）

所以 PostgreSQL 被称为：

> “最像 Oracle 的开源数据库”

------

### 二、PostgreSQL 的核心特点

------

#### 1. 完整 ACID 事务

支持：

- Atomicity（原子性）
- Consistency（一致性）
- Isolation（隔离性）
- Durability（持久性）

支持：

```sql
BEGIN;
UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

失败时：

```sql
ROLLBACK;
```

------

#### 2. MVCC（多版本并发控制）

PostgreSQL 的核心。

作用：

- 读写不阻塞
- 高并发
- 减少锁竞争

------

##### 原理

每行数据都有：

```text
xmin
xmax
```

表示：

- 哪个事务创建
- 哪个事务删除

查询时：

- 根据事务快照判断可见性

因此：

- SELECT 不会阻塞 UPDATE
- UPDATE 不会阻塞 SELECT

这是 PostgreSQL 高并发的核心。

------

### 三、PostgreSQL 架构

------

#### 1. 进程模型

PostgreSQL 是：

> 多进程模型（不是多线程）

客户端连接：

```text
Client -> Postmaster -> Backend Process
```

每个连接：

- 一个独立进程

优点：

- 稳定
- 崩溃隔离

缺点：

- 连接过多会占内存

所以通常配合：

- PgBouncer
- Odyssey

做连接池。

------

#### 2. 共享内存

核心区域：

##### shared_buffers

数据库缓存。

类似 MySQL InnoDB Buffer Pool。

------

##### WAL Buffer

Write Ahead Log 缓冲区。

------

##### work_mem

排序、Hash Join 内存。

------

### 四、WAL（预写日志）

PostgreSQL 最重要机制之一。

------

#### 原理

修改数据前：

先写 WAL 日志。

流程：

```text
修改数据
   ↓
写 WAL
   ↓
WAL fsync
   ↓
COMMIT 成功
```

因此：

即使宕机：

- 也能恢复数据

------

#### WAL 的作用

##### 1. 崩溃恢复

Crash Recovery

------

##### 2. 主从复制

Streaming Replication

------

##### 3. 时间点恢复（PITR）

Point In Time Recovery。

------

### 五、存储结构

------

#### 1. Database

数据库。

------

#### 2. Schema

模式。

类似：

```text
Java package
```

例如：

```sql
public.user
sales.order
```

------

#### 3. Table

表。

------

#### 4. Page（页）

PostgreSQL 默认：

```text
8KB
```

数据以页存储。

------

#### 5. Tuple（元组）

一行记录。

------

### 六、索引系统

PostgreSQL 索引极强。

------

#### 1. B-Tree

默认索引。

适合：

- 等值查询
- 范围查询
- 排序

```sql
CREATE INDEX idx_user_name ON users(name);
```

------

#### 2. Hash Index

哈希索引。

适合：

- 精确匹配

较少使用。

------

#### 3. GIN

Generalized Inverted Index。

非常重要。

适合：

- JSONB
- 数组
- 全文搜索
- 向量

例如：

```sql
CREATE INDEX idx_json ON doc USING GIN(data);
```

------

#### 4. GiST

适合：

- GIS
- 范围查询

------

#### 5. BRIN

Block Range Index。

适合：

- 超大时序表

例如：

- 日志
- IoT

------

### 七、JSON 能力（非常强）

PostgreSQL 不只是关系型数据库。

还支持：

```sql
JSON
JSONB
```

------

#### JSONB

二进制 JSON。

支持：

- 索引
- 高效查询

例如：

```sql
CREATE TABLE user_profile(
    id BIGSERIAL,
    data JSONB
);
```

插入：

```sql
INSERT INTO user_profile(data)
VALUES ('{"name":"Tom","age":18}');
```

查询：

```sql
SELECT *
FROM user_profile
WHERE data->>'name' = 'Tom';
```

------

#### 为什么强

现在很多系统：

- 半结构化数据
- 动态字段

非常适合 PostgreSQL。

因此：

PostgreSQL：

> 同时具备 MySQL + MongoDB 一部分能力

------

### 八、全文搜索

PostgreSQL 内置全文搜索。

不一定需要 Elasticsearch。

------

#### 示例

```sql
SELECT *
FROM article
WHERE to_tsvector(content)
@@ to_tsquery('database');
```

支持：

- 分词
- 排名
- 倒排索引

------

### 九、主从复制

------

#### Streaming Replication

主库：

```text
WAL -> Standby
```

从库实时同步 WAL。

------

#### 同步复制

主库必须等待从库 ACK。

优点：

- 强一致

缺点：

- 延迟增加

------

#### 异步复制

默认模式。

性能更高。

------

### 十、高可用（HA）

常见方案：

------

#### 1. Patroni

PostgreSQL HA 事实标准。

支持：

- 自动故障转移
- Leader 选举

------

#### 2. etcd / Consul

存储集群状态。

------

#### 3. Keepalived + VIP

虚拟 IP 漂移。

------

### 十一、分区表

PostgreSQL 原生支持：

```sql
PARTITION BY RANGE
PARTITION BY HASH
PARTITION BY LIST
```

例如：

```sql
CREATE TABLE log (
    id BIGINT,
    create_time TIMESTAMP
)
PARTITION BY RANGE(create_time);
```

适合：

- 大表
- 时序数据

------

### 十二、查询优化器

PostgreSQL 优化器非常强。

支持：

- Cost Based Optimizer（CBO）

会自动选择：

- 索引扫描
- Seq Scan
- Hash Join
- Merge Join
- Nested Loop

------

#### 查看执行计划

```sql
EXPLAIN ANALYZE
SELECT * FROM user WHERE id = 1;
```

非常重要。

------

### 十三、PostgreSQL 与 MySQL 对比

| 对比项   | PostgreSQL     | MySQL |
| -------- | -------------- | ----- |
| SQL 标准 | 更完整         | 较弱  |
| 事务能力 | 很强           | 强    |
| JSON     | 极强           | 一般  |
| 扩展性   | 极强           | 一般  |
| GIS      | 强             | 一般  |
| 向量检索 | 好             | 较弱  |
| 全文搜索 | 强             | 一般  |
| OLTP     | 强             | 强    |
| OLAP     | 更适合复杂分析 | 一般  |
| 学习成本 | 较高           | 较低  |

------

### 十四、PostgreSQL 在 AI 时代的价值

现在 PostgreSQL 非常热门。

因为：

------

#### 1. pgvector

PostgreSQL 可以直接做：

- 向量数据库

例如：

```sql
CREATE EXTENSION vector;
```

存 embedding：

```sql
embedding vector(1536)
```

做：

```sql
ORDER BY embedding <-> query_vector
```

因此：

- RAG
- AI Agent
- 语义检索

都能做。

------

#### 2. LangChain / Spring AI 支持

很多 AI 框架直接支持 PostgreSQL。

包括：

- LangChain
- Spring AI
- LangGraph
- LlamaIndex

------

### 十五、PostgreSQL 常见生态

------

#### 管理工具

##### pgAdmin

官方 GUI。

------

##### DBeaver

非常流行。

------

##### DataGrip

JetBrains。

------

#### 连接池

##### PgBouncer

最常用。

------

##### ORM

##### Hibernate

##### MyBatis

##### JPA

------

### 十六、PostgreSQL 企业级最佳实践

------

#### 1. 必开 WAL 归档

用于 PITR。

------

#### 2. 使用连接池

不要直连。

------

#### 3. 大表分区

超过千万行建议分区。

------

#### 4. 定期 VACUUM

因为 MVCC 会产生死元组。

------

#### 5. 使用 EXPLAIN ANALYZE

优化 SQL。

------

### 十七、VACUUM 机制（核心）

因为 MVCC：

旧版本不会立刻删除。

因此：

会产生：

```text
dead tuple
```

需要：

```sql
VACUUM;
```

否则：

- 表膨胀
- 索引膨胀

------

#### Autovacuum

PostgreSQL 自动清理机制。

非常重要。

生产环境必须正确配置。

------

### 十八、PostgreSQL 为什么越来越火

因为它：

- 免费
- 开源
- 功能强
- 可扩展
- AI 友好
- 云原生友好

很多公司：

正在：

```text
Oracle -> PostgreSQL
MySQL -> PostgreSQL
```

迁移。

------

### 十九、PostgreSQL 适合什么场景

非常适合：

- 企业核心系统
- 金融系统
- SaaS
- AI Agent
- RAG
- GIS
- 大数据分析
- 时序数据

------

### 二十、学习路线（推荐）

建议顺序：

------

#### 第一阶段

学习：

- SQL
- CRUD
- JOIN
- 索引
- 事务

------

#### 第二阶段

学习：

- MVCC
- WAL
- EXPLAIN
- VACUUM
- 锁机制

------

#### 第三阶段

学习：

- 主从复制
- 分区
- 高可用
- 性能调优

------

#### 第四阶段

学习：

- JSONB
- pgvector
- 全文搜索
- TimescaleDB

------

### 二十一、PostgreSQL 核心一句话总结

PostgreSQL 本质上是：

> 一个“可扩展的企业级数据平台”

它不仅是关系型数据库，还逐渐演化成：

- 文档数据库
- 向量数据库
- GIS 数据库
- 时序数据库
- AI 数据平台

的统一底座。

## PostgreSQL 使用教程（从入门到企业开发）

PostgreSQL 的学习建议：

```text
安装 → 基础SQL → 表设计 → 索引 → 事务 → Java连接 → 高级特性
```

下面按企业开发流程带你完整学习。

------

### 一、安装 PostgreSQL

------

#### 1. Windows 安装

官方下载：

[PostgreSQL Downloads](https://www.postgresql.org/download/?utm_source=chatgpt.com)

安装时注意：

```text
记住：
- superuser: postgres
- password
- port: 5432
```

安装完成后：

```text
pgAdmin
PostgreSQL Service
psql
```

都会自动安装。

------

### 二、启动 PostgreSQL

------

#### Windows

服务：

```text
postgresql-x64-17
```

启动：

```bash
net start postgresql-x64-17
```

------

#### Linux

启动：

```bash
sudo systemctl start postgresql
```

查看状态：

```bash
sudo systemctl status postgresql
```

------

### 三、连接 PostgreSQL

------

#### 1. 使用 psql

进入：

```bash
psql -U postgres
```

指定数据库：

```bash
psql -U postgres -d testdb
```

------

#### 2. 常用命令

查看数据库：

```sql
\l
```

切换数据库：

```sql
\c testdb
```

查看表：

```sql
\dt
```

查看表结构：

```sql
\d users
```

退出：

```sql
\q
```

------

### 四、创建数据库

------

#### 创建数据库

```sql
CREATE DATABASE theater;
```

进入数据库：

```sql
\c theater
```

------

### 五、创建表

------

#### 用户表

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(100),
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

------

#### 解释

------

#### BIGSERIAL

自增主键：

```text
1
2
3
4
```

类似 MySQL：

```sql
AUTO_INCREMENT
```

------

#### PRIMARY KEY

主键索引。

------

#### DEFAULT CURRENT_TIMESTAMP

默认当前时间。

------

### 六、插入数据

------

#### 插入单条

```sql
INSERT INTO users(username, age, email)
VALUES ('Tom', 18, 'tom@test.com');
```

------

#### 插入多条

```sql
INSERT INTO users(username, age)
VALUES
('Alice', 20),
('Bob', 25),
('Jack', 30);
```

------

### 七、查询数据

------

#### 查询全部

```sql
SELECT * FROM users;
```

------

#### 条件查询

```sql
SELECT *
FROM users
WHERE age > 20;
```

------

#### 排序

```sql
SELECT *
FROM users
ORDER BY age DESC;
```

------

#### 分页

```sql
SELECT *
FROM users
LIMIT 10 OFFSET 0;
```

等价：

```text
第一页，每页10条
```

------

### 八、更新数据

------

#### 更新

```sql
UPDATE users
SET age = 28
WHERE id = 1;
```

------

### 九、删除数据

------

#### 删除

```sql
DELETE FROM users
WHERE id = 1;
```

------

### 十、事务使用

PostgreSQL 的事务能力非常强。

#### 

#### 示例

```sql
BEGIN;

UPDATE users SET age = age - 1 WHERE id = 1;

UPDATE users SET age = age + 1 WHERE id = 2;

COMMIT;
```

回滚：

```sql
ROLLBACK;
```

------

### 十一、索引使用

------

#### 创建索引

```sql
CREATE INDEX idx_user_name
ON users(username);
```

------

#### 查看执行计划

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE username = 'Tom';
```

非常重要。

企业里每天都在用。

------

### 十二、JSONB 使用（PostgreSQL 核心优势）

------

#### 创建 JSONB 表

```sql
CREATE TABLE user_profile (
    id BIGSERIAL PRIMARY KEY,
    profile JSONB
);
```

------

#### 插入 JSON

```sql
INSERT INTO user_profile(profile)
VALUES (
    '{"name":"Tom","skills":["Java","Python"]}'
);
```

------

#### 查询 JSON

```sql
SELECT *
FROM user_profile
WHERE profile->>'name' = 'Tom';
```

------

#### 查询数组

```sql
SELECT *
FROM user_profile
WHERE profile->'skills' ? 'Java';
```

------

### 十三、GIN 索引（JSONB 必备）

------

#### 创建 GIN 索引

```sql
CREATE INDEX idx_profile_gin
ON user_profile
USING GIN(profile);
```

否则 JSON 查询会很慢。

------

### 十四、全文搜索

------

#### 创建文章表

```sql
CREATE TABLE article (
    id BIGSERIAL,
    content TEXT
);
```

------

#### 搜索

```sql
SELECT *
FROM article
WHERE to_tsvector(content)
@@ to_tsquery('database');
```

------

### 十五、分页优化

不要：

```sql
LIMIT 10 OFFSET 100000
```

因为：

```text
OFFSET 越大越慢
```

推荐：

------

#### 游标分页

```sql
SELECT *
FROM users
WHERE id > 1000
LIMIT 10;
```

企业级标准方案。

------

### 十六、PostgreSQL 用户管理

------

#### 创建用户

```sql
CREATE USER theater_user
WITH PASSWORD '123456';
```

------

#### 授权

```sql
GRANT ALL PRIVILEGES
ON DATABASE theater
TO theater_user;
```

------

### 十七、Java 连接 PostgreSQL

------

#### Maven 依赖

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.4</version>
</dependency>
```

------

#### JDBC

```java
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/theater",
    "postgres",
    "123456"
);
```

------

### 十八、Spring Boot 集成 PostgreSQL

------

#### application.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/theater
    username: postgres
    password: 123456
    driver-class-name: org.postgresql.Driver
```

------

#### JPA

```java
@Entity
@Table(name = "users")
public class User {
}
```

------

#### MyBatis

```xml
<select id="findAll" resultType="User">
    select * from users
</select>
```

------

### 十九、Docker 安装 PostgreSQL

企业开发非常常见。

------

#### 启动容器

```bash
docker run -d \
--name postgres \
-e POSTGRES_PASSWORD=123456 \
-p 5432:5432 \
postgres:17
```

------

#### 进入容器

```bash
docker exec -it postgres bash
```

------

#### 进入数据库

```bash
psql -U postgres
```

------

### 二十、主从复制（了解）

企业生产环境：

```text
Primary
   ↓
Replica
```

用于：

- 高可用
- 读写分离
- 灾备

------

### 二十一、VACUUM（非常重要）

PostgreSQL 使用 MVCC。

会产生：

```text
dead tuple
```

所以必须清理。

------

#### 手动清理

```sql
VACUUM;
```

------

#### 自动清理

```text
autovacuum
```

生产环境必须开启。

------

### 二十二、企业最佳实践

------

#### 1. 一定加索引

高频查询字段：

```text
WHERE
JOIN
ORDER BY
```

都要考虑索引。

------

#### 2. 避免 SELECT *

使用：

```sql
SELECT id, username
```

------

#### 3. 大表分区

超过千万行：

```text
Partition Table
```

------

#### 4. 使用连接池

推荐：

- HikariCP
- PgBouncer

------

### 5. 用 EXPLAIN ANALYZE 调优

```sql
EXPLAIN ANALYZE
SELECT ...
```

这是 PostgreSQL 性能优化核心。

------

### 二十三、PostgreSQL AI 场景（非常热门）

------

#### 安装 pgvector

```sql
CREATE EXTENSION vector;
```

------

#### 创建向量列

```sql
embedding vector(1536)
```

------

#### 向量检索

```sql
SELECT *
FROM doc
ORDER BY embedding <-> '[1,2,3]'
LIMIT 5;
```

现在：

- RAG
- AI Agent
- LangChain
- Spring AI

大量使用 PostgreSQL。

------

### 二十四、学习路线（推荐）

------

#### 第一阶段（SQL）

学习：

- CRUD
- JOIN
- GROUP BY
- ORDER BY
- 索引

------

#### 第二阶段（核心原理）

学习：

- MVCC
- WAL
- 锁
- VACUUM
- 执行计划

------

#### 第三阶段（企业级）

学习：

- 主从复制
- 高可用
- 分区
- 调优

------

#### 第四阶段（高级）

学习：

- JSONB
- 全文搜索
- pgvector
- TimescaleDB

------

### 二十五、推荐工具

------

#### GUI 工具

#### pgAdmin

官方工具。

------

#### DBeaver

非常推荐。

------

#### DataGrip

企业里很多人用。

------

### 二十六、一套完整企业开发流程

```text
Spring Boot
    ↓
HikariCP
    ↓
PostgreSQL
    ↓
Redis 缓存
    ↓
Kafka / RocketMQ
```

PostgreSQL 往往是：

```text
核心事务数据库
```

------

### 二十七、最终总结

PostgreSQL 最核心优势：

```text
强事务
强扩展
强SQL
强JSON
强AI能力
```

因此现在 PostgreSQL 正逐渐成为：

```text
新一代企业级默认数据库
```

## Spring Boot 集成 PostgreSQL（完整企业级教程）

使用：

- Spring Boot
- PostgreSQL

是现在企业开发非常常见的组合。

典型架构：

```text
Spring Boot
    ↓
HikariCP
    ↓
PostgreSQL
```

下面从：

```text
创建项目
→ 配置数据源
→ 建表
→ JPA/MyBatis
→ CRUD
→ 企业最佳实践
```

完整带你实现。

------

### 一、创建 PostgreSQL 数据库

先进入 PostgreSQL：

```bash
psql -U postgres
```

创建数据库：

```sql
CREATE DATABASE theater;
```

创建用户：

```sql
CREATE USER theater_user
WITH PASSWORD '123456';
```

授权：

```sql
GRANT ALL PRIVILEGES
ON DATABASE theater
TO theater_user;
```

------

### 二、创建 Spring Boot 项目

推荐：

- Java 17+
- Spring Boot 3.x

------

### 三、引入依赖

------

#### 方式1：JPA

##### pom.xml

```xml
<dependencies>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.4</version>
    </dependency>

</dependencies>
```

------

#### 方式2：MyBatis（企业更常见）

##### pom.xml

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>3.0.3</version>
    </dependency>

    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.4</version>
    </dependency>

</dependencies>
```

------

### 四、配置 application.yml

------

#### application.yml

```yaml
spring:

  datasource:
    url: jdbc:postgresql://localhost:5432/theater
    username: theater_user
    password: 123456
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: update

    show-sql: true

    properties:
      hibernate:
        format_sql: true
```

------

### 五、创建表

------

#### PostgreSQL 建表

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50),
    age INT,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

------

### 六、JPA 方式（完整 CRUD）

------

#### 1. 创建实体类

##### User.java

```java
@Entity
@Table(name = "users")
@Data
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    private Integer age;
}
```

------

#### 2. Repository

##### UserRepository.java

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

------

#### 3. Service

##### UserService.java

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public User save(User user) {
        return userRepository.save(user);
    }

    public List<User> list() {
        return userRepository.findAll();
    }
}
```

------

#### 4. Controller

##### UserController.java

```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public User save(@RequestBody User user) {
        return userService.save(user);
    }

    @GetMapping
    public List<User> list() {
        return userService.list();
    }
}
```

------

### 七、MyBatis 方式（企业推荐）

很多企业：

```text
Spring Boot + MyBatis + PostgreSQL
```

------

#### 1. Entity

```java
@Data
public class User {

    private Long id;

    private String username;

    private Integer age;
}
```

------

#### 2. Mapper

##### UserMapper.java

```java
@Mapper
public interface UserMapper {

    @Insert("""
        insert into users(username, age)
        values(#{username}, #{age})
    """)
    void save(User user);

    @Select("""
        select *
        from users
    """)
    List<User> findAll();
}
```

------

##### 3. Service

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserMapper userMapper;

    public void save(User user) {
        userMapper.save(user);
    }

    public List<User> list() {
        return userMapper.findAll();
    }
}
```

------

### 八、测试接口

------

#### 插入数据

```http
POST /user
Content-Type: application/json

{
  "username":"Tom",
  "age":18
}
```

------

#### 查询数据

```http
GET /user
```

------

### 九、PostgreSQL 与 MySQL 差异（重要）

------

#### 1. 自增主键

MySQL：

```sql
AUTO_INCREMENT
```

PostgreSQL：

```sql
BIGSERIAL
```

------

#### 2. 分页

PostgreSQL：

```sql
LIMIT 10 OFFSET 0
```

------

#### 3. 大小写问题

PostgreSQL：

```text
默认转小写
```

所以：

```sql
user_name
```

比：

```sql
userName
```

更推荐。

------

### 十、事务管理

Spring Boot 默认支持事务。

------

#### 使用方式

```java
@Transactional
public void transfer() {

}
```

------

#### PostgreSQL 事务非常强

支持：

- MVCC
- 行锁
- Serializable

------

### 十一、JSONB 使用（非常重要）

PostgreSQL 最大优势之一。

------

#### 建表

```sql
CREATE TABLE user_profile (
    id BIGSERIAL PRIMARY KEY,
    profile JSONB
);
```

------

#### 插入

```sql
INSERT INTO user_profile(profile)
VALUES (
'{"name":"Tom","skills":["Java","AI"]}'
);
```

------

#### 查询

```sql
SELECT *
FROM user_profile
WHERE profile->>'name' = 'Tom';
```

------

#### Spring Boot 使用

实体：

```java
@Column(columnDefinition = "jsonb")
private String profile;
```

------

### 十二、分页查询

------

#### JPA 分页

```java
Page<User> page =
userRepository.findAll(
    PageRequest.of(0, 10)
);
```

------

#### MyBatis 分页

推荐：

- PageHelper
- MyBatis Plus

------

### 十三、连接池（企业必须）

Spring Boot 默认：

```text
HikariCP
```

性能非常高。

------

#### 配置

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
```

------

### 十四、Docker PostgreSQL 连接

如果 PostgreSQL 在 Docker：

------

#### docker run

```bash
docker run -d \
--name postgres \
-e POSTGRES_PASSWORD=123456 \
-p 5432:5432 \
postgres:17
```

------

#### Spring Boot 配置

```yaml
url: jdbc:postgresql://localhost:5432/postgres
```

即可。

------

### 十五、Flyway 数据库版本管理（企业推荐）

企业里：

```text
绝对不要手动改表
```

推荐：

Flyway

------

#### Maven

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

------

#### 创建 SQL

```text
resources/db/migration
```

例如：

```text
V1__create_user.sql
```

启动自动执行。

------

### 十六、日志 SQL 输出

------

#### application.yml

```yaml
logging:
  level:
    org.hibernate.SQL: debug
```

------

### 十七、性能优化（企业重点）

------

#### 1. 必加索引

```sql
CREATE INDEX idx_user_name
ON users(username);
```

------

#### 2. 查看执行计划

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE username='Tom';
```

------

#### 3. 避免 SELECT *

推荐：

```sql
SELECT id, username
```

------

### 十八、AI 场景（现在非常热门）

PostgreSQL + pgvector：

已经成为：

```text
AI Agent
RAG
Embedding Search
```

主流方案。

------

#### pgvector

```sql
CREATE EXTENSION vector;
```

------

#### 向量列

```sql
embedding vector(1536)
```

------

### 十九、企业推荐技术栈

推荐：

```text
Spring Boot
    ↓
MyBatis
    ↓
PostgreSQL
    ↓
Redis
    ↓
Kafka/RocketMQ
```

这是现在大量互联网公司的标准方案。

------

### 二十、最终总结

Spring Boot 集成 PostgreSQL 的核心：

```text
1. 引入驱动
2. 配置 datasource
3. 建表
4. ORM(MyBatis/JPA)
5. CRUD
```

真正企业核心：

```text
事务
索引
执行计划
连接池
JSONB
性能调优
```