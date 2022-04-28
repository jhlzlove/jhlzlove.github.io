---
title: MySQL数据库
music:
  server: netease
  type: song
  id: 373114
categories:
  - Java
  - MySQL
abbrlink: 9e299c6e
---
MySQL整理

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true}-->

<!-- code_chunk_output -->

- [MySQL数据库](#mysql数据库)
  - [1. 数据库安装](#1-数据库安装)
    - [1.1 Mysql5.7 版本安装](#11-mysql57-版本安装)
    - [1.2 MySQL8.0安装](#12-mysql80安装)
  - [数据库操作：](#数据库操作)
    - [DQL DML DDL DCL](#dql-dml-ddl-dcl)
    - [数据库分区](#数据库分区)
  - [MySQL数据库优化](#mysql数据库优化)
    - [数据库事务的四个特性](#数据库事务的四个特性)
    - [事务的四种隔离级别：](#事务的四种隔离级别)
    - [数据库索引分类：](#数据库索引分类)
    - [数据表结构优化：](#数据表结构优化)
    - [表分区](#表分区)
    - [SQL 语句优化](#sql-语句优化)
  - [数据库锁](#数据库锁)
    - [悲观锁、乐观锁](#悲观锁乐观锁)
  - [架构](#架构)
  - [MySQL主从复制](#mysql主从复制)
  - [集群架构](#集群架构)

<!-- /code_chunk_output -->
# MySQL数据库

## 1. 数据库安装

官网下载安装版或者解压版，安装版一般不会有什么错误，环境变量如果安装的时候勾选加入 Path 中也不用配置，比较简单。这里介绍解压版的安装使用。

### 1.1 Mysql5.7 版本安装

下载压缩包解压，设置好环境变量 MYSQL_HOME=解压目录，在path加上`%MY_HOME%\bin`。
然后执行以下三条命令：
```bash{.line-numbers}
mysqld -install
mysqld --initialize-insecure --user=mysql
net start mysql
```

**附：**
```bash{.line-numbers}
# 安装 MySQL 服务
mysqld -install
# 移除 MySQL 服务
mysqld -remove
# 初始化，建立data文件
mysqld --initialize
```


### 1.2 MySQL8.0安装
解压出来配置好环境变量；然后和上面一样。
```bash{.line-numbers}
# 安装服务并启动
mysqld --install
mysqld --initialize-insecure --user=mysql
net start mysql
# 进入 MySQL 环境，初始无密码，直接回车就 ok
mysql -u root -p
# 修改密码为 123456，当然也可以修改成其他的
alter user 'root'@'localhost' identified by '123456';
```
如果连接sqlyog报错，进入到 MySQL 的环境(mysql -u root -p)，然后执行以下命令：
```bash
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
> Java程序中连接8.0以上的数据库时区设置：`serverTimezone=Asia/Shanghai`
> UTC时间和中国时间相差8小时。

## 数据库操作：

### DQL DML DDL DCL

1. DQL 数据查询语言：
`select filed from table where *`


2. DML 数据操纵语言：
数据操纵语言DML主要有三种形式，这也是与各种语言交互使用的：
   1) 插入：INSERT
   2) 更新：UPDATE
   3) 删除：DELETE

3. DDL 数据定义语言
数据定义语言DDL用来管理数据库中的各种对象-----表、视图、索引、同义词、聚簇等。

```sql{.line-numbers}
create
alter
drop
-- 修改属性 
alter table 表名 modify 属性名 set 属性

-- 修改已有字段名及属性
alter table 表名 change old_字段 new_字段 set 属性

-- 删除字段
alter table 表名 drop 字段名
```

4. DCL 数据控制语言
数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
1) GRANT：授权。

2) ROLLBACK：回滚命令使数据库状态回到上次最后提交的状态。

3) COMMIT [WORK]：提交。


### 数据库分区

MySQL 可以在建表时最后可以创建表分区，通过 partition by 关键字创建。如下示例:
```sql{.line-numbers}
-- 注意，这种方式括号内必须是整数字段名或返回确定整数的函数
-- 如果想要定义非整型和多列范围，需要加入 columns，
-- 而且括号内必须是列名，不支持函数。
partition by range(字段名)

-- 这种方式支持非整型和多列
partition by range columns(a, b)
```

也可以在建表完成后创建分区，例如：
```sql{.line-numbers}
-- 创建分区
alter TABLE table_name add PARTITION(PARTITION partition_name VALUES LESS THAN (20180630) ENGINE = InnoDB
);
-- 删除分区
ALTER TABLE table_name DROP PARTITION partition_name；

```

MySQL 支持四种分区方式：RANGE、LIST、HASH、KEY


## MySQL数据库优化

阿里巴巴手册：SQL优化标准是到ref，最差是range，达到const最好。

### 数据库事务的四个特性
1. A 原子性
2. C 一致性
3. I 隔离性
4. D 持久性


### 事务的四种隔离级别：
隔离性中包含了四种隔离级别，分别是：
1. 读未提交内容 
2. 读已提交内容 脏读
3. 幻读（重复读） 读已提交
4. 可串行化 解决幻影读


### 数据库索引分类：
主键索引、唯一索引、普通索引（前缀索引）、复合索引

### 数据表结构优化：

1. 选择合适的数据类型
（1）使用可存下数据的最小的数据类型。
（2）使用简单地数据类型，int要比varchar类型在mysql处理上更简单。
（3）尽可能使用not null定义字段，这是由innodb的特性决定的，因为非not null的数据可能需要一些额外的字段进行存储，这样就会增加一些IO。可以对非null的字段设置一个默认值。
（4）尽量少用text，非用不可最好分表，将text字段存放到另一张表中，在需要的时候再使用联合查询，这样可提高查询主表的效率。

2. LIKE关键字匹配'%'开头的字符串,不会使用索引。含有 or 的查询子句，如果要利用索引，那么 or 之间的每一个字段都必须是索引字段。or 之间的任何一个字段没有索引的话所有涉及的索引都不会被利用。

### 表分区
MySQL5.1开始支持分区功能，在不同版本中查看当前数据库是否支持的命令也不相同。
|版本|命令|
|--|--|
|5.6以下|show variables like '%partition%';|支持会显示名称和值|
|5.6之后（包括5.6）|show plugins;|如果包含`partition`选项代表支持|
|8.0之后|只有InnoDB支持表分区||

### SQL 语句优化

索引失效：
全值匹配最喜欢，最左前缀规矩严。
带头大哥不能死，中间兄弟不能断。
索引列上少计算，范围之后全完蛋。
LIKE百分最右边，覆盖索引*全不见。
不等空值还有OR，你建索引也失效。
VAR引号要出现，SQL高级也不难。

建立索引口诀：
主键索引自动建，频繁查询索引现；
查询关联其他表，外键索引也要看；
频繁更新引不建，where条件用到算；
单键/组合选择难，高并发下组合建；
若要速度有体现，排序字段索引建；
统计、分组你咋看？索引索引建建建。



## 数据库锁

### 悲观锁、乐观锁

悲观锁(synchronized)解决超卖问题：启用事务的话，在调用处加锁。（不建议，影响系统性能，耗时长，降低用户体验）
使用乐观锁（数据库层面解决超卖问题）：使用数据库中定义的 `version` 字段及数据库中的 `事务` 去解决。


## 架构

## MySQL主从复制

1. 修改MySQL的配置文件：`vim /etc/my.cnf`
2. 分别在配置文件中加入如下配置

```bash{.line-numbers}
  # 两台及其的 server-id 不能一样
  mysql(master)
  server-id=1
  log-bin=mysql-bin
  log-slave-updates
  slave-skip-errors=all

  mysql(slave)
  server-id=2
  log-bin=mysql-bin
  log-slave-updates
  slave-skip-errors=all
```

登录主节点执行 `show master status` 命令。
登录从节点执行以下命令：
```bash
  change master to master_host='master主机地址',
  master_user='master用户名',
  master_password='master密码',
  master_log_file='log文件名',
  master_log_pos=上面命令的 position;
```
开启从节点:
start/stop slave

> `show slave status\G` 如果出现 1593 的报错，那么先关闭 MySQL 服务，然后删除 /var/lib/mysql/auto.cnf 文件，然后启动 MySQL 服务即可。

## 集群架构
基于主从复制发展而来，主从架构节点数据库只能实现冗余备份，不能进行操作，但是集群架构可以操作从库。

mycat相当于一个路由，把写的操作转发给写库，把读的操作分发给读库。

mycat不仅会根据 SQL 语句去分析检测此次操作是发给读库还是写库，还会根据是否具有事务属性区分是读还是写。

如果查询也开启了事务，那么查询也会走主库。

