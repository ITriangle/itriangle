---
layout: post
title: Spark Mlib Guid
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---

# Spark机器学习库(MLlib)指南
MLlib是Spark里的机器学习库.它的目标是使实用的机器学习算法可扩展并容易使用.它提供如下工具：

1. 机器学习算法：常规机器学习算法包括分类、回归、聚类和协同过滤.
2. 特征工程：特征提取、特征转换、特征选择以及降维.
3. 管道：构造、评估和调整的管道的工具.
4. 存储：保存和加载算法、模型及管道
5. 实用工具：线性代数,统计,数据处理等.

# API 转变说明
**基于DataFrame的API是主要的API,基于MLlib RDD的API现在处于维护模式**

从Spark 2.0开始,spark.mllib软件包中基于RDD的API已进入维护模式. Spark的主要机器学习API现在是spark.ml包中的基于DataFrame的API.

### 这一转变包含哪些信息？

1. MLlib将继续在spark.mllib中支持基于RDD的接口.
2. MLlib不会向基于RDD的接口中继续添加新的特征.
3. 在Spark2.0以后的版本中,将继续向基于数据框的接口添加新特征以缩小与基于RDD接口的差异.
4. 当两种接口之间达到特征相同时(初步估计为Spark2.2),基于RDD的接口将被废弃.
5. 基于RDD的接口将在Spark3.0中被移除.

### 为什么MLlib转向数据框接口？

1. 数据框架可以提供比RDD更容易掌握使用的接口.数据框架的主要优点包括Spark数据源来源、结构化查询语言的数据框架查询、各编程语言之间统一的接口.
2. 基于数据框架的MLlib接口为多种机器学习算法与编程语言提供统一的接口.
3. 数据框架有助于实现机器学习管道,特别是特征转换.[管道指南中可查看详细信息](http://spark.apache.org/docs/latest/ml-pipeline.html)

### Spark ML 是什么?
Spark ML 不是一个很官方的名称,但是基于 MLlib DataFrame-based API.因为基于DataFrame的API使用的org.apache.spark.ml Scala包名称,以及我们最初想用来强调管道概念的 "Spark ML Pipelines" 术语.

### MLlib 是否已被弃用?
没有, Mlib 包含 RDD-base API 和 DataFrame-base API, RDD-base API 现在是维护模式, 但没有一个 API 被弃用,MLib 也不能代表所有了.

### 依赖(Dependencies)
MLlib使用线性代数包Breeze,它依赖于netlib-java进行优化的数值处理.如果本地库在运行时不可用,您将看到一条警告消息,并将使用纯JVM实现.

考虑到运行专有二进制文件的许可问题,我们默认不包括netlib-java的本地代理. 要配置netlib-java / Breeze以使用系统优化的二进制文件,请将com.github.fommil.netlib：all：1.1.2(或使用-Pnetlib-lgpl构建Spark)作为项目的依赖项,并阅读netlib-java文档 为您的平台的额外安装说明.

要在Python中使用MLlib,您需要 [NumPy](http://www.numpy.org/) 版本1.4或更高版本.

## 迁移指南(Migration guide)
MLlib正在积极发展.标记为Experimental / DeveloperApi的API在将来的版本中可能会更改,下面的迁移指南将解释版本之间的所有更改.

### From 2.0 to 2.1

### 剧烈的变化
如下方法被移除

- setLabelCol in feature.ChiSqSelectorModel
- numTrees in classification.RandomForestClassificationModel (This now refers to the Param called numTrees)
- numTrees in regression.RandomForestRegressionModel (This now refers to the Param called numTrees)
- model in regression.LinearRegressionSummary
- validateParams in PipelineStage
- validateParams in Evaluator

### 弃用和修改的调整

- 弃用
    - [SPARK-18592](https://issues.apache.org/jira/browse/SPARK-18592): Deprecate all Param setter methods except for input/output column Params for DecisionTreeClassificationModel, GBTClassificationModel, RandomForestClassificationModel, DecisionTreeRegressionModel, GBTRegressionModel and RandomForestRegressionModel 
- 更改
    - [SPARK-17870](https://issues.apache.org/jira/browse/SPARK-17870): Fix a bug of ChiSqSelector which will likely change its result. Now ChiSquareSelector use pValue rather than raw statistic to select a fixed number of top features.
    - [SPARK-3261](https://issues.apache.org/jira/browse/SPARK-3261): KMeans returns potentially fewer than k cluster centers in cases where k distinct centroids aren’t available or aren’t selected.
    - [SPARK-17389](https://issues.apache.org/jira/browse/SPARK-17389): KMeans reduces the default number of steps from 5 to 2 for the k-means|| initialization mode.

## 以前的版本
查看以前版本的更改,[查看链接](http://spark.apache.org/docs/latest/ml-migration-guides.html)

## 参考
1. [Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-guide.html)