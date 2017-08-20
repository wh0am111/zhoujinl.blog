---
title: Elasticsearch 监控
date: 2017-08-17 20:30:45
tags: 
 - Elasticsearch
 - Monitor 
category: 技术 
---
ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

# Elasticsearch集群状态监控
集群状态
`curl -XGET http://192.168.17.201:9200/_cluster/health?pretty 2>/dev/null|grep "status"|awk -F: '{print $2}'|awk -F \" '{print $2}'`

<!-- more -->

# Elasticsearch集群节点数目
`curl -XGET http://192.168.17.201:9200/_cluster/health?pretty 2>/dev/null|grep "number_of_nodes"|awk -F: '{print $2}'|awk -F\, '{print $1}'`
