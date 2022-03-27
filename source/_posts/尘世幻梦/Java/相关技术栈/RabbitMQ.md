---
title: RabbitMQ整理
categories:
  - Java
  - RabbitMQ
abbrlink: 2d31c306
---

RabbitMQ整理

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=false}-->

<!-- code_chunk_output -->

- [RabbitMq的学习整理](#rabbitmq的学习整理)
    - [RabbitMq 的常用命令](#rabbitmq-的常用命令)
    - [镜像队列：（使用最多）](#镜像队列使用最多)

<!-- /code_chunk_output -->

# RabbitMq的学习整理
### RabbitMq 的常用命令
```bash{.line-numbers}
rabbitmqctl list_users    # 查看用户列表
rabbitmqctl add_user admin 123456  #添加用户名和密码
rabbitmqctl set_permissions -p /admin".*" ".*" ".*" #修改权限
rabbitmqctl set_user_tags admin administrator  #添加用户角色

rabbitmq-server -detached  #守护模式启动（后台运行）
rabbitmqctl stop    # 停止服务
rabbitmqctl status  # 查看状态
```
重启 rabbitmq 服务
```bash{.line-numbers}
rabbitmqctl stop
rabbitmq-server restart
```
> 5672 端口用于客户端使用，15672 用于可视化网页控制，使用这个功能时需要开启插件 `rabbitmq-plugins enable rabbitmq_management`。

### 镜像队列：（使用最多）
```bash
# 策略说明
rabbitmqctl set_policy [-p <vhost>] [--priority <priority>] [--apply-to <apply-to>] <name> <pattern> <definition>
# 查看当前策略：
rabbitmqctl list_policies
# 删除队列：
rabbitmqctl clear_policy ha-all
```

1. -p vhost 可选参数，针对指定 vhost 下的 queue 进行设置
    name: policy 的名称
    pattern: queue 的匹配模式（正则表达式）
    definition: 镜像定义，包括三个部分ha-mode、ha-params、ha-sync-mode
2. ha-mode 指明镜像队列的模式，有效值 all/exactly/nodes
all 表示在集群所有的节点上进行镜像
exactly 表示在指定个数的节点上进行镜像，节点的个数由 ha-params 指定
nodes 表示在指定的节点上进行镜像，节点名通过 ha-params 指定

3. ha-params: ha-mode 模式需要用到的参数
ha-sync-mode: 进行队列中消息的同步方式，有效值为 automatic 和 manual
priority: 可选参数，policy 的优先级。