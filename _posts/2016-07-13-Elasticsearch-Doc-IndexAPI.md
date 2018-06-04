---
layout: post
title: Index API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

# Index API(索引API)
索引 API 允许你创建自定义的 JSON 文档存储到Elasticsearch.

## 一.创建 JSON 文档
创建 JSON 文档的方式有多中:

- 用 `byte[]` 或者 `String` 手动创建
- 用 `Map` 创建, `Map` 会自动转化为等价的 JSON 
- 用第三方库序列化自定义的类对象,比如 `Jackson` 库
- 用内置的工具类方法 `XContentFactory.jsonBuilder()` 创建  

在内部, 每种方式都要转化为 `byte[]` . 因此,如果已经是 `byte[]` 格式,就可以直接使用. `jsonBulider` 直接构造为 `byte[]`,所以 `jsonBulider` 是效率最高的方式.

### 1.手动创建
手动创建没有任何的难度,只是要注意 [Date Fromat](https://www.elastic.co/guide/en/elasticsearch/reference/5.x/mapping-date-format.html) `yyyy-MM-dd`  

```java
String json = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
    "}";
```

### 2.使用 Map
Map 是 `key:value` 的数据集合. 它呈现为 JSON 的结构:

```java
Map<String, Object> json = new HashMap<String, Object>();
json.put("user","kimchy");
json.put("postDate",new Date());
json.put("message","trying out Elasticsearch");
```

### 3.用 Jackson 序列化自定义对象
你可以使用 [Jackson](http://wiki.fasterxml.com/JacksonHome) 将自定义对象序列化为 JSON.使用时,在 Maven 中添加依赖.然后就可以使用 `ObjectMapper` 去序列化自定义对象了:

```java
import com.fasterxml.jackson.databind.*;

// instance a json mapper
ObjectMapper mapper = new ObjectMapper(); // create once, reuse

// generate json
byte[] json = mapper.writeValueAsBytes(yourbeaninstance);
```

### 4.用 Elasticsearch 内置的类
Elasticsearch 提供了内置的类 `XContentFactory` ,帮助你去创建 JSON 文档.

```java
import static org.elasticsearch.common.xcontent.XContentFactory.*;

XContentBuilder builder = jsonBuilder()
    .startObject()
        .field("user", "kimchy")
        .field("postDate", new Date())
        .field("message", "trying out Elasticsearch")
    .endObject()
```

**注意**: 你同样可以使用 `startArray(String)` 和 `endArrary()` 方法.同样的, `field` 方法接受任何对象类型. 包括: 数值,日期和其他的 `XContentBuilder` 对象.  

使用 `stirng()` 方法创建 JSON String 对象: 

```java
String json = builder.string();
```

## 二.索引 JSON 文档
下面的例子, 用 `prepareIndex` API 创建一个 JSON 文档: `index` 为 `twitter`, `type` 为 `tweet`, `id` 为 `1`

```java
import static org.elasticsearch.common.xcontent.XContentFactory.*;

IndexResponse response = client.prepareIndex("twitter", "tweet", "1")
        .setSource(jsonBuilder()
                    .startObject()
                        .field("user", "kimchy")
                        .field("postDate", new Date())
                        .field("message", "trying out Elasticsearch")
                    .endObject()
                  )
        .get();
```

**注意**:你可以创建一条记录,但不说明 `id`,会以自增的方式分配一个 `id`:

```java
String json = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
    "}";

IndexResponse response = client.prepareIndex("twitter", "tweet")
        .setSource(json)
        .get();
```

## 三.获取应答状态
`IndexResponse` 对象包含应答的信息:

```java
// Index name
String _index = response.getIndex();
// Type name
String _type = response.getType();
// Document ID (generated or not)
String _id = response.getId();
// Version (if it's the first time you index this document, you will get: 1)
long _version = response.getVersion();
// status has stored current instance statement.
RestStatus status = response.status();
```

## 参考链接
1. [Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-index.html#java-docs-index-generate)