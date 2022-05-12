---
title: MongoDB总结整理
categories:
  - Java
  - MongoDB
abbrlink: 1a01f215
---

MongoDB整理，5.x版本。与之前的版本有些许变化。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [MongoDB的三个默认库](#mongodb的三个默认库)
- [基础操作](#基础操作)
  - [1.库操作](#1库操作)
  - [2.集合操作](#2集合操作)
  - [3.文档操作](#3文档操作)
- [索引](#索引)
  - [1. 基本操作](#1-基本操作)
  - [2. 复合索引](#2-复合索引)
  - [3. 聚合查询（aggregate）](#3-聚合查询aggregate)
- [SpringBoot 整合 MongoDB](#springboot-整合-mongodb)
- [MongoDB架构](#mongodb架构)
  - [1. 主从复制（4.0 版本废弃，了解即可）](#1-主从复制40-版本废弃了解即可)
  - [2. 副本集](#2-副本集)
  - [3. 分片（sharding）](#3-分片sharding)
  - [4. 环境搭建](#4-环境搭建)

<!-- /code_chunk_output -->

## MongoDB的三个默认库

admin： root 数据库
local： 这个数据永远不会复制，可以用来存储限于本地单台服务器的任意集合。
config： 当 Mongo 用于分片设置时，config 数据库在内部使用，用于保存分片的相关信息。
> 这三个库一般不需要操作。

## 基础操作

### 1.库操作

```js{.line-numbers}
//  查看所有库
show databases; / show dbs;
//  切换/创建指定库(不存在即创建)
use 库名;
//  查看当前所在库
db
//  删除当前库
db.dropDatabase();
```

> 1.使用 use 创建新库时，如果库内没有集合数据时，使用 `show databases` 是看不到新建的库的，但是可以使用 `db` 查看当前所在库。
> 2.使用删除库的命令时，删除后使用 `db` 命令还是在已删除的库中的，但是该库确实已经删除，因为该库已经没有数据了。

### 2.集合操作

集合：就类似与 MySQL 数据库中的表的概念，一个集合对应一张表。在向不存在的集合插入数据时，会自动创建该集合并插入数据。

```js{.line-numbers}
// 创建库
use 库名;
// 查看集合
show collections; / show tables;
// 创建集合
db.createCollection("集合名称");
// 删除集合
db.集合名称.drop();
```

> 如果集合并不存在，使用文档操作插入数据时，会隐式的创建不存在的集合并插入该数据

### 3.文档操作

[官方文档](https://docs.mongodb.com/manual/reference/method/)

文档：就是一条条的数据，增删改查就是针对文档的。
每条文档在放入集合的时候，MongoDB会自动的维护一个 `_id`，当然，我们可以手动指定，该 id 不允许重复。

- 单条文档插入

```js{.line-numbers}
// 查询所有文档
db.集合名称.find();
// 插入单条文档
db.集合名称.insert({});
// 删除文档：第二个参数可选
db.集合名称.remove({}, {
        # 只删除一条
        justOne: true
    });

// 查询文档总条数：
db.user.count();
// 查询符合条件的文档条数：
db.user.find({age: 18}).count();
```

> 新版本使用 insert 也可以直接插入多条文档

- 多条插入

```js{.line-numbers}
db.集合名称.insertMany(
    [{}, {}],
    {
        // 写入策略，默认为1，要求确认写操作;0不要求
        writeConcern: 1,
        // 是否按顺序写入，默认true，按顺序插入
        ordered: true   
    }
);

db.集合名称.insert([
    {},
    {}
]);
```

- 脚本插入

```bash{.line-numbers}
for (let i = 1; i < 10; i++) {
    db.集合名称.insert({name: "小明" + i, age: i+10, bir: "2021-10-0" + i});
}
```

- 文档删除

```js{.line-numbers}
// 删除所有文档
db.集合名称.remove({});
// 删除指定条件的文档
db.集合名称.remove({条件});
```

- 文档更新：最后一个参数是可选的

```js{.line-numbers}
db.集合名称.update(
    {查询指定文档(类似 where)} 
    {目标更新值(set 后面的值)},
    {
        // 如果查不到更新的文档，是否将当前文档插入集合中，
        // 默认false不插入
        upsert: <boolean>,
        // 是否只更新查询到的第一条记录。默认false，只更新第一条记录
        multi: <boolean>,
        // 抛出异常的级别
        riteConcern: <document>
    }
);
```

```js{.line-numbers}
// 默认更新方式：先删除，后插入 结果不保留其它字段
db.user.update({name: "小明"}, {age: 19, likes: ["看书", "听音乐"]});
// 如果想在原始记录上直接更新：使用 $set
db.user.update({name: "小明"}, {$set:{age: 19, likes: ["吃饭", "睡觉"]}});
```

> 默认的更新方式是先删除再插入，如果想保留其它的字段，在原始的记录上进行修改，需要使用 `$set`，例如：`db.user.update({name:"小明"}, {$set:{name:"小黑"}})`。

- 文档查询

```js{.line-numbers}
// 查询所有文档(默认显示前 20 条数据)
db.集合名称.find();
// 查询文档数量
db.集合名称.count();
db.集合名称.find(条件).count();
// 格式化后展示数据
db.集合名称.find().pretty();
// 查询年龄大于20的
db.集合名称.find({age: {$gt: 20}});
// and 且查询：名字年龄同时满足
db.集合名称.find({name: '小明', age: 24});
// or 或查询：名字年龄有一个满足
db.集合名称.find({$or: [{nmae: "小明"}, {age: 20}]});
// $size 按照数组长度查询
db.集合名称.find({likes: {$size: 2}});

// 排序（1：升序；-1：降序）
// 按年龄升序排列输出
db.集合名称.find().sort({name: 1, age: 1});

// 模糊查询(使用正则实现，不能使用引号，使用 /)
db.集合名称.find({name: /小/});

// 分页查询 skip：起始条数 limit：每页条数
db.集合名称.find({}).skip(0).limit(2);

// 去重
db.集合名称.distinct('age');

// 查询时指定返回字段(1：返回，0;不返回)。0和1不能同时用
// 默认返回 _id
db.集合名称.find({}, {name: 1});

// $type 查询，只查询字段为指定类型的文档数据
db.集合名称.find({key: {$type: 2}});
db.集合名称.find({key: {$type: 'String'}});
```

|符号|含义|
|:--:|:--:|
|$lt | 小于|
|$gt | 大于|
|$lte| 小于等于|
|$gte| 大于等于|
|$eq | 等于|
|$ne | 不等于|

|类型|对应数字|说明|
|--|--|--|
|Double|1|
|String|2||
|Object|3||
|Array|4||
|Binary data|5||
|Undefined|6|已废弃|
|Object id|7||
|Boolean|8||
|Date|9||
|Null|10||
|Regular Expression|11||
|JavaScript|13||
|Symbol|14||
|JavaScript(with scope)|15||
|32-bit integer|16||
|Timestamp|17||
|64-bit integer|18||
|Min key|255||
|Max key|127||
> **注意：** And查询时，如果同一个条件出现多次，那么后面的值会覆盖前面的。例如 `db.user.find({name: '小明',name: '小红'});` 那么，会查找 name 为小红的数据。
> 使用正则表达式实现近似的模糊查询时，需要使用 `/` 作为正则匹配符。
> $type 查询的 undefined 类型在新版本中已经废弃。
> MoongoDB中数字类型默认按 Double 类型存储，不论小数或者整数。
> 查询时只返回指定的字段时，0和1不能同时使用。

## 索引

### 1. 基本操作

```js{.line-numbers}
// 创建索引(1:升序；-1:降序)：
db.集合名称.createIndex({name: 1});
// 创建复合索引(左前缀原则):
db.集合名称.createIndex({name: 1, age: -1})
// 查看所有索引：
db.集合名称.getIndexes();
// 查看索引大小(字节大小)：
db.集合名称.totalIndexSize();

// 创建升序索引：
db.集合名称.createIndex({name:1})
// 创建降序索引：
db.集合名称.ensu
// 创建索引指定索引名称：
db.集合名称.createIndex({name:1}, {name: "name_index"})
// 创建唯一索引：
db.集合名称.createIndex({name:1}, {unique: true})
// 创建指定时间过期的索引，单位：秒
db.集合名称.createIndex({name: 1}, {expireAfterSeconds: 20});

// 删除指定索引
db.集合名称.dropIndex("索引名称");
// 删除所有索引，不包含 _id
db.集合名称.dropIndexes();
```

### 2. 复合索引

一个索引的值由多个key共同维护的索引称为复合索引。
MongoDB的复合索引和关系型数据库一样，也满足最左匹配原则。

### 3. 聚合查询（aggregate）

主要用于处理数据。
|表达式|描述|
|--|--|
|$sum|计算总和|
|$avg|计算平均值|
|$min|获取集合中所有文档对应值的最小值|
|$max|获取集合中所有文档对应值的最大值|
|$push|将值加入一个数组中，不会判断是否只有重复的值|
|$addToSet|将值加入一个数组中，会判断是否有重复的值，若该值数组中已经存在，则不加入|
|$first|根据资源文档的排序获取第一个文档数据|
|$last|根据资源文档的排序获取最后一个文档数据|

举例：

```js{.line-numbers}
// _id 不是文档的id，而是聚合查询的标识符
db.test.aggregate([{$group: {_id: '$by_user', 'avg_by_user': {$avg: '$likes'}}}])
//聚合查询：查询喜欢该作者的人数的平均值、最大值、最小值。
// 即获取 liskes 列的平均值，最大值，最小值
db.test.aggregate([{$group: {_id: '$by_user', 'avg_by_user': {$avg: '$likes'}}}])
db.test.aggregate([{$group: {_id: '$by_user', 'max_by_user': {$max: '$likes'}}}])
db.test.aggregate([{$group: {_id: '$by_user', 'min_by_user': {$min: '$likes'}}}])
```

> **注意：** 在使用以上的聚合查询时，使用的 `_id` 不是文档默认维护的id，而是以哪个字段为标识符进行分组查询。上面的例子就是以 `by_user` 字段为标识符。类似MySQL中的 `order by xxx`。

## SpringBoot 整合 MongoDB

1. 引入依赖：

    ```xml{.line-numbers}
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
        <version>2.6.4</version>
    </dependency>
    ```

2. 配置 properties 或者 yaml
3. 编写程序

## MongoDB架构

### 1. 主从复制（4.0 版本废弃，了解即可）

备份、故障恢复、读扩展
生产环境中，如果使用的是老版本的 MongoDB，推荐的从节点不超过 12 个。

从节点：仅仅负责数据同步，冗余备份，不能进行自动故障转移。

启动主节点：
./mongod --port 27017 --dbpath /root/data/master --master --bind_ip 0.0.0.0 --oplogSize 100
启动多个从节点：
./mongod --port 27018 --dbpath /root/data/slave1 --slave --bind_ip 0.0.0.0 --source 192.168.10.1:27017 --only 100 --autoresync
./mongod --port 27019 --dbpath /root/data/slave2 --slave --bind_ip 0.0.0.0 --source 192.168.10.1:27017 --only 100

--source： 指定主库的位置
--only： 指定复制哪个库

默认的从节点也不能查看，开启从节点的查看权限： rs.slaveOK();

> yum install -y lrzsz 文件的上传和下载

### 2. 副本集

为了解决主从复制中自动转移的问题，尽管主从在新版本中已经废弃，MongoDB 又提出了新的结构，就是副本集。可以理解为带有自动故障转移的主从复制架构。推荐的副本集为奇数个。

副本集必须进行初始化。

```bash{.line-numbers}
mongod --port 27017 --dbpath ../repl/data1 --bind_ip 0.0.0.0 --replSet myreplace/[localhost:27018,localhost:27019]
mongod --port 27018 --dbpath ../repl/data2 --bind_ip 0.0.0.0 --replSet myreplace/[localhost:27017,localhost:27019]
mongod --port 27019 --dbpath ../repl/data3 --bind_ip 0.0.0.0 --replSet myreplace/[localhost:27017,localhost:27018]
```

> --replRet 副本集，myreplace 副本集名称/集群中其它节点的主机和端口。
> 对副本集进行查看时，如果不允许查询，使用 `rs.secondaryOk()`后再进行查询。
> 多个副本集组成的的名称必须一样
> 推荐使用本机IP地址，尽量不要使用 localhost

配置副本集，连接任意节点：use admin
初始化副本集：

```bash{.line-numbers}
# 初始化配置：如果错了可以多次修改，以最后一次为准
> var config = {
    _id:"myreplace",
    members:[
        # 副本集主机地址：端口
        # 推荐使用本机的IP地址，不要使用 localhost
        {_id:0, host: "localhost: 27017"},
        {_id:1, host: "localhost: 27018"},
        {_id:2, host: "localhost: 27019"},
    ]
}
# 调用初始化函数
> rs.initiate(config);
```

当集群超过半数以上的节点宕机时，集群无法对外提供服务。

### 3. 分片（sharding）

目的：解决单点压力问题（并发访问）

Shard：用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组成一个replica set承担，防止主机单点故障。
Config Server：mongod实例，存储了整个ClusterMetadata。
Query Routers：前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。
Shard Key：片键，设置分片时需要在集合中选择一个片键作为拆分数据的依据。片键的选取决定了数据散列是否均匀。

### 4. 环境搭建

MongoDB4.0版本之后要求Config Server必须以副本集方式部署，以保证系统的高可用。也建议每一个Shard也为一个副本集，目前没有强制要求必须为副本集。

创建三个分片，每一个分片创建一个副本集，一共是6个。再创建3个Config Server和一个Router Process，一共10个端口，9个数据目录。

1.启动s0 r0

```bash{.line-numbers}
# 分片
./mongod --port 27017 --dbpath ../cluster/shard/s0 --bind_ip 0.0.0.0 shardsvr --replSet r0/121.5.167.13:27018
# 对应副本集
./mongod --port 27018 --dbpath ../cluster/shard/s0-repl --bind_ip 0.0.0.0 shardsvr --replSet r0/121.5.167.13:27017

# 登录任意节点执行
use admin
# 编写配置
config = {_id: "r0", members:[
    {_id: 0, host: "121.5.167.13:27017"},
    {_id: 1, host: "121.5.167.13:27018"},
]}
# 初始化，在初始化之前的配置信息可以多次执行，以最后一次为准
rs.initiate(config);
```

2.启动s1 r1

```bash{.line-numbers}
./mongod --port 27019 --dbpath ../cluster/shard/s1 --bind_ip 0.0.0.0 shardsvr --replSet r1/121.5.167.13:27020

./mongod --port 27020 --dbpath ../cluster/shard/s1-repl --bind_ip 0.0.0.0 shardsvr --replSet r1/121.5.167.13:27019

# 登录任意节点执行
use admin
# 编写配置
config = {_id: "r1", members:[
    {_id: 0, host: "121.5.167.13:27019"},
    {_id: 1, host: "121.5.167.13:27020"},
]}
# 初始化
rs.initiate(config);
```

3.启动s2 r2

```bash{.line-numbers}
./mongod --port 27021 --dbpath ../cluster/shard/s2 --bind_ip 0.0.0.0 shardsvr --replSet r3/121.5.167.13:27022

./mongod --port 27022 --dbpath ../cluster/shard/s2-repl --bind_ip 0.0.0.0 shardsvr --replSet r2/121.5.167.13:27021

# 登录任意节点执行
use admin
# 编写配置
config = {_id: "r2", members:[
    {_id: 0, host: "121.5.167.13:27021"},
    {_id: 1, host: "121.5.167.13:27022"},
]}
# 初始化
rs.initiate(config);
```

4.启动3个config服务

```bash{.line-numbers}
./mongod --port 27023 --dbpath ../cluster/shard/config1 --bind_ip 0.0.0.0 --replSet config/[121.5.167.13:27024,121.5.167.13:27025] --configsvr

./mongod --port 27024 --dbpath ../cluster/shard/config2 --bind_ip 0.0.0.0 --replSet config/[121.5.167.13:27023,121.5.167.13:27025] --configsvr

./mongod --port 27025 --dbpath ../cluster/shard/config3 --bind_ip 0.0.0.0 --replSet config/[121.5.167.13:27023,121.5.167.13:27024] --configsvr

# 初始化config
use admin
# 编写配置
config = {_id: "config", configsvr: true, members:[
    {_id: 0, host: "121.5.167.13:27023"},
    {_id: 1, host: "121.5.167.13:27024"},
    {_id: 1, host: "121.5.167.13:27025"},
]}
# 初始化
rs.initiate(config);
```

5.启动路由服务

```bash{.line-numbers}
./mongod --port 27026 --configdb config/121.5.167.13:27023,121.5.167.13:27024,121.5.167.13:27025 --bind_ip 0.0.0.0
```

6.登录 mongos 服务

```bash{.line-numbers}
./mongo --port 27026
use admin
# 添加分片信息
db.runCommand({addshard: "121.5.167.13:27017", "allowLocal":true});
db.runCommand({addshard: "121.5.167.13:27019", "allowLocal":true});
db.runCommand({addshard: "121.5.167.13:27021", "allowLocal":true});
# 启用指定数据库的分片
db.runCommand({enablesharding: "xxx"});
# 设置库的片键信息 xxx库的xxxx集合，使用 _id 作为片键（不推荐）
db.runCommand({shardcollection: "xxx.xxxx", key: {_id: 1}});
# 使用 xxx库xxxx集合的 _id 的哈希作为散列（推荐）
db.runCommand({shardcollection: "xxx.xxxx", key: {_id: "hashed"}});
```
