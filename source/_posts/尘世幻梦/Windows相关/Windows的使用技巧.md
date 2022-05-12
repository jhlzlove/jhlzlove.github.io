---
layout: docs
title: Windows相关问题
categories: window
---

window 使用技巧合集，更新待续

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [查看windows版本(关于windows)](#查看windows版本关于windows)
- [永久关闭win10自动更新](#永久关闭win10自动更新)
- [删除bios的多余启动项（Linux同样适用）](#删除bios的多余启动项linux同样适用)
- [快捷方式](#快捷方式)
- [WSL相关问题](#wsl相关问题)
- [VMWare虚拟机配置双网卡](#vmware虚拟机配置双网卡)
- [powershell不能运行脚本](#powershell不能运行脚本)

<!-- /code_chunk_output -->

### 查看windows版本(关于windows)

window+r   输入winver，回车。

### 永久关闭win10自动更新

按Win键+R键调出运行，输入“gpedit.msc”点击“确定”，调出“本地计算机策略”，计算机配置→管理模板→Windows组件→Windows更新。在右面找到“配置自动更新”，并双击，点击禁用。关掉“本地组策略编辑器”，重启电脑。

### 删除bios的多余启动项（Linux同样适用）

```bash
# 查看启动项
efibootmgr

# 删除对应的项
efibootmgr -b 0001(项序号) -B
```

### 快捷方式

win10系统 按下 win 后，直接输入想要的结果就行，比如直接输入控制，就会出现最佳匹配控制面板。(~~win10的易用性简直不要太好~~)

控制面板：
win + r 输入 `appwiz.cpl` 回车
环境变量：
win + r 输入 `sysdm.cpl` 回车 或者 Fn+win+pause break(笔记本没有pause break键可以使用字母P)
磁盘管理
win + r 输入 `diskmgmt.msc` 回车
服务
win + r 输入 `services.msc` 回车

### WSL相关问题

在Windows的Docker desktop下使用ES，启动时会遇到内存不足的问题。调整的方式是通过命令行wsl进入docker-desktop的终端，然后通过sysctl命令调整系统参数。

```bash
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

也可以编辑文件 `vi /etc/sysctl.conf` 加入 `sysctl -w vm.max_map_count=262144`,保存退出执行 `sysctl -p` 即可。

### VMWare虚拟机配置双网卡

使用 VMWare 时，推荐创建的虚拟机配置双网卡，默认的使用 NAT 模式，新添加一个使用 仅主机 模式，然后进行修改配置。这样就算以后连接无数个无线网，电脑依然能上网，但是使用 SSH 远程连接时依然不用修改 Linux 的网络配置文件。实现了一次配置，永久使用的效果。配置也很简单：

在 VMWare 里创建一个虚拟机后，打开改虚拟机的设置，硬件下方的”添加“，然后在弹出的列表里选择”网络适配器“，之后点击新添加的网络适配器，选择”仅主机“。

然后打开网络适配器修改 VMWare Network Adapter VMnet1 的 IP 为简单的地址，Linux 里的网络配置文件的 IP 地址不要和这个一样，因为 VM1 是本机和 Linux 虚拟机通信的 IP 地址，不能一样，推荐 VM1 10.0.0.1，Linux 配置为 10.0.0.2，简单好记。

> **注意：** 如果是新建的虚拟机，在安装之前按照以上方法配置，安装后查看网络配置文件时或许会有两个不同的文件，比如：ens33，ens34。如果是之前已经安装过的单网卡想要配置双网卡，可以把已经存在网络配置文件复制一份，修改文件里面的内容即可。如果新建的虚拟机也是只有一个配置文件，也可以直接复制原有的文件进行修改。

另外，第一次安装后网络服务是正常的，之后换了一个无线网连接就出现了问题，可能是因为 NetworkManager 的问题，可以关闭此服务。然后重启网络。

```bash
# 关闭服务
systemctl stop NetworkManager
# 禁止开机启动
systemctl disable NetworkManager
# chkconfig NetworkManager off

# 重启网络
systemctl restart network
```

{% link 其它原因请参考::https://blog.csdn.net/weistin/article/details/80676955 %}

### powershell不能运行脚本

管理员身份运行`set-executionpolicy RemoteSigned`，然后选择 Y.

powershell有四种执行策略：

- Restricted 禁止运行任何脚本和配置文件（默认）
- AllSigned 可以运行脚本，但要求所有脚本和配置文件由可信发布者签名，包括在本地计算机上编写的脚本
- RemoteSigned 可运行脚本，但要求从网络上下载的脚本和配置文件由可信发布者签名；不要求对已经运行和本地计算机编写的脚本进行数字签名
- Unrestricted 可以运行未签名的脚本