---
title: Open-falcon 监控体系介绍
date: 2018-04-27 12:51:34
tags: 
 - Open-falcon
 - Go
categories: 技术
---

## 1.Openfalcon安装部署

### 简介

[官方文档 v2.0](https://book.open-falcon.org/zh_0_2/intro/)

- 强大灵活的数据采集：自动发现，支持falcon-agent、snmp、支持用户主动push、用户自定义插件支持、opentsdb data model like（timestamp、endpoint、metric、key-value tags）
- 水平扩展能力：支持每个周期上亿次的数据采集、告警判定、历史数据存储和查询
- 高效率的告警策略管理：高效的portal、支持策略模板、模板继承和覆盖、多种告警方式、支持callback调用
- 人性化的告警设置：最大告警次数、告警级别、告警恢复通知、告警暂停、不同时段不同阈值、支持维护周期
- 高效率的graph组件：单机支撑200万metric的上报、归档、存储（周期为1分钟）
- 高效的历史数据query组件：采用rrdtool的数据归档策略，秒级返回上百个metric一年的历史数据
- dashboard：多维度的数据展示，用户自定义Screen
- 高可用：整个系统无核心单点，易运维，易部署，可水平扩展
- 开发语言： 整个系统的后端，全部golang编写，portal和dashboard使用python编写。

### 框架

![func_tro](/img/openfalcon-func.png)



<!-- more -->

### 后端安装(docker)

/opt/openfalcon/falcon-plus

[官方教程](https://github.com/open-falcon/falcon-plus/blob/master/docker/README.md)

```shell
## Running falcon-plus container

    docker pull openfalcon/falcon-plus:0.2.0
    docker run -itd -p 8081:8081 openfalcon/falcon-plus:0.2.0 bash /run.sh hbs

## Running falcon-plus container with docker-compose

    docker-compose -f init.yml up -d falcon-plus

## Running mysql and redis container

    docker-compose -f init.yml up -d mysql redis

## Stop and Remove containers

    docker-compose -f init.yml rm -f
```

init.yml

```yaml
version: '2'
services:
  mysql:
    environment:
    - MYSQL_ROOT_PASSWORD=password
    extends:
      file: common.yml
      service: template
    hostname: docker-mysql
    image: mysql:5.7
    labels:
      owl: mysql
    ports:
    - 3306:3306
    restart: always
    volumes:
    - ../scripts/mysql/db_schema:/docker-entrypoint-initdb.d
    - ./mysql.cnf:/etc/mysql/conf.d/mysql.cnf:ro
  redis:
    command: redis-server /redis.conf
    extends:
      file: common.yml
      service: template-backend
    hostname: docker-redis
    image: redis:3.0
    labels:
      owl: redis
    ports:
    - 6379:6379
    restart: always
    volumes:
    - ./redis.conf:/redis.conf
  falcon-plus:
    command: bash /run.sh hbs
    image: openfalcon/falcon-plus:0.2.0
    ports:
    - 8081:8081
    restart: always
```

**注意**  需要打包mysql初始化脚本到这个路径下```../scripts/mysql/db_schema``` ，具体文件[在这里](https://github.com/open-falcon/falcon-plus/tree/master/scripts/mysql/db_schema)

mysql登陆

```
#默认mysql密码  password
docker exec -it <容器id> mysql -p 
```

web 登陆 http://192.168.14.165:8081   新建用户

![web](/img/openfalcon-web.png)

### 前端安装(docker)

/opt/openfalcon/dashboard

[官方教程](https://github.com/open-falcon/dashboard/blob/master/README.md)

```shell
# make the image，run commands under dir of dashboard:
docker build -t falcon-dashboard:v1.0 .

# start the container
docker run -itd --name aaa --net host \
	-e API_ADDR=http://127.0.0.1:8080/api/v1 \
	-e PORTAL_DB_HOST=127.0.0.1 \
	-e PORTAL_DB_PORT=3306 \
	-e PORTAL_DB_USER=root \
	-e PORTAL_DB_PASS=123456 \
	-e PORTAL_DB_NAME=falcon_portal \
	-e ALARM_DB_PASS=123456 \
	-e ALARM_DB_HOST=127.0.0.1 \
	-e ALARM_DB_PORT=3306 \
	-e ALARM_DB_USER=root \
	-e ALARM_DB_PASS=123456 \
	-e ALARM_DB_NAME=alarms \
	falcon-dashboard:v1.0
```

Dockerfile

```
FROM centos:7.3.1611

RUN yum clean all && yum install -y epel-release && yum -y update && \
yum install -y git python-virtualenv python-devel openldap-devel mysql-devel && \
yum groupinstall -y "Development tools"

RUN export HOME=/home/work/ && mkdir -p $HOME/open-falcon/dashboard && cd $HOME/open-falcon/dashboard
WORKDIR /home/work/open-falcon/dashboard
ADD ./ ./
RUN virtualenv ./env && ./env/bin/pip install -r pip_requirements.txt -i http://pypi.douban.com/simple

ADD ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```





## 2.Openfalcon使用

### 采集模式

[入门手册](https://book.open-falcon.org/zh_0_2/usage/getting-started.html)

falcon-agent自动开始采集各项指标，主动上报，不需要用户在server做任何配置（这和zabbix有很大的不同），这样做的好处，就是用户维护方便，覆盖率高。当然这样做也会server端造成较大的压力，不过open-falcon的服务端组件单机性能足够高，同时都可以水平扩展，所以自动多采集足够多的数据，反而是一件好事情，对于SRE和DEV来讲，事后追查问题，不再是难题。

另外，falcon-agent提供了一个proxy-gateway，用户可以方便的通过http接口，push数据到本机的gateway，gateway会帮忙高效率的转发到server端。



**机器负载信息**

这部分比较通用，我们提供了一个agent部署在所有机器上去采集。不像zabbix，要采集什么数据需要在服务端配置，falcon无需配置，只要agent部署到机器上，配置好heartbeat和Transfer地址，就自动开始采集了，省去了用户配置的麻烦。目前agent只支持64位Linux，Mac、Windows均不支持。

**硬件信息**

硬件信息的采集脚本由系统组同学提供，作为plugin依托于agent运行，plugin机制介绍请看[这里](http://book.open-falcon.org/zh_0_2/philosophy/plugin.html)。

**服务监控数据**

服务的监控指标采集脚本，通常都是跟着服务的code走的，服务上线或者扩容，这个脚本也跟着上线或者扩容，服务下线，这个采集脚本也要相应下线。公司里Java的项目有不少，研发那边就提供了一个通用jar包，只要引入这个jar包，就可以自动采集接口的调用次数、延迟时间等数据。然后将采集到的数据push给监控，一分钟push一次。目前falcon的agent提供了一个简单的http接口，这个jar包采集到数据之后是post给本机agent。向agent推送数据的一个简单例子，如下：

```
curl -X POST -d '[{"metric": "qps", "endpoint": "open-falcon-graph01.bj", "timestamp": 1431347802, "step": 60,"value": 9,"counterType": "GAUGE","tags": "project=falcon,module=graph"}]' http://127.0.0.1:1988/v1/push

```

**各种开源软件的监控指标**

这都是大用户，比如DBA自己写一些采集脚本，连到各个MySQL实例上去采集数据，完事直接调用server端的jsonrpc汇报数据，一分钟一次，每次甚至push几十万条数据，比较好的发送方式是500条数据做一个batch，别几十万数据一次性发送。



由上可知，falcon-agent本身仅支持主机类的指标采集，对于其他业务系统、开源软件，需独立的使用采集脚本，将数据按照规定格式，发送到falcon-agent 所在的监听端口。最终再由falcon-agent 转发到服务端。

