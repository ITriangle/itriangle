---
layout: post
title: Gson 
categories: JSON
description: Gson JSON 解析又快又好
keywords: Gson,JSON
---
# Gson

## 任务目标
Gson 好用不贵,谁用谁知道

## 一.Maven构建项目
在享受Gson解析的高度封装带来的便利时，有时可能会遇到一些特殊情况，比如json数据中的字段key是动态可变的时候，由于Gson是使用静态注解的方式来设置实体对象的，因此我们很难直接对返回的类型来判断。但Gson在解析过程中如果不知道解析的字段，就会将所有变量存储在一个Map中，我们只要实例化这个map就能动态地取出key和value了。然后,在用 `Gson` 转化为对象是极好用的.

## 参考链接
1. [你真的会用Gson吗?Gson使用指南](http://www.jianshu.com/p/e740196225a4)
2. [Gson解析JSON中动态未知字段key的方法](http://blog.csdn.net/Chaosminds/article/details/49049455)