---
title: 在移动硬盘或者U盘中安装 wintogo 系统
categories: window相关
abbrlink: 72ef477f
---


先附上参考链接：[参考网址](https://social.technet.microsoft.com/Forums/zh-CN/d80d0f84-72f4-46ee-9f66-bd7412bf13eb/229142030920174win10319953247923433350133823620687259912021420013?forum=win10itprogeneralCN)

<!-- more -->

## 在移动硬盘或者U盘中安装 wintogo 系统

### 我直接装的专业版，未果，其它没试，听说只有企业版才行。请自行尝试。。。

官网下载器下载 win10 iso映像，双击挂载或者使用工具解压，进入挂载目录source文件夹下，哎～我的 install.wim 文件呢？不会吧，我下载的为什么没有呢？怎么办呢？还是百度问大佬去吧！咦～正要打开浏览器的你突然瞥到了 install.esd 文件；这个文件怎么和我想要的那么像呢？你不禁在思索他们之间的关系。。。思索了一阵，还是打开了浏览器。。。

的确，如果找不到 wim 文件，我们确实可以使用 install.esd 来生成。

首先，我么在C盘根目录创建esd文件夹，再把source下的 install.esd 复制过来。其实在哪个地方建，叫什么名字全凭自己的喜好来就行，我这是做是为了方便。少侠甚至可以直接放到c盘根目录。
之后以管理员身份运行cmd,进入到esd文件夹，运行以下命令：
`dism /Get-WimInfo /WimFile:install.esd`

根据打印的信息找到你想要的系统的索引，例如浪子选专业版，它的索引是 4，就运行以下命令，根据自己需要选择其他索引，索引是在 `/SourceIndex:` 这里写，各位少侠不要弄错了哦。
`dism /export-image /SourceImageFile:install.esd /SourceIndex:4 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity`

等到命令执行完就可以在esd文件夹看到wim文件了，这时我们就可以使用wtg辅助工具去安装了。
总结一下：
1.在C 盘（系统盘）下，创建一个新的文件夹命名为esd, 将install.esd文件放在该文件夹下。

2.使用管理员身份打开命令行工具。

3.运行命令：cd c:\esd 

4.运行命令以显示esd文件中的内容：dism /Get-WimInfo /WimFile:install.esd

5.找到你想要导出的文件内容序号，比如index:1，然后运行命令进行提取:

dism /export-image /SourceImageFile:install.esd /SourceIndex:1 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity

6.在文件夹c:\esd中你将会发现新生成的一个文件install.wim

> 移动硬盘、U盘安装系统后想要更换或者升级版本不能直接升级，还要使用这个方法，下载需要的镜像版本，提取 wim 文件，使用 wintogo 辅助工具部署。