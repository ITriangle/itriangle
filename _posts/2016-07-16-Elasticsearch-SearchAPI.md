---
layout: post
title: Search API
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch,Java,API
---

# Search API
`Search API` 允许你执行一个搜索,并获取搜索结果中 `hits` 匹配的文档. `Search API` 可以执行一个或者多个 `index` 和 一个或者多个 `type`. `Search API` 能使用基于 [Query API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/2.4/java-query-dsl.html).

`Search API` 也同样有两类型:
1. 查询 `Query API`
2. 过滤 `Filter API`

**引入依赖:**

```java
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchType;
import org.elasticsearch.index.query.QueryBuilders.*;
```

```java
SearchResponse response = client.prepareSearch("index1", "index2")
        .setTypes("type1", "type2")
        .setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
        .setQuery(QueryBuilders.termQuery("multi", "test"))                 // Query
        .setPostFilter(QueryBuilders.rangeQuery("age").from(12).to(18))     // Filter
        .setFrom(0).setSize(60).setExplain(true)
        .execute()
        .actionGet();
```

## 1.Using scrolls in Java
`scroll API` 不是用于实时用户的请求,而是单次请求大量的数据结果.所以,当数据被实时更新时,会影响最后的结果.

`scroll API` 有两个参数,分别是:
1. `time` 超时时间
2. `size` 每一个分片 `hits` 到 `size` 大小的数据就返回

```java
import static org.elasticsearch.index.query.QueryBuilders.*;

QueryBuilder qb = termQuery("multi", "test");

SearchResponse scrollResp = client.prepareSearch(test)
        .addSort(SortParseElement.DOC_FIELD_NAME, SortOrder.ASC)
        .setScroll(new TimeValue(60000))
        .setQuery(qb)
        .setSize(100).execute().actionGet(); //100 hits per shard will be returned for each scroll

//Scroll until no hits are returned
while (true) {

    for (SearchHit hit : scrollResp.getHits().getHits()) {
        //Handle the hit...
    }
    scrollResp = client.prepareSearchScroll(scrollResp.getScrollId()).setScroll(new TimeValue(60000)).execute().actionGet();
    //Break condition: No hits are returned
    if (scrollResp.getHits().getHits().length == 0) {
        break;
    }
}
```

## 2.Multi Search API
`Multi Search API` 允许执行一连串的请求在同一个 `index` 中.

```java
SearchRequestBuilder srb1 = client
    .prepareSearch().setQuery(QueryBuilders.queryStringQuery("elasticsearch")).setSize(1);
SearchRequestBuilder srb2 = client
    .prepareSearch().setQuery(QueryBuilders.matchQuery("name", "kimchy")).setSize(1);

MultiSearchResponse sr = client.prepareMultiSearch()
        .add(srb1)
        .add(srb2)
        .execute().actionGet();

// You will get all individual responses from MultiSearchResponse#getResponses()
long nbHits = 0;
for (MultiSearchResponse.Item item : sr.getResponses()) {
    SearchResponse response = item.getResponse();
    nbHits += response.getHits().getTotalHits();
}
```

## 3. Using Aggregations
下面的代码显示了如何在 `search` 中添加两个 `aggregation`

```java
SearchResponse sr = client.prepareSearch()
    .setQuery(QueryBuilders.matchAllQuery())
    .addAggregation(
            AggregationBuilders.terms("agg1").field("field")
    )
    .addAggregation(
            AggregationBuilders.dateHistogram("agg2")
                    .field("birth")
                    .interval(DateHistogramInterval.YEAR)
    )
    .execute().actionGet();

// Get your facet results
Terms agg1 = sr.getAggregations().get("agg1");
DateHistogram agg2 = sr.getAggregations().get("agg2");
```

## 4.Terminate After
为每一个分片搜索的文档达到最大时,将会提前结束查询,即使还有符合条件的文档.

```java
SearchResponse sr = client.prepareSearch(INDEX)
    .setTerminateAfter(1000)    //Finish after 1000 docs
    .get();

if (sr.isTerminatedEarly()) {   //如果提前终止,将会执行
    // We finished early
}
```

# Query DSL API
全文搜索和过滤

## 1.Match All Query

```java
QueryBuilder qb = matchAllQuery();
```

## 2.Full text queries

### 1.Match Query

```java
QueryBuilder qb = matchQuery(
    "name",                  //field
    "kimchy elasticsearch"   //text
);
```

### 2.Multi Match Query

```java
QueryBuilder qb = multiMatchQuery(
    "kimchy elasticsearch",  //text
    "user", "message"        //fields,多个字段
);
```

### 3.Common Terms Query 
指定 `field` 的 某一个单词为 `common term`

```java
QueryBuilder qb = commonTermsQuery("name",    //field
                                   "kimchy"); //value
```

### 4.Query String Query
自己拼接搜索的 `text`

```java
QueryBuilder qb = queryStringQuery("+kimchy -elasticsearch"); 

QueryBuilder qb = simpleQueryStringQuery("+kimchy -elasticsearch");
```


## 3.Term level queries

### 1.Term Query

```java
QueryBuilder qb = termQuery(
    "name",    //field
    "kimchy"   //value
);
```

### 2.Terms Query

```java
QueryBuilder qb = termsQuery("tags",    //field
    "blue", "pill");                    //values
```

### 3.Range Query

```java
QueryBuilder qb = rangeQuery("age")     //field
    .gte("10")                          //大于等于
    .lt("20");                          //小于  
```

### 4.Exists Query

```java
QueryBuilder qb = existsQuery("name");  //field 为不空
```

### 5.Missing Query
使用 `exists` 和 `must_not` 

### 6.Prefix Query

```java
QueryBuilder qb = prefixQuery(
    "brand",            //field
    "heine"             //前缀
);
```

### 7.Wildcard Query

```java
QueryBuilder qb = wildcardQuery("user", "k?mc*");
```

### 8.Regexp Query

```java
QueryBuilder qb = regexpQuery(
    "name.first",        //field
    "s.*y");             //正则
```

### 9.Fuzzy Query

```java
QueryBuilder qb = fuzzyQuery(
    "name",     
    "kimzhy"    
);
```

### 10.Type Query

```java
QueryBuilder qb = typeQuery("my_type"); 
```

### 11.Ids Query

```java
QueryBuilder qb = idsQuery("my_type")
    .addIds("1", "4", "100");

QueryBuilder qb = idsQuery() 
    .addIds("1", "4", "100");
```

## 4.Compound queries

### 1.Bool Query

```java
QueryBuilder qb = boolQuery()
    .must(termQuery("content", "test1"))    
    .must(termQuery("content", "test4"))    
    .mustNot(termQuery("content", "test2")) 
    .should(termQuery("content", "test3"));     
```

## 5.Joining queries
## 6.Geo queries
## 7.Specialized queries
## 8.Span queries


## 参考
1. [elasticsearch-guide](https://endymecy.gitbooks.io/elasticsearch-guide-chinese/content/index.html)