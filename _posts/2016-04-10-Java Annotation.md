---
layout: post
title: Java Annotation
categories: [Java,Annotation]
description: 简单工厂属于创造模型类
keywords: Annotation,Java
---

# 注解 Annotation
注解实质上是一种动态配置.用来简化程序中的配置

## Annotation分类

- 元注解：自定义注解时，用来修饰注解的
    - 描述注解的作用范围
    - 描述注解的生命周期
    - 描述注解的继承特性
    - 描述注解是否在javadoc中出现
- 原生注解：JDK自带的注解，如`@Overwried`
- 第三方注解:第三方框架自定义的注解。如Spring中的`@Autowried`、`@Service`
- 自定义注解:自己定义的注解

## Annotation生命周期

> 注解被读取的时间点

- 源码阶段
- 编译阶段
- 运行阶段

## Annotation作用范围

- 包
- 类：DI注入
- 字段：类的成员变量
- 方法：类的方法
- 方法的参数：
- 局部变量：



## 参考
1. [深入理解Java：注解（Annotation）基本概念](http://www.cnblogs.com/peida/archive/2013/04/23/3036035.html)
2. [深入理解Java：注解（Annotation）自定义注解入门](http://www.cnblogs.com/peida/archive/2013/04/24/3036689.html)
3. [深入理解Java：注解（Annotation）--注解处理器](http://www.cnblogs.com/peida/archive/2013/04/26/3038503.html)
4. [Java深度历险（六）——Java注解](http://www.infoq.com/cn/articles/cf-java-annotation)