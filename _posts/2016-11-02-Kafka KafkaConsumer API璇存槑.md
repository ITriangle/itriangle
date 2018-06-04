---
layout: post
title: Kafka KafkaConsumer API 说明
categories: [大数据,Kafka]
description: 简单工厂属于创造模型类
keywords: 大数据,Kafka,Java
---

# Kafka KafkaConsumer API 说明
Kafka client 从 Kafka Cluster 消费记录.

它将透明地处理Kafka集群中服务器的故障，并透明地适应它在集群中提取的数据分区。 该客户端还与服务器交互以允许消费者组使用消费者组来负载均衡消费（如下所述）。

消费者维护与必要代理的TCP连接以提取数据。 使用后不能关闭消费者会泄漏这些连接。 消费者不是线程安全的。 有关更多详细信息，请参阅多线程处理。

## 一.Offsets and Consumer Position
Kafka为分区中的每个记录维护一个数字偏移量。该偏移用作该分区内的记录的一种唯一标识符，并且还表示分区中的消费者的位置。也就是说，具有位置5的消费者已经使用具有偏移量0到4的记录，并且将接下来接收具有偏移量5的记录。实际上存在与消费者的用户相关的位置的两个概念。

消费者的位置给出了将被给出的下一个记录的偏移量。它将是一个大于消费者在该分区中看到的最高偏移量。它每次消费者接收数据调用poll（long）和接收消息时自动前进。

提交的位置是已安全保存的最后一个偏移。如果进程失败并重新启动，这是它将恢复到的偏移。消费者可以周期性地自动提交偏移;或者它可以选择通过调用commitSync手动控制这个提交的位置，这将阻塞，直到偏移已成功提交或在提交过程中发生致命错误，或commitAsync是非阻塞的，并将触发OffsetCommitCallback成功提交或致命失败。

这种区别使消费者能够控制何时将记录视为已消耗。下面进一步详细讨论。

## 二.Consumer Groups and Topic Subscriptions


## 参考
1. [Class KafkaConsumer<K,V>](https://kafka.apache.org/090/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)