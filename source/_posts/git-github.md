---
title: Git/Github
date: 2015-05-20 20:07:50
tags: 
 - Git
 - GitHub 
category: 技术 
---

# Git 
* win7+   
在wind7下搭建git环境。需下载windows 版本的git工具。见附件。选择完整安装。安装目录 D:\Program Files (x86)\Git\
并将该路径添加至PATH环境。之后在cmd 环境下就可以使用了。后续如何使用git 就参考git 使用手册了，跟linux环境下的使用方法类似。
在windows 下生成id_rsa ，免密码连接github。使用 进入D:\Program Files (x86)\Git\git-bash.exe 命令，进入控制台窗口后，输入
`ssh-keygen -t rsa -b 4096 -C "zhoujinl@126.com"`
然后一直按回车默认，直至完成。
最后可以在C:\Users\zhoujl\.ssh 目录下生成ssh 密钥和公钥。 
然后到.ssh目录中，将生成的id_rsa.pub里面的内容，复制到github网站中的 SSH KEYs 设置中。
git 全局设置
>git config --global user.name zhoujinl
git config --global user.email zhoujinl@126.com  

在公司内部，有时需要通过设置代理才能连接上外网。
设置代理
`git config --global http.proxy http://user:pwd@server.com:port`
取消代理
`git config --global (or --system or --local)  --unset  http.proxy `
* Unix  
这里主要介绍linux版本。其余的类linux应该类似。以CentOs环境为例，通过yum命令即可快速安装。
`yum install git`

<!-- more -->

# GitHub
GitHub是一个以git作为版本控制器的代码托管社区。我们可以在上面创建项目，然后通过git来进行开发维护和协作。具体的使用方法，可参考网站：http://www.worldhello.net/gotgithub/index.html 有详细介绍。
PS:在设置完user.name和user.email之后，通过命令生成id_ras.pub，
`ssh-keygen -t rsa -b 4096 -C "zhoujinl@126.com"`
并且上传到github 官网中。然后通过ssh 方式，从github上面clone项目到本地。由于已经将id_ras.pub上传到到github中，因此可以免密码登陆下载。
验证是否创建成功，如出现以下结果则说明ssh免密码设置成功。
>$  ssh -T git@github.com
Hi zhoujinl! You've successfully authenticated, but GitHub does not provide shell access.

然后即可从github上面clone项目下来
`git clone git@github.com:zhoujinl/zhoujinl.github.io.git`
注：Are you sure you want to continue connecting (yes/no)?  会要求你是否确认连接，输入yes

另外也可以使用https的方式。从github上克隆项目下来，并且push的时候，要求每次都输入用户名密码
`git clone https://github.com/zhoujinl/zhoujinl.github.io.git`


# Git使用技巧：
>git pull <远程主机名> <远程分支名>:<本地分支名>
比如，取回origin主机的next分支，与本地的master分支合并，需要写成下面这样。
git pull origin next:master
git push <远程主机名> <本地分支名>:<远程分支名>
本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建
git push origin master
注意，分支推送顺序的写法是<来源地>:<目的地>，所以git pull是<远程分支>:<本地分支>，而git push是<本地分支>:<远程分支>。

* Git撤销操作：
    HEAD 最近一个提交
    HEAD^ 上一次
1. 处于add之后，未commit之前  
git reset HAED <file>  
2. commit之后，还未push之前  
>a.先看日志 git log ,找到要回退之前的版本的commit-id
λ git log
commit 992de5dde4f29aa35eb642b8fd55089838572c2d   ###此次错误提交的版本
Author: zhoujl <zhoujl@ffcs.cn>
Date:   Wed Aug 9 16:40:59 2017 +0800
Local change
commit 931c4a4e7678b760faa6849e16c849d7c679ac9c       ##上一个版本，应该使用该id
Author: arganzheng <arganzheng@gmail.com>
Date:   Thu Jun 18 20:21:51 2015 +0800
small modify
b.执行撤销操作
`git reset --hard 931c4a4e7678b760faa6849e16c849d7c679ac9c `    
**注意：**工 作区和暂存区的内容都会被重置到指定提交的时候，如果不加--hard则只移动HEAD的指针，不影响工作区和暂存区的内容。
3. push之后，如何回退：--待确认
	在步骤2的基础上，执行   git push origin HEAD --force
