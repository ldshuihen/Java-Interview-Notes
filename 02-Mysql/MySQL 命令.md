# MySQL Windows‑CMD 完整常用命令操作教程

> 前提：已经安装 MySQL、配置好环境变量（可以直接 cmd 识别 mysql 命令）；没配置环境变量需要进入 MySQL 的 `bin` 目录执行命令

## 一、登录 MySQL

### 1.基础登录

```
mysql -u root -p
```

- `-u`：指定用户名，默认管理员 `root`
- `-p`：回车之后输入密码（密码输入时屏幕不显示）

### 带密码直接登录（不推荐，明文暴露）

```
mysql -u root -p123456
```

### 指定端口登录（MySQL 修改过端口，默认3306）

```
mysql -u root -p -P 3307
```

### 远程连接

```
mysql -h 192.168.1.100 -P 3306 -u root -p
```

------

## 二、MySQL 服务启停（Windows cmd，管理员模式）

```
# 启动服务，服务名一般 MySQL80（看你安装版本）
net start MySQL80

# 停止服务
net stop MySQL80
```

> 查看本机mysql服务名：`services.msc`

------

## 三、数据库操作（登录成功之后在 mysql> 终端内执行）

### 1.查看所有数据库

```
show databases;
```

### 2.创建数据库

```
create database testdb;
-- 指定字符集
create database testdb default character set utf8mb4;
```

### 3.进入/使用数据库

```
use testdb;
```

### 4.查看当前正在使用的数据库

```
select database();
```

### 5.删除数据库

```
drop database testdb;
```

------

## 四、数据表操作

### 1.查看当前库所有表

```
show tables;
```

### 2.新建数据表

```
create table user(
id int primary key auto_increment,
name varchar(50) not null,
age int
)engine=InnoDB default charset=utf8mb4;
```

### 3.查看表结构

```
desc user;
-- 完整建表语句
show create table user;
```

### 4.修改表名

```
rename table user to t_user;
```

### 5.删除数据表

```
drop table t_user;
```

------

## 五、表数据 CRUD

### 新增

```
insert into user(name,age) values ('张三',20);
```

### 查询

```
select * from user;
select id,name from user where age>18;
```

### 更新

```
update user set age=22 where id=1;
```

### 删除

```
delete from user where id=1;
```

------

## 六、用户与密码管理

### 1.修改 root 密码（MySQL8.0+）

```
alter user 'root'@'localhost' identified by '新密码';
```

### 5.x版本修改密码

```
set password for root@localhost=password('新密码');
```

### 创建新用户、授权

```
-- 创建用户
create user 'test'@'localhost' identified by '123456';
-- 授予testdb全部权限
grant all privileges on testdb.* to 'test'@'localhost';
-- 刷新权限
flush privileges;
```

------

## 七、导入导出 sql 备份（**新开普通cmd，不要登录mysql**）

### 1.导出整个数据库备份（mysqldump）

```
mysqldump -u root -p testdb > D:\sql\testdb.sql
```

### 2.导入数据库

方式一 cmd：

```
mysql -u root -p testdb < D:\sql\testdb.sql
```

方式二 mysql终端：

```
source D:/sql/testdb.sql;
```

------

## 八、常用实用命令（mysql>环境）

```
-- 查看mysql版本
select version();
-- 查看字符集
show variables like 'character%';
-- 查看端口
show variables like 'port';
-- 退出mysql终端
exit;
quit;
```

------

## 九、常见报错解决

1. cmd提示mysql不是内部命令： 把 `MySQL安装路径\bin` 添加到Windows系统环境变量‑Path
2. 3306端口被占用：关闭占用程序或者修改my.ini端口
3. 忘记root密码：跳过权限验证重置密码

需要我整理一份 MySQL‑cmd 速查表可复制清单吗？