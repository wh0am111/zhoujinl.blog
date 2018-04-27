---
title: TICK 监控体系
date: 2018-04-27 12:33:38
tags: 
 - TICK
 - InfluxDB 
 - Telegraf
categories: 技术
---

# Tick

## 1.Tick安装部署

### Tick简介

TICK 是由 InfluxData 开发的一套运维工具栈，由 Telegraf, InfluxDB, Chronograf, Kapacitor 四个工具的首字母组成。
这一套组件将收集数据和入库、数据库存储、展示、告警四者囊括了。

![TICK框架图](/img/tick.png "TICK") 

<!-- more -->

Telegraf

是一个数据收集和入库的工具。提供了很多 input 和 output 插件，比如收集本地的 cpu、load、网络流量等数据，然后写入 InfluxDB 、Kafka或者OpenTSDB等。相当于ELK栈中的 logstash 功能。

InfluxDB

InfluxDB 是一个开源的GO语言为基础的数据库, 用来处理时间序列数据,提供了较高的可用性。与opentsdb类似，支持HTTP API方式，写入和读取数据。相当于ELK栈中的elasticsearch功能。

Chronograf

从InfluxDB时间序列数据的数据可视化工具，负责从InfluxDB收集数据，并将数据图表以web的形式发布。它使用简单,包括模板和库可以快速构建实时数据的可视化仪表板，轻松地创建报警和自动化的规则。相当于ELK栈中的kibana功能。

Kapacitor

Kapacitor是InfluxDB的数据处理引擎，主要作用是时间序列数据处理、监视和警报。

### TICK安装（DOCKER版）

[官方安装介绍](https://portal.influxdata.com/downloads)

本地安装版本分别是telegraf1.5,influxdb1.42,chronograf1.4,kapacitor1.4

```
docker pull telegraf
docker pull influxdb
docker pull quay.io/influxdb/chronograf
docker pull kapacitor
```



### TICK配置与运行

[TICK Docker file](https://github.com/influxdata/influxdata-docker)



#### InfluxDB

[How to run](https://github.com/docker-library/docs/blob/master/influxdb/README.md)

参数介绍：

- 8086 HTTP API port
- 8083 Administrator interface port, if it is enabled
- 2003 Graphite support, if it is enabled

启动server 可修改默认配置文件等，详见[How to run]

```bash
docker run --name influxdb -d -p 8086:8086 -p 8083:8083  -e INFLUXDB_ADMIN_ENABLED=true  -v /var/lib/influxdb:/var/lib/influxdb  influxdb
```

初始化数据库

```bash
docker run --rm \
      -e INFLUXDB_DB=db0 -e INFLUXDB_ADMIN_ENABLED=true \
      -e INFLUXDB_ADMIN_USER=admin -e INFLUXDB_ADMIN_USER=admin \
      -e INFLUXDB_USER=telegraf -e INFLUXDB_USER_PASSWORD=telegraf \
      -v /var/lib/influxdb:/var/lib/influxdb \
      influxdb /init-influxdb.sh
```

启动client

```bash
docker run --rm --link=influxdb -it influxdb influx -host influxdb
```

其他命令

创建数据库

```shell
curl -G http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"
```

插入数据

```shell
curl -XPOST "http://localhost:8086/write?db=mydb" \
-d 'cpu,host=server01,region=uswest load=42 1434055562000000000'

curl -XPOST "http://localhost:8086/write?db=mydb" \
-d 'cpu,host=server02,region=uswest load=78 1434055562000000000'

curl -XPOST "http://localhost:8086/write?db=mydb" \
-d 'cpu,host=server03,region=useast load=15.4 1434055562000000000'
```

查询数据

```shell
curl -G "http://localhost:8086/query?pretty=true" --data-urlencode "db=mydb" \
--data-urlencode "q=SELECT * FROM cpu WHERE host='server01' AND time < now() - 1d"
```

分析数据

```shell
curl -G "http://localhost:8086/query?pretty=true" --data-urlencode "db=mydb" \
--data-urlencode "q=SELECT mean(load) FROM cpu WHERE region='uswest'"
```

[InfluxDB概念介绍](https://docs.influxdata.com/influxdb/v1.4/concepts/key_concepts/#field-key)

#### Telegraf

[Telegraf Docker file](https://github.com/influxdata/influxdata-docker/blob/ae56003075c8f39b6c265294ca8235c2c5235cc6/telegraf/1.5/alpine/Dockerfile)
[How to run](https://github.com/docker-library/docs/blob/master/telegraf/content.md)

参数介绍：

- 8125 StatsD
- 8092 UDP
- 8094 TCP

```bash
docker run --name telegraf -d -p 8125:8125 -p 8092:8092 -p 8094:8094  -v ~/telegraf.conf:/etc/telegraf/telegraf.conf telegraf
```

#### Chronograf

Exposed Ports

- 8888TCP -- Listen  endpoint

```shell
docker run -d --name chronograf -p 8888:8888 -v /var/lib/chronograf:/var/lib/chronograf quay.io/influxdb/chronograf chronograf
```



#### Kapacitor

Exposed Ports

- 9092 TCP -- HTTP API endpoint

需要修改默认的kapacitor.conf url地址，默认是配置localhost，而容器进程肯定不是localhost.

```shell
docker run -d --name kapacitor -p 9092:9092 -v /var/lib/kapacitor:/var/lib/kapacitor  -v ~/kapacitor.conf:/etc/kapacitor/kapacitor.conf  kapacitor 
```

查看帮助

```shell
docker exec  kapacitor kapacitor help
docker exec  kapacitor kapacitor list tasks 
docker exec  kapacitor kapacitor show CPURule

```

常见问题：

容器告警因为ip原因无法，注册到influxdb时，ip是容器id，如下，导致Influxdb 无法识别该地址，性能数据无法发送到kapacitor。因此临时解决方案是修改容器的hostname

```
> show SUBSCRIPTIONS
name: telegraf
retention_policy name                                           mode destinations
---------------- ----                                           ---- ------------
autogen          kapacitor-8ca0780c-5208-41d5-91ec-05b28d13b2b8 ANY  [http://c9816b89f375:9092]

name: _internal
retention_policy name                                           mode destinations
---------------- ----                                           ---- ------------
monitor          kapacitor-8ca0780c-5208-41d5-91ec-05b28d13b2b8 ANY  [http://c9816b89f375:9092]

name: chronograf
retention_policy name                                           mode destinations
---------------- ----                                           ---- ------------
autogen          kapacitor-8ca0780c-5208-41d5-91ec-05b28d13b2b8 ANY  [http://c9816b89f375:9092]

```



```
 docker run -d --name kapacitor -e KAPACITOR_HOSTNAME=192.168.14.165 -p 9092:9092 -v /var/lib/kapacitor:/var/lib/kapacitor  -v ~/kapacitor.conf:/etc/kapacitor/kapacitor.conf  kapacitor
```



## 2.Telgraf重点介绍

Telegraf是一个用Go编写的代理程序，用于收集，处理，汇总和编写度量标准。

Telegraf是插件驱动，并有4个不同的插件的概念：

1. [输入插件](https://github.com/influxdata/telegraf#input-plugins)从系统，服务或第三方API收集指标。
2. [处理器插件](https://github.com/influxdata/telegraf#processor-plugins)转换，修饰和/或过滤度量标准
3. [聚合器插件](https://github.com/influxdata/telegraf#aggregator-plugins)创建聚合度量（例如平均值，最小值，最大值，分位数等）
4. [输出插件](https://github.com/influxdata/telegraf#output-plugins)将指标写入各个目标

Telegraf 二进制文件包含对所有支持的采集对象的采集功能，只是可以通过配置选择性的开启。如果更新某个对象的采集指标，则需要重新编译整个二进制文件。



### 测试数据/帮助命令

~/telegraf.conf 测试用配置文件，可定义输入插件

```shell
docker run -v ~/telegraf.conf:/etc/telegraf/telegraf.conf --rm telegraf telegraf --config /etc/telegraf/telegraf.conf  --test
```

```
docker run --rm telegraf telegraf --help
```

### 数据收集方式

push 模式，由telegraf 根据input配置主动采集并发送到output配置的后端。一般是InfluxDB数据库。TICK并无服务端的概念，在TICK监控部署文档的架构图中可知。

[input-output-plugins](https://github.com/influxdata/telegraf/blob/master/README.md)



### InfluxDB Line Protocol 数据格式

InfluxDB Line Protocol 格式

Telegraf metrics, like InfluxDB [points](https://docs.influxdata.com/influxdb/v0.10/write_protocols/line/), are a combination of four basic parts:

```shell
measurement_name[,tag1=val1,...]  field1=val1,field2=val2,...
```

1. Measurement Name

2. Tags

3. Fields

4. Timestamp

   ​

### 输入数据格式

[telegraf有六种采集数据输入格式](https://docs.influxdata.com/telegraf/v1.5/concepts/data_formats_input/)

Telegraf is able to parse the following input data formats into metrics:

1. [InfluxDB Line Protocol](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#influx)
2. [JSON](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#json)
3. [Graphite](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#graphite)
4. [Value](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#value), ie: 45 or "booyah"
5. [Nagios](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#nagios) (exec input only)
6. [Collectd](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md#collectd)

默认是第一种：InfluxDB Line Protocol。Graphite、Nagios、Collected都是其他开源监控项目，其中Nagios是比较完整的监控系统，其格式只能用在exec输入插件中；Collected一般用作系统守护进程，采集系统统计信息，并且通过网络开放数据；Graphite则是另外一套监控系统，也是C-S模式。

如果是配置成其他格式，那么采集回来的数据（受限于采集方可吐出的数据），将会被Telegraf  转换为其可识别的InfluxDB Line Protocol。



### 输出数据格式

[telegraf有三种采集数据输出格式](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md)

Telegraf is able to serialize metrics into the following output data formats:

1. [InfluxDB Line Protocol](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md#influx)
2. [JSON](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md#json)
3. [Graphite](https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md#graphite)

一般而言，输出格式如果是其他格式，例如Json，那么输出的对象则一般为file类型。从实际应用中来看，即通过telegraf采集回数据，生成json文件，发送至其他系统进行处理。

以下是采集结果示例

```shell
* Plugin: inputs.cpu, Collection 1
* Plugin: inputs.cpu, Collection 2
> cpu,cpu=cpu0,host=0336dcb23579 usage_system=0,usage_idle=100,usage_nice=0,usage_iowait=0,usage_irq=0,usage_guest=0,usage_user=0,usage_softirq=0,usage_steal=0,usage_guest_nice=0 1515338025000000000
> cpu,cpu=cpu1,host=0336dcb23579 usage_user=0,usage_guest_nice=0,usage_steal=0,usage_guest=0,usage_system=0,usage_idle=100,usage_nice=0,usage_iowait=0,usage_irq=0,usage_softirq=0 1515338025000000000
> cpu,cpu=cpu2,host=0336dcb23579 usage_idle=100,usage_nice=0,usage_steal=0,usage_guest=0,usage_user=0,usage_system=0,usage_iowait=0,usage_irq=0,usage_softirq=0,usage_guest_nice=0 1515338025000000000
> cpu,host=0336dcb23579,cpu=cpu3 usage_softirq=0,usage_guest_nice=0,usage_idle=100,usage_nice=0,usage_iowait=0,usage_irq=0,usage_steal=0,usage_guest=0,usage_user=0,usage_system=0 1515338025000000000
> cpu,cpu=cpu-total,host=0336dcb23579 usage_guest_nice=0,usage_idle=100,usage_nice=0,usage_irq=0,usage_steal=0,usage_guest=0,usage_user=0,usage_system=0,usage_iowait=0,usage_softirq=0 1515338025000000000

```

cpu：measurements ，类似oracle的表名

cpu=cpu-total,host=0336dcb23579 ： cpu、host为tag，索引；

usage_guest_nice=0,usage_idle=100,usage_nice=0... : usage_guest_nice等为field，即被采集的指标字段。field在measurements是不可或缺的，tag可无， tag与field用空格隔开，field放置在后面

1515338025000000000： time 时间戳



### 数据过滤/聚合

Measurement Filtering

![数据处理](/img/agg-pro.jpg)

Filters can be configured per input, output, processor, or aggregator, see below for examples.

- **namepass**: An array of glob pattern strings. Only points whose measurement name matches a pattern in this list are emitted.
- **namedrop**: The inverse of `namepass`. If a match is found the point is discarded. This is tested on points after they have passed the `namepass` test.
- **fieldpass**: An array of glob pattern strings. Only fields whose field key matches a pattern in this list are emitted. Not available for outputs.
- **fielddrop**: The inverse of `fieldpass`. Fields with a field key matching one of the patterns will be discarded from the point. This is tested on points after they have passed the `fieldpass` test. Not available for outputs.
- **tagpass**: A table mapping tag keys to arrays of glob pattern strings. Only points that contain a tag key in the table and a tag value matching one of its patterns is emitted.
- **tagdrop**: The inverse of `tagpass`. If a match is found the point is discarded. This is tested on points after they have passed the `tagpass` test.
- **taginclude**: An array of glob pattern strings. Only tags with a tag key matching one of the patterns are emitted. In contrast to `tagpass`, which will pass an entire point based on its tag, `taginclude` removes all non matching tags from the point. This filter can be used on both inputs & outputs, but it is *recommended* to be used on inputs, as it is more efficient to filter out tags at the ingestion point.
- **tagexclude**: The inverse of `taginclude`. Tags with a tag key matching one of the patterns will be discarded from the point.

输入数据示例

```shell
[[inputs.cpu]]
  percpu = true
  totalcpu = false
  fielddrop = ["cpu_time"]
  # Don't collect CPU data for cpu6 & cpu7
  [inputs.cpu.tagdrop]
    cpu = [ "cpu6", "cpu7" ]

[[inputs.disk]]
  [inputs.disk.tagpass]
    # tagpass conditions are OR, not AND.
    # If the (filesystem is ext4 or xfs) OR (the path is /opt or /home)
    # then the metric passes
    fstype = [ "ext4", "xfs" ]
    # Globs can also be used on the tag values
    path = [ "/opt", "/home*" ]
```

输出数据示例

与输入（采集）数据过滤类似，可定义多个输出源

```shell
[[outputs.influxdb]]
  urls = [ "http://localhost:8086" ]
  database = "telegraf"
  # Drop all measurements that start with "aerospike"
  namedrop = ["aerospike*"]

[[outputs.influxdb]]
  urls = [ "http://localhost:8086" ]
  database = "telegraf-aerospike-data"
  # Only accept aerospike data:
  namepass = ["aerospike*"]

[[outputs.influxdb]]
  urls = [ "http://localhost:8086" ]
  database = "telegraf-cpu0-data"
  # Only store measurements where the tag "cpu" matches the value "cpu0"
  [outputs.influxdb.tagpass]
    cpu = ["cpu0"]
```

Aggregator Plugins

- [basicstats](https://github.com/influxdata/telegraf/blob/master/plugins/aggregators/basicstats)
- [minmax](https://github.com/influxdata/telegraf/blob/master/plugins/aggregators/minmax)
- [histogram](https://github.com/influxdata/telegraf/blob/master/plugins/aggregators/histogram)

### 部署方式

客户端部署方式，可结合Ansible 进行部署，以及配置管理。官方说明并未看到。



### 任务配置

客户端配置文件配置 /etc/telegraf/telegraf.conf，可配置指标采集周期、过滤指标等。



### 告警处理

与Telegraf配合使用的告警处理程序是[kapacitor](https://github.com/influxdata/kapacitor) 

Open source framework for processing, monitoring, and alerting on time series data。