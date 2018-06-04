---
layout: post
title: SpringBoot Redis
categories: [SpringBoot]
description: Listener 有助于完成其他的功能
keywords: SpringBoot,Java
---

# SpringBoot Redis
内存型数据库中, Redis 丰富的数据结构确实很便利,谁用谁知道.


## 任务目标
利用Spring Boot提供的数据访问框架Spring- Data-Redis(基于Jedis),去访问Redis。

## 样例实现

### 1.添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

### 2.Redis参数配置
按照惯例在application.properties中加入Redis服务端的相关配置,如下:

```properties
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

### 3.简单连接测试
Redis 中写入 string 类型的 `key` 和`value`

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
public class ApplicationTests {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Test
    public void test() throws Exception {
        // 保存字符串
        stringRedisTemplate.opsForValue().set("key", "value");
        Assert.assertEquals("value", stringRedisTemplate.opsForValue().get("key"));
    }
}
```

### 4.Redis中存入类对象
创建一个User类,并通过序列化和反序列化写入Redis

1. 创建要存储的对象：User
2. 实现对象的序列化接口
3. 配置针对User的RedisTemplate实例
4. 编写对应的测试类

[参考代码SpringBoot/Chapter2_0/](https://github.com/ITriangle/SpringBoot/tree/master/Chapter2_0)


## 总结分析

- 简单连接测试中,未手动创建redis连接,采用Spring Boot的默认连接,装配`stringRedisTemplate`

- Redis中存入类对象中,自己创建redis连接模板

1. 因为Spring Boot不支持直接使用 RedisTemplate<String, User>，需要我们自己实现RedisSerializer<T>接口来对传入对象进行序列化和反序列化
2. 配置针对User的RedisTemplate实例,需要自己手动读取`application.properties`中的配置

## 参考链接:
1. [Spring Data Redis](http://docs.spring.io/spring-data/redis/docs/1.6.2.RELEASE/reference/html/)