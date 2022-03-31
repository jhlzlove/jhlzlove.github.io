---
title: Arch安装软件
categories:
  - Linux
  - Arch
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/blogCover/arch.jpeg
abbrlink: 8ecfb81b
---

Arch 安装软件可以去官网搜索，然后安装。
{% link Arch WIKI::https://wiki.archlinux.org/ %}

这里做一个浪子常用的整理吧，Arch Linux 可以安装的软件一般 Manjaro 也是可以安装使用的。

这里只是总结一些常用的软件安装方式。另外软件的更新比较快，不保证任何包名称以及版本等相关问题。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [Arch 安装一些常用软件](#arch-安装一些常用软件)
      - [1. AUR 助手 yay：](#1-aur-助手-yay)
      - [2. 中文输入法](#2-中文输入法)
      - [3. 字体](#3-字体)
      - [4. 网易云音乐](#4-网易云音乐)
      - [5. 火焰截图](#5-火焰截图)
      - [6. 谷歌浏览器](#6-谷歌浏览器)
      - [7. QQ、微信待测](#7-qq微信待测)
      - [8. 录屏软件](#8-录屏软件)
      - [9. 动图录制工具](#9-动图录制工具)
      - [10. 安装WPS](#10-安装wps)
      - [11. VSCode](#11-vscode)
      - [12. Edge](#12-edge)
      - [开发工具](#开发工具)

<!-- /code_chunk_output -->

# Arch 安装一些常用软件

#### 1. AUR 助手 yay：
yay 是使用 AUR 的命令，AUR中包含了一些 Window 的软件，有需要的就可以安装这个命令，然后使用该命令去安装软件。类似于 CentOS 的 yum，也是Arch特有的包管理工具。[详细了解](https://zhuanlan.zhihu.com/p/363666022) https://blog.zhullyb.top/2021/04/04/yay-more/
```bash{.line-numbers}
sudo pacman -S yay
```
yay 常用命令：
```bash{.line-numbers}
# 更新软件库
yay -Syu
# 这个命令将卸载该软件和没有被使用的依赖
pacman/yay -Rs package-name
# 清除缓存
yay -Sc
# 更新指定包软件
yay -u package-name
# 查看安装的软件
yay -Ps
```

#### 2. 中文输入法
```bash{.line-numbers}
yay -S fcitx5-im 
yay -S fcitx5-chinese-addons fcitx5-rime
```
编辑 `/etc/environment` 文件，没有的话可以手动创建，添加以下内容：
```bash{.line-numbers}
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS=@im=fcitx5
export SDL_IM_MODULE=fcitx5
```

如果喜欢也可以安装搜狗输入法
```bash{.line-numbers}
sudo pacman -S fcitx-sogoupinyin
sudo pacman -S fcitx-im
sudo pacman -S fcitx-configtool
```
编辑 `/etc/environment` 文件，添加以下内容：
```bash{.line-numbers}
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

#### 3. 字体
```bash{.line-numbers}
# 文泉驿字体
yay -S wqy-bitmapfont wqy-microhei wqy-zenhei wqy-microhei-lite

# 思源字体
yay -S noto-fonts-cjk adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
```

#### 4. 网易云音乐
```bash{.line-numbers}
yay -S netease-cloud-music
```

#### 5. 火焰截图
```bash{.line-numbers}
yay -S flameshot
```
可以给截图设置自启动和快捷键，这样使用方便，比如 `Alt+A`。

#### 6. 谷歌浏览器
```bash{.line-numbers}
yay -S google-chrome
```

#### 7. QQ、微信待测
```bash{.line-numbers}
```
QQ 微信的包有好多个，不同的机器安装不同版本会产生不同的问题，这个等我换了主力机之后实测更新。

#### 8. 录屏软件
```bash{.line-numbers}
yay -S vokoscreen
```

#### 9. 动图录制工具
```bash{.line-numbers}
yay -S peek
```

#### 10. 安装WPS
```bash{.line-numbers}
yay -S wps-office-cn
# 安装缺失字体
yay -S ttf-wps-fonts
# 安装中文语言包
yay -S wps-office-mui-zh-cn
```

#### 11. VSCode
```bash{.line-numbers}
yay -S visual-studio-code-bin
```

#### 12. Edge
```bash{.line-numbers}
git clone https://aur.archlinux.org/microsoft-edge-dev-bin.git

cd microsoft-edge-dev-bin

makepkg -si
```

#### 开发工具
```bash{.line-numbers}
yay -S git vim docker npm node yarn
yay -S jdk11-openjdk
# 如果有多个版本，设置JDK11为默认
sudo archlinux-java set java-11-openjdk
```

未完待续。。。