---
layout: post
title: SpringBoot 快速入门
categories: SpringBoot
description: Spring Boot 作为Spring 配置的简化项目,具有很强的实用性
keywords: SpringBoot
---

## 任务目标
用Maven构建Spring Boot基础项目，并且实现一个简单的Http请求处理，通过这个例子对 Spring Boot 有一个初步的了解，并体验其结构简单、开发快速的特性。

## 一.Maven构建项目
1. 通过 [`SPRING INITIALIZR`](http://start.spring.io/) 工具产生基础项目, `Spring Boot` 版本 `1.3.8`
2. 解压后,IDE导入工程,默认选择即可


## 二.项目结构

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- web
      |  +- CustomerController.java
      |
```

- root package结构：com.example.myproject
- 应用主类Application.java置于root package下，通常我们会在应用主类中做一些框架配置扫描等配置，我们放在root package下可以帮助程序减少手工配置来加载到我们希望被Spring加载的内容
- 实体（Entity）与数据访问层（Repository）置于com.example.myproject.domain包下
- 逻辑层（Service）置于com.example.myproject.service包下
- Web层（web）置于com.example.myproject.web包下

## 三.编写实例:
> Spring Boot 是完全遵从Spring的启动原则的,只是优化了配置(约定优先于配置)  

创建 `Example` 类，内容如下:

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

启动主程序，打开浏览器访问`http://localhost:8080`，可以看到页面输出Hello World
  

## 四.启动说明  

### 1. 注解配置

`@RestController`  

`@EnableAutoConfiguration`

`@RequestMapping`


### 2. 启动流程  

1. 程序入口`main`
2. `main` 方法通过调用 `run`，将业务委托给了 `Spring Boot` 的 `SpringApplication` 类
3. `SpringApplication` 将引导应用启动:
    1. 启动Spring
    2. 相应地启动被自动配置的Tomcat web服务器
4. `Example.class` 作为参数传递给 `run` 方法，**以此告诉SpringApplication谁是主要的Spring组件**，并传递 `args` 数组以暴露所有的命令行参数。

## 五.创建可运行的`jar`

为了创建可执行的jar，我们需要将spring-boot-maven-plugin添加到pom.xml中，在dependencies节点后面插入以下内容：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```


PS:`spring-boot-starter-parent` POM 包含绑定到repackage目标的<executions>配置。如果不使用parent POM，你需要自己声明该配置，具体参考[插件文档](http://docs.spring.io/spring-boot/docs/1.4.1.BUILD-SNAPSHOT/maven-plugin/usage.html)。

### 运行 `jar` 包 
在工程目录下运行一下命令:

- 打包jar

```
mvn package
```

- 运行jar

```
java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

## 参考链接
1. [Spring Boot参考指南](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/content/I.%20Spring%20Boot%20Documentation/3.%20First%20steps.html)
2. [Spring Boot快速入门](http://blog.didispace.com/spring-boot-learning-1/)