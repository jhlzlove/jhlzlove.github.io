---
title: ArchLinux安装教程
categories:
  - Linux
  - Arch
  - 教程
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogcover/arch.jpeg
music:
  server: netease
  type: song
  id: 29045586
abbrlink: 8166567f
---

ArchLinux，很不错的一款 Linux 发行版，不过的它的安装可能让很多人望而却步，不过还有背靠 Arch 的另一个子系统，Manjaro，它的安装就和 Window 一样，简单容易。本篇文章是基于之前的大佬教我而写。
{% link 官方文档::https://wiki.archlinux.org/title/Installation_guide#Boot_the_live_environment::https://cdn.jsdelivr.net/gh/xaoxuu/cdn-assets@master/logo/256/safari.png %}

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=5 orderedList=true}-->

<!-- code_chunk_output -->

- [一、国内镜像列表](#一国内镜像列表)
- [二、前言](#二前言)
- [三、准备工作](#三准备工作)
- [四、开始安装](#四开始安装)
  - [1. 分区](#1-分区)
  - [2. 格式化分区](#2-格式化分区)
  - [3. 挂载分区](#3-挂载分区)
  - [4. 设置下载镜像源，提升我们后续的下载速度](#4-设置下载镜像源提升我们后续的下载速度)
  - [5. 安装 linux 基本组件](#5-安装-linux-基本组件)
  - [6. 执行以下命令](#6-执行以下命令)
  - [7. 切换到挂载点](#7-切换到挂载点)
  - [8. 设置时间](#8-设置时间)
  - [9. 设置root用户密码](#9-设置root用户密码)
  - [10. 设置语言和本机名](#10-设置语言和本机名)
  - [11. 退出](#11-退出)
  - [12. 前期准备](#12-前期准备)
  - [13. 安装驱动(请确保是网络连接正常)](#13-安装驱动请确保是网络连接正常)
  - [14. 安装桌面(这里安装的Gnome)](#14-安装桌面这里安装的gnome)
  - [15. 尾声](#15-尾声)
- [可能遇到的问题](#可能遇到的问题)

<!-- /code_chunk_output -->

如果是懒人或者小白，可以直接使用 Manjaro 系统，它是 Arch 的子项目，安装简单，Arch有的功能它基本都有，背靠 Arch，可以使用 AUR 仓库，配置简单。安装就和 Window 或者其它的 Linux 一样，只需要一个 ISO 镜像就可以，浪子推荐 Manjaro-kde 版本。如果你的硬件配置过低，又想充分利用资源的话，推荐使用 Manjaro-xfce 版本。KDE 桌面美观，占用资源多；xfce占用资源低，详情可以自行百度。

## 一、国内镜像列表

```bash{.line-numbers}
##
## Arch Linux repository mirrorlist
## Generated on 2021-07-27
##
## China
#Server = http://mirrors.163.com/archlinux/$repo/os/$arch
#Server = http://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.hit.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.hit.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirror.redrock.team/archlinux/$repo/os/$arch
#Server = https://mirror.redrock.team/archlinux/$repo/os/$arch
#Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
```

## 二、前言

下面的内容其实到第 ***13*** 步各位少侠就可以去找别的适合自己的教程了，这部分是需要结合自己电脑的配置情况去输入命令的。第 ***14*** 步安装桌面少侠们也可以去找自己喜欢的桌面风格样式。这里做个说明节省大家的时间。

## 三、准备工作

1. 镜像
2. 确保网络良好 **(最简单的就是使用手机数据线连接电脑，开启USB共享；条件允许使用网线最佳)**

## 四、开始安装

**注意，以下命令都是紧接着的（可以不看文字，跟着命令走，确保联网正常，命令输入正确），分步是为了给自己做个说明，也便于理解。我也加入了注释，分区部分的注释建议大家一定要看，浪子尽力做到能让第一次安装的少侠看明白。单个字母就是执行了 `fdisk /dev/sda` 命令后，我们手动输入的命令。**

### 1. 分区

```bash{.line-numbers}
fdisk /dev/sda    # 进入分区命令行，一直到最后的 w 命令，否则一直处于分区命令模式中

g                 # 该命令创建一个新的 gpt 分区表

n                 # 该命令新建分区，默认是1，直接回车即可

                  # 这一步是选择柱面，直接回车即可

+500M             # 给这个新建的gpt分区 500M 的空间，如果硬盘空间足够可以多给，后面我们会把分区格式化

n                 # 新建第二个分区，像上面一样，两次回车，大小给 +8G

+8G               # 给该分区分配 8G 内存，这个分区稍后我们将格式为交换分区

n                 # 建立第三个分区，然后三次回车键；分配剩余所有的空间，如果有需求可以再添加分区

p                 # 上面分区完了后，输入该命令可以看到刚才的分区信息

w                 # 确认无误后，w 保存退出；分区完成

```

> **注：** 上面三个分区中 500M 的引导分区是必须的，8G 的交换分区如果内存足够也可以不分，建议都分上，它会在内存不足的时候使用硬盘的部分空间当作虚拟内存使用。基本每种系统都有。剩下的分区就是我们操作的分区了。

### 2. 格式化分区

```bash{.line-numbers}

mkfs.fat -F32 /dev/sda1       # 上面分配的 500M 的分区格式，用来存放系统信息

mkswap /dev/sda2              # 上面分配的 8G 的分区格式为交换分区

mkfs.ext4 /dev/sda3           # ext4 文件格式的分区，这个分区就是我们直接操作的空间了
```

> **注：** 上面的 `sdax` 需要根据自己的实际分区情况来定，我这里仅仅是示例，可以看后面的注释部分，中途有确认项选择 `y` 即可。

### 3. 挂载分区

```bash{.line-numbers}
mount /dev/sda3 /mnt          # 把sda3挂载到mnt上

mkdir -p /mnt/boot/efi        # 创建多级目录

mount /dev/sda1 /mnt/boot/efi # 把sda1挂载到/mnt/boot/efi下

swapon /dev/sda2              # 这个好像是激活swap分区，我忘了；执行命令后，大家可以去查一下
```

### 4. 设置下载镜像源，提升我们后续的下载速度

```bash{.line-numbers}
# 把系统文件先做个备份
cp /etc/pacman.d/mirrorlist/etc/pacman.d/mirrorlist.bak
# 向文件添加内容，不理解没关系，执行命令就好
echo "https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch" > /etc/pacman.d/mirrorlist
```

### 5. 安装 linux 基本组件

```bash{.line-numbers}
pacstrap /mnt base base-devel linux linux-firmware dhcpcd iwd neworkmanager grub efibootmgr vim
```

### 6. 执行以下命令

```bash{.line-numbers}
genfstab -U /mnt >> /mnt/etc/fstab

# 查看一下是不是和自己开始的分区个数一致，一样代表写入成功；可以执行后续操作；
# 如果失败，请重新再来。一般来说，按照顺序执行正确命令到这里理论不会失败
# 在这里这么说是因为这一步比较重要，如果失败可以重新尝试一下
cat /mnt/etc/fstab
```

### 7. 切换到挂载点

```bash{.line-numbers}
arch-chroot /mnt
```

### 8. 设置时间

```bash{.line-numbers}
ln -sf /usr/share/zoneinfo/Asia/Shanghai/etc/localtime

hwclock --systohc
```

### 9. 设置root用户密码

```bash{.line-numbers}
passwd      # 之后输入想设置的密码
```

### 10. 设置语言和本机名

```bash{.line-numbers}
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" >> /etc/locale.conf

# 以上三行是设置语言，设置为英文是确保没有乱码，
# 因为刚装完的系统是没有中文环境的，如果这里设置中文，安装了以后没有中文字体，将会出现乱码。
# 安装完了后可以安装中文环境，然后进行设置

# 下面开始写入本机信息
echo "本机名，想要啥自己输入" >> /etc/hostname

grub-install --target=x86_64-efi --efi-directory=/boot/efi

grub-mkconfig -o /boot/grub/grub.cfg

# 创建一个新用户，以username为例(-m 自动生成用户主目录，-G 加入一个不存在的组 wheel)
useradd -m -G wheel username

# 设置密码
passwd username

# 然后编辑一个文件
vim /etc/sudoers

# 找到 %wheel ALL=(ALL)ALL (vim 编辑时输入 `/#%` 会跳到这儿)
去掉前面的注释 `#` ，完成后保存退出。
```

### 11. 退出

```bash{.line-numbers}
# 退出 arch-chroot 状态
exit

# 取消挂载
umount /mnt/boot/efi

umount /mnt

# 重启系统
reboot(记得拔掉 U 盘)
```

### 12. 前期准备

```bash{.line-numbers}
# 重启之后登录进系统，这时我们需要进行安装桌面系统的前期准备
sudo cp /etc/pacman.d/mirrorlist/etc/pacman.d/mirrorlist.bak

sudo vim /etc/pacman.d/mirrorlist
# 编辑文件加入下面的一行内容。加入的内容依然是我们之前提升下载速度所加入的
# 如果还是之前的设置内容，就不用添加了，保存退出即可。
Server = "https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch"

# 然后编辑 pacman.conf 文件
sudo vim /etc/pacman.conf
# 直接到最后，把带有下面的标签部分的的两行的注释取消掉，然后再加入以下内容
[multilib]
Include = /etc/pacman.d/mirrorlist

[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

# 按顺序执行以下命令吧
sudo systemctl start iwd

sudo systemctl start dhcpcd

sudo systemctl enable NetworkManager

sudo systemctl enable dhcpcd

sudo systemctl enable iwd

# 编辑文件
sudo vim /etc/NetworkManager/NetworkManager.conf
# 加入以下内容，保存退出
[device]
wifi.backend=iwd

# 输入 ip a 查看现在连上网没有
其实，如果按照我刚开始的联网方式，这里配置完也是连着网的。
```

### 13. 安装驱动(请确保是网络连接正常)

这一步大家可以根据自己的电脑 **硬件型号** 去选择，k可以百度也可以上 ArchWiki 查询；下面的桌面系统部分也可以选择自己喜欢的。

```bash{.line-numbers}
# haveged 是做 GPG 签名的
sudo pacman -Syu haveged

sudo systemctl start haveged

sudo systemctl enable haveged

sudo rm -rf /etc/pacman.d/gnupg

sudo pacman-key --init

sudo pacman-key --populate archlinux

sudo pacman -S archlinuxcn-keyring

# 我的CPU、显卡是intel 5 代核显，执行以下命令。
# 如果是其他的CPU 显卡请 自行 去官网查看或者百度，这个我也不懂，哈哈哈
# 显卡驱动
sudo pacman -S vulkan-intel lilb32-vulkan-intel mesa lib32-mesa

# 声卡驱动
sudo pacman -S alsa alsa-utils pulseaudio pulseaudio-alsa
```

### 14. 安装桌面(这里安装的Gnome)

```bash{.line-numbers}
sudo pacman -S cinnamon gnome gnome-extra
之后一路回车，出现选项选择 y
等待安装完成。

sudo pacman -S sddm

sudo systemctl enable sddm

sudo pacman -S way-microhei way-zenhei ttf-dejavu

sudo pacman -S google-chrome

现在就可以重启了，让我们怀着高兴紧张又期待的心情输入 reboot 回车吧！
```

### 15. 尾声

感谢教我的大佬，还记得大佬对我说，到这里基本的桌面和谷歌浏览器都帮我弄好了，剩下的就要靠我自己了。不知道为什么，当时听到这句话有一点么想哭。
然后我听大佬的用的 cinnamon,大佬还给我发了截图。我看过去的第一眼，哇塞！好漂亮！！！然后大佬对我说，你需要自己去设置，默认的很丑，哈哈哈哈。
犹记大佬最后对我说 **你要学会靠自己，要会自己解决问题**，我会尽力的。
确实，自己手动安装一遍的收获真的是很大啊，我很佩服教我的大佬，并不仅仅是因为他教我安装 arch。感谢大佬，比心 \^o^

## 可能遇到的问题

1. 如果你插入了一个 **ntfs** 格式的硬盘，Linux 识别不了，不能挂载的话，请安装 **ntfs-3g** ；命令：`yay -S ntfs-3g`
2. 未完待续。。。
