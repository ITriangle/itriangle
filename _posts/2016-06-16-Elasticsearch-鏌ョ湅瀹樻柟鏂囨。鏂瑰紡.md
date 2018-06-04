---
layout: post
title: 如何查看 Elasticsearch Reference
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

# Elasticsearch Reference

## 如何学习 Elasticsearch
查看官方文档,如果你英语不错的话!比如我,英语稍微差一点,先可以尝试看一下官方文档,是否可以懂.如果不行,可以先找好一点的中文文档了解一下知识背景.再去看英文文档.

## 什么是 Elasticsearch Reference
Elasticsearch Reference 就是 Elasticsearch 官网的参考手册.我查看的是 [Elasticsearch Reference 2.4](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/index.html).写了如何使用 Elasticsearch.

## 如何阅读 Elasticsearch Reference
对于英文文档,可以先了解整体的文档结构,这样对知识体系有一个详细了解,就可以有条理,有顺序,有需求的读英文手册

我是属于任务型驱动的.所以,一般是针对某一个问题去寻找解决方案.这对于一个新知识是不可取的.至少需要对该组件有一个整体的认识后,再进行单个深入.那么,最好的方式就是官方手册的简介和目录.

### 接下来,就详细分析一下目录:
1. Getting Started : 开始
2. Setup : 步骤
3. Breaking changes : 版本更新说明
4. API Conventions : API 大体说明
5. Document APIs : 文档API
6. Search APIs : 搜索API(Elasticsearch的作用就是搜索,重要程度可想而知)
7. Aggregations : 聚合API(聚合分析)
8. Indices APIs : 索引API(index 操作)
9. cat APIs : cat API(用作应答信息的格式为表格,便于人阅读,自然就是各种信息的展示)
10. Cluster APIs : 集群API(cluster 操作)
11. Query DSL : 查询 DSL
12. Mapping : mapping
13. Analysis : 分词
14. Modules : 模型
15. Index Modules : 索引模型
16. Testing : 测试
17. Glossary of terms
18. Release Notes : 版本说明

### 目录分析完毕,针对需求有的放矢

#### 基础部分
1. 1-3部分,可以快速的略过,一般找个中文的安装测试文档就可以解决
2. 4 部分,要看下,毕竟是大体认识
3. 12 部分,我认为比较重要,在对 Elasticsearch 有一定体会后,可以先看,毕竟索引的字段是基本

#### 需求部分
1. 针对需求就是:CURD,那么从创建,到搜索
2. 5,6,8,11 包含了 CURD 的相关操作
3. 进一步需求,用 Elasticsearch 做聚合分析, 自然 7 部分就要开始了

#### 性能优化
这个就是比较难的了,需要深入理解一下原理性的问题

1. 集群信息:9,10 部分
2. 原理基础:13,14,15部分

## 总结
到这里,基本描述完了我看 Elasticsearch Reference 方法.在看的基础上,再配合练.基本上就没有问题了.英文,并不可怕,可怕的是不知道如何有效的学习.

其实,读其他类型的英文文档,也基本是这种方式,没有什么巧妙的技巧,读多了就自然了.
