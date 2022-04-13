---
title: Git总结
categories: Git
cover: true
headimg: https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/git_command.jpg
abbrlink: f26bc7f6
---

Git 总结

<!-- more -->

# Git总结
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=true} -->

<!-- code_chunk_output -->

- [Git总结](#git总结)
  - [Git 配置](#git-配置)
  - [Git命令](#git命令)
    - [1. 远程代码托管平台相关（Github、Gitee、Gitlab）：](#1-远程代码托管平台相关githubgiteegitlab)
    - [2. 日常操作：](#2-日常操作)
  - [Git配置多个SSH-Key](#git配置多个ssh-key)
    - [1. 生成gitee使用的SSH-Key](#1-生成gitee使用的ssh-key)
    - [2. 生成Github用的SSH-Key](#2-生成github用的ssh-key)
    - [3. 配置](#3-配置)
    - [4. 用ssh命令分别测试](#4-用ssh命令分别测试)
  - [一张图总结](#一张图总结)

<!-- /code_chunk_output -->

## Git 配置

Git 下载安装略过，一路 next 即可。教程一般都需要配置 SSH KEY，这样不用每次 pull、push 都要输入账号密码。注意，是每次与Github、Gitee等这样的远程托管平台交互时都会弹出一个对话框让你输入。详细了解可以参看下方链接：

{% link Git官方文档::https://git-scm.com/book/zh/v2 %}
{% link 了解更多::https://zhuanlan.zhihu.com/p/347114235#:~:text=%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E5%88%86%20%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%20%E5%92%8C%20%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%20%EF%BC%8C%E5%85%B6%E4%B8%AD%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E7%9A%84%E5%8A%A0%E5%AF%86%E4%B8%8E%E8%A7%A3%E5%AF%86%20%E5%AF%86%E9%92%A5%E7%9B%B8%E5%90%8C%20%EF%BC%8C%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E7%9A%84%E5%8A%A0%E5%AF%86%E5%AF%86%E9%92%A5%E4%B8%8E%E8%A7%A3%E5%AF%86%20%E5%AF%86%E9%92%A5%E4%B8%8D%E5%90%8C,%E5%92%8C%20%E6%8E%A5%E6%94%B6%20%E5%8F%8C%E6%96%B9%E9%83%BD%E4%BD%BF%E7%94%A8%E8%BF%99%E4%B8%AA%E5%AF%86%E9%92%A5%E5%AF%B9%E6%95%B0%E6%8D%AE%E8%BF%9B%E8%A1%8C%20%E5%8A%A0%E5%AF%86%20%E5%92%8C%20%E8%A7%A3%E5%AF%86%20%E3%80%82%20%E8%BF%99%E5%B0%B1%E8%A6%81%E6%B1%82%E5%8A%A0%E5%AF%86%E5%92%8C%E8%A7%A3%E5%AF%86%E6%96%B9%E4%BA%8B%E5%85%88%E9%83%BD%E5%BF%85%E9%A1%BB%E7%9F%A5%E9%81%93%E5%8A%A0%E5%AF%86%E7%9A%84%E5%AF%86%E9%92%A5%E3%80%82 %}

```bash{.line-numbers}
$ git config --global user.name "你的名字或昵称"
$ git config --global user.email "你的邮箱"

# 列出所有配置信息：
$ git config --list
# 列出某一项配置：
$ git config user.email
# 取消某一项全局配置
$ git config --global --unset user.name

# 乱码问题，这个问题出现了可以参考这个设置，安装后不用立即设置
# 文件名显示错误
$ git config --global core.quotepath false
# 输出日志乱码：
$ git config --global i18n.commitEncoding utf-8
$ git config --global i18n.logOutputEncoding utf-8

# core.autocrlf 这个设置主要是换行符问题，不过新版的 Git 不用设置也行
# 浪子使用的 2.34 版本，没有任何问题。可以酌情配置
# 注意自己使用的操作系统，不同的系统配置不同。
# Window 设置 (提交时自动转换为LF，检出时LF转为CRLF)
$ git config --global core.autocrlf true
# Mac、Linux 设置 (提交时CRLF替换为LF，检出时无操作)
$ git config --global core.autocrlf input
```
> 检出: 可以理解为下载/克隆/pull命令


## Git命令

### 1. 远程代码托管平台相关（Github、Gitee、Gitlab）：
```bash{.line-numbers}
# 克隆远程库到本地
$ git clone https://github.com/个性地址/仓库名称.git
# 编辑文件提交
# 将当前目录所有文件添加到git暂存区，可以一次提交多个文件，使用空格分开
$ git add .
# 提交到版本库并备注提交信息
$ git commit -m "my first commit" 
# 将本地提交推送到远程仓库,如果本地和远程的分支名称相同，后面的可以省略
$ git push origin master:master # 第一个 master 是本地分支，后面的一个是远程分支

# 关联远程仓库
$ git remote add origin https://gitee.com/用户个性地址/HelloGitee.git
# 取消远程关联
$ git remote remove origin

```

### 2. 日常操作：
```bash{.line-numbers}
# 删除提交到暂存区的内容：
$ git rm --cache 文件名

# 将已提交的版本库（commit）、暂存区（add）、工作区（本地）的内容恢复到指定的版本：
$ git reset --hard 版本ID

# 只撤销已提交的版本库、暂存区的内容，不会修改工作区：
$ git reset --mixed 版本ID

# 只撤销已提交的版本库，不会修改暂存区和工作区
$ git reset --soft 版本库ID

# 克隆指定的分支
$ git clone -b 分支名称 仓库地址

# 查看文件差异：
# 查看工作目录与暂存区的不同
$ git diff
# 查看暂存区与已提交的文件不同            
$ git diff --cached
# 比较工作区和当前最近历史提交   
$ git diff HEAD         
# 移除文件
# 从暂存区移除文件，会把工作区相应文件一并删除，回收站无法还原。如果该文件没有提交，则无法删除，可以使用 ***==-f==*** 选项强制删除。
$ git rm
# 只删除暂存区文件，保留工作区文件            
$ git rm --cached  
  
# 查看提交历史：
$ git log
# 图形显示
$ git log --graph 
# 单行显示、短格式、长格式
$ git log --pretty=oneline/short/full

# 切换分支：
$ git checkout 分支名 / git switch branch_name
# 删除本地分支
$ git branch -d branch_name
# 删除远程分支
$ git push origin --delete branch_name
# 把本地分支推送到远程存储库，远程不存在分支
$ git push -u origin branch_name
# 把本地分支推送到远程存储库，远程不存在分支，自定义远程分支名
$ git push -u origin 本地分支名:远程分支名

# 查看本地所有分支
$ git branch -a
# 查看远程分支
$ git branch -r
# 查看本地与远程分支的一映射关系
$ git branch -vv
# 创建并切换到改分支并建立映射关系
$ git checkout -b 本地分支名x origin/远程分支名x
```

在新建仓库时，如果在 Gitee 平台仓库上已经存在 readme 或其他文件，在提交时可能会存在冲突，这时用户需要选择的是保留线上的文件或者舍弃线上的文件，如果您舍弃线上的文件，则在推送时选择强制推送，强制推送需要执行下面的命令(默认不推荐该行为)，如果是团队开发，那么执行此条命令可能面临生命危险：
`$ git push origin master -f`

如果您选择保留线上的 readme 文件,则需要先执行：
`$ git pull origin master`

> 如果是多人合作，而且恰好你又是 Git 新手的话，推荐每次推送时先执行 pull 命令更新，然后执行 push 命令。


## Git配置多个SSH-Key

生成多个SSH-KEY与不同的远程托管库交互。
### 1. 生成gitee使用的SSH-Key
```bash
# Linux Mac
$ ssh-keygen -t rsa -C 'xxxxx@company.com' -f ~/.ssh/gitee_id_rsa
# window
$ ssh-keygen -t rsa -C 'xxxxx@company.com' -f C:\Users\用户名\.ssh\gitee_id_rsa
```
### 2. 生成Github用的SSH-Key
```shell
# Linux Mac
$ ssh-keygen -t rsa -C 'xxxxx@qq.com' -f ~/.ssh/github_id_rsa
# window
$ ssh-keygen -t rsa -C 'xxxxx@qq.com' -f C:\Users\用户名\.ssh\github_id_rsa
```
### 3. 配置

在 ~/.ssh 目录下新建一个 `config` 文件，添加如下内容（其中Host和HostName填写git服务器的域名，IdentityFile指定私钥的路径）

```bash{.line-numbers}
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa

# github1
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
# github2
Host xyzgithub.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```
> 如果像上面一样配置了多个Github账号的SSH密钥，那么在进行 clone 或者 remote 关联时，地址需要做一些改变，否则会报错。
> 克隆时 git@github.com 需要替换为对应的 `Host` 内容。例如使用 github2 的账户关联远程仓库：
`git remote add origin git@xyzgithub.com:XXX/repository.git`
> Window 下 IdentityFile 指定的路径示例：`IdentityFile C:\Users\用户名\.ssh\github_id_rsa`，其实可以完全和 Linux 一样的路径。

### 4. 用ssh命令分别测试
```bash
$ ssh -T git@gitee.com
$ ssh -T git@github.com
$ ssh -T git@xyzgithub.com
```
这里以gitee为例，成功的话会返回下图内容：
![](https://images.gitee.com/uploads/images/2018/0921/161137_b71ef6be_967230.png)

## 一张图总结

{% gallery %}
![git命令总结](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/git_command.jpg "git命令总结")
{% endgallery %}