---
title: Docker安装常用服务
music:
  server: netease
  type: song
  id: 373114
categories:
  - Java
  - docker安装常用服务
abbrlink: 4bb2ae3a
---

使用 Docker 安装常用服务，可以去 [Docker Hub](https://registry.hub.docker.com/) 上搜索镜像以及版本，确定好自己的目标镜像之后就可以拉取了。镜像后面带有 `offical image` 标签的都是官方镜像。其它的可能是用户自己打包上传的镜像。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=2 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->
# Docker安装常用服务

使用 `--restart=always` 参数可以在 Docker 服务启动时自动启动对应的docker容器。如果在 docker 容器已经启动后，想要设置容器跟随 docker 服务自启动，可以使用更新的命令：`docker update --restart=always 容器名称`，比如我想让MySQL容器自动启动，那么命令就是`docker update --restart=always mysql`，这样，容器名称为 `mysql` 的容器就会跟随Docker服务而自启动了。

### 1. MySQL

镜像拉取：`docker pull mysql:5.7.32`

##### 1.1 使用自定义的配置文件跟随 Docker 的启动自动启动 MySQL 容器:
```bash
docker run -d -p 3306:3306 --name mysql -v /root/docker/mysql:/etc/mysql/conf.d -v /root/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --restart=always mysql:5.7.32
```

##### 1.2 使用自定义的配置文件并且启动时创建一个数据库:
```bash
docker run -d -p 3306:3306 --name mysql -v /root/docker/mysql:/etc/mysql/conf.d -v /root/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -v MYSQL_DATABASE=数据库名称 mysql:5.7.32
```

> 导入备份数据到容器：docker cp /root/docker/test.sql mysql:/
> 进入 MySQL 的 bash 环境: docker exec -it mysql bash
> 登录数据库，并选择数据库加载数据： mysql -u root -p && use test
> 加载数据：source test.sql;


### 2. Redis

Redis默认开启的是快照模式(RDB)，可以开启AOF持久化(最多丢1s内数据):
##### 2.1 开启AOF持久化方式启动
```bash
docker run --name redis -p 6379:6379 -v /root/docker/redis:/data -d redis:版本号 redis-server --apaendonly yes
```

##### 2.2 自定义配置文件启动
下载官方安装包获取配置文件，然后放入宿主机系统中：`/root/redisconf/redis.conf`。修改 `redis.conf` 配置，启动容器时加载指定的配置文件。
```bash
docker run --name redis -p 6379:6379 -v /root/docker/redis:/data -v /root/docker/redis/conf/redis.conf:/etc/redis.conf -d redis:版本号 redis-server /etc/redis.conf
```

### 3. Nginx

##### 3.1 直接启动 部署 dist 
```bash
docker run -d --name nginx -p 80:80 -v /root/docker/nginx/dist:/usr/share/nginx/html:ro nginx:1.20 
```

##### 3.2 使用自定义配置文件启动:
```bash
docker run --name nginx-customer -d -p 80:80 -v /root/docker/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.20
```

> 注意：nginx的配置文件必须和版本一致。
> `ro` 代表只读(read only): 外部的改变能够影响内部，内部的改变不会影响外部。

复制容器内部的配置文件到宿主机：`docker cp nginx:/etc/nginx/nginx.conf /root/nginx.conf`

修改配置文件实现反向代理。
```xml{.line-numbers}
server {
    localhost / {
        proxy_pass http://nacos-servers/;
    }
}
```

### 4. RabbitMQ

##### 4.1 直接启动
```bash
docker run -d --name RabbitMQ -p 15672:15072 -p 5672:5672 rabbitmq:3.8-management
```
5672端口是Java进行通信的，rabbitmq:3.8-management 是带有管理界面的插件(默认的账号密码：guest/guest)，rabbitmq:3.8 是没有管理界面的。
如果在启动时指定用户名、密码可以加上以下内容。
```bash{.line-numbers}
-e RABBITMQ_DEFAULT_VHOST=/ems
-e RABBITMQ_DEFAULT_USER=root
-e RABBITMQ_DEFAULT_PASS=root
```

##### 4.2 使用自定义配置信息启动
```bash
docker run -d --name RabbitMQ -p 15672:15072 -p 5672:5672 -v /root/docker/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf rabbitmq:3.8-management
```

### 7. MongoDB

`docker run --name mongo -d -p 27017:27017  mongo:5.0.5`

### 6. ES

##### 6.1 直接启动
```bash
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:7.0
```

> 直接启动 ES 容器会报错，解决方案：
> 1. 编辑宿主机 vim /etc/sysctl.conf 加入 `vm.max_map_count=262144`，保存退出
> 2. sysctl -p

##### 6.2 持久化数据到宿主机方式启动
```bash{.line-numbers}
docker run -d --name es -p 9200:9200 -p 9300:9300 -v /root/docker/es/data:/usr/share/elasticsearch/data elasticsearch:7.0
```

##### 6.3使用自定义配置启动
```bash
docker run -d --name es -p 9200:9200 -p 9300:9300 -v /root/docker/es/data:/usr/share/elasticsearch/data -v /root/docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:7.0
```

##### 6.4 加载 IK 分词器启动
```bash
docker run -d --name es -p 9200:9200 -p 9300:9300 -v /root/docker/es/data:/usr/share/elasticsearch/data -v /root/docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /root/docker/es/ik:/usr/share/elasticsearch/plugins/ik elasticsearch:7.0
```

### 7. Kibana
```bash
# 直接启动
docker run -d --name kibana -p 5601:5601 -e ELASTICSEARCH_URL=http://es的IP地址:9200 Kibana:7.0

# 使用自定义配置文件启动
docker run -d --name kibana -p 5601:5601 -v /root/docker/kibana.yml:/usr/share/kibana/config/kibana.yml Kibana:7.0
```