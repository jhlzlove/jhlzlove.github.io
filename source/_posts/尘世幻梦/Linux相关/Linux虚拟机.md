---
title: Linux虚拟机问题
categories:
  - Java
  - Linux
  - 虚拟机
abbrlink: 75091ad3
---

centos7
问题描述：之前内网都通，使用xshell等终端工具突然连不上虚拟机了也连不上外网了

解决方法：
```bash
# 1.将networkmanager服务停掉并禁止其自启
systemctl stop NetworkManager
systemctl disable NetworkManager

# 2.重启网卡服务
systemctl restart network
```

[原博客地址](https://blog.csdn.net/weixin_44695793/article/details/108089356)
