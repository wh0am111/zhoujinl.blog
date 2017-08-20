---
title: Zookeeper 监控
date: 2017-08-20 20:07:50
tags: 
 - Zookeeper
 - Monitor 
category: 技术 
---
ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。
# Zookeeper 命令行监控方法  
[zookeeper monitor](http://zookeeper.apache.org/doc/r3.4.6/zookeeperAdmin.html#sc_zkCommands "zookeeper 4字节命令监控")

常见四字命令：
* `echo stat|nc 127.0.0.1 2181 `来查看哪个节点被选择作为follower或者leader
* `echo ruok|nc 127.0.0.1 2181 `测试是否启动了该Server，若回复imok表示已经启动。
* `echo cons | nc 127.0.0.1 2181 `列出所有连接到服务器的客户端的完全的连接/会话的详情。
* `echo wchs | nc 127.0.0.1 2181 `列出服务器 watch 的详细信息。

<!-- more -->

# Zookeeper Jmx监控方法  
需先启用JMX端口。修改zookeeper的启动脚本vim zkServer.sh。找到启动参数ZOOMAIN，修改为下面值。其中local.only=false，设为false才能在远程建立连接。
可使用Jconsole 通过IP、jmx端口连接到Zookeeper。