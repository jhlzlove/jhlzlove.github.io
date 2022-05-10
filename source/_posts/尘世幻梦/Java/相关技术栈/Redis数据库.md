---
title: Redis数据库
music:
  server: netease
  type: song
  id: 430793672
categories:
  - Java
  - Redis
abbrlink: eb52a7b2
---

Redis 数据库整理

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [Redis](#redis)
  - [Redis 基础](#redis-基础)
  - [Redis 的几种模式](#redis-的几种模式)

<!-- /code_chunk_output -->

# Redis

## Redis 基础
1. Redis的数据结构
   String, list, set, Sorted Set, hash
2. Redis 的持久化方式
   将内存中的数据异步写入硬盘中，两种方式：RDB和AOF
   RDB持久化原理：通过 bgsave 命令触发，然后父进程执行fork操作创建子进程，子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换(定时一次性将所有数据进行快照生成一份副本存储在硬盘中)
   **优点：** 是一个紧凑压缩的二进制文件，Redis加载RDB恢复数据远远快于AOF的方式。
   **缺点：** 由于每次生成RDB开销较大，非实时持久化，

   AOF持久化原理：开启后，Redis每执行一个修改数据的命令，都会把这个命令添加到AOF文件中。
   **优点：** 实时持久化。
   **缺点：** 所以AOF文件体积逐渐变大，需要定期执行重写操作来降低文件体积，加载慢.
3. 内存淘汰策略
   Redis 的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。

4. 缓存雪崩的解决方案：
   原因：一次性加入缓存的数据过多，导致内存过高，从而影响内存的使用导致宕机。
   解决方法：
   (1)redis集群，通过集群的方式将数据放置。
   (2)后端服务降级和限流：当一个接口请求过多，那么就会添加过多数据，可以对服务进行限流，限制访问的数量，这样就可以减少问题的出现。

Redis 有 16384 个 slot，存储的时候会是使用hash均匀的散列到这些 slot 上。

## Redis 的几种模式
Redis：
1. 主从模式
2. 哨兵模式(建立在主从模式上)
3. 集群模式

一台主机搭建时，复制多个配置文件，配置不同端口启动即可；

哨兵的作用，就是监控主、从数据库的状态，当主数据库挂掉以后，哨兵会在一定的时间内去判断，然后在从数据库中选举出一个去顶替主数据库，从而实现redis数据的高可用。


redis 主从复制，从节点默认是只读的，当 master 服务挂掉之后，从节点不能代替主节点；主从复制架构只是一个数据的备份。

集群：CRC16算法
特点:
1. 对集群模式下的所有key进行CRC16计算，计算的结果始终在0-16383之间
2. 对客户端的key进行CRC16算法计算时，同一个key经过多次计算，计算结果始终一致。
3. 对客户端的不同的key进行CRC16计算，会出现不同的key计算结果一致。

Q: 什么时候重新分配 slot？
A: 往集群中添加新的物理节点时会重新分配


搭建 redis 集群实现 session 共享：
修改不同端口的配置文件：
```bash{.line-numbers}
# 启动端口号
port 7000
# 允许远程连接
bind 0.0.0.0
# rdb 方式持久化数据的文件名
dbfilename dump-7000.rdb
# 开启守护进程(即后台运行)
daemonize yes
# 开启 aof 缓存并指定 aof 方式持久化数据的文件名
appendonly yes
appendonlyfilename "appendonly-7000.aof"
# 开启集群配置
cluster-enabled yes
# 配置集群节点名称
cluster-config-file nodes-7000.conf
# 集群超时时间 5s
cluster-node-timeout 5000
```

配置主从数据库后使用 redis-cli 进行连接后，使用 `info replication` 命令查看当前数据库的 role，“master” 是主节点，“slave” 是从节点。默认从库是只读的，不能进行写操作。