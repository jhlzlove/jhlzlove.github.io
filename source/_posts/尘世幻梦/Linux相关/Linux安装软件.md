---
title: Linux安装软件
author: 江湖浪子
categories:
  - Linux
  - 软件安装
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/linux.jpeg
abbrlink: 5e42d40d
---

虽然 Ubantu 的 deb 包和 CentOS 的 rpm 包方便下载与安装配置，但是在某些开源工具时，我们可能还是需要去编译一些源码安装，比如 Redis。使用 Docker 固然方便，但是使用他的代价确是繁琐的持久化配置。说实话，个人感觉只要理解 Docker 的原理，项目中会使用就好。安装软件还是直接安装吧，反正各有各的优点不是吗？

<!-- more -->

# Linux 安装软件
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=true}-->

<!-- code_chunk_output -->

- [Linux 安装软件](#linux-安装软件)
  - [Redis](#redis)
  - [Git](#git)
  - [Docker](#docker)
  - [Node](#node)
  - [RabbitMQ](#rabbitmq)
  - [Python](#python)
  - [Nginx](#nginx)
  - [MongoDB](#mongodb)
  - [MySQL](#mysql)
      - [*數據庫問題*](#數據庫問題)

<!-- /code_chunk_output -->

## Redis

Linux 下官网下载 redis 是源码，我们需要编译安装；但是系统可能缺少运行的依赖。如果缺少，以 root 身份执行以下的命令：
```bash{.line-numbers}
    # 安装编译 redis 的工具
    yum -y install gcc automake autoconf libtool make
    # 进入解压目录，进行编译，直到编译完成
    # （编译完成后解压目录下会有一个 bin 目录）
    make MALLOC=libc
    # 安装到指定路径
    make install PREFIX=/usr/local/redis
```

另外如果按照上面的方法执行后，编译时仍然出现错误，找不到文件或者目录；原因可能是之前编译失败的缓存，我们需要清理缓存之后再重新编译。[参考](https://blog.csdn.net/wcnmlgb888/article/details/82713106)
```bash{.line-numbers}
    # 清除缓存
    make distclean
    # 编译
    make
```


配置主从数据库后使用 redis-cli 进行连接后，使用 `info replication` 命令查看当前数据库的 role，“master” 是主节点，“slave” 是从节点。默认从库是只读的，不能进行写操作。

设置开机自启：`chkconfig redis-auto on`

## Git

[参考网址](https://www.cnblogs.com/wulixia/p/11016684.html)

先安装编译 Git 源码的依赖：`yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker`

官网下载源码：https://github.com/git/git/releases
```bash{.line-numbers}
    # 卸载旧版
    yum -y remove git

    # 编译git源码
    make prefix=/usr/local/git all

    # 安装git至/usr/local/git路径
    make prefix=/usr/local/git install

    # 配置环境变量
    vi /etc/profile 
    export PATH=$PATH:/usr/local/git/bin

    # 刷新环境变量
    source /etc/profile

    # 查看Git是否安装完成
    git --version
```

## Docker

如果有旧版本先卸载：
`yum remove docker  docker-common docker-selinux docker-engine`

之后执行以下命令：
```bash{.line-numbers}
    yum install -y yum-utils
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    yum install docker-ce
```
设置自启动：`systemctl enable docker`

## Node

```bash{.line-numbers}
  # CentOS、RHEL
  sudo yum -y remove nodejs
  curl -fsSL https://rpm.nodesource.com/setup_16.x | sudo -E bash -
  sudo yum -y install nodejs

  #  Ubuntu、Debian
  sudo apt -y remove nodejs
  curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
  sudo apt -y install nodejs
```

到这个网站 https://npmmirror.com/mirrors/node/v16.13.1/ 下载 `node-v16.13.1-linux-x64.tar.gz` 文件的即可。然后建立全局链接。
```bash{.line-numbers}
    tar  zvxf node-v16.13.1-linux-x64.tar.gz -C /usr/local/
    cd /usr/local/
    mv node-v16.13.1-linux-x64/ nodejs
    ln -s /usr/local/nodejs/bin/node /usr/local/bin
    ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```


## RabbitMQ
参考网址：https://blog.csdn.net/weixin_36041939/article/details/116908138

一、安装Erlang环境

1、在安装erlang之前先安装下依赖文件(这一步不要忘掉了，不然后面./configure的时候要报错)：

`yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto`

2、到erlang官网去下载erlang安装包

https://www.erlang.org/downloads

接下来解压：
```bash{.line-numbers}
    tar -zxvf otp_src_24.1.7.tar.gz

    cd otp_src_24.1.7/

    ./configure --prefix=/usr/local/erlang
    # 编译安装
    make && make install
    # 测试安装是否成功：
    cd /usr/local/erlang/bin/

    ./erl
```

3、配置环境变量

编辑文件：`vim /etc/profile`
加入内容：`export PATH=$PATH:/usr/local/erlang/bin`
刷新环境变量：`source /etc/profile`

二、安装rabbitmq

1、到官网下载最新安装包：https://www.rabbitmq.com/install-generic-unix.html

解压：
```bash{.line-numbers}
  tar -Jxvf rabbitmq-server-generic-unix-3.9.11.tar.xz
  mv rabbitmq_server-3.9.11 /usr/local/rabbitmq
```

2、配置rabbitmq的环境变量

编辑文件：`vim /etc/profile`
加入内容：`export PATH=$PATH:/usr/local/rabbitmq/sbin`
刷新环境变量：`source /etc/profile`

3、rabbitmq的基本操作：

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

安装完后启动服务并启用插件：`rabbitmq-plugins enable rabbitmq_management`
启用这个插件之后才可以使用 `http://IP地址:15672` 访问管理页面，查看具体信息。默认登录的初始账号密码为 `guest`。

在较新的版本中默认的账号只能以本地 `localhost` 的方式访问，远程操作时登录时可能会出现 `User can only log in via localhost` 的情况，解决办法就是新添加一个超级管理员账户，使用这个新添加的账户登录。
> 5672 用于客户端使用，15672 用于网页控制。


## Python
[参考网址](https://www.jianshu.com/p/15f40edefb13)
[Python官网](https://www.python.org/downloads/)

## Nginx
1. [官网](https://nginx.org/en/)下载安装包；
2. 解压：`tar -zvxf nginx-1.20.2.tar.gz`，之后进入解压目录
3. 执行以下命令：
```bash
    ./configure --prefix=/usr/local/nginx
    make && make install
    # 进入安装目录查看是否安装成功
    cd /usr/local/nginx
```
./nginx -s stop 
./nginx -s reload

在Nginx的配置文件中，location后面使用的路径与模块内使用root或者alias有关，如果使用的是alias，那么可以为任意名称，alias后跟绝对目标文件路径，而且，末尾必须加上“/”。如果使用的是root，那么location后面的路径是目标路径，root后目标路径的上级路径，末尾的“/“可加可不加。
例如：
```
    <!-- 使用root，location 后必须为目标路径 -->
    location /share {
        <!-- 目标路径的上级目录 -->
        root /data/xxxx;
        autoindex on;
        autoindex_localtime on;
    }
    <!-- 使用alias -->
    location /share {
        <!-- 绝对路径，末尾的 / 必须加上，否则403 -->
        alias /data/xxxx/share/;
        autoindex on;
        autoindex_localtime on;
    }
```

## MongoDB

[参考](https://www.cnblogs.com/yangmingxianshen/p/11279405.html)
[官网下载](https://www.mongodb.com/try/download/community)
解压：
`tar -xvzf mongodb-linux-x86_64-rhel62-3.4.22.tgz`
创建数据存储目录、工作目录以及日志目录：

```bash
mv mongodb-linux-x86_64-rhel62-3.4.22 /usr/local/mongodb
cd /usr/local/mongodb/
mkdir conf
mkdir data
mkdir log
```
配置环境变量/etc/profile：
```bash
export MONGODB_HOME=/usr/local/mongodb  
export PATH=$PATH:$MONGODB_HOME/bin 
```

使环境变量生效：`source /etc/profile`

编辑启动文件：
```bash
dbpath = /usr/local/mongodb/data/db  #数据存储目录
logpath = /usr/local/mongodb/log/mongodb.log  #日志存储目录
port = 27017 #指定端口号
fork = true  #以守护进程的方式启动，即在后台运行
bind_ip = 0.0.0.0  #可以连接的端口号
```

启动：`./mongod --config /usr/local/mongodb/conf/mongodb.conf`

如果需要开启密码验证，则需要添加--auth参数：`./mongod --config /usr/local/mongodb/conf/mongodb.conf --auth`

关闭：`./mongod -shutdown -dbpath=/usr/local/mongodb/data/db`
当然你也可以通过kill -9直接将进程杀死。

 
2.远程登陆

如果你希望进行远程登陆，那么在启动的配置文件中，你必须放开bind_ip的配置。

如果你没有开启密码验证：`./mongo --host 172.31.237.186`

如果你开启了密码验证：`./mongo --host 172.31.237.186/admin -uadmin -p123`
需要注意的是，开启验证之后，即使在本机操作，也需要指定host：`mongo -u admin -p123 127.0.0.1/admin`


## MySQL
1. 卸载 mariadb
2. 添加官方的yum源 创建并编辑mysql-community.repo文件
a) vi /etc/yum.repos.d/mysql-community.repo
b) 粘贴以下内容到源文件中
```bash
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```
 c) 注意:如果需要安装 mysql5.7 只需要将baseurl修改即可
`baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/`
3. MySQL 5.7 之后有了默认密码，密码在 `/var/log/mysqld.log` 文件中，注意，需要启动后才有log文件。可以使用 `grep 'temporary password' /var/log/mysqld.log` 查找。

```bash
systemctl start mysqld
systemctl status mysqld
systemctl stop mysqld
```

开启远程连接：
1. 关闭防火墙
2. `grant all privileges on *.* to 'root'@'%' identified by '数据库密码' with grant option;`
3. flush privileges;


#### *數據庫問題*
> 下載navicat后連接數據庫時使用 localhost 連接失敗，顯示沒有權限。
此時可以考慮使用 127.0.0.1 進行連接
若出現以下問題：
  ```bash
  mysql> use dbname;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A 
  ```
出現以上原因是因爲數據庫採用了預讀處理，我们进入mysql 时，没有使用-A参数；
即我们使用
==mysql -hhostname -uusername -ppassword -Pport== 的方式进入数据，
而没有使用
==mysql -hhostname -uusername -ppassword -Pport  -A==的方式进入数据库。
当我们打开数据库，即use   dbname时，要预读数据库信息，当使用==-A==参数时，就不预读数据库信息。 

這種情況我們進入數據庫時可以加上 **`-A`** 選項，如果覺得每次輸入麻煩的話，可以在 **`my.cnf`** 文件里加上如下內容：
```bash
    [mysql]
    no-auto-rehash
```
然後重啓一下mysql，再次連接的時候就好了

- [卸載重新安裝數據庫](https://blog.lanluo.cn/8662)
<br>
> 完全卸載：
> sudo rm -rf /var/lib/mysql
> sudo rm -rf /etc/mysql
> sudo apt-get autoremove mysql* --purge
> sudo apt-get remove apparmor
> 安裝：
> sudo apt-get update
> sudo apt-get install mysql-server

使用cat命令查看默認用戶名密碼：
> sudo cat /etc/mysql/debian.cnf 
 

1. Mysql登录授权：
sudo mysql -u root -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY "123";
（% 表示对全部非本地主机授权）
有些软件不允许使用 root 用户登录，这时需要创建一个新帐户，并为其授权
`create user wasd identified by ‘密码’;`
`grant all privileges on *.* to ‘wasd’@’%’ identified by ‘密码’;`
删除帐户命令：
`drop user username@’%’;`


2. 查看所有的安装软件（包括卸载）：
`dpkg --list;`
找到你所想要卸载的软件，在终端输入命令：
`sudo apt-get --purge remove 包名`
（--purge是可选项，写上这个属性是将软件及其配置文件一并删除，如不需要删除配置文件，直接执行sudo apt-get remove 包名即可）
执行过程中会问你是否要删除，按y即可
参考博客：https://blog.csdn.net/luckydog612/article/details/80877179

centos7问题描述：
用的好好的虚拟机，之前内网都通，突然xshell连不上虚拟机了也连不上外网了，这时候怎么办呢？
> 解决方法：
1.将networkmanager服务停掉
systemctl stop NetworkManager
systemctl disable NetworkManager
2.重启网卡
systemctl restart network
如上操作，就可以啦

[原博客地址](https://blog.csdn.net/weixin_44695793/article/details/108089356)