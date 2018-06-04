---
layout: post
title: Elasticsearch 安装
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---
## 系统环境

> Centos6.4

## 安装步骤

- 下载源码包

```shell
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.1.tar.gz
```

- 解压并创建执行链接

```shell
tar zxf elasticsearch-1.7.1.tar.gz
sudo ln -s /home/wl/lib/elk/elasticsearch-1.7.1/bin/elasticsearch /usr/bin/elasticsearch
```

## 测试

```shell
elasticsearch start -d #可以用` -d`参数打入后台运行
curl -X GET http://localhost:9200
```


出现200返回码表示OK

```shell
{
  "status" : 200,
  "name" : "Wasp",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.7.1",
    "build_hash" : "b88f43fc40b0bcd7f173a1f9ee2e97816de80b19",
    "build_timestamp" : "2015-07-29T09:54:16Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```

## 集群和节点

- 节点(node)是一个运行着的Elasticsearch实例。
- 集群(cluster)是一组具有相同 cluster.name的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一
个节点也可以组成一个集群。
- 建立集群：通过修改 config/ 目录下的 elasticsearch.yml 文件，然后重启ELasticsearch来做到这一点。两台es服务同时起来，配置文件中均默认cluster.name=elasticsearch，所以这两台机器自动构建成一个集群，集群名字为elasticsearch。
- 当Elasticsearch在前台运行，可以使用 Ctrl-C 快捷键终止，或者你可以调
用 shutdown API来关闭：

```shell
curl -XPOST 'http://localhost:9200/_shutdown'
```

- 注意：最好找一个合适的名字来替代 cluster.name 的默认值，这样可以防止一个新启动的节点加入到相同网络中的另一个同名的集群中。

## 参考链接
1. [ElasticSearch入门 —— 集群搭建](https://my.oschina.net/xiaohui249/blog/228748)