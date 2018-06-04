---
layout: post
title: Elasticsearch 搜索一:简要说明简单搜索API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

# Elasticsearch 搜索
Elasticsearch 搜索分为 **简易查询** 和 **结构化查询**,分别对应两种请求方式: **请求参数方式** 和 **请求体方式**。

## 一.造数据样例
这里提供了一份官网上的数据，[accounts.json](http://pan.baidu.com/s/1pJvOlbP).

首先开启你的ES，然后执行下面的命令，windows下需要自己安装curl,也可以使用cygwin模拟curl命令,或者GitBash也是可以的:

```shell
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary  @accounts.json
```

### 说明格式说明
1. 需要在accounts.json所在的目录运行curl命令。 
2. localhost:9200是ES得访问地址和端口
3. bank是索引的名称
4. account是类型的名称
5. 索引和类型的名称在文件中如果有定义，可以省略；如果没有则必须要指定
6. _bulk是rest得命令，可以批量执行多个操作（操作是在json文件中定义的，原理可以参考之前的翻译）
7. pretty是将返回的信息以可读的JSON形式返回。

**注意**:这里使用的是 `Elasticsearch` 批量操作命令,暂时可不做了解

### 验证样例数据
执行完上述的命令后，可以通过下面的命令查询：

```shell
curl 'localhost:9200/_cat/indices?v'
health index pri rep docs.count docs.deleted store.size pri.store.size
yellow bank    5   1       1000            0    424.4kb        424.4kb
```

当然,如果在安装 Elasticsearch 时,能安装 head 插件验证会更加的方便.

## 查询

### 简易查询

> 一种是“简易版”的查询字符串(query string)将所有参数通过查询字符串定义 

- 单个匹配:例如这个语句查询所有类型为tweet并在tweet字段中包含elasticsearch字符的文档,q后面跟着搜索的条件：q=*表示查询所有的内容

```shell
GET /_all/tweet/_search?q=tweet:elasticsearch
```

- 多个匹配:查找name字段中包含"john"和tweet字段包含"mary"的结果。根据URL编码需要将字符串变形."+"前缀表示语句匹配条件必须被满足。类似的"-"前缀表示条件必须不被满足。所有条件如果没有+或-表示是可选的——匹配越多，相关的文档就越多。

```shell
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```

### 结构化查询

> 使用JSON完整的表示请求体(request body)，这种富搜索语言叫做结构化查询语句（DSL）

### 查询 bank 中所有记录

- 简易查询,请求参数方式

```shell
curl 'localhost:9200/bank/_search?q=*&pretty'
```

- 结构化查询,请求体方式（推荐这种方式）

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'
```

**返回的内容**

```

{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "13",
```

**返回的内容大致可以如下讲解**
- took：是查询花费的时间，毫秒单位
- time_out：标识查询是否超时
- _shards：描述了查询分片的信息，查询了多少个分片、成功的分片数量、失败的分片数量等
- hits：搜索的结果，total是全部的满足的文档数目，hits是返回的实际数目（默认是10）
- _score是文档的分数信息，与排名相关度有关，参考各大搜索引擎的搜索结果，就容易理解。

## 其他查询说明

## 多索引(index)和多类型(type)
> 例如:`user`和`tweet`的类型文档来自于不同的索引`us`和`gb`  

- `/_search` 在所有索引的所有类型中搜索
- `/gb/_search` 在索引gb的所有类型中搜索
- `/gb,us/_search` 在索引gb和us的所有类型中搜索
- `/g*,u*/_search` 在以g或u开头的索引的所有类型中搜索
- `/gb/user/_search` 在索引gb的类型user中搜索
- `/gb,us/user,tweet/_search` 在索引gb和us的类型为user和tweet中搜索
- `/_all/user,tweet/_search` 在所有索引中搜索user和tweet类型文档

## 搜索结果分页显示
> 和SQL使用`LIMIT`关键字返回只有一页的结果一样，Elasticsearch使用`from`和`size`参数控制每次显示的数目  

- `size`: 结果数，默认10
- `from`: 跳过开始的结果数，默认0
- 如果向搜索结果:每页显示5个结果，显示3页，那请求如下：

```shell
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```
