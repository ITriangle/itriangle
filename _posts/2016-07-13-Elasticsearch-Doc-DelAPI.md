---
layout: post
title: Delete API & Delete By Query API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

# Delete API
Delete API 可以删除指定 `index` 的记录通过 `id`.下面的实例是,在 `index` 中删除 `id` 为 1 的记录:

```java
DeleteResponse response = client.prepareDelete("twitter", "tweet", "1").get();
```

## 操作线程
同 GET API , DELETE API 也可以设置线程模型,并且含义是一致的.

```java
DeleteResponse response = client.prepareDelete("twitter", "tweet", "1")
        .setOperationThreaded(false)
        .get();
```

# Delete By Query API
Delete By Query API,可以基于查询的结果集,删除相应的记录.

```java
BulkIndexByScrollResponse response =
    DeleteByQueryAction.INSTANCE.newRequestBuilder(client)
        .filter(QueryBuilders.matchQuery("gender", "male")) //查询
        .source("persons")                                  //索引
        .get();                                             //执行操作

long deleted = response.getDeleted();//删除返回的文档
```

## 异步处理应答
因为 Delete By Query API 可能是一很长的操作.如果你想去异步的操作,那么可以用 `execute` 代替 `get ` , 并提供一个监听器:

```java
DeleteByQueryAction.INSTANCE.newRequestBuilder(client)
    .filter(QueryBuilders.matchQuery("gender", "male"))              //查询    
    .source("persons")                                               //索引
    .execute(new ActionListener<BulkIndexByScrollResponse>() {       //监听器
        @Override
        public void onResponse(BulkIndexByScrollResponse response) { 
            long deleted = response.getDeleted();                    //删除会的文档    
        }
        @Override
        public void onFailure(Exception e) {
            // Handle the exception
        }
    });
```

## 参考链接:
1. [Delete API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-delete.html)
2. [Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-delete-by-query.html)