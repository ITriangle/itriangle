---
layout: post
title: Elasticsearch 简述和基本概念
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

# Elasticsearch 简述

- Elasticsearch是一个分布式的文档(document)存储引擎。它可以实时存储并检索复杂数据结构——序列化的 JSON文档。换言说，一旦文档被存储在Elasticsearch中，它就可以在集群的任一节点上被检索。
- 在Elasticsearch中，每一个字段的数据都是默认被索引的。也就是说，每个字段专门有一个反向索引用于快速检索。而且，与其它数据库不同，它可以在同一个查询中利用所有的这些反向索引，以惊人的速度返回结果。

## 基于HTTP协议，以JSON为数据交互格式的RESTful API 

### 命令格式

```shell
curl -i -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

### 命令格式说明

+ `VERB`:指明HTTP方法：GET , POST , PUT , HEAD , DELETE
    + `GET` 查询
    + `DELTE` 删除
    + `HEAD` 存在
    + `POST` `PUT` 创建,修改
        + 修改倾向于使用`PUT`
        + 创建:自动`ID`创建用`PSOT`,  必须指定`ID`创建用`PUT`
+ `PROTOCOL`: http或者https协议（ 只有在Elasticsearch前面有https代理的时候可用）
+ `HOST`: Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
+ `PORT`:Elasticsearch HTTP服务所在的端口，默认为9200
+ `PATH`:API路径（ 例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
+ `QUERY_STRING`:一些可选的查询请求参数，例如 ?pretty 参数将使请求返回更加美观易读的JSON数据
+ `BODY`:一个JSON格式的请求主体（ 如果请求需要的话）
+ `i`:显示头部信息
+ 举例说明，为了计算集群中的文档数量，我们可以这样做：

```shell
curl -i -XGET 'http://localhost:9200/_count?pretty' -d '
{
"query": {
"match_all": {}
}
} '
```

## Elasticsearch面向于文档

- 在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库：

```
Relational DB -> Databases  -> Tables -> Rows       -> Columns
Elasticsearch -> Indices    -> Types  -> Documents  -> Fields
```

- Elasticsearch集群可以包含多个索引(indices)（ 数据库），每一个索引可以包含多个类型(types)（ 表） ，每一个类型包含多个文档(documents)（ 行） ，然后每个文档包含多个字段(Fields)（ 列） 。

## 搜索

- 简单搜索：搜索特定的名字，通过年龄筛选等
    - `query match`  模糊搜索
    - `filter range` 精确搜索
- 全文搜索：默认情况下，Elasticsearch根据结果相关性评分来对结果集进行排序，所谓的「 结果相关性评分」 就是文档与查询条件的匹配程度。
    - `query match about`
- 短语搜索：确切的匹配若干个单词或者短语(phrases)。例如我们想要查询同时包含"rock"和"climbing"（ 并且是相邻的） 的员工记录。
    - `query match_phrase about`
- 高亮搜索：用**highlight**匹配关键字，用 <em></em> 来标识匹配到的部分
    - `highlight fields about`

## 分析
允许管理者在职员目录中进行一些分析。 Elasticsearch有一个功能叫做聚合(aggregations)，它允许你在数据上生成复杂的分析统计。它很像SQL中的 `GROUP BY` 但是功能更强大。

## 数据

- 输入，输出的数据为：Json格式的文档
- 文档元数据
    - `_index` 文档存储的地方
    - `_type`  文档代表的对象的类
    - `_id`    文档的唯一标识

## CURD

### 索引文档（创建文档）
- put创建和更新对象，需要指明对象ID
- post可以自动增加对象ID.自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally uniqueidentifiers, 或者叫 UUIDs。

### 检索文档（查询文档）
- GET：获取文档
    - pretty：在任意的查询字符串中增加 pretty参数，类似于上面的例子。会让Elasticsearch美化输出(pretty-print)JSON响应以便更加容易阅读。 _source 字段不会被美化，它的样子与我们输入的一致。
- HEAD：文档是否存在  

### 删除文档
- DELETE

## 处理并发冲突

### 并发控制方式
* 悲观并发控制（ Pessimistic concurrency control）：这在关系型数据库中被广泛的使用，假设冲突的更改经常发生，为了解决冲突我们把访问区块化。典型的例子是在读一行数据前锁定这行，然后确保只有加锁的那个线程可以修改这行数据。
* 乐观并发控制（ Optimistic concurrency control） ：被Elasticsearch使用，假设冲突不经常发生，也不区块化访问，然而，如果在读写过程中数据发生了变化，更新操作将失败。这时候由程序决定在失败后如何解决冲突。实际情况中，可以重新尝试更新，刷新数据（ 重新读取） 或者直接反馈给用户

#### 乐观并发控制
指定版本更新_version，采用CAS，当比较 _version失败时，则更新失败。`PUT /website/blog/1?version=1`
* 顺序无关的操作冲突：对于多用户的局部更新，文档被修改了并不要紧。例如，两个进程都要增加页面浏览量，增加的顺序我们并不关心——如果冲突发生，我们唯一要做的仅仅是重新尝试更新既可。
    * 可以通过 retry_on_conflict 参数设置重试次数来自动完成，这样 update 操作将会在发生错误前重试——这个值默认为0。
    * `POST /website/pageviews/1/_update?retry_on_conflict=5`

#### 使用外部版本控制系统:
* 描述：一种常见的结构是使用一些其他的数据库做为主数据库，然后使用Elasticsearch搜索数据，这意味着所有主数据库发生变化，就要将其拷贝到Elasticsearch中。如果有多个进程负责这些数据的同步，就会遇到上面提到的并发问题。
* 如果主数据库有版本字段，或一些类似于 timestamp 等可以用于版本控制的字段。可用外部数据库的版本指定ES数据库中文档的版本号。
* 不同于乐观并发控制。不再检查 _version 是否与请求中指定的一致，而是检查是否小于指定的版本。如果请求成功，外部版本号就会被存储到 _version 中。
* 注意：外部版本号不仅在索引和删除请求中指定，也可以在创建(create)新文档中指定。`PUT /website/blog/2?version=5&version_type=external`

# 基础概念

- `Index` 对应一个逻辑数据库。一个index是一个索引的集合。
- `Mapping` 对应数据库里的表定义。Mapping是对于index上每种type的定义
- `Type` 则是数据库里的一个表。是index上的一类document
- `Document` 是数据库里的一个行。对应一个type的一个实例。
- `Node(节点)` :节点是属于集群的elasticsearch的正在运行的实例。为了测试目的，可以在单个服务器上启动多个节点，但通常每个服务器应有一个节点。在启动时，节点将使用单播（或多播，如果指定）发现具有相同集群名称的现有集群，并尝试加入该集群
- `Cluster(集群)` :集群由共享相同集群名称的一个或多个节点组成。每个集群都有一个主节点，由集群自动选择，如果当前主节点发生故障，则可以替换该主节点
- `Shard(分片)`:分片是一个单一的Lucene实例。它是一个低级“工作”单元，由elasticsearch自动管理。 索引是一个指向主分片和副本分片的逻辑命名空间。除了定义索引应具有的主分片和副本分片的数量之外，您不需要直接引用分片。相反，你的代码应该只处理一个索引。Elasticsearch在集群中的所有节点之间分布分片，并且可以在节点故障的情况下将分片从一个节点自动移动到另一个节点，或者添加新节点
    - `primary shard(主分片)`:每个文档存储在单个主分片中。索引文档时，首先在主分片上查找，然后在主分片的所有副本上对其进行索引。默认情况下，索引具有5个主分片。您可以指定更少或更多的主分片来缩放索引可处理的文档数。创建索引后，无法更改索引中的主分片数。 
    - `replica Shard(副分片)`:每个主分片可以有零个或多个副分片。 副分片是主分片的副本.默认情况下，每个主分片具有一个副分片，但可以在现有索引上动态更改副本数。 副分片永远不会与其主分片在相同的节点上启动
    - 分为主分片和副分片,具有两个目的：
        - 增加故障转移：如果主分片故障，可以将副分片提升到主分片
        - 提高性能：获取和搜索请求可以由主或副本分片处理。