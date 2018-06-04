---
layout: post
title: Elasticsearch Plugin 安装
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---
# Elasticsearch Plugin
工欲善其事必先利其器,安装 Elasticsearch 插件 `Head` 、`Kopf` 与 `Bigdesk`

1. Elasticsearch 版本 `2.4.1`
2. elasticsearch-head版本: 2.x(支持elasticsearch 2.x)
3. elasticsearch-kopf版本: 2.1.2(支持elasticsearch 2.x)

## 一.ElasticSearch-Head
ElasticSearch-Head 是一个与Elastic集群（Cluster）相交互的Web前台。ES-Head的主要作用:

1. 它展现ES集群的拓扑结构，并且可以通过它来进行索引（Index）和节点（Node）级别的操作
2. 它提供一组针对集群的查询API，并将结果以json和表格形式返回
3. 它提供一些快捷菜单，用以展现集群的各种状态


### 1.在线安装
在安装目录下的 `bin` 中执行以下命令,进行在线安装

```shell
plugin install mobz/elasticsearch-head
```

### 2.手动安装
在 `Github` 下载文件 `elasticsearch-head-master.zip`.然后通过指定文件路径来安装:

```
plugin install file:///yourpath/elasticsearch-head-master.zip
```

### 3.验证
浏览器访问: `http://localhost:9200/_plugin/head/` 进行验证,可看到如下界面:
![](http://images2015.cnblogs.com/blog/613455/201602/613455-20160224102545802-1222195555.png)

## 二.ElasticSearch-Kopf
Kopf是一个ElasticSearch的管理工具，它也提供了对ES集群操作的API。

## 三.ElasticSearch-Bigdesk
Bigdesk为Elastic集群提供动态的图表与统计数据。


## 参考链接
1. [Installing Plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/2.4/installation.html)
2. [elasticsearch-head](https://github.com/mobz/elasticsearch-head)
3. [elasticsearch-kopf](https://github.com/lmenezes/elasticsearch-kopf)
4. [bigdesk](https://github.com/lukas-vlcek/bigdesk)
5. [ElasticSearch 2 (6) - 插件安装Head、Kopf与Bigdesk](http://www.cnblogs.com/richaaaard/p/5212044.html)