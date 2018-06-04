---
layout: post
title: SpringBoot Restful API
categories: SpringBoot
description: Spring Boot 作为Spring 配置的简化项目,具有很强的实用性
keywords: SpringBoot,Restful
---

## 任务目标
设计如下 RESTful API：

请求类型  |   URL   |   功能说明
---|---|---
GET     | /users      | 查询用户列表
POST    | /users      | 创建一个用户
GET     | /users/id   | 根据id查询一个用户
PUT     | /users/id   | 根据id更新一个用户
DELETE  | /users/id   | 根据id删除一个用户

## 项目结构参照 [Chapter1_0](https://github.com/ITriangle/SpringBoot/tree/master/Chapter1_0)

## 用到的注解
- @Controller：修饰class，用来创建处理http请求的对象
- @RestController：Spring4之后加入的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接用@RestController替代@Controller就不需要再配置@ResponseBody，默认返回json格式。
- @RequestMapping：配置url映射

参数绑定的注解参照 [HTTP 请求参数绑定的注解](http://triangleidea.com/2016/09/11/Annotation-Http/)

## [GitHub代码地址](https://github.com/ITriangle/SpringBoot/tree/master/Chapter1_1)