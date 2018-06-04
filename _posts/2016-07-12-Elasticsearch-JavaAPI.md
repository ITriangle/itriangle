---
layout: post
title: Elasticsearch Java Client
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

## 任务目标
1. 向 Elasticsearch 写入数据
2. 从 Elasticsearch 读出数据

## 环境:
1. 下载 [Elasticsearch 5.1.1](https://www.elastic.co/downloads/elasticsearch)
2. 安装,为了测试方便,在本机安装 Window 版本.当然也可以在 Linux 上安装
3. JDK 版本 1.8

## 流程分析
1. Elasticsearch 提供的是 HTTP Resetful API.
2. 数据格式是 Json.
3. Elasticsearch 连入方式:
    1. `Transport Client` 并不加入集群 
    2. `Connecting a Client to a Coordinating Only Node` 作为节点,加入集群

## Maven 管理依赖
时刻注意,驱动与 Elasticsearch 间的版本问题:

```xml
<dependencies>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>5.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
```

## 编写连接实例
创建 `TestEsClient` 类:

```java
//设置集群名称
Settings settings = Settings.builder().put("cluster.name", "elasticsearch").build();
//创建client
TransportClient client = null;
try {
    client = new PreBuiltTransportClient(settings)
            .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300));
} catch (UnknownHostException e) {
    e.printStackTrace();
}
//创建数据
String json = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
IndexResponse responseIndex = client.prepareIndex("twitter", "tweet", "1")
        .setSource(json)
        .get();

//搜索数据
GetResponse responseGet = client.prepareGet("twitter", "tweet", "1").execute().actionGet();
//输出结果
System.out.println(responseGet.getSourceAsString());
//关闭client
client.close();
```

## 结果验证
输出结果如下,与写入数据保持一致.

```
no modules loaded
loaded plugin [org.elasticsearch.index.reindex.ReindexPlugin]
loaded plugin [org.elasticsearch.percolator.PercolatorPlugin]
loaded plugin [org.elasticsearch.script.mustache.MustachePlugin]
loaded plugin [org.elasticsearch.transport.Netty3Plugin]
loaded plugin [org.elasticsearch.transport.Netty4Plugin]
{"user":"kimchy","postDate":"2013-01-30","message":"trying out Elasticsearch"}
```

[GitHub 代码链接](https://github.com/ITriangle/Elasticsearch-master/tree/master/Elasticsearch)

## 参考链接
1. [Elasticsearch Java API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/transport-client.html)