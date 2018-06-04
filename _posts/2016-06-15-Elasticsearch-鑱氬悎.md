---
layout: post
title: Elasticsearch 聚合
categories: Elasticsearch
description: Elasticsearch 在实际中的作用是越来越强了
keywords: Elasticsearch
---

## 聚合[API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)  

聚合的作用:**统计数据集合的各项参数指标**  

- bucketing API的使用:得到数据集合
- metric API的使用:统计指定数据集合中类属性的各项参数指标  

### 聚合API
```
"aggregations" : {                  // 表示聚合操作，可以使用aggs替代
    "<aggregation_name>" : {        // 聚合名，可以是任意的字符串。用做数据集合参数值对应的key，便于快速取得正确的响应数据。
        "<aggregation_type>" : {    // 聚合指标，数据集合的参数指标，如min取最小,terms分组
            <aggregation_body>      // 聚合体,数据集合中需要求参数的具体字段,排布方式,等属性要求
        }
        [,"aggregations" : { [<sub_aggregation>]+ } ] // 嵌套的子聚合，可以有0或多个
    }
    [,"<aggregation_name_2>" : { ... } ]* // 另外的聚合，可以有0或多个
}
```

### 度量类型（metric）聚合
- `min`:数据集合,指定属性的最小值
- `max`:数据集合,指定属性的最大值
- `sum`:数据结合,指定属性的总和
- `avg`:数据集合,指定属性的平均值
- `stats`:数据集合,指定属性的基础参数 
- `grade_stats`:数据集合,指定属性的增强参数
- `cardinality`:数据集合,指定属性去重
- `percentiles`:数据集合,指定属性的所占的百分比
- `top_hits`:数据集合,指定属性的前n条数据

### 桶类型（bucketing）聚合   

#### Terms Aggregation
按照指定的1或多个字段将数据划分成若干个小的区间，计算落在每一个区间上记录数量，并按指定顺序进行排序。下面统计每个班的学生数，并按学生数从大到小排序，取学生数靠前的2个班级。

```
curl -XPOST "192.168.1.101:9200/student/student/_search?search_type=count" -d 
'
{
  "aggs": {
    "terms_classNo": {
      "terms": {
        "field": "classNo",            // 按照班号进行分组 
        "order": { "_count": "desc"},  // 按学生数从大到小排序
        "size": 2                      // 取前两名
      }
    }
  }
}
'
```

#### Range Aggregation
自定义区间范围的聚合，我们可以自己手动地划分区间，ES会根据划分出来的区间将数据分配不同的区间上去。

#### Date Range Aggregation
时间区间聚合专门针对date类型的字段，它与Range Aggregation的主要区别是其可以使用时间运算表达式。主要包括+（加法）运算、-（减法）运算和/（四舍五入）运算，每种运算都可以作用在不同的时间域上面，下面是一些时间运算表达式示例。

now+10y：表示从现在开始的第10年。
now+10M：表示从现在开始的第10个月。
1990-01-10||+20y：表示从1990-01-01开始后的第20年，即2010-01-01。
now/y：表示在年位上做舍入运算。今天是2015-09-06，则这个表达式计算结果为：2015-01-01。

#### Histogram Aggregation
直方图聚合，它将某个number类型字段等分成n份，统计落在每一个区间内的记录数。它与前面介绍的Range聚合非常像，只不过Range可以任意划分区间，而Histogram做等间距划分。既然是等间距划分，那么参数里面必然有距离参数，就是interval参数。

#### Date Histogram Aggregation
时间直方图聚合，专门对时间类型的字段做直方图聚合。这种需求是比较常用见得的，我们在统计时，通常就会按照固定的时间断（1个月或1年等）来做统计。

#### Missing Aggregation
值缺损聚合，它是一类单桶聚合，也就是最终只会产生一个“桶”。


## 嵌套使用
聚合操作是可以嵌套使用的。通过嵌套，可以使得metric类型的聚合操作作用在每一“桶”上。我们可以使用ES的嵌套聚合操作来完成稍微复杂一点的统计功能。下面统计每一个班里最大的年龄值。  

```
curl -XPOST "192.168.1.101:9200/student/student/_search?search_type=count" -d
'
{
  "aggs": {
    "missing_address": {
      "terms": {
        "field": "classNo"
      },
      "aggs": {                 // 在这里嵌套新的子聚合
        "max_age": {
          "max": {              // 使用max聚合
            "field": "age"
          }
        }
      }
    }
  }
}
'
```

## 参考链接
[实时搜索引擎Elasticsearch（4）——Aggregations （聚合）API的使用](http://blog.csdn.net/xialei199023/article/details/48298635)