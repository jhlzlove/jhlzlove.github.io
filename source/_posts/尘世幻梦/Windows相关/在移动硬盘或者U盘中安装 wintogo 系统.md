---
title: 在移动硬盘或者U盘中安装 wintogo 系统
categories: 
    - window
    - 系统
abbrlink: 72ef477f
---


{% link 参考网址::https://social.technet.microsoft.com/Forums/zh-CN/d80d0f84-72f4-46ee-9f66-bd7412bf13eb/229142030920174win10319953247923433350133823620687259912021420013?forum=win10itprogeneralCN %}

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

## 在移动硬盘或者U盘中安装 wintogo 系统

> 浪子直接装的专业版，可以安装的上，也可以启动使用，但是每次重新插入使用的话，都需要重新走一遍重新装机的流程，有时还会出现系统错误，并且之前的内容都不存在。其它版本没试，据说只有企业版才行。请自行尝试。。。

官网下载器下载 win10 ISO映像，双击挂载或者使用工具解压，进入挂载目录source文件夹下，哎～我的 install.wim 文件呢？不会吧，明明是按照网上教程来的，我下载的镜像为什么没有呢？难不成官网的镜像有问题？怎么办呢？还是百度问大佬去吧！咦～正要打开浏览器的你突然瞥到了 install.esd 文件；这个文件怎么和我想要的那么像呢？你不禁在思索他们之间的关系。。。思索了一阵，还是打开了浏览器。。。

的确，如果找不到 `wim` 文件，我们确实可以使用 `install.esd` 来生成。

首先，我么在任意位置创建一个空文件夹，推荐在桌面创建，方便查找，再把source下的 `install.esd` 文件复制过来。再以管理员身份运行cmd,进入到刚才创建的文件夹，运行以下命令：

`dism /Get-WimInfo /WimFile:install.esd`

根据打印的信息找到你想要的系统的索引，例如浪子选专业版，它的索引是 4，就运行以下命令，根据自己需要选择其他索引，索引就是在 `/SourceIndex:` 后面的数字，各位少侠不要弄错了哦。

`dism /export-image /SourceImageFile:install.esd /SourceIndex:4 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity`

等到系统执行完就可以在刚刚的文件夹看到 `wim` 文件了，这时我们就可以使用wtg辅助工具去安装了。
总结一下：

1.将install.esd文件放到一个文件夹中，最好不要有其他文件，便于查找。

2.使用管理员身份打开命令行工具并进入到 install.esd 的文件目录中。

4.运行命令：`dism /Get-WimInfo /WimFile:install.esd`

5.找到你想要导出的系统内容序号，比如index:1，然后运行命令进行提取:
`dism /export-image /SourceImageFile:install.esd /SourceIndex:1 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity`

6.在文件夹会新生成的一个文件 `install.wim`。

> 移动硬盘/U盘安装系统后想要更换或者升级版本不能直接升级，还要使用这个方法，下载最新的镜像版本，提取 wim 文件，再使用 wintogo 辅助工具部署。
