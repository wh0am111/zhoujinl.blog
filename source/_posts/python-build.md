---
title: Python编程环境搭建
date: 2016-08-19 22:13:30
tags: Python
categories: 技术
---

# 简介  #
假如新建了一台CentOs虚拟机，作为纯净的操作系统，我们需要搭建基本的开发环境。CentOs6.x一般都是java1.5版本，python2.6，并且很多开发工具都是没有默认安装的，比如gcc。下面重点介绍Python环境的搭建，主要涉及Python升级，pip安装，virtualenv安装。
# yum #
yum 源更新，可选操作，可替换国内的yum源。
yum groupinstall 。必选操作，编译python需要编译工具。还有一些必要的库文件需要安装，例如openssl,zlib,这些都需要额外安装。pip要安装python2.7版本，因此需要zlib需要更新到最新版本1.2.8，而CentOs默认是1.2.3。否则会报错，错误信息后面提示。
```
[root@jinqiu opt]# yum -y update
[root@jinqiu opt]# yum groupinstall "Development tools"
[root@jinqiu opt]# yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
```
顺便简单提下，快速免验证安装jdk
```
1. wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm  
2. rpm -ivh jdk-8u131-linux-x64.rpm
```
# python #
保存2.6版本的python
下载2.7版本的python源码文件，解压，然后配置，编译，安装
```
[root@jinqiu~]$ cd /opt
[root@jinqiuopt]$ sudo wget --no-check-certificate https://www.python.org/ftp/python/2.7.6/Python-2.7.6.tar.xz
[root@jinqiuopt]$ sudo tar xf Python-2.7.6.tar.xz 
[root@jinqiuopt]$ cd Python-2.7.6
[root@jinqiuPython-2.7.6]$ sudo ./configure --prefix=/usr/local
[root@jinqiuPython-2.7.6]$ sudo make && sudo make altinstall
```
<!-- more -->

创建软链接到2.7版本的python
修改yum文件，由于yum只支持2.6版本的python，因此需要将/usr/bin/yum中的默认python解释器修改为2.6版本。
```
[root@jinqiu opt]# mv /usr/bin/python /usr/bin/python2.6
[root@jinqiu opt]# ln -s /usr/bin/python2.7  /usr/bin/python
```

从一开始，如果要做一些实际Python开发，你一定会用到一些第三方包。
在Linux系统上至少有3种安装第三方包的方法。
* 使用系统自带的包管理系统(deb, rpm, 等)
* 通过社区开发的各种工具，例如 pip ， easy_install 等
* 从源文件安装
这三个方面，几乎完成同样的事情。即：安装依赖，编译代码（如果需要的话），将一个包含模块的包复制的标准软件包搜索位置。

# 安装pip #
```
[root@jinqiu opt]# yum install python-pip
[root@jinqiu opt]# pip install --upgrade pip
```
通过yum安装，可能安装的pip版本比较低。因此也可以通过网络上最新的pip源进行安装
```
curl https://raw.githubusercontent.com/pypa/pip/master/contrib/get-pip.py | python2.7 -
```
pip用法这里不再赘述，man pip 即可。

# virtualenv #
安装virtualenv，virtualenv作为python环境的隔离器，可以保证在这个环境下安装的第三方python库，都是在这个环境下的，而不会安装到系统路径上面去。
我们需要处理的基本问题是包的依赖、版本和间接权限问题。想象一下，你有两个应用，一个应用需要libfoo的版本1，而另一应用需要版本2。如何才能同时使用这些应用程序？如果您安装到的/usr/lib/python2.7/site-packages（或任何平台的标准位置）的一切，在这种情况下，您可能会不小心升级不应该升级的应用程序。
```
sudo pip install virtualenv
mkdir my_project_venv
virtualenv --distribute my_project_venv
```
** The output will something like:**
New python executable in my_project_venv/bin/python
Installing distribute.............................................done.
Installing pip.....................done.

这里只列出了将被讨论的目录和文件
```
|-- bin
|   |-- activate  # <-- 这个virtualenv的激活文件
|   |-- pip       # <-- 这个virtualenv的独立pip
|   |-- python    # <-- python解释器的一个副本
|-- lib
|-- python2.7 # <-- 所有的新包会被存在这
```
通过下面的命令激活这个virtualenv：
```
cd my_project_venv
source bin/activate
```
执行完毕后，提示可能是这个样子的：
```
(my_project_venv)$ # the virtualenv name prepended to the prompt
通过 deactivate 命令离开virtualenv环境
(my_project_venv)$ deactivate
```
注意：一定要先进行yum步骤的操作，安装必要的openssl库和zlib库，否则在编译之后的python2.7，会有问题。例如pip2.7会报错，zlib异常。

# 异常处理：
pythonImportError: No module named zlib2.7，报错如下：
Traceback (most recent call last):
File "/usr/local/bin/pip", line 9, in <module>
   load_entry_point('pip==1.4.1', 'console_scripts', 'pip')()
File "build/bdist.linux-x86_64/egg/pkg_resources.py", line 378, in load_entry_point
File "build/bdist.linux-x86_64/egg/pkg_resources.py", line 2566, in load_entry_point
File "build/bdist.linux-x86_64/egg/pkg_resources.py", line 2260, in load
File "/usr/local/lib/python2.7/site-packages/pip-1.4.1-py2.7.egg/pip/__init__.py", line 10, in <module>
   from pip.util import get_installed_distributions, get_prog
File "/usr/local/lib/python2.7/site-packages/pip-1.4.1-py2.7.egg/pip/util.py", line 17, in <module>
   from pip.vendor.distlib import version
File "/usr/local/lib/python2.7/site-packages/pip-1.4.1-py2.7.egg/pip/vendor/distlib/version.py", line 13, in <module>
   from .compat import string_types
File "/usr/local/lib/python2.7/site-packages/pip-1.4.1-py2.7.egg/pip/vendor/distlib/compat.py", line 31, in <module>
   from urllib2 import (Request, urlopen, URLError, HTTPError,
ImportError: cannot import name HTTPSHandle
类似的错误还有：
ImportError: No module named zlib
Traceback (most recent call last):
  File "/usr/bin/pip", line 5, in <module>
    from pkg_resources import load_entry_point
ImportError: No module named pkg_resources
都是因为前面最早没有对系统进行升级，要进行Yum install openssl 和zlib 的安装升级。然后再重新安装python2.7，这样高版本的python 才不会有问题。
