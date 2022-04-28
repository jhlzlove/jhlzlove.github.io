---
title: Docker入门
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/docker.jpeg
music:
  server: netease
  type: song
  id: 373114
categories:
  - Java
  - docker
abbrlink: 8699d8dc
---
Docker入门

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->

<!-- code_chunk_output -->

- [Docker入门](#docker入门)
    - [为什么使用 Docker](#为什么使用-docker)
  - [Docker入门体验](#docker入门体验)
    - [Docker 配置下载镜像加速](#docker-配置下载镜像加速)
    - [Windows Docker-Desktop 启动 ElasticSearch 失败](#windows-docker-desktop-启动-elasticsearch-失败)
    - [docker常用命令](#docker常用命令)
  - [Docker进一步理解](#docker进一步理解)
    - [Docker为什么提供网络功能？](#docker为什么提供网络功能)
    - [docker数据卷（volume）](#docker数据卷volume)
  - [dockerfile](#dockerfile)
  - [docker-compose](#docker-compose)
    - [portainer 可视化工具](#portainer-可视化工具)

<!-- /code_chunk_output -->

# Docker入门

### 为什么使用 Docker
一个镜像可以去创建多个容器，各个容器之间互不干扰。

## Docker入门体验

为了避免使用普通用户运行 docker 的相关命令时出现报错，我们可以在docker命令前加上sudo去运行，但是每次都加显然很麻烦。那么在安装完docker后，可以运行以下命令：
```bash{.line-numbers}
# 创建 docker 用户组
sudo groupadd docker
# 将普通用户加入 docker 组中
sudo gpasswd -a $USER docker
# 更新 docker 组
newgrp docker
# 测试命令
docker ps
```

### Docker 配置下载镜像加速
为 docker 设置下载国内镜像源，提高镜像拉取的速度和上手效率。执行命令：`dockerd --registry-mirror=https://registry.docker-cn.com`

Docker配置下载镜像加速(vim /etc/docker/daemon.json)，设置完毕重启docker。
1、Docker官方的中国镜像加速器：https://registry.docker-cn.com  **不用注册**
2、中科大的镜像加速器：https://docker.mirrors.ustc.edu.cn/   **不用注册（推荐）**
3、阿里云的镜像加速器：登录阿里云的容器hub服务，镜像加速器那一栏里会为你独立分配一个加速器地址 **要注册**
4、DaoCloud的镜像加速器：登录DaoCloud的加速器获取脚本，该脚本可以将加速器添加到守护进程的配置文件中 **要注册**

### Windows Docker-Desktop 启动 ElasticSearch 失败
在Windows下，使用 Docker 启动 ES 服务，如果遇到内存不足的问题。调整的方式是通过命令行的 `wsl` 命令进入 docker-desktop 的终端，然后通过 `sysctl` 命令调整系统参数。
```bash
# 使用 cmd 进入 docker
wsl -d docker-desktop
# 调整相应的参数，docker 启动 ES 服务时会有打印，修改为指定的即可
sysctl -w vm.max_map_count=262144
```
也可以编辑文件 `vi /etc/sysctl.conf` 加入 `vm.max_map_count=262144`，保存退出执行 `sysctl -p` 后重启 es 服务即可。

> **`Q:`docker 拉取的镜像为什么比我们直接下载的文件体积大？**
`A:`一个镜像不仅仅是原来的软件包，它还包含了软件包运行所需的操作系统依赖、软件自身依赖等。所以随着我们的使用，使用的镜像越多，新的镜像下载会越来越快，因为有些依赖已经存在，后续的镜像如果对存在的依赖有使用的话，它会复用已经存在的依赖，而不会去再次下载。

### docker常用命令
```bash{.line-numbers}
# 镜像操作
docker info         # docker信息
docker images -a    # 列出本地所有的镜像
docker images --digests # 显示镜像的摘要信息
docker images --no-trunc    # 显示完整的镜像信息

docker search 镜像名称         # 根据镜像名称查找镜像
docker pull 镜像名称:版本号       # 拉取镜像，注意对应的版本号
docker load -i 镜像          # 加载自己下载的tar包到docker本地仓库
docker rmi 名称或者id        # 删除镜像，-f 选项强制删除

# 容器操作
docker ps   # 列出当前正在运行的容器
docker ps -a    # 列出所有的容器
docker ps -q    # 列出所有的容器ID
docker stop 容器名称或容器ID    # 停止容器运行
docker start 容器名称或容器ID     # 启动容器
docker restart 容器名称或容器ID     # 重启容器
docker rm 容器名称或容器ID    # 删除容器，-f 选项强制删除

docker logs 容器名称或ID    # 查看容器服务运行日志
docker logs -f 容器名称或ID # 实时监听服务运行日志
docker logs -t 容器名称或ID # 为服务运行日志加入时间戳

# 进入容器
docker exec -it 容器名称或ID bash

# 查看容器内部信息
docker inspect 容器名称或ID

# 将容器打包成一个新的镜像
docker commit -m "描述信息" -a "作者信息" 容器名称或者ID 打包的镜像名称:标签
# 将镜像备份,备份为 .tar 文件
docker save 镜像名称:标签 -o 文件名
# 退出
exit
```

常用的命令就这些，接下来以MySQL为例来使用docker，请按以下命令敲一边，体会一下docker。使用之前确定网络连接良好。
```bash{.line-numbers}
# 查找docker hub中是否有MySQL镜像
docker search mysql

# 拉取MySQL8.0版本
docker pull mysql:8.0

# 查看拉取的镜像
docker images -a

# 创建并运行容器
docker run -d -p 3306:3306 --name mysql-dev -v /opt/docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0
# 创建容器时，最后mysql:8.0表示mysql镜像的版本，可以写，表示指定该版本；
# 如果不写也可以，docker会自动在本地检测有没有最新的，如果没有会自动去dockerhub上去下载。

#    run              运行一个docker容器
#    --name           生成的容器的名字 mysql-dev
#    -p 3306:3306     代表端口映射，格式为 宿主机映射端口:容器运行端口；表示这个容器中使用3306（第二个）映射到本机的端口号也为3306（第一个） 这样外部的 mysql 可视化工具(例如navicat)可以通过 3306 端口访问容器mysql
#    -e               代表添加环境变量 MYSQL_ROOT_PASSWORD=123456 初始化root用户的密码
#    -d               表示使用守护进程运行，即服务挂在后台运行
    
# 查看容器
docker ps -a

# 查看容器信息
docker inspect mysql-dev

# 进入容器
docker exec -it mysql-dev bash

# 进入 mysql 命令行
mysql -u root -p

# 退出 bash，退出容器
exit

# 查看正在运行的容器
docker ps

# 停止容器
docker stop mysql-dev

# 再次查看正在运行的容器
docker ps

# 查看本地所有的容器
docker ps -a

# 删除容器
docker rm mysql-dev

# 再次查看本地所有的容器
docker ps -a

# 删除镜像
docker rmi mysql:8.0

# 查看镜像
docker images
```

## Docker进一步理解

### Docker为什么提供网络功能？

Docker允许通过外部访问容器或容器互联的方式来提供网络服务，方便了不同容器间进行通信。一般在使用docker网桥(bridge)实现容器与容器通信时，都是站在一个应用角度进行容器通信。

```bash{.line-numbers}
# 查看网桥配置
docker network ls
# 创建网桥
docker network create 自定义网桥名称
# 删除网桥
docker network rm 网桥名称或ID
# 查看网桥详情
docker inspect 网桥名称或ID
# 运行容器时使用 --network 网桥名称 指定该容器在那个网桥段
docker run -d --name mysql -p 3306:3306 --network ems mysql:8.0
```
**注意：** 如果这个网桥不存在，那么运行容器时指定网桥会导致容器运行失败，不会自动创建网桥。必须先创建网桥，再指定。
一旦在容器启动时指定了网桥后，日后可以在任何这个网桥关联的容器中使用容器名字与其它容器通信。
比如：
```bash
docker run -d --name tomcat01 -p 8081:8080 --network ems tomcat:8.0-jre8
docker run -d --name tomcat02 -p 8082:8080 --network ems tomcat:8.0-jre8
# 之后可以通过名称去访问tomcat02的主页信息
curl http://tomcat02:8082
```

### docker数据卷（volume）
实现容器与宿主机之间数据共享。数据卷就是上面创建mysql时的 `-v` 指定的映射目录，可以把容器内的数据持久化，这样删除容器时我们不必担心数据丢失。但是注意，这样做外部的改变和容器内部的改变会互相影响。

如果我们希望外部影响内部，但是内部操作不影响外部的话，在创建容器时我们可以这样指定： `-v /usr/mysql/data:/var/lib/mysql:ro`，其中 `ro` 代表只读(read only).

前面的映射目录我们也可以使用数据卷简称，这个简称可以随便起名，例如：`-v aa:/var/lib/mysql`，这样docker会自动帮我们创建aa，可以使用  `docker volume inspect aa`查看它的路径。

**数据卷的特性：**
1. 数据卷的修改会立即影响到容器；
2. 对数据的更新修改，不会影响镜像；
3. 数据卷默认一直存在，即使容器被删除

```bash{.line-numbers}
# 查看数据卷
docker volume ls
# 查看数据卷细节
docker volume inspect 卷名
# 创建数据卷
docker volume create 卷名
# 删除数据卷
docker volume rm 卷名
# 删除没有使用的数据卷，删除前需要我们手动确认
docker volume prune
```


## dockerfile

{% link 官方文档::https://docs.docker.com/engine/reference/builder/ %}

使用 Docker 中的docker image build命令会读取 Dockerfile，并将应用程序容器化。

Dockerfile 由一行行命令语句组成，并支持以 # 开头的注释行。

使用 `-t` 参数为镜像打标签，使用 `-f` 参数指定 Dockerfile 的路径和名称，使用 `-f` 参数可以指定位于任意路径下的任意名称的 Dockerfile。

构建上下文是指应用文件存放的位置，可能是本地 Docker 主机上的一个目录或一个远程的 Git 库。

FROM 指令用于指定要构建的镜像的基础镜像。它通常是 Dockerfile 中的第一条指令。

RUN 指令用于在镜像中执行命令，这会创建新的镜像层。每个 RUN 指令创建一个新的镜像层。

COPY 指令用于将文件作为一个新的层添加到镜像中。通常使用 COPY 指令将应用代码赋值到镜像中。

EXPOSE 指令用于记录应用所使用的网络端口。

ENTRYPOINT 指令用于指定镜像以容器方式启动后默认运行的程序。

Dockerfile文件内容示例(dockerfile的命令必须使用大写)
```dockerfile{.line-numbers}
FROM 目标镜像:版本号 # 必须写在第一行
WORKDIR /data # 指定目录后，下面的命令都是基于此目录进行的
WORKDIR ss
# 设置目录变量
ENV BASE_DIR /etc/
ENTRYPOINT cat $BASE_DIR/profile
```

然后执行 `docker build -t 自定义名称:版本号 .` 来创建一个自定义的镜像，注意最后的 ` . ` 不要漏掉。

使用 dockerfile 创建 Java 镜像在容器运行，首先创建一个空的文件夹，在空文件夹中创建 DockerFile 文件，编辑：
```dockerfile{.line-numbers}
FROM openjdk:8-jre
# 指定目录后，下面的命令都是基于此目录进行的
WORKDIR /application
# 基于 WORKDIR 的路径,/application/aa
WORKDIR aa
# 该文件上传后，直接改名为 app.jar
ADD app-0.0.1-SNAPSHOT.jar app.jar
# 暴露的端口
EXPOSE 8081
# 运行时执行的命令
ENTRYPOINT java -jar app.jar
```
执行构建：`docker build -t demo:1.0 .`
运行容器：`docker run -d -p 8081:8081 --name demo demo:1.0`


## docker-compose

docker-compose 学起来也是非常的简单，和 Dockerfile 差不多，写一个文件，然后使用命令去启动这个文件。
和 docker 的不同：docker是面向容器的，而 docker-compose 是面向服务的，这也是两者指令的根本。

如果时Window系统，安装 Docker for Window 后就有对应的 dcoker-compose工具了；如果是 Linux 系统，那么需要安装 docker-compose。
{% link 官网安装 docker-compose 示例::https://docs.docker.com/compose/install/ %}

**docker-compose.yml 模板：**
```yaml{.line-numbers}
version: "3.8" # 4.0以下
services: 
    tomcat: # 唯一服务名称
        image: tomcat:8.0-jre8 # 使用哪个镜像创建
        # 用来指定 dockerfile 所在目录。
        # 和 image 的不同：先根据该文件构建镜像，之后再运行容器
        build: 
            # dockerfile 文件所在的目录
            context: demo
            dockerfile: Dockerfile
        container_name: tomcat # 指定容器名称
        ports: # 宿主机与容器的的端口映射
            - "8080:8081"
        volumes: # 宿主机与容器的目录映射
            #- /root/data:/usr/local/tomcat/
            # 自定义映射
            - tomcatwebapps:/usr/local/tomcat/webapps
        networks: # 指定容器启动后使用的哪个网桥
            - hello
    mysql: 
        image: mysql:8.0
        container_name: mysql
        ports: 
            - "3306:3306"
        volumes: 
            - /root/mysql/data:/var/lib/mysql
            - /root/mysql/config:/etc/mysql
        enviroment:
            - MYSQL_ROOT_PASSWORD=root
        # 可以把 environment 的内容写到配置文件中
        env_file: # 配置文件必须以 .env 结束
            - mysql.env
        # 代表这个容器启动时依赖哪些模块，服务名称,不是容器名称
        depends_on: 
            - tomcat
            - redis
        # 用来修改容器内部参数；因为如果不修改的话容器可能无法启动
        sysctls: 
            - net.core.somaxconn=1024
            - net.ipv4.tcp.syncookies=0
        # 修改容器中系统内部进程数限制；根据当前容器运行服务要求更改
        ulimits: 
            nproc: 65535
            nofile: 
                soft: 20000
                hard: 40000
    redis: 
        image: redis:6.2.5
        container_name: redis
        ports: 
            - "6379:6379"
        volumes: 
            - redisdata:/data
        # 启动时覆盖容器的默认命令
        command: "redis-server --appendonly yes"

volumes: 
    tomcatwebapps: # 自定义映射卷标需要显式声明
        # 是否确定使用指定卷标
        # false 会使用 项目名 + 卷标名
        external: 
            true
    redisdata: 

networks: 
    # 定义上面服务用到的网桥名称，默认就是 bridge
    hello: 
        # 是否确定使用指定名称的网桥
        # false 会使用 项目名 + 网桥名称
        external: 
            true
```
> 同一网络中的服务可以使用服务名称进行通信

**docker-compose 常用指令：**
```bash{.line-numbers}
# 查看帮助
docker-compose --help

# 启动服务的命令,默认前台启动；如果文件名称是 `docker-compose.yml` 可以省略，其它名称不可省略。
docker-compose up docker-compose.yml 或者直接 docker-compose up
docker-compose up docker-test.yml

# 后台启动
docker-compose up -d

# 停止所有 up 启动的服务,并移除网络
docker-compose down

# 进入指定的容器，注意写的是服务名称
docker-compose exec

# 列出当前 docker-compose 运行的所有容器
docker-compose ps

# 重启项目中服务，后面不写服务名代表重启所有服务。还有 start stop 指令，与之类似，略过。
docker-compose restart

# 强制删除（-f）并删除数据卷（-v，慎用-v选项）
docker-compose rm -f -v

# 查看每个服务容器内的进程
docker-compose top

# 唤醒服务
docker-compose unpause

# 暂停服务
docker-compose pause

# 查看服务日志
docker-compose 服务 logs  
```

### portainer 可视化工具
1. 下载可视化工具
    `docker pull portainer/portainer`
2. 启动 portanier，需要开放两个端口，8000 为监听服务的端口，9000 为向外部提供服务的端口。
    `docker -d -p 8000:8000 -p 9000:9000 --name portanier --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer`
3. 浏览器访问 http://localhost:9000

**附：** 一个 docker-compose 对应一个 stack。