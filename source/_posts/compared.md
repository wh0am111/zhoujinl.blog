---
title: 开源监控体系对比
date: 2018-05-16 23:10:07
categories: 技术
---

## Prometheus、TICK、Sensu、Openfalcon 对比

**目的**

- 研究开源监控系统架构，完善现有的后台采集结构
- 利用开源监控体系中的采集插件，节省KM研究、开发新监控对象（各类分布式组件）的时间
- 使用开源监控系统为独立的一套监控系统，作为端到端等分布式应用场景的监控系统
- 与现有网管后台结合，将开源监控系统作为一类采集数据源



 OpenTSDB、Graphite 偏向数据存储，并无完整的监控体系（采集、存储、告警、展示）。

## 对比表格

以下列出几个重要的关注点

|                              |               Prometheus               |     TICK     |        Sensu         |      Openfalcon      |
| :--------------------------: | :------------------------------------: | :----------: | :------------------: | :------------------: |
|             开发语言             |                   Go                   |      Go      |         ruby         |          Go          |
| <a href="#数据收集方式">数据收集方式</a> |            Pull/Pushgateway            |     Push     |         Push         |         Push         |
|            采集客户端             |               各种Exporter               |   telegraf   | sensu-client<br>各种脚本 | Falcon-agent<br>各种脚本 |
|   <a href="#数据模型">数据模型</a>   |                key+tag                 |   key+tag    |         json         |       key+tag        |
|              存储              | 本地存储 <br>远程数据库(influxDB可读写/OpenTSDB只写) |   influxDB   |        redis         |    mysql/opentsdb    |
|          agent部署方式           |                   人工                   |      人工      |          人工          |          人工          |
|            任务更新策略            |                   无                    |      无       |          无           | dashbord可关联主机和pluins |
|   <a href="#数据查询">数据查询</a>   |                HTTP API                | SQL/HTTP API |  HTTP API/redis-cli  |         sql          |

<!-- more -->

## 数据收集方式

Prometheus 与众不同的采用Pull方式，由服务端主动从被监控机器上面抓取数据。Prometheus 也提供Pushgateway的方式，支持其他数据源主动推送数据到Pushgateway，Pushgateway只做数据缓存，之后仍然是等待服务端抓取。Exporter作为被监控对象暴露指标的客户端，运行在被监控对象上面，并且在Prometheus 服务端配置作为Job，需配置抓取地址端口和间隔。针对各种监控对象，可部署各种Exporter。

Pull和Push的主要区别在于Pull方式下，监控系统可以方便的增加删除被监控对象，但是需要被监控对象主动开放端口，这对防火墙/NAT方式下的被监控对象是特殊的要求。而Push模式下，每个采集客户端需要知道服务端的信息，才能上报数据。相比之下，网管后台使用Rabbitmq隔离了服务端的信息，但是仍然需要客户端配置Rabbitmq信息。

TICK使用telegraf作为数据采集端，由telegraf 根据input配置主动采集并发送到output配置的后端，一般是InfluxDB数据库。telegraf还能对采集数据做聚合过滤等处理。telegraf是Go开发的二进制可执行对象，各种对象的采集均打包在一起，通过指定input插件类型，可以指定采集某类指标。相比之下，网管后台使用Java+KM的方式，由Java负责进程调度，KM负责指定采集对象。

Sensu 也是采用Push 方式，Sensu由Client发送采集数据到transport，服务端再去transport 读取数据，transport一般使用Rabbitmq。这点跟网管有点类似。Client利用插件采集数据，是ruby脚本，需要额外安装。安装后会在/opt/sensu/embedded/bin/ 路径下下载安装rubu解释器和check插件。process-checks 是进程检测插件，cpu-checks 是cpu检测指标插件。采集配置较为繁琐。

Openfalcon 也是采用Push 方式，由Client发送采集数据到transport，与Sensu  不同的是，transport 会做一些数据规整，检查之后，转发到多个后端系统去处理。transfer目前支持的业务后端，有三种，judge、graph、opentsdb。judge是告警判定组件，graph作为高性能数据存储、归档、查询组件，opentsdb主要用作数据存储服务。falcon-agent本身仅支持主机类的指标采集，对于其他业务系统、开源软件，需独立的使用采集脚本，将数据按照规定格式，发送到falcon-agent 所在的监听端口。最终再由falcon-agent 转发到服务端。



综上，其实介绍了四种主要的采集逻辑。以两种不同对象的采集作为示例说明

- 主机类指标CPU
- 开源软件Redis

<a href="#对比表格">返回目录</a>

## 数据模型

架构的分层隔离，依赖于各层之间接口的规范。对于采集数据而言，Pull/Push 体现的是传输方式，而数据格式，则是数据组织的规范。以下是各个系统的数据格式。

- openfalcon  采用和OpenTSDB相似的数据格式：metric、endpoint加多组key value tags

```
{
    metric: load.1min,
    endpoint: open-falcon-host,
    tags: srv=falcon,idc=aws-sgp,group=az1,
    value: 1.5,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
```

其中，metric是监控指标名称，endpoint是监控实体，tags是监控数据的属性标签，counterType是Open-Falcon定义的数据类型(取值为GAUGE、COUNTER)，step为监控数据的上报周期，value和timestamp是有效的监控数据。

- prometheus    <metric name>{<label name>=<label value>, ...}

  prometheus 有四种类型的指标：counter、gauge、[summary、histogram](https://prometheus.io/docs/practices/histograms/) ，[示例](http://192.168.7.176:9090/metrics)

```
# HELP http_requests_total Total number of HTTP requests made.
# TYPE http_requests_total counter
http_requests_total{code="200",handler="alerts",method="get"} 2
http_requests_total{code="200",handler="config",method="get"} 1
http_requests_total{code="200",handler="flags",method="get"} 2

# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 235

# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 4.5975e-05
go_gc_duration_seconds{quantile="0.25"} 8.3846e-05
go_gc_duration_seconds{quantile="0.5"} 0.000100729
go_gc_duration_seconds{quantile="0.75"} 0.000123377
go_gc_duration_seconds{quantile="1"} 0.026837959
go_gc_duration_seconds_sum 1.990353198
go_gc_duration_seconds_count 10956

# HELP prometheus_tsdb_compaction_chunk_range Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range histogram
prometheus_tsdb_compaction_chunk_range_bucket{le="100"} 336
prometheus_tsdb_compaction_chunk_range_bucket{le="400"} 336
prometheus_tsdb_compaction_chunk_range_bucket{le="1600"} 336
prometheus_tsdb_compaction_chunk_range_bucket{le="6400"} 336
prometheus_tsdb_compaction_chunk_range_bucket{le="25600"} 2336
prometheus_tsdb_compaction_chunk_range_bucket{le="102400"} 269009
prometheus_tsdb_compaction_chunk_range_bucket{le="409600"} 271929
prometheus_tsdb_compaction_chunk_range_bucket{le="1.6384e+06"} 4.2524554e+07
prometheus_tsdb_compaction_chunk_range_bucket{le="6.5536e+06"} 4.3107242e+07
prometheus_tsdb_compaction_chunk_range_bucket{le="2.62144e+07"} 4.3107877e+07
prometheus_tsdb_compaction_chunk_range_bucket{le="+Inf"} 4.3107877e+07
prometheus_tsdb_compaction_chunk_range_sum 4.4190173791382e+13
prometheus_tsdb_compaction_chunk_range_count 4.3107877e+07
```

不同类型的指标，有不同的查询方法，需根据指标含义选择合适的查询方法

histogram_quantile(0.9, rate(prometheus_tsdb_compaction_chunk_range_bucket[10m]))

- Telegraf 支持多种输入输出格式，重点讲解InfluxDB Line Protocol，另外两种是json/Graphite

```
weather,location=us-midwest temperature=82 1465839830100400200
  |    -------------------- --------------  |
  |             |             |             |
  |             |             |             |
+-----------+--------+-+---------+-+---------+
|measurement|,tag_set| |field_set| |timestamp|
+-----------+--------+-+---------+-+---------+

> cpu,cpu=cpu0,host=0336dcb23579 usage_system=0,usage_idle=100,usage_nice=0,usage_iowait=0,usage_irq=0,usage_guest=0,usage_user=0,usage_softirq=0,usage_steal=0,usage_guest_nice=0 1515338025000000000
> cpu,cpu=cpu-total,host=0336dcb23579    usage_guest_nice=0,usage_idle=100,usage_nice=0,usage_irq=0,usage_steal=0,usage_guest=0,usage_user=0,usage_system=0,usage_iowait=0,usage_softirq=0 1515338025000000000
```

cpu：measurements ，类似oracle的表名  

cpu=cpu-total,host=0336dcb23579 ： cpu、host为tag，索引；    内部以，分割 

usage_guest_nice=0,usage_idle=100,usage_nice=0... : usage_guest_nice等为field，即被采集的指标字段。

***field在measurements是不可或缺的，tag可无， tag与field用空格隔开，field放置在后面***

1515338025000000000： time 时间戳

这里顺便提下 influx metric -> graphite 的转换格式

```
template = "host.tags.measurement.field"
```

```
cpu,cpu=cpu-total,dc=us-east-1,host=tars usage_idle=98.09,usage_user=0.89 1455320660004257758
=>
tars.cpu-total.us-east-1.cpu.usage_user 0.89 1455320690
tars.cpu-total.us-east-1.cpu.usage_idle 98.09 1455320690
```

- Sensu 指标数据，实际上是check的返回结果。应该是json字符串

```shell
{
    "client": "tick-client",
    "check": {
      "type": "metric",
      "command": "metrics-cpu.rb",
      "subscribers": [
        "production"
      ],
      "interval": 10,
      "handler": "tcp_socket",
      "standalone": true,
      "name": "cpu_metrics",
      "issued": 1525270655,
      "executed": 1525270655,
      "duration": 0.074,
      "output": "tick.cpu.total.user 738228 1525270655\n...",
      "status": 0,
      "history": [
        0,
		...
      ]
    }
}

```

output 就是chcek返回结果。由不同的指标采集插件返回，默认的返回格式是Graphite 格式。

```
sensu.cpu.user 0.50 1515534170
sensu.cpu.nice 0.00 1515534170
sensu.cpu.system 0.00 1515534170
sensu.cpu.idle 99.50 1515534170
sensu.cpu.iowait 0.00 1515534170
sensu.cpu.irq 0.00 1515534170
sensu.cpu.softirq 0.00 1515534170
sensu.cpu.steal 0.00 1515534170
sensu.cpu.guest 0.00 1515534170
```



<a href="#对比表格">返回目录</a>

## 数据查询

- Prometheus   HTTP API接口查询，查询结果类型："resultType": "matrix" | "vector" | "scalar" | "string"

  matrix ： 返回某个时间段内的指标数据集合

  vector：返回单个时间点的指标数据

  ```
  # Instant queries
  curl 'http://192.168.7.176:9090/api/v1/query?query=elasticsearch_node_stats_up&time=1525313513.452&_=1525313461038' -s | python -m json.tool

  # Range queries
  curl 'http://192.168.7.176:9090/api/v1/query_range?query=prometheus_tsdb_compaction_chunk_range_bucket&start=1525307102.216&end=1525307302.216&step=28&_=1525313461049' -s|python -m json.tool

  ```

  ​

- TICK  HTTP API查询/登陆数据库查询sql

  ```
  curl -G "http://192.168.7.176:8086/query?pretty=true" --data-urlencode "db=mydb" \
  --data-urlencode "q=SELECT * FROM cpu WHERE host='server01' AND time < now() - 1d"

  ##==>
  SELECT * FROM cpu WHERE host='server01' AND time < now() - 1d
  ```

  ​

- Sensu   HTTP API查询/登陆redis查询sql

  ```
  curl -s http://127.0.0.1:4567/clients | jq .
  curl -s http://localhost:4567/events | jq .
  curl -s http://localhost:4567/results | jq .
  curl -s http://localhost:4567/checks | jq .

  ###===>
  redis-cli
  keys * 
  get result:tick-client:cpu_metrics
  ```

  ​

- Openfalcon 

  从mysql数据库中取数

  ​

<a href="#对比表格">返回目录</a>

## 网管对接

1. 跟云资源采集类似，将上述监控系统作为一个数据采集中心。开发一个转储接口程序，从监控系统中（数据库或者REST API）读取配置、性能数据等，转换数据格式，存入网管系统。

   优点：可以方便对接第三方开源系统，利用其成熟的监控指标和稳定的监控体系。对原有的网管系统也不需要过多的改造，相当于改造重心在转储接口程序。

   ​ 缺点：需要额外维护多套监控系统以及对应的转换程序；对该转换程序尽可能做得通用，因为可以同时对接多套监控系统；对于指标的定义、配置，仍然需要沿用原来的方法，即还要开发KM、配置表单，这部分是无法避免的。



2. 仅复用监控系统的采集客户端，类似现在的Agent对接开源组件的模式。例如prometheus，可以复用其exporter。即prometheus exporter独立运行并暴露数据至指定监听端口，Agent开发http接口定时扫描读取该端口暴露的指标即可，然后由KM负责解析转换，最终存入网管系统。

   优点：仅仅用到第三方开源系统的采集客户端，抓取我们想要的数据。对原有的网管系统也不需要过多的改造，仅需要新增KM，对这类采集结果做格式解析处理。exporter暴露的数据指标相对统一，处理起来也比较方便。

   缺点：需要额外维护exporter，且需要开通exporter监听端口的访问权限；exporter的定义配置如何通知给网管系统；对于指标的定义、配置，仍然需要沿用原来的方法，即还要开发KM、配置表单，这部分是无法避免的。



3. 复用监控系统的采集客户端，同时改造网管系统部分功能。例如telegraf，telegraf支持输出结果到influxdb,或者生成本地json文件。如果是本地Json文件，则与prometheus类似，由KM解析即可。如果是influxdb，则需开发程序从influxdb读取解析数据(相当于另一个数据源)。

   优点：仅仅用到第三方开源系统的采集客户端，抓取我们想要的数据。对原有的网管系统也不需要过多的改造，仅需要新增KM或者influxdb读取程序，对这类采集结果做格式解析处理。telegraf暴露的数据指标相对统一，处理起来也比较方便。

   缺点：需要额外维护telegraf进程；telegraf的定义配置如何通知给网管系统；对于指标的定义、配置，仍然需要沿用原来的方法，即还要KM、配置表单，这部分是无法避免的。

<a href="#对比表格">返回目录</a>



## 支持的采集对象

- Prometheus

https://prometheus.io/docs/instrumenting/exporters/



- Telegraf

https://github.com/influxdata/telegraf/tree/master/plugins/inputs

jmx方式采集：kafka

https://github.com/influxdata/telegraf/tree/master/plugins/inputs/jolokia2

https://docs.confluent.io/current/kafka/monitoring.html



- Sensu

https://github.com/sensu-plugins



- Openfalcon

https://book.open-falcon.org/zh_0_2/usage/

[https://github.com/iambocai/falcon-monit-scripts](https://github.com/iambocai/falcon-monit-scripts)



- Datadog 

https://app.datadoghq.com/account/settings#integrations



## 结论

采用第二种方案，结合开源组件的客户端和采集脚本，有Agent负责调度，解析结果并返回。

被监控对象：ES /kakfa

实现的客户端：telegraf/openfaclon   客户端

目的：从前台配置开始，最终采集入库



jolokia2 采集分为两种模式，一种是jvm代理，由java应用（kafka）启动时修改参数引入jolokia2 包```-javaagent:/usr/hdp/2.6.1.0-129/kafka/libs/jolokia-jvm-1.5.0-agent.jar=port=8778,host=0.0.0.0```

第二种是Proxy模式，由于第一种应用的限制，proxy模式不需要在应用端修改

