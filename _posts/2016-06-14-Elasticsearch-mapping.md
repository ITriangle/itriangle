---
layout: post
title: Elasticsearch 基础 mapping
categories: [Elasticsearch]
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

## 什么是mapping
如果将Elasticsearch理解为一个RDBMS（关系型数据库，比如MySQL），那么index 就相当于数据库实例，type可以理解为表,这样mapping可以理解为表的结构和相关设置的信息（当然mapping有更大范围的意思）。  

默认情况不需要显式的定义mapping， 当新的type或者field引入时，Elasticsearch会自动创建并且注册有合理的默认值的mapping(毫无性能压力)， 只有要覆盖默认值时才必须要提供mapping定义。

**官方定义**
> A mapping defines the fields within a type, the datatype for each field, and how the field should be handled by Elasticsearch. A mapping is also used to configure metadata associated with the type.  

Mapping定义了type中的诸多字段的数据类型以及这些字段如何被Elasticsearch处理，比如一个字段是否可以查询以及如何分词等.

## 如何自定义mapping
下面是一个简单的Mapping定义：

```shell
curl -XPUT 'http://localhost:9200/megacorp' -d '
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }, 
    "mappings": {
        "employee": {
            "properties": {
                "first_name": {
                    "type": "string"
                }, 
                "last_name": {
                    "type": "string"
                }, 
                "age": {
                    "type": "integer"
                }, 
                "about": {
                    "type": "string"
                }, 
                "interests": {
                    "type": "string"
                }, 
                "join_time": {
                    "type": "date", 
                    "format": "dateOptionalTime", 
                    "index": "not_analyzed"
                }
            }
        }
    }
}
'
```

其中 **employee是type(/index/type/id),相当于关系数据库中的表** ，在`employee`中我们定义了`first_name`、`last_name`、`age`、`about`、`interests`、`join_time`这6个属性。

指定mapping的分片和拷贝

```
"settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
}
```

## 具体分析`field`

### `field`的类型`type`
`type`表示`field`的数据类型，下例中`interests`的`type`为`string`表示为普通文本。 

```
"interests": {
    "type": "string"
}
```

### Elasticsearch支持以下数据类型：
- 文本: string
- 数值: byte, short, integer, long
- 浮点数: float, double
- 布尔值: boolean
- Date: date

### `field`最重要属性:`index` `analyzer`

#### `index`
`index` 属性控制 `string` 如何被索引，它有三个可选值:
- analyzed: First analyze the string, then index it. In other words, index this field as full text.
- not_analyzed:: Index this field, so it is searchable, but index the value exactly as specified. Do not analyze it.
- no: Don’t index this field at all. This field will not be searchable.

对于 `string` 类型的 `filed index` 默认值是： `analyzed`.如果我们想对进行精确查找, 那么我们需要将它设置为： `not_analyzed`。 

```
"interests": {
  "type": "string",
  "index": "not_analyzed"
}
```

#### `analyzer `
对于 `string` 类型的字段, 我们可以使用 `analyzer `属性来指定在搜索阶段和索引阶段使用哪个分词器. 默认, Elasticsearch 使用 `standard analyzer`, 你也可以指定Elasticsearch内建的其它分词器，比如 `whitespace`, `simple`, `or english`:

```
"interests": {
  "type": "string",
  "analyzer": "english"
}
```

### 查询索引

```shell
curl -XGET 'http://localhost:9200/megacorp/_mapping?pretty'
{
  "megacorp": {
    "mappings": {
      "employee": {
        "properties": {
          "about": {
            "type": "string"
          },
          "age": {
            "type": "long"
          },
          "first_name": {
            "type": "string"
          },
          "interests": {
            "type": "string"
          },
          "last_name": {
            "type": "string"
          }
        }
      }
    }
  }
}

```

## 参考链接
1. [Elasticsearch实战系列-mapping 设置](http://blog.csdn.net/top_code/article/details/50767138)