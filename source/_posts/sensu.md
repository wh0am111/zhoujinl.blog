---
title: Sensu 监控体系介绍
date: 2018-03-27 12:51:23
tags: 
 - Sensu
 - Ruby
categories: 技术
---

## 1.Sensu 安装部署

### 主要特性

[首页介绍](https://sensuapp.org/docs/1.2/overview/what-is-sensu.html)

Sensu was designed to provide a comprehensive monitoring platform for monitoring infrastructure (servers), services, application health, and business KPIs — and to collect and analyze custom metrics. 

主要包含以下功能与特性：

- 检查系统、服务和程序的运行状态。
- 基于分布式的设计，能够轻松的动态伸缩规模。
- 支持通过插件的形式自定义检查的内容，拥有丰富的插件库。
- 收集信息，获取被监控节点上的各项数据指标等。
- 可视化的操作界面，提供实时的 GUI 用于显示和操作相关信息。
- 内置的集成工具，可用于和其它系统集成，如 PagerDuty、Graphite、Email 等。
- 提供丰富的 API 接口，支持通过 API 调用访问事件和客户端信息，触发检测等。
- 加密的安全通信，支持各种复杂的网络拓扑。

包括有类似我们拨测的功能，可对服务可用性进行检查，例如指定页面进行访问。

结合其他软件：Sensu 从监控节点获取数据，将数据格式化成 Graphite 要求的格式，然后通过调用 Graphite 的接口将数据发给 Graphite。Graphite 将数据存储在时序数据中供 Grafana 绘图使用。最终用户通过在 Grafana 中[定制](http://www.liuhaihua.cn/archives/tag/private)需要显示的数据及显示的方式，获得最终的可视化图表。

------

### 架构

Sensu 由服务器、客户端、RabbitMQ、Redis 和 API 这五个部分构成。图 1 展示了这些组件之间的关系以及通信的数据流。如图所示，RabbitMQ 用于组件之间的通信，Redis 用于持久化 Sensu 服务器和 Sensu API 的数据。因为客户端都是通过文件进行配置，并且不需要在服务器端配置客户端的信息，所以可以很轻易的增加和减少客户端的数量。由于 Sensu 的服务器和 API 原生支持多节点部署，所以不存在效率的瓶颈问题。从图中可以看到，为了解耦服务器和客户端，通信都是通过 RabbitMQ 进行的。

![组成部分](/img/sensu组成部分.png)

核心代码由Ruby编写。[Github地址](https://github.com/sensu/sensu)

实际安装中，不一定要使用RabbitMq作为消息中间件。直接使用redis即可。以下安装使用redis。

<!-- more -->

### 安装部署

server/clent 支持平台有

![支持平台](/img/sensu-support-platforms.png)



Server安装部署：

安装配置yum源 /etc/yum.repos.d/sensu.repo

```shell
[sensu]
name=sensu
baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
gpgcheck=0
enabled=1
```

```shell
yum install redis -y
yum install sensu -y
```

sensu server 配置 /etc/sensu/config.json

```
{
  "transport": {
    "name": "redis"
  },
  "api": {
    "host": "127.0.0.1",
    "port": 4567
  }
}
```

sensu client/server配置transport.redis

```shell
{
  "redis":
    {
       "host": "127.0.0.1",
       "port": 6379
    }
}
```

sensu client 配置 /etc/sensu/conf.d/client.json

```
{
  "client": {
    "name": "rhel-client",
    "address": "127.0.0.1",
    "environment": "development",
    "subscriptions": [
      "dev",
      "rhel-hosts"
    ],
    "socket": {
      "bind": "127.0.0.1",
      "port": 3030
    }
  }
｝
```

sensu dashboard配置 /etc/sensu/dashboard.json

```
{
  "sensu": [
    {
      "name": "sensu",
      "host": "127.0.0.1",
      "port": 4567
    }
  ],
  "dashboard": {
    "host": "0.0.0.0",
    "port": 3000
  }
}
```

启动进程

```shell
systemctl enable sensu-{server,api,client} 
systemctl start sensu-{server,api,client}
systemctl restart sensu-{server,api,client}
```

uchiwa 图形展示界面，需要额外安装。sensu提供了两个版本的展示界面，一个是企业版，一个就是uchiwa 。这里使用uchiwa，[下载链接](https://uchiwa.io/#/download)。 下载完成后，使用yum命令安装

```
yum -y localinstall uchiwa-1.1.1-1.x86_64.rpm
systemctl enable uchiwa
systemctl start uchiwa
```

配置好 uchiwa  /etc/sensu/uchiwa.json

```shell
{
  "sensu": [
    {
      "name": "Sensu-Server-1",
      "host": "127.0.0.1",
      "port": 4567,
      "timeout": 10
    }
  ],
  "uchiwa": {
    "host": "0.0.0.0",
    "port": 3000,
    "refresh": 10
  }
}
```

[URL](http://192.168.14.165:3000)

![图例](/img/uchiwa.png)



uchiwa 只是简单的展示功能，并不能对client等进行增删操作，即相当于对Sensu-API 只实现了Get相关的功能

[HTTP API用例](https://sensuapp.org/docs/1.2/api/overview.html)

### 部署过程中遇到的问题

- uchiwa无法展示client和server，大部分都是json配置文件没有配置清楚导致。可查看日志/var/log/uchiwa，/var/log/sensu/sensu-client.log



## 2.Sensu 使用

### Sensu Client

Sensu Client 就是采集客户端agent。

[介绍博文](https://blog.csdn.net/houzhe_adore/article/details/48518825)

### Proxy Clients



### Sensu Check

Sensu checks allow you to monitor server resources, services, and application health, as well as collect & analyze metrics; they are executed on servers running the Sensu client. Sensu checks allow you to monitor server resources, services, and application health, as well as collect & analyze metrics; they are executed on servers running the Sensu client. 

首先安装check插件

```shell
sensu-install -p process-checks	
sensu-install -p cpu-checks
```

插件是ruby脚本，安装后会在/opt/sensu/embedded/bin/ 路径下下载安装rubu解释器和check插件。process-checks 是进程检测插件，cpu-checks 是cpu检测指标插件。

[github插件脚本](https://github.com/sensu-plugins/sensu-plugins-process-checks)

配置check 服务端路径 /etc/sensu/conf.d/

检查sensu进程个数 check_cron.json

```shell
{
  "checks": {
    "cron": {
      "command": "check-process.rb -p cron",
      "standalone": true,
      "subscribers": [
        "production"
      ],
      "interval": 60
    }
  }
}
```

收集cpu指标 metrics_cpu.json

```
{
  "checks": {
    "cpu_metrics": {
      "type": "metric",
      "command": "metrics-cpu.rb",
      "subscribers": [
        "production"
      ],
      "interval": 10,
      "handler": "tcp_socket"
    }
  }
}

```

其中 "standalone": true 表示允许client 单独调度运行，不需要server 调度。"type": "metric" 表明这是指标类型的采集。

在/var/log/sensu/sensu-client.log 、sensu-server.log、HTTP API 可以看到执行结果。后续可再对结果进行处理，需借助另外的插件[**sensu-plugins-graphite**](https://github.com/sensu-plugins/sensu-plugins-graphite) . 

*采集一个指标展示，真累。。。*

接着通过redis查看cpu指标数据

```shell
redis-cli
keys * 
get result:tick-client:cpu_metrics
"{\"type\":\"metric\",\"command\":\"metrics-cpu.rb\",\"subscribers\":[\"production\"],\"interval\":10,\"handler\":\"tcp_socket\",\"standalone\":true,\"name\":\"cpu_metrics\",\"issued\":1524824239,\"executed\":1524824239,\"duration\":0.073,\"output\":\"tick.cpu.total.user 8057 1524824240\\n...\",\"status\":0}"
```



也可以直接到脚本安装路径，运行ruby脚本，直接看看采集结果

```shell
[root@tick bin]# /opt/sensu/embedded/bin/ruby  check-cpu.rb
CheckCPU TOTAL OK: total=0.25 user=0.0 nice=0.0 system=0.25 idle=99.75 iowait=0.0 irq=0.0 softirq=0.0 steal=0.0 guest=0.0 guest_nice=0.0
```



### Sensu data output

Metric collection checks are used to collect measurements from server resources, services, and applications. Metric collection checks can output metric data in a variety of metric formats:

- [Graphite plaintext](http://graphite.readthedocs.org/en/latest/feeding-carbon.html#the-plaintext-protocol)
- [Nagios Performance Data](http://nagios.sourceforge.net/docs/3_0/perfdata.html)
- [OpenTSDB](http://opentsdb.net/docs/build/html/user_guide/writing.html)
- [Metrics 2.0 wire format](http://metrics20.org/spec/)

The output produced by the check `command` ，貌似不同的插件返回不同的结果集。



### Sensu event processor

The Sensu server provides a scalable event processor. Event processing involves conversion of [check results](https://sensuapp.org/docs/latest/reference/checks.html#check-results) into Sensu events, and then applying any defined [event filters](https://sensuapp.org/docs/latest/reference/filters.html), [event data mutators](https://sensuapp.org/docs/latest/reference/mutators.html), and [event handlers](https://sensuapp.org/docs/latest/reference/handlers.html). All event processing happens on a Sensu server system.



### Sensu event handlers

Sensu event handlers are for taking action on [events](https://sensuapp.org/docs/latest/reference/events.html) (produced by check results), such as sending an email alert, creating or resolving a PagerDuty incident, or storing metrics in Graphite. There are several types of handlers: `pipe`, `tcp`, `udp`, `transport`, and `set`.

- **Pipe handlers** execute a command and pass the event data to the created process via `STDIN`.
- **TCP and UDP handlers** send the event data to a remote socket.
- **Transport handlers** publish the event data to the Sensu transport (by default, this is RabbitMQ).
- **Set handlers** are used to group event handlers, making it easier to manage many event handlers.

类似于告警。

这里使用TCP handlers 作为验证，需安装nc 命令。开启两个终端，分别运行以下命令，一个是作为server端，监听6000端口，一个是客户端，往监听端口发送数据。

```shell
 nc -l -k -4 -p 6000      
 echo "testing" | nc localhost 6000
```

新建TCP 配置文件 /etc/sensu/conf.d/tcp_socket.json

```sh
{
  "handlers": {
    "tcp_socket": {
      "type": "tcp",
      "socket": {
        "host": "localhost",
        "port": 6000
      }
    }
  }
}
```

注意，需修改对应的check 配置，把handler 修改为 *tcp_socket*  

例如，cron 进程检测的异常情况，即进程不存在![Check Error Event](/img/check-error-event.png)



### Sensu API

常用的API 

```shell
curl -s http://127.0.0.1:4567/clients | jq .
curl -s http://localhost:4567/events | jq .
curl -s http://localhost:4567/results | jq .
curl -s http://localhost:4567/client | jq .
```



## 3.关注事项

1. 数据收集方式（Pull、Push）   

   默认都是Push，由Client发送采集数据到transport，服务端再去transport 读取数据。

2. 数据格式

   check-result-cpu.json

3. 数据存储方式

   数据存储在transport 中，可以固化到文件系统

4. 数据输出方式

   checks 返回的结果是有一定格式的，其中，output字段存储的是对应的实际插件执行的命令输出结果。不同插件返回的命令结果不同。

5. agent 部署方式

   并未提供部署方案。

6. 任务下发方式

   客户端配置任务。无法从通过sensu-server直接下发到客户端。