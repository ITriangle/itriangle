---
layout: post
title: Elasticsearch 搜索二:搜索API分类对比
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

# Elasticsearch 查询 `DSL` 语句说明
`Query DSL` 为 `_search API` 提供了查询的参数

### 查询条件由三部分构成:
1. 查询的字段:全部字段,某类字段,指定字段
2. 查询字段符合的条件:是否为空,是否包含,范围内
3. 查询字段条件间逻辑组合方式:与或非

### 根据查询条件,查询又可以分为:
1. 模糊查询:解决,文档符不符合的问题
2. 精确匹配(过滤):解决,文档是不是的问题

## `query DSL`(查询) 和 `filter DSL`(过滤) 简要说明
首先过滤，然后查询剩余的文档(Filter first ， then query remaining docs).

两者对比图:

queries | filters
---| ---
relevance | boolean yes/no
full text | exact values
not cached| cached
slower    | faster

### query DSL

> 在查询上下文中，查询会回答这个问题——“这个文档匹不匹配这个查询，相关度是多少？”  

- 关心文档的关联度高不高（relevance），会计算分值_score
- 可以模糊匹配
- 无缓存
- 因为查询结果无缓存，下一次查询速度较慢

### filter DSL

> 在过滤器上下文中，查询会回答这个问题——“这个文档与查询条件匹不匹配？”  

- 匹配则是yes，不匹配是no，不关心分值
- 精确匹配
- 有缓存
- 因为有缓存，下一次查询速度较快

## 字段匹配说明


## 参考
1. [Query and filter context](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-filter-context.html)