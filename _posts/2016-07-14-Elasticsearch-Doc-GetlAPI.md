---
layout: post
title: Get API & Multi GET API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

## Get API
GET API 可以从 Elasticsearch 获取指定 `id` 的 JSON 文档.下面的实例时,获取一个JSON文档: `index` 为 `twitter`, `type` 为 `tweet`, `id` 为 `1` :

```java
GetResponse response = client.prepareGet("twitter", "tweet", "1").get();
```

### 操作线程(Operation Threading)
GET API 允许设置线程模型.你在同一个节点上执行 API 时,会运行不同的线程模型.

可以选择不同的线程执行 API. `operationThreaded ` 默认设置为 `true`,就是 默认执行的是不同的线程.下面演示的是设置为 `false` 在相同的线程执行:

```java
GetResponse response = client.prepareGet("twitter", "tweet", "1")
        .setOperationThreaded(false)
        .get();
```


## Multi GET API 批量获取请求
Multi GET API 允许获取多个文档.基于 `index` `type` `id`:

```java
MultiGetResponse multiGetItemResponses = client.prepareMultiGet()
    .add("twitter", "tweet", "1")                   //获取单个id
    .add("twitter", "tweet", "2", "3", "4")         //获取过个id
    .add("another", "type", "foo")                  //获取另外的index
    .get();

for (MultiGetItemResponse itemResponse : multiGetItemResponses) {  //iterate 查询结果集
    GetResponse response = itemResponse.getResponse();             
    if (response.isExists()) {                               //检查文档食肉存在
        String json = response.getSourceAsString();          //获取文档内容
    }
}
```


## 参考链接:
1. [GET API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-get.html)
2. [Multi GET API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-multi-get.html)