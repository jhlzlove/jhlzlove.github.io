---
title: Linux安装软件
categories:
  - Linux
  - 软件安装
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/linux.jpeg
abbrlink: 5e42d40d
---

Linux 下基本有源码编译安装、二进制包解压缩安装（可以使用包管理器安装，方便快捷）、Appimage格式的软件等，当然，最后一个一般在服务器上也用不到。
整理这个就是为了防止有一天浪子在没网络的时候，比较无聊，搭个环境在本地写写代码什么的。我相信大家都喜欢使用各个系统相应的包管理工具（rpm、apt-get、pacman等）直接安装，一个字，爽的不要不要的，不需要担心，缺啥装啥就行了。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=true}-->

<!-- code_chunk_output -->

- [Linux源码安装](#linux源码安装)
  - [1. Redis](#1-redis)
  - [2. Git](#2-git)
  - [3. RabbitMQ](#3-rabbitmq)
  - [4. Python](#4-python)
  - [5. Nginx](#5-nginx)
  - [6. Btop++](#6-btop)
- [Linux通用二进制文件安装](#linux通用二进制文件安装)
  - [1. Docker](#1-docker)
  - [2. Node](#2-node)
  - [3. MongoDB](#3-mongodb)
  - [4. MySQL](#4-mysql)
    - [密码正确但是进不去 bash 环境](#密码正确但是进不去-bash-环境)
    - [预读处理](#预读处理)
- [Some Questions](#some-questions)
  - [1. 关于源码编译安装失败](#1-关于源码编译安装失败)

<!-- /code_chunk_output -->

**TIPS：** 如果下面的命令不是使用超级用户执行的话，可能不能安装成功，这个时候可以添加 `sudo` 选项进行重试。

## Linux源码安装

### 1. Redis

{% link 官网源码下载::https://redis.io/download/ %}

```bash{.line-numbers}
# 安装编译 redis 需要的工具
sudo yum -y install gcc automake autoconf libtool make
# 进入解压目录，进行编译，直到编译完成
make MALLOC=libc
# 安装到指定路径
make install PREFIX=/usr/local/redis
```

> Redis 的默认的配置文件在源码解压后的目录中

### 2. Git

{% link 官网源码下载::https://github.com/git/git/tags %}

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

### 3. RabbitMQ

RabbitMQ 是使用 Erlang 语言编写的中间件，联想一下 Java，我们可以猜到它需要先搭建 Erlang 环境。主要是这个环境需要编译源码，RabbitMQ 本身官网提供了二进制压缩包。

{% link Erlang源码下载::https://www.erlang.org/downloads %}

一、Erlang环境搭建

```bash{.line-numbers}
# 安装编译 Erlang 的相关依赖
yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto
# 解压源码包
tar -zxvf otp_src_24.1.7.tar.gz

cd otp_src_24.1.7/
# 指定安装目录
./configure --prefix=/usr/local/erlang
# 编译安装
make && make install
# 测试安装是否成功：
cd /usr/local/erlang/bin/
./erl

# 安装成功后配置环境变量 vim /etc/profile
export PATH=$PATH:/usr/local/erlang/bin

# 添加完成保存退出，刷新使其生效
source /etc/profile
```

二、安装Rabbitmq

通过第一步，我们就搭建好了 Erlang 环境，接下来就是安装 RabbitMQ 了，这个还是比较简单的，因为它已经编译好了，我们可以下载直接配置，无需编译。

{% link 二进制文件包下载::https://www.rabbitmq.com/install-generic-unix.html %}

```bash{.line-numbers}
# 解压
tar -Jxvf rabbitmq-server-generic-unix-3.9.11.tar.xz
# 移动
mv rabbitmq_server-3.9.11 /usr/local/rabbitmq

# 添加环境变量：vim /etc/profile
export PATH=$PATH:/usr/local/rabbitmq/sbin
# 刷新变量
source /etc/profile
```

### 4. Python

Linux 下基本不需要配置 Python 的环境，有个别的 ISO 镜像版本比较老，比如 CentOS7 的 mini ISO 镜像是2.x的，我们可以通过各大系统的包管理工具进行安装，也可以自己通过源码编译安装。

{% link 源码下载::https://www.python.org/downloads/ %}

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

### 5. Nginx

{% link 官网源码下载::https://nginx.org/en/ %}

```bash{.line-numbers}
# 解压
tar -zvxf nginx-1.20.2.tar.gz
# 配置安装路径
./configure --prefix=/usr/local/nginx
# 编译安装
make && make install
# 进入安装目录查看是否安装成功
cd /usr/local/nginx

# 启动 停止 重启
./nginx start
./nginx -s stop
./nginx -s reload
```

### 6. Btop++

Btop++ 是一个 Linux 资源监视器，显示处理器、内存、磁盘、网络和进程的使用情况和统计资料，界面美观，使用简单。

{% link Github地址::https://github.com/aristocratos/btop %}
{% link Gitee同步仓库::https://gitee.com/mirrors/btop %}

```bash{.line-numbers}
# 安装、升级相关依赖工具
yum install coreutils sed build-essential -y
yum install centos-release-scl -y
yum install devtoolset-10 -y
scl enable devtoolset-10 bash 
echo "source /opt/rh/devtoolset-10/enable" >> /etc/profile

# 克隆源码编译安装
git clone https://gitee.com/mirrors/btop.git
cd btop
make && make install
```

## Linux通用二进制文件安装

通用二进制文件一般使用包管理工具即可快速安装使用，写这个主要是因为没有网络的情况下，在自己虚拟机上搭建相应的环境去耍。以下都是推荐使用各个平台的包管理工具去耍。以下都是使用的 yum，之前用的 CentOS 做实验。

### 1. Docker

推荐使用包管理工具安装 Docker 和 docker-compose。

```bash{.line-numbers}
# 如果有旧版本先卸载
yum remove docker docker-common docker-selinux docker-engine

# 安装相关工具
yum install -y yum-utils
# 添加阿里云镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装 docker 引擎
yum install docker-ce

# 设置开机自启
systemctl enable docker
```

### 2. Node

{% link 二进制包下载::https://nodejs.org/en/download/ %}

Node 可以直接下载二进制文件，解压配置环境变量即可。

```bash{.line-numbers}
# 官网下载二进制压缩包是 xz 结尾的，和下面的镜像站下载的略有不同
tar -Jvxf node-v16.15.0-linux-x64.tar.xz -C /usr/local/
cd /usr/local/
mv node-v16.15.0-linux-x64/ nodejs
ln -s /usr/local/nodejs/bin/node /usr/local/bin
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```

如果嫌弃官网下载速度慢可以到这里 https://registry.npmmirror.com/binary.html?path=node/latest-v16.x/ 选择合适的版本下载。

### 3. MongoDB

{% link 二进制包下载::https://www.mongodb.com/try/download/community %}

```bash{.line-numbers}
# 解压
tar -xvzf mongodb-linux-x86_64-rhel62-3.4.22.tgz
mv mongodb-linux-x86_64-rhel62-3.4.22 /usr/local/mongodb

# 创建数据存储目录、工作目录以及日志目录
cd /usr/local/mongodb/
mkdir conf
mkdir data
mkdir logs

# 添加环境变量: vim /etc/profile
export MONGODB_HOME=/usr/local/mongodb  
export PATH=$PATH:$MONGODB_HOME/bin

# 使环境变量生效
source /etc/profile
```

顺带说一下配置，编辑 MongoDB 的配置文件 `mongodb.conf`，修改以下内容：

```bash{.line-numbers}
# 数据存储目录
dbpath = /usr/local/mongodb/data/db
# 日志存储目录
logpath = /usr/local/mongodb/logs/mongodb.log
# 指定端口号
port = 27017
# 以守护进程的方式启动，即后台运行
fork = true
# 可以连接的地址，开启远程登录
bind_ip = 0.0.0.0
# 启用密码验证，可根据实际情况配置
# auth = true
```

mongodb安装好后第一次启动服务是不需要密码的，也没有任何用户，通过shell命令可直接进入，cd到mongodb目录下的bin文件夹，执行命令 `./mongo` 即可。如果配置了环境变量，可以在任意路径执行 `mongo` 也可以。

```bash{.line-numbers}
# 指定配置文件启动服务
./mongod --config /usr/local/mongodb/conf/mongodb.conf
# 进入系统数据库
use admin
# 创建 root 用户
db.createUser( {user: "root",pwd: "xxx",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]});
# 查看创建的用户(以下两个命令二选一)
show users;
db.system.users.find();
# 关闭服务
db.shutdownServer();

# 启动时开启密码验证，也可以在配置文件中加入 auth = true 在启动时开启验证，这样就不用 --auth 选项了
./mongod --config /usr/local/mongodb/conf/mongodb.conf --auth
```

### 4. MySQL

MySQL 说实话我觉得通用的二进制文件包安装较为简单，卸载是最简单的，比使用包管理工具安装的简单的多。下面给两个包管理工具安装的参考链接，我没怎么用过，仅作参考。

apt-get安装：https://blog.lanluo.cn/8662
yum安装：https://zhuanlan.zhihu.com/p/87069388

接下来进入正题，使用通用的 MySQL8.x 版本的二进制压缩包进行安装。至于卸载，就把有关 MySQL 创建的几个文件夹删掉就行了，`/etc/my.cnf` 是默认自带的，卸载的时候删不删都没有问题，没有的也不必担心，新建也是可以的。

```bash{.line-numbers}
# 检查mysql用户组和用户是否存在，如果没有，则创建
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql
# 创建 mysql 组
groupadd mysql
# 新建 mysql 用户并加入 mysql 群组
useradd -r -g mysql mysql

# 安装所需依赖(需要安装 libaio-devel.x86_64 numactl 这两个依赖)
yum -y install libaio-devel.x86_64 numactl

# 解压二进制文件包到 /usr/local/mysql 目录
tar -C /usr/local/mysql
cd /usr/local/mysql/bin
# 初始化数据，需要记录最后 root@localhost: 后的字符串，它是后面进入 bash 环境的初始密码
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql

# vim /etc/my.cnf(没有该文件手动创建) 修改内容
datadir=/usr/local/mysql/data
port = 3306

# 启动MySQL服务 启动成功会有 Starting MySQL.. SUCCESS! 提示
cd /usr/local/mysql/support-files/
mysql.server start

# 添加软链接并重启服务
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
```

至此，安装任务基本完成，下面需要添加用户并分配权限，进入 MySQL 的 bash 环境需要前面我们进行初始化时生成的密码。

```bash{.line-numbers}
# 登录 MySQL，密码使用初始化成功时 root@localhost: 后的字符串
mysql -uroot -p

# 修改密码 毕竟那么不好记
# 而且如果不修改 它会报 ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement. 的错误
alter user root@'localhost' identified by 'newpassword';
flush privileges;

# 开放远程连接
use mysql;
# 允许所有主机，都可以通过用户为root用户，密码为默认数据库登录密码，进行数据库操作
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
{% gallery %}
![初始化成功截图](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/Linux_MySQL初始化成功截图.png)
{% endgallery %}

> 记录日志最末尾位置 `root@localhost:` 后的字符串，此字符串为mysql管理员临时登录密码。

如果启动失败查看err日志，根据日志定位修复问题。

#### 密码正确但是进不去 bash 环境

```bash{.line-numbers}
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES/NO) 
```

解决方法：

1. 使用 kill 命令停止 mysqld 相关服务
2. cd /usr/local/mysql/bin/，运行命令： `mysqld_safe --user=mysql --skip-grant-tables --skip-networking &`
3. 使用密码登录数据库 mysql -u root -p 并切换到 mysql 数据库
4. 执行命令：`update user set host='%' where user='root';`

#### 预读处理

```bash{.line-numbers}
mysql> use dbname;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A 
```

这个问题是之前使用的 apt-get 管理工具安装的 MySQL 出现的，原因是因爲數據庫採用了預讀處理。解决办法就是在我们进入MySQL的bash环境时，需要加入 `-A` 参数，不让其预读数据库信息，`mysql -u root -p -A`。如果覺得每次进入 bash 环境都要添加参数比较麻烦，也可以在 **`my.cnf`** 文件里加上如下內容：

```bash{.line-numbers}
[mysql]
no-auto-rehash
```

## Some Questions

### 1. 关于源码编译安装失败

如果源码编译失败，先确认所需依赖是否全部成功安装，然后清除上一次编译的缓存，之后再次编译，不然会一直失败。[参考网址](https://blog.csdn.net/wcnmlgb888/article/details/82713106)

```bash{.line-numbers}
# 清除上一次编译失败的缓存
make distclean
# 再次编译
make
```
