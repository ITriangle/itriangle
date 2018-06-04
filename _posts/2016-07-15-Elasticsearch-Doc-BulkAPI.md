---
layout: post
title: Bulk API & Using Bulk Processor
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

## Bulk API
Bulk API 允许 `index` 和 `delete` 多个各自的文档在一个请求中.下面实例:

```java
import static org.elasticsearch.common.xcontent.XContentFactory.*;

BulkRequestBuilder bulkRequest = client.prepareBulk();

// either use client#prepare, or use Requests# to directly build index/delete requests
bulkRequest.add(client.prepareIndex("twitter", "tweet", "1")
        .setSource(jsonBuilder()
                    .startObject()
                        .field("user", "kimchy")
                        .field("postDate", new Date())
                        .field("message", "trying out Elasticsearch")
                    .endObject()
                  )
        );

bulkRequest.add(client.prepareIndex("twitter", "tweet", "2")
        .setSource(jsonBuilder()
                    .startObject()
                        .field("user", "kimchy")
                        .field("postDate", new Date())
                        .field("message", "another post")
                    .endObject()
                  )
        );

BulkResponse bulkResponse = bulkRequest.get();
if (bulkResponse.hasFailures()) {
    // process failures by iterating through each bulk response item
}
```

## Using Bulk Processor
`BulkProcessor` 类提供了简单的接口自动更新操作.依据参数有:

1. number  请求个数
2. size    请求大小
3. period  更新间隔时间

使用 `BulkProcessor` 实例:

```java
import org.elasticsearch.action.bulk.BackoffPolicy;
import org.elasticsearch.action.bulk.BulkProcessor;
import org.elasticsearch.common.unit.ByteSizeUnit;
import org.elasticsearch.common.unit.ByteSizeValue;
import org.elasticsearch.common.unit.TimeValue;

BulkProcessor bulkProcessor = BulkProcessor.builder(
        client,   //Elasticsearch client
        new BulkProcessor.Listener() {
            
            /**
             * 在 bulk 执行前调用.可通过 request.numberOfActions()查看numberOfActions
             */
            @Override
            public void beforeBulk(long executionId,
                                   BulkRequest request) { ... } 

            /**
             * 在 bulk 执行后调用.可通过response.hasFailures() 查看是否有失败的操作
             */
            @Override
            public void afterBulk(long executionId,
                                  BulkRequest request,
                                  BulkResponse response) { ... } 

            /**
             * 在 bulk 执行失败后并且有异常抛出时调用
             */
            @Override
            public void afterBulk(long executionId,
                                  BulkRequest request,
                                  Throwable failure) { ... } 
        })
        .setBulkActions(10000) //每10000个请求执行一次Bulk
        .setBulkSize(new ByteSizeValue(5, ByteSizeUnit.MB)) //每5MB执行一次
        .setFlushInterval(TimeValue.timeValueSeconds(5))  //每间隔5S执行一次
        .setConcurrentRequests(1) //可异步执行多个 Bulk
        .setBackoffPolicy(
            BackoffPolicy.exponentialBackoff(TimeValue.timeValueMillis(100), 3)) //等待100ms,并尝试3次
        .build();
```

`BulkProcessor` 默认情况下:
1. 请求个数 1000
2. 大小 5mb
3. 不存在刷新间隙
4. `setConcurrentRequests(1)` 异步刷新操作
5. 补偿机制: 指数级增长,8次尝试,初始值延时为50ms,总延时大约5.1s

### 添加请求
初始化 `bulkProcessor` 后,直接添加请求:

```java
bulkProcessor.add(new IndexRequest("twitter", "tweet", "1").source(/* your doc here */));
bulkProcessor.add(new DeleteRequest("twitter", "tweet", "2"));
```

### 关闭 Bulk Processor

1. 等待关闭 `bulkProcessor.awaitClose(10, TimeUnit.MINUTES);`
2. 立即关闭 `bulkProcessor.close();`

### Bulk Processor 使用实例:
测试时,最好设置 `setConcurrentRequests(0)` ,可以同步执行,便于观察

```java
BulkProcessor bulkProcessor = BulkProcessor.builder(client, new BulkProcessor.Listener() { /* Listener methods */ })
        .setBulkActions(10000)
        .setConcurrentRequests(0)
        .build();

// Add your requests
bulkProcessor.add(/* Your requests */);

// Flush any remaining requests
bulkProcessor.flush();

// Or close the bulkProcessor if you don't need it anymore
bulkProcessor.close();

// Refresh your indices
client.admin().indices().prepareRefresh().get();

// Now you can start searching!
client.prepareSearch().get();
```

## 参考
1. [Bulk API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-bulk.html#java-docs-bulk)
2. [Using Bulk Processor](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.1/java-docs-bulk-processor.html)