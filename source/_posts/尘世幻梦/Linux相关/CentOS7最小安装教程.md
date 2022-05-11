---
title: CentOS7最小安装教程
categories:
  - Linux
  - CentOS
  - 教程
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/linux.jpeg
music:
  server: netease
  type: song
  id: 29045586
abbrlink: 5d8de8e6
---

一般都选 CentOS/Ubantu 作为服务器，这里介绍使用 CentOS7.9 最小化安装。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=false}-->

<!-- code_chunk_output -->

- [1、配置网卡](#1配置网卡)
- [2、关闭防火墙以及 Linux 的一些安全策略](#2关闭防火墙以及-linux-的一些安全策略)
- [3、配置本地 yum 源](#3配置本地-yum-源)
- [4、安装常用工具](#4安装常用工具)
- [5、安装依赖关系](#5安装依赖关系)
- [6、修改yum源](#6修改yum源)
- [rpm命令](#rpm命令)
- [问题](#问题)

<!-- /code_chunk_output -->

采用最小安装的方式安装后没有 `vim` 命令。我们需要先使用 `vi`，这个命令是原生就有的。

## 1、配置网卡

先进行网络的连接，编辑网络配置文件（`vi /etc/sysconfig/network-scripts/ifcfg-ens32`），不同的机器最后的文件名称可能不同，一般都是 `ifcfg-` 开头。

```bash{.line-numbers}
# 空着的部分自定义即可
BOOTPROTO=static
ONBOOT=yes
IPADDR=
NETMASK=
GATEWAY=
DNS1=
```

设置完成后，保存退出，使用命令 `systemctl restart network` 重启网卡。

## 2、关闭防火墙以及 Linux 的一些安全策略

```bash{.line-numbers}
systemctl stop firewalld
systemctl disable firewalld

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

setenforce 0
```

## 3、配置本地 yum 源

```bash{.line-numbers}
# 进入目录
cd /etc/yum.repos.d/
# 创建 备份 文件夹
mkdir bak
# 移动该目录下的所有文件到备份文件夹
mv * bak
# 拷贝一份文件进行编辑
cp bak/CentOS-Media.repo /etc/yum.repos.d/CentOS-Media.repo
```

编辑刚才我们拷贝的文件：`vi CentOS-Media.repo`，这就是安装软件时读取的安装源配置，加入以下内容，先使用本地镜像安装。

```bash{.line-numbers}
    [linux]
    name=linux
    baseurl=file:///media/
    gpgcheck=0
    enabled=1
```

清除yum缓存：`yum -y clean all`
重建yum缓存：`yum makecache`

## 4、安装常用工具

`yum -y install curl telnet vim wget lrzsz net-tools`

修改vim配置（可以不修改，按照默认的即可，这里仅仅是偏好）

```bash{.line-numbers}
vim ~/.vimrc
set encoding=utf-8      " 文件编码
set number              " 显示行号
set tabstop=4           " tab宽度为4
set softtabstop=4        " 设置一次可以删除4个空格
set expandtab           " tab转换为空格
set nowrap              " 不自动换行
set showmatch          " 显示括号配对情

syntax on              " 开启语法高亮   
```

## 5、安装依赖关系

`yum -y install gcc gcc-c++ make autoconf wget lrzsz libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel libxslt-devel libevent-devel libtool libtool-ltdl bison gd gd-devel vim-enhanced pcre-devel zip unzip ntpdate sysstat patch bc expect rsync`

## 6、修改yum源

```bash{.linr-numbers}
# 复制文件
cp /etc/yum.repos.d/bak/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
# 进入目录
cd /etc/yum.repos.d/
# 添加 网易 的下载源
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
# 重建缓存
yum makecache
# 更新源
yum -y update
```

如果不想使用 `wget http://mirrors.163.com/.help/CentOS6-Base-163.repo` 的话，可以自己编辑：`/etc/yum.repos.d/CentOS-Base.repo` 这个文件(以下是用的清华源)。编辑完成之后更新库，这时需要网络。

```bash{.line-numbers}
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

## rpm命令

安装：rpm -ivh xxx.rpm
卸载：rpm -evh xxx.rpm
更新：rpm -Uvh xxx.rpm
显示所有已安装软件：rpm -qa
CentOS安装lrzsz工具：sz下载，rz上传

## 问题

centos7问题描述：
用的好好的虚拟机，之前内网都通，突然xshell连不上虚拟机了也连不上外网了，这时候怎么办呢？
> 解决方法：
1.将networkmanager服务停掉
systemctl stop NetworkManager
systemctl disable NetworkManager
2.重启网卡
systemctl restart network
如上操作，就可以啦

{% link 参考博客地址::https://blog.csdn.net/weixin_44695793/article/details/108089356 %}
<!-- [原博客地址](https://blog.csdn.net/weixin_44695793/article/details/108089356) -->