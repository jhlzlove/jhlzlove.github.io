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
  - [Btop++](#btop)
  - [MySQL](#mysql)
    - [其它问题](#其它问题)
      - [*可视化工具连接問題*](#可视化工具连接問題)

<!-- /code_chunk_output -->

## Redis

{% link 官网下载源码::https://redis.io/download/ %}

以 root 身份执行以下的命令：
```bash{.line-numbers}
# 安装编译 redis 需要的工具
yum -y install gcc automake autoconf libtool make
# 进入解压目录，进行编译，直到编译完成
make MALLOC=libc
# 安装到指定路径
make install PREFIX=/usr/local/redis
```
> 配置文件在解压的 redis 包下，编译安装后在 /usr/local/redis/bin 下是没有默认的 redis.conf 配置文件的。可以拷贝一份过去。

另外如果按照上面的方法执行后，编译时如果出现错误，找不到文件或者目录；原因可能是之前编译过，但是编译失败留下的缓存，我们需要清理缓存之后再重新编译。[参考网址](https://blog.csdn.net/wcnmlgb888/article/details/82713106)
```bash{.line-numbers}
# 清除缓存
make distclean
# 编译
make
```

<!-- 设置开机自启：`chkconfig redis-auto on` -->

配置主从数据库后使用 redis-cli 进行连接后，使用 `info replication` 命令查看当前数据库的 role，“master” 是主节点，“slave” 是从节点。默认从库是只读的，不能进行写操作。


## Git

{% link 选择版本下载源码::https://github.com/git/git/tags %}

```bash{.line-numbers}
# 安装编译 Git 源码的工具和依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

# 卸载旧版 Git
yum -y remove git

# 编译 Git 源码
make prefix=/usr/local/git all

# 安装git至指定的路径
make prefix=/usr/local/git install

# 配置环境变量： 编辑文件 vi /etc/profile 
export PATH=$PATH:/usr/local/git/bin

# 刷新环境变量
source /etc/profile

# 查看Git是否安装完成
git --version
```
[参考网址](https://www.cnblogs.com/wulixia/p/11016684.html)


## Docker

```bash{.line-numbers}
# 如果有旧版本先卸载
yum remove docker  docker-common docker-selinux docker-engine

# 安装相关工具
yum install -y yum-utils
# 添加阿里云镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装 docker 引擎
yum install docker-ce

# 设置开机自启动
systemctl enable docker
```


## Node

{% link 官网下载::https://nodejs.org/en/download/ %}

```bash{.line-numbers}
# 官网下载压缩包是 xz 结尾的，和下面的镜像站下载的略有不同
tar -Jvxf node-v16.14.2-linux-x64.tar.xz -C /usr/local/
cd /usr/local/
mv node-v16.14.2-linux-x64/ nodejs
ln -s /usr/local/nodejs/bin/node /usr/local/bin
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```

如果下载速度慢可以到这里 https://npmmirror.com/mirrors/node/v16.13.1/ 下载 `node-v16.13.1-linux-x64.tar.gz` 文件的即可。然后解压并建立全局链接。
```bash{.line-numbers}
tar  zvxf node-v16.13.1-linux-x64.tar.gz -C /usr/local/
cd /usr/local/
mv node-v16.13.1-linux-x64/ nodejs
ln -s /usr/local/nodejs/bin/node /usr/local/bin
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```

直接使用命令行安装，简便快捷
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


## RabbitMQ

一、安装Erlang环境

1、在安装erlang之前先安装下依赖文件(这一步不要忘掉了，不然后面./configure的时候要报错)：

`yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto`

2、到erlang官网去下载erlang安装包

{% link 官网下载::https://www.erlang.org/downloads %}

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

1、到官网下载安装包：https://www.rabbitmq.com/install-generic-unix.html

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

[参考网址](https://blog.csdn.net/weixin_36041939/article/details/116908138)

## Python

Linux系统自带Python环境，但是版本基本都是2.x，我们可以手动安装新版本。

{% link Python官网下载最新安装包::https://www.python.org/downloads/ %}

```bash{.line-numbers}
# 安装相关工具和依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel

# 解压压缩包
tar -zxvf Python-3.10.2.tgz  

# 进入文件夹
cd Python-3.10.2

# 配置安装位置
./configure prefix=/usr/local/python3

# 编译并安装
make && make install

# 使用 python3 验证是否安装成功
python3 -V

#添加python3的软链接 
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python3 

#添加 pip3 的软链接 
ln -s /usr/local/python3/bin/pip3.8 /usr/bin/pip3
```

[参考网址](https://www.jianshu.com/p/15f40edefb13)



## Nginx

{% link 官网下载::https://nginx.org/en/ %}

1. 解压：`tar -zvxf nginx-1.20.2.tar.gz`，之后进入解压目录
2. 执行以下命令：
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
设置自启动：https://www.jianshu.com/p/ca5ee5f7075c


## MongoDB


{% link 官网下载::https://www.mongodb.com/try/download/community %}
解压：
`tar -xvzf mongodb-linux-x86_64-rhel62-3.4.22.tgz`
创建数据存储目录、工作目录以及日志目录：

```bash
mv mongodb-linux-x86_64-rhel62-3.4.22 /usr/local/mongodb
cd /usr/local/mongodb/
mkdir conf
mkdir data
mkdir logs
```
配置环境变量/etc/profile：
```bash
export MONGODB_HOME=/usr/local/mongodb  
export PATH=$PATH:$MONGODB_HOME/bin 
```

使环境变量生效：`source /etc/profile`

编辑启动文件：
```bash
# 数据存储目录
dbpath = /usr/local/mongodb/data/db
# 日志存储目录
logpath = /usr/local/mongodb/logs/mongodb.log
# 指定端口号
port = 27017
# 以守护进程的方式启动，即在后台运行
fork = true
# 可以连接的地址，开启远程登录
bind_ip = 0.0.0.0
```

mongodb安装好后第一次进入是不需要密码的，也没有任何用户，通过shell命令可直接进入，cd到mongodb目录下的bin文件夹，执行命令 `./mongo` 即可。如果配置了环境变量，可以在任意路径执行 `mongo` 也可以。

命令：
```bash{.line-numbers}
# 启动服务
./mongod --config /usr/local/mongodb/conf/mongodb.conf
# 进入系统数据库
use admin
# 创建 root 用户
db.createUser( {user: "root",pwd: "root123",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]});
# 查看创建的用户
show users; / db.system.users.find();
# 关闭服务
db.shutdownServer();

# 启动时开启密码验证，也可以在配置文件中加入 auth = true 在启动时开启验证，这样就不用 --auth 选项了
./mongod --config /usr/local/mongodb/conf/mongodb.conf --auth
```


## Btop++
Btop++ 是一个 Linux 资源监视器，显示处理器、内存、磁盘、网络和进程的使用情况和统计资料。

{% link Github地址::https://github.com/aristocratos/btop %}

如果下载速度较慢，可以使用Gitee同步仓库：https://gitee.com/mirrors/btop

CentOS安装:
```bash{.line-numbers}
# 安装依赖
yum install coreutils sed build-essential -y

# 升级gcc（10及以上版本）
yum install centos-release-scl -y
yum install devtoolset-10 -y
scl enable devtoolset-10 bash 
echo "source /opt/rh/devtoolset-10/enable" >> /etc/profile

# 克隆源码编译安装
git clone https://gitee.com/mirrors/btop.git
cd btop
make && make install
```

## MySQL

{% link Ubantu系统安装MySQL::https://blog.lanluo.cn/8662 %}

CentOS等常用的一键安装网络上有许多参考，这里使用通用版的压缩包方式来安装。
{% link 参考网址::https://zhuanlan.zhihu.com/p/87069388 %}

```bash{.line-numbers}
# 检查mysql用户组和用户是否存在，如果没有，则创建
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql
groupadd mysql
useradd -r -g mysql mysql

# 安装所需依赖(需要安装 libaio-devel.x86_64 numactl 这两个依赖)
yum -y install libaio-devel.x86_64 numactl

# 初始化数据
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql

# 编辑配置文件 my.cnf 修改内容：
datadir=/usr/local/mysql/data
port = 3306

# 启动MySQL服务 启动成功会有 Starting MySQL.. SUCCESS! 提示
cd /usr/local/mysql/support-files/
mysql.server start

# 添加软链接重启服务
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
service mysql restart
# systemctl restart mysql

# 添加开机自启
# 1、将服务文件拷贝到init.d下，并重命名为mysql
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
# 2、赋予可执行权限
chmod +x /etc/init.d/mysqld
# 3、添加服务
chkconfig --add mysqld
# 4、显示服务列表
chkconfig --list

# 登录 MySQL，密码使用初始化成功时 root@localhost: 后的字符串
mysql -uroot -p

# 修改密码 不然在 bash 环境总报 ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement. 的错误
alter user root@'localhost' identified by 'newpassword';
flush privileges;
# update user set authentication_string='' where user='root';
# ALTER user 'root'@'localhost' IDENTIFIED BY 'newpassword';

# 开放远程连接
use mysql;
# 允许所有主机，都可以通过用户为root用户，密码为默认数据库登录密码，进行访问数据库
update user set host='%' where user='root';

# ① 适用于 MySQL 8.0之前的版本，可以直接授权
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
# ② 适用于 MySQL 8.0之后的版本，需要先创建一个用户，再进行授权【推荐方式②】
create user root@'%' identified by 'root';
grant all privileges on *.* to root@'%' with grant option;
# 刷新权限，这一句很重要，使修改生效，如果没有写，则还是不能进行远程连接。这句表示从mysql数据库的grant表中重新加载权限数据，因为MySQL把权限都放在了cache中，所以，做完修改后需要重新加载。
flush privileges;
```

初始化成功截图：
![初始化成功截图](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/Linux_MySQL初始化成功截图.png)
记录日志最末尾位置 `root@localhost:` 后的字符串，此字符串为mysql管理员临时登录密码。

如果启动失败查看err日志，根据日志定位修复问题。

### 其它问题
```bash{.line-numbers}
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES/NO) 
```
解决方法：
1. 使用 kill 命令停止 mysqld 相关服务
2. cd /usr/local/mysql/bin/,运行命令： `mysqld_safe --user=mysql --skip-grant-tables --skip-networking &`
3. 使用密码登录数据库 mysql -u root -p 并切换到 mysql 数据库
4. 执行命令：`update user set host='%' where user='root';`

查看MySQL的binlog命令
/usr/bin/mysqlbinlog --no-defaults -v --base64-output=decode-rows binlog.000001 > nov3.sql


根据binlog的内容恢复数据
/usr/bin/mysqlbinlog binlog.000007 | mysql -uroot -p -v -f > /opt/1.txt
然后输入密码开始恢复数据，
-f是为了跳过执行错误，
-v是展开执行的详细信息
如果文件中存在不能执行的指令，可以按照时间进行执行比如：
加上时间段参数 `--start-datetime=‘2022-01-06 14:14:37’ --stop-datetime=‘2022-01-24 13:04:44’` 恢复指定时间段的数据。


#### *可视化工具连接問題*

错误提示：
```bash
mysql> use dbname;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A 
```

出現以上原因是因爲數據庫採用了預讀處理，解决办法就是在我们进入MySQL的bash环境时，需要加入 `-A` 参数，不让其预读数据库信息，`mysql -u root -p -A`。

如果覺得每次进入bash环境添加麻烦，也可以在 **`my.cnf`** 文件里加上如下內容：
```bash
    [mysql]
    no-auto-rehash
```
 

1. Mysql登录授权：
sudo mysql -u root -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY "123";
（% 表示对全部非本地主机授权）
有些软件不允许使用 root 用户登录，这时需要创建一个新帐户，并为其授权
`create user wasd identified by ‘密码’;`
`grant all privileges on *.* to ‘wasd’@’%’ identified by ‘密码’;`
删除帐户命令：
`drop user username@’%’;`



