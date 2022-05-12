---
title: MySQL整理总结中……
music:
  server: netease
  type: song
  id: 373114
categories:
  - Java
  - MySQL
abbrlink: 9e299c6e
---

MySQL数据库整理

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true}-->

<!-- code_chunk_output -->

- [MySQL 安装（win）](#mysql-安装win)
  - [1. Mysql5.7 版本安装](#1-mysql57-版本安装)
  - [2. MySQL8.0安装](#2-mysql80安装)
- [MySQL基础](#mysql基础)
  - [1. 认识DQL DML DDL DCL](#1-认识dql-dml-ddl-dcl)
  - [2. 数据库索引分类](#2-数据库索引分类)
  - [3. 数据库事务的四个特性](#3-数据库事务的四个特性)
  - [4. 事务的四种隔离级别](#4-事务的四种隔离级别)
  - [5. 数据库分区](#5-数据库分区)
- [MySQL优化](#mysql优化)
  - [1. 数据表结构优化](#1-数据表结构优化)
  - [2. 表分区](#2-表分区)
  - [3. MySQL语句优化口诀](#3-mysql语句优化口诀)
- [数据库锁](#数据库锁)
  - [1. 悲观锁、乐观锁](#1-悲观锁乐观锁)
  - [2. 死锁的原因](#2-死锁的原因)
    - [产生死锁的四个必要条件](#产生死锁的四个必要条件)
    - [死锁的解除](#死锁的解除)
- [MySQL架构](#mysql架构)
  - [1. MySQL主从复制](#1-mysql主从复制)
  - [2. 集群架构](#2-集群架构)
- [MySQL进阶操作](#mysql进阶操作)

<!-- /code_chunk_output -->

## MySQL 安装（win）

官网下载安装版或者解压版，安装版一般不会有什么错误，环境变量如果安装的时候勾选加入 Path 中也不用配置，比较简单。这里介绍解压版的安装使用。

### 1. Mysql5.7 版本安装

下载压缩包解压，设置好环境变量 MYSQL_HOME=解压目录，在path加上`%MY_HOME%\bin`，然后执行以下三条命令：

```bash{.line-numbers}
mysqld -install
mysqld --initialize-insecure --user=mysql
net start mysql
```

```bash{.line-numbers}
# 安装 MySQL 服务
mysqld -install
# 移除 MySQL 服务
mysqld -remove
# 初始化，建立data文件
mysqld --initialize
```

### 2. MySQL8.0安装

基本和上面一样。

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

如果连接sqlyog报错，进入到 MySQL 的环境(mysql -u root -p)，然后执行命令：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

> Java 程序中连接 8.0 以上的数据库时区设置：`serverTimezone=Asia/Shanghai`
> UTC 时间和中国时间相差 8 小时。

## MySQL基础

### 1. 认识DQL DML DDL DCL

1. DQL 数据查询语言：select
2. DML 数据操纵语言：insert update delete
3. DDL 数据定义语言：create alter drop
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
    3) COMMIT：提交。

### 2. 数据库索引分类

1. 主键索引
2. 唯一索引
3. 普通索引（前缀索引）
4. 复合索引

### 3. 数据库事务的四个特性

1. A 原子性
2. C 一致性
3. I 隔离性
4. D 持久性

### 4. 事务的四种隔离级别

1. 读未提交内容
2. 读已提交内容 脏读
3. 幻读（重复读） 读已提交
4. 可串行化 解决幻影读

### 5. 数据库分区

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

## MySQL优化

### 1. 数据表结构优化

1. 选择合适的数据类型
（1）使用可存下数据的最小的数据类型。
（2）使用简单地数据类型，int要比varchar类型在mysql处理上更简单。
（3）尽可能使用not null定义字段，这是由innodb的特性决定的，因为非not null的数据可能需要一些额外的字段进行存储，这样就会增加一些IO。可以对非null的字段设置一个默认值。
（4）尽量少用text，非用不可最好分表，将text字段存放到另一张表中，在需要的时候再使用联合查询，这样可提高查询主表的效率。

2. LIKE关键字匹配'%'开头的字符串,不会使用索引。含有 or 的查询子句，如果要利用索引，那么 or 之间的每一个字段都必须是索引字段。or 之间的任何一个字段没有索引的话所有涉及的索引都不会被利用。

### 2. 表分区

MySQL5.1开始支持分区功能，在不同版本中查看当前数据库是否支持的命令也不相同。
|版本|命令|
|--|--|
|5.6以下|show variables like '%partition%';|支持会显示名称和值|
|5.6之后（包括5.6）|show plugins;|如果包含`partition`选项代表支持|
|8.0之后|只有InnoDB支持表分区||

### 3. MySQL语句优化口诀

阿里巴巴手册：SQL优化标准是到ref，最差是range，达到const最好。
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

### 1. 悲观锁、乐观锁

悲观锁：使用系统层面的加锁方式
乐观锁：数据库层面的锁，通过增加 `version` 字段以及使用数据库的 **事务** 的方式加锁

可以解决超卖等问题，一般使用乐观锁，乐观锁的性能相较于悲观锁要好得多，这样在不影响用户的体验下保证了应用的性能。

### 2. 死锁的原因

死锁是由于两个或以上的线程互相持有对方需要的资源，导致这些线程处于等待状态，无法执行。

#### 产生死锁的四个必要条件

1.互斥性：线程对资源的占有是排他性的，一个资源只能被一个线程占有，直到释放。

2.请求和保持条件：一个线程对请求被占有资源发生阻塞时，对已经获得的资源不释放。

3.不剥夺：一个线程在释放资源之前，其他的线程无法剥夺占用。

4.循环等待：发生死锁时，线程进入死循环，永久阻塞。

#### 死锁的解除

1.抢占资源，从一个或多个进程中抢占足够数量的资源，分配给死锁进程，以解除死锁状态。

2.终止（或撤销）进程，终止（或撤销）系统中的一个或多个死锁进程，直至打破循环环路，使系统从死锁状态解脱出来.

## MySQL架构

### 1. MySQL主从复制

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

开启从节点: `start/stop slave`

> `show slave status\G` 如果出现 1593 的报错，那么先关闭 MySQL 服务，然后删除 /var/lib/mysql/auto.cnf 文件，然后启动 MySQL 服务即可。

### 2. 集群架构

基于主从复制发展而来，主从架构节点数据库只能实现冗余备份，不能进行操作，但是集群架构可以操作从库。可以使用阿里开源的 mycat 作为 MySQL 的统一管理中心。

mycat相当于一个路由，把写的操作转发给写库，把读的操作分发给读库。

mycat不仅会根据 SQL 语句去分析检测此次操作是发给读库还是写库，还会根据是否具有事务属性区分是读还是写。

如果查询也开启了事务，那么查询也会走主库。

## MySQL进阶操作

查看MySQL的binlog命令
/usr/bin/mysqlbinlog --no-defaults -v --base64-output=decode-rows binlog.000001 > nov3.sql

根据binlog的内容恢复数据
/usr/bin/mysqlbinlog binlog.000007 | mysql -uroot -p -v -f > /opt/1.txt
然后输入密码开始恢复数据，
-f是为了跳过执行错误，
-v是展开执行的详细信息
如果文件中存在不能执行的指令，可以按照时间进行执行比如：
加上时间段参数 `--start-datetime=‘2022-01-06 14:14:37’ --stop-datetime=‘2022-01-24 13:04:44’` 恢复指定时间段的数据。
