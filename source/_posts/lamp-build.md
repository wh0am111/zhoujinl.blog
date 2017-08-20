---
title: LAMP站点搭建过程分析
date: 2017-03-20 19:34:19
tags: LAMP
category: 技术 
---
LAMP 即 LINUX + APACHE + MYSQL +PHP 搭建网站的四大组件。下面详细介绍其普通的安装方法和配置。

# LINUX 
咳，操作系统安装，可以使用VirtualBox创建虚拟机即可。这个就不细说了。推荐使用Centos6+

# Apache httpd  
1. 安装
`yum install httpd`
2. 配置
查找出Httpd相关配置文件 `rpm -ql httpd|grep '/etc'`
根据上述命令，查询主要的配置文件有/etc/httpd/conf/httpd.conf。 logs是日志链接目录。modules是支持的模块库目录。run是进程pid文件。
3. 可执行文件
>/usr/sbin/apachectl  #这个就是 Apache 的主要执行档，这个执行档其实是 shell script 而已， 他可以主动的侦测系统上面的一些设定值，好让你启动 Apache 时更简单！
/usr/sbin/httpd     #这个才是主要的 Apache 二进制执行文件啦！
/usr/bin/htpasswd   #(Apache 密码保护) 在某些网页当你想要登入时你需要输入账号与密码对吧！那 Apache 本身就提供一个最基本的密码保护方式， 该密码的产生就是透过这个指令来达成的！相关的设定方式我们会在 WWW 进阶设定当中说明的。
4. 网页数据
/var/www 目录下，html是放置网页内容的目录，error是错误信息的内容，icons是自带的一些图标目录，cgi-bin是默认网页可执行程序放置的地方。

<!-- more -->

# MYSQL   
1. 安装 
`yum install mysql mysql-server`
2. 配置 
查找配置文件与Apache Httpd 类似的，这里就不赘述了。主要配置文件是/etc/my.cnf 通过查看该文件，可看到日志的信息等。
`cat /etc/my.cnf `
>[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
#Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# PHP 
1. 安装 
`yum install php php-mysql`
2. 配置
/etc/php.ini
>/etc/php.ini #就是 PHP 的主要配置文件，包括你的 PHP 能不能允许使用者上传档案？能不能允许某些低安全性的标志等等， 都在这个配置文件当中设定的啦！
/usr/lib64/httpd/modules/libphp5.so # PHP 这个软件提供给 Apache 使用的模块！这也是我们能否Apache 网页上面设计 PHP 程序语言的最重要的咚咚！ 务必要存在才行！
/etc/php.d/mysql.ini, /usr/lib64/php/modules/mysql.so #你的 PHP 是否可以支持 MySQL 接口呢？就看这两个东西啦！这两个咚咚是由php-mysql 软件提供的呢！
/usr/bin/phpize, /usr/include/php/ #如果你未来想要安装类似 PHP 加速器以让浏览速度加快的话，那么这个档案与目录就得要存在， 否则加速器软件可无法编译成功喔！这两个数据也是php-devel 软件所提供的啦！

>[root@jinqiu mysql]# rpm -ql php
/etc/httpd/conf.d/php.conf
/usr/lib64/httpd/modules/libphp5.so
/var/lib/php/session
/var/www/icons/php.gif  

/etc/httpd/conf.d/php.conf 是与apache相关联的配置文件，不需要手动写入/etc/httpd/conf/httpd.conf文件中，httpd服务会自动到/etc/httpd/conf.d 目录下读取该文件。

以上就是基本的LAMP的搭建过程。具体的httpd.conf要再去查阅相关文档。建议安装http-manual软件，提供操作手册,可查询。
同时为支持其他脚本语言，也可安装mrtg画图软件，mod-perl ,mod-python,mod-ssl。

# 初试网站搭建
当上述四个步骤都操作完成之后，接下来就是验证功能咯。各位请看
* 静态网页   

在/var/www/html 目录下新建php文件，演示如下。并通过浏览器查询界面。
>[root@jinqiu html]# pwd
/var/www/html
[root@jinqiu html]# ls
hello  index.html  phpinfo.php
[root@jinqiu html]# cat phpinfo.php 
<?php phpinfo(); ?>

`<?php ... ?> `是嵌入在 HTML 档案内的 PHP 程序语法，在这两个标签内的就是 PHP 的程序代码。那么 phpinfo(); 就是 PHP 程序提供的一个函式库，这个函式库可 以显示出你 WWW 服务器内的相关服务信息， 包括主要的 Apache 信息与 PHP 信息等等  
* 简单动态网页
修改配置执行perl 脚本。
>vi /etc/httpd/conf/httpd.conf
AddHandler cgi-script .cgi .pl
<Directory "/var/www/html/cgi">
        Options +ExecCGI
        AllowOverride None
        Order allow,deny
        Allow from all
</Directory>


在/var/www/html/cgi 目录下新建perl文件
>[root@www ~]# mkdir /var/www/html/cgi  
[root@www ~]# vim /var/www/html/cgi/helloworld.pl
#!/usr/bin/perl
print "Content-type: text/html\r\n\r\n";
print "Hello, World.";
[root@www ~]# chmod a+x /var/www/html/cgi/helloworld.pl  #赋权 非常重要



# phpBB3 搭建论坛  
首先到官网上下载zip安装包，和汉化包。然后将安装包解压到/var/www/html apache网站目录下，重命名解压后的文件夹名称phpBB3。然后通过浏览器url: http://localhost:80/phpBB3/install/index.php 安装界面进行安装。 
在新建论坛之后，要记得对论坛赋权限。然后还有把网站下的install文件删除或者重命名。否则看不见论坛，处于关闭状态。

# Wordpress 搭建个人博客
首先到官网上下载zip安装包。与phpBB3一样，将其解压后的文件夹复制到/var/www/html/目录下，重命名为wordpress。然后到通过浏览器访问http://localhost:80/wordpress/ 访问，即可自动跳转至配置界面，进行数据库的配置，最后会在/wordpress目录下生成wp_config.php配置文件，然后进入安装界面。配置用户名密码即可完成安装。
ps:在配置过程中，如果出现没有权限生成wp_config.php。手动复制文件内容自己创建也可。我在安装过程中出现问题，就是配置完文件后，并且生成wp_config.php后，一直又重复回到配置界面。仿佛就像他没有检测到生成的wp_config.php文件一样，及时我重启httpd服务也是如此。后来我把/wordpress 目录下的文件全部删除，重新配置一把就可以了。
