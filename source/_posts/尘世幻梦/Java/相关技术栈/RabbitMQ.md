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

- [RabbitMQ的常用命令](#rabbitmq的常用命令)
- [镜像队列：（使用最多）](#镜像队列使用最多)

<!-- /code_chunk_output -->

### RabbitMQ的常用命令

```bash{.line-numbers}
rabbitmqctl list_users    # 查看用户列表
rabbitmqctl add_user admin 123456  #添加用户名和密码
rabbitmqctl set_permissions -p /admin".*" ".*" ".*" #修改权限
rabbitmqctl set_user_tags admin administrator  #添加用户角色


#守护模式启动（后台运行）
rabbitmq-server -detached
# 停止服务
rabbitmqctl stop
# 查看状态
rabbitmqctl status 
# 重启 rabbitmq 服务
rabbitmq-server restart
```

启用插件的同时并启动服务：`rabbitmq-plugins enable rabbitmq_management`，这个插件是 RabbitMQ 自带的，便于我们查看系统的状态数据，启用这个插件之后才可以使用 `http://IP地址:15672` 访问网页管理页面，查看具体信息。默认登录的初始账号密码为 `guest`。

> **注意：** 在较新的版本中，默认的账号只能以本地 `localhost` 的方式访问，如果服务在远程启动，本地进行远程访问操作时登录可能会出现 `User can only log in via localhost` 的情况，解决办法就是新添加一个超级管理员账户，使用这个新添加的账户登录。
> 5672 用于客户端使用，15672 用于网页控制。

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
