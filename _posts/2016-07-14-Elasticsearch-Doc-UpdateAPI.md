---
layout: post
title: Update API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

## Update API

### 1.使用 `UpdateRequest` 对象发送更新请求

```java
UpdateRequest updateRequest = new UpdateRequest();
updateRequest.index("index");
updateRequest.type("type");
updateRequest.id("1");
updateRequest.doc(jsonBuilder()
        .startObject()
            .field("gender", "male")
        .endObject());
client.update(updateRequest).get();
```

### 2.使用 `prepareUpdate()` 方法

#### 1. 通过脚本更新

```java
UpdateRequest updateRequest = new UpdateRequest("ttl", "doc", "1")
        .script(new Script("ctx._source.gender = \"male\""));
client.update(updateRequest).get();
```

上面使用得是内部脚本.如果需要使用自定义脚本,需要设置 `ScriptService.ScriptType.FILE` :

```java
client.prepareUpdate("ttl", "doc", "1")
        .setScript(new Script("ctx._source.gender = \"male\""  , ScriptService.ScriptType.FILE, null, null))
        .get();
```

#### 2.通过已存在文档合并

```java
UpdateRequest updateRequest = new UpdateRequest("index", "type", "1")
        .doc(jsonBuilder()
            .startObject()
                .field("gender", "male")
            .endObject());
client.update(updateRequest).get();
```

## Upsert 更新插入
`upsert` 支持更新插入,当文档不存在时候,就变成添加,存在就是更新:

```java
IndexRequest indexRequest = new IndexRequest("index", "type", "1")
        .source(jsonBuilder()
            .startObject()
                .field("name", "Joe Smith")
                .field("gender", "male")
            .endObject());
UpdateRequest updateRequest = new UpdateRequest("index", "type", "1")
        .doc(jsonBuilder()
            .startObject()
                .field("gender", "male")
            .endObject())
        .upsert(indexRequest);              
client.update(updateRequest).get();
```



## 参考链接:
1. [Update API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-update.html#java-docs-update)