---
layout: post
title: Elasticsearch 搜索三:搜索API详解
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

# Elasticsearch 查询 `DSL` 语句说明
`Query DSL` 为 `_search API` 提供了查询的参数

### Full text queries
高级全文查询通常用于在全文字段（如电子邮件正文）上运行全文查询。 他们了解如何分析要查询的字段，并在执行之前将每个字段的分析器（或search_analyzer）应用于查询字符串.

#### 1.Match Query
接受文本/数字/日期的匹配查询族，分析它们并构造查询。
1. match 单词
2. match_phrase 短语
3. match_phrase_prefix 短语前缀模糊

```shell
#分词匹配,只要address 中包含 502 , Baycliff ,Terrace
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"match": {"address": "502 Baycliff Terrace"}}}'

#不分词匹配,相当于term查询不分词的string
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"match": {"address": {"query": "502 Baycliff Terrace", "operator": "and"}}}}'

#不分词匹配,相当于term查询不分词的string
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"match_phrase": {"address": "502 Baycliff Terrace"}}}'

#不分词匹配,相当于match_phrase,多了一个允许最后一个词能前缀模糊匹配
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"match_phrase_prefix": {"address": "502 Baycliff T"}}}'

```

#### 2.Multi Match Query
`multi_match` 查询建立在 `match` 查询上.允许多个字段同一条件的查询

```shell
## 分别在"firstname", "lastname"上查询
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"multi_match" : {"query":"Mullen","fields": [ "firstname", "lastname" ]}}}'

## 分别在"firstname", "lastname"上查询,模糊匹配字段
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"multi_match" : {"query":"Mullen","fields": [ "*tname"]}}}'

```

#### 3.Common Terms Query
`Common Terms Query` 把一部分词当做停用词(不加入索引),来提高查询的速度.针对全文搜索分词的情况.

把每一个查询的条件分为两类:
1. 在文档中,频率低的,比较重要
2. 在文档中,频率高的,不重要

每次查询,先搜索频率低的条件,再在查询结果中查询频率高的条件.

下面的实例: `this is bonsai cool` 是 `string` 分词了, `this` 和 `is` 在文档中出现频率高于 `0.1%`,自动被识别为 `common terms`(不重要的条件).

```shell
{
  "common": {
    "body": {
      "query": "this is bonsai cool",
      "cutoff_frequency": 0.001
    }
  }
}
```

### Term level queries
虽然全文查询将在执行之前分析查询字符串，但 `term-level` 查询是对存储的倒排索引进行精确的匹配.

通常用于查询:numbers,dates,enums的结构化数据,而不是全文查询的数据.

#### 1.Term Query
`term` query 将查找包含在反向索引中指定的确切条件的文档,对比 mapping 中的 field 属性 : 可以分词,或者不分词,或者不加入索引.

```shell
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"term": {"balance": "42368"}}}'
```

#### 2.Terms Query
`terms` query 将匹配数组中的多个条件值

```shell
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"terms": {"balance": ["42368","49119"]}}}'
```

#### 3.Range Query
匹配具有某个范围内字词的字段的文档。 Lucene查询的类型取决于字段类型，对于字符串字段，TermRangeQuery，而对于number / date字段，查询是一个NumericRangeQuery

```shell
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"range": {"balance": {"gte" : 42368, "lte":49119}}}'
```

**范围配置 date 属性字段**
当匹配 date 属性字段是,可以使用 date 类型的计算方式: [the section called “Date Mathedit”](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/common-options.html#date-math)

#### 4.Exists Query
返回该字段不为空的文档,类似于 `SQL IS NOT NULL`

```shell
## 查询 balance 不为空的记录
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"exists":{"field":"balance"}}}'

```

相反,可以查询 为空的记录
```
"bool": {
    "must_not": {
        "exists": {
            "field": "balance"
        }
    }
}
```

#### 5.Missing Query
不推荐用 `missing` ,可以用 `bool` + `must+not` + `exitsts` 代替.在最新的 5.2 中已经不存在了.

```shell
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"bool": {"must_not": {"exists": {"field": "balance"}}}}}'

```


#### 6.Prefix Query
匹配字段包含指定前缀的文档,该字段不能分词.

```shell
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"prefix":{"firstname" : "Mu"}}}'
```

#### 7.Wildcard Query
字段通配符匹配,通配符有两种:
1. `*`:多个
2. `?`:单个

该两种通配符不能用在开头,这种查询比较慢值得注意.

```shell
##单个字符
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"wildcard" : { "firstname" : "Nanet?e" }}}'

##多个字符
curl -XGET 'http:/localhost:9200/bank/_search?pretty' -d'{"query": {"wildcard" : { "firstname" : "Na*e" }}}'
```

#### 8.Regexp Query
正则匹配查询

#### 9.Fuzzy Query
模糊查询使用基于Levenshtein编辑距离的字符串字段的相似性，以及数字和日期字段的+/-边距

参考链接:https://scsundefined.gitbooks.io/elasticsearch-the-definitive-guide-cn/content/s05/07_02_fuzzy_query.html

#### 10.Type Query
过滤与所提供的文档/映射类型匹配的文档

```
{
    "type" : {
        "value" : "my_type"
    }
}
```

#### 11.Ids Query
过滤只提供了 uid 的文档

```
{
    "ids" : {
        "type" : "my_type",
        "values" : ["1", "4", "100"]
    }
}
```

## 参考
1. [Query and filter context](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-filter-context.html)