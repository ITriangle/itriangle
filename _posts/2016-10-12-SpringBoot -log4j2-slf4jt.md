---
layout: post
title: SpringBoot log4j2 + slf4j
categories: SpringBoot
description: Spring Boot 作为Spring 配置的简化项目,具有很强的实用性
keywords: SpringBoot,log
---

# 日志
日志在应用程序中的重要性是可想而知的.接下来就介绍 log4j2 + slf4j 在 SpringBoot 中的使用.

## 一.Maven依赖
这个还是很要注意版本问题的.

```xml
<!--去掉springboot本身日志依赖-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <!--log4j2-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>
```

## 二.添加 log4j2.xml 文件(与 application.properties 同级)
可以见我 log4j2 中详细配置的说明.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--启动项设置为 trace，加载 springboot 启动内部各种详细输出-->
<Configuration status="trace">
    <Appenders>
        <!--添加一个控制台追加器-->
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout>
                <pattern>[%-5p] %d %c - %m%n</pattern>
            </PatternLayout>
        </Console>
        <!--添加一个文本追加器，文件位于根目录下，名为log.log-->
        <File name="File" fileName="log.log">
            <PatternLayout>
                <pattern>[%-5p] %d %c - %m%n</pattern>
            </PatternLayout>
        </File>
    </Appenders>
    <Loggers>
        <Logger name="org.apache.catalina.util.LifecycleBase" level="error" />
        <Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn" />
        <Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn" />
        <Logger name="org.hibernate.validator.internal.util.Version" level="warn" />
        <Logger name="org.springframework" level="warn" />
        <!--记录 com.wangl 包及其子包 debug 及其以上的记录，并输出到文件中-->
        <Logger name="com.wangl" level="debug">
            <!-- AppenderRef 可以控制文件输出对象-->
            <AppenderRef ref="File" />
        </Logger>
        <!--根记录全部输出到控制台上-->
        <Root level="debug">
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

## 三.配置 application.properties 
在 application.properties 中加入配置文件的扫描位置,启用自己的配置

```xml
logging.config=classpath:log4j2.xml 
```

## 实例代码

```java

import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class TestType {

//    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    private final Logger logger = LoggerFactory.getLogger(TestType.class);

//     private static final Logger logger = LoggerFactory.getLogger(TestType.class);

    @Test
    public void test1(){
        logger.debug("This is a debug message");
        logger.info("This is an info message");
        logger.warn("This is a warn message");
        logger.error("This is an error message");

        logger.error("error {}", "param");//slf4j 支持的写法,不使用字符串拼接的方式,效率更高
    }
}
```

## 参考链接
1. [Spring Boot日志管理](http://blog.didispace.com/springbootlog/)