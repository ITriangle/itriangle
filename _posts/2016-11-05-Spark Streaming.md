---
layout: post
title: Spark Streaming
categories: [Spark,大数据]
description: Spark Streaming 作为 Spark 流数据处理的核心组件
keywords: 大数据,Spark,Java
---

# Spark Streaming

Spark Streaming 类似于 Apache Storm，用于流式数据的处理。根据其官方文档介绍，Spark Streaming 有高吞吐量和容错能力强这两个特点。Spark Streaming 支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。另外 Spark Streaming 也能和 MLlib（机器学习）以及 Graphx 完美融合。

在 Spark Streaming 中，处理数据的单位是一批而不是单条，而数据采集却是逐条进行的，因此 Spark Streaming 系统需要设置间隔使得数据汇总到一定的量后再一并操作，这个间隔就是批处理间隔。批处理间隔是 Spark Streaming 的核心概念和关键参数，它决定了 Spark Streaming 提交作业的频率和数据处理的延迟，同时也影响着数据处理的吞吐量和性能。

## StreamingContext

## DStream 与 RDD

## 1.测试实例

## 特点
批处理,并行化


## 参考
1. [Spark Streaming](https://www.ibm.com/developerworks/cn/opensource/os-cn-spark-streaming/index.html#icomments)