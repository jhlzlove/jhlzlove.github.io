---
title: ZooKeeper 总结
categories:
  - Java
  - ZooKeeper
music:
  server: netease
  type: song
  id: 373114
abbrlink: 6fff97ea
---
Zookeeoer 总结

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [1 Zookeeper的结构认识](#1-zookeeper的结构认识)
  - [1.1 节点](#11-节点)
    - [1. 特点](#1-特点)
    - [2. 节点监听机制](#2-节点监听机制)
- [2 ZK的配置文件](#2-zk的配置文件)
- [3 客户端基本指令](#3-客户端基本指令)
- [4 ZK伪集群搭建示例](#4-zk伪集群搭建示例)

<!-- /code_chunk_output -->

Zookeeper 简称 ZK，是基于 Java 语言编写的开放源码的分布式应用程序协调服务，可以使用 jps 命令查询相关进程。

ZooKeeper主要服务于分布式系统：统一配置管理、统一命名服务、分布式锁、集群管理。

- dubbo、springcloud 框架使用 ZK 作为注册中心；
- Hadoop、Hbase组件使用 ZK 作为集群管理者；
- redis 可以使用 ZK 实现分布式锁。

## 1 Zookeeper的结构认识

ZK 的内存结构和 Linux 文件系统非常相似。

ZK中的节点称为 ZNode，节点模型的特点：

- 每个子目录都被称为一个节点，这个节点是它所在路径的唯一标识。
- 节点可以有子节点目录，每个子节点都可以存储数据。
- 节点是有版本的，每个子节点中的数据都可以有多个版本，也就是一个访问路径可以存储多份数据。
- 节点可以被监控，包括这个目录节点中存储的数据的修改，节点目录的变化等，一旦变化可以通知监听的客户端。

### 1.1 节点

ZK 节点分类：

- 持久性节点：PERSISTENT
- 持久性顺序节点：PERSISTENT_SEQUENTIAL
- 临时节点：EPHEMERAL
- 临时顺序节点：EPHEMERAL_SEQUENTIAL

#### 1. 特点

- 临时/临时顺序节点上不可以包含任何的子节点。

- ZK 默认在根路径中有一个 zookeeper 节点，zookeeper 节点下有一个 quota 子节点。

#### 2. 节点监听机制

一个 Watch 事件是一个一次性的触发器，当被设置了 Watch 的数据和目录发生了改变的时候，则服务器将这个改变发送给设置了 Watch 的客户端以便通知他们。节点的监听是一次性的，监听一次就会失效。

```bash{.line-numbers}
# 监听路径变化
ls /node true

# 监听数据变化
get /node true
```

## 2 ZK的配置文件

```bash{.line-numbers}
# 集群节点之间的心跳时间
tickTime=2000
# 初始化集群时集群节点的同步超时时间，10代表 10 个滴答声，20秒
initLimit=10
# 同步时间限制 5 个滴答声
syncLimit=5
# 持久化数据存放目录
dataDir=/tmp/zookeeper
# ZK 服务监听端口号
clientPort=2181
```

## 3 客户端基本指令

指定配置文件启动：`./bin/zkServer.sh start ./conf/zoo.cfg`
客户端连接：`./bin/zkCli.sh -server localhost:2181`

```bash{.line-numbers}
# 查看路径节点
ls /node

# 创建一个持久顺序节点：
create -s /node jhlz

# 创建一个临时数据节点
create -e /node jhlz

# 创建一个临时顺序节点
create -e -s /node jhlz

# 查看节点的状态
stat /node

# 查看节点数据和状态
get /node

# 设置节点的值
set /node xiaoming

# 组合 相当于 ls / + stat /
ls2 /node

# 删除节点，只能删除没有子节点的节点
delete /node

# 递归删除节点
rmr /node

# 退出当前会话
quit
```

> 创建节点时，默认创建的是持久化节点。create /node jhlz，这样就创建了一个 jhlz 的持久化节点

## 4 ZK伪集群搭建示例

集群的搭建和其它集群的搭建都是一样的，非常简单：创建不同的配置文件，在服务启动时指明需要的配置文件即可。

1. 首先建立三个文件夹： zk1 zk2 zk3
在每一个数据目录中创建一个 myid 文件。并在每一个 myid 文件中添加标号。

    ```bash{.line-numbers}
    mkdir zk1 zk2 zk3

    touch zk1/myid /zk2/myid zk3/myid

    echo "1" > zk1/myid
    echo "2" > zk2/myid
    echo "3" > zk3/myid
    ```

2. 修改各自的配置

    ```bash{.line-numbers}
    # 编辑第一个
    vim zk1/zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/tmp/zk1
    clientPort=3001
    server.1=localhost:3002:3003
    server.2=localhost:4002:4003
    server.3=localhost:5002:5003
    # 编辑第二个
    vim zk1/zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/tmp/zk1
    clientPort=4001
    server.1=localhost:3002:3003
    server.2=localhost:4002:4003
    server.3=localhost:5002:5003
    # 编辑第三个
    vim zk3/zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/tmp/zk1
    clientPort=5001
    server.1=localhost:3002:3003
    server.2=localhost:4002:4003
    server.3=localhost:5002:5003
    ```

    > 其中，server.1 的区分就是上面创建的 myid 文件的编号，ZK 会自动根据 myid 的编号去识别，但是我们在集群配置中需要写明。
    > 3002: 数据同步使用的端口号。
    > 3003： leader 选举使用的端口号。

3. 指定配置文件启动服务

    ```bash{.line-numbers}
    ./bin/zkServer.sh start zk1/zoo.cfg

    ./bin/zkServer.sh start zk2/zoo.cfg

    ./bin/zkServer.sh start zk3/zoo.cfg
    ```
