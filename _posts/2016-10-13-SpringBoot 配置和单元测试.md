---
layout: post
title: SpringBoot IOC 结合服务器配置和单元测试
categories: [SpringBoot]
description: SpringBoot服务器应用和配置
keywords: SpringBoot,Java
---

# SpringBoot 服务器配置
spring Boot 其默认是集成web容器的，启动方式由像普通Java程序一样，main函数入口启动。其内置Tomcat容器或Jetty容器，具体由配置来决定（默认Tomcat）。当然你也可以将项目打包成war包，放到独立的web容器中（Tomcat、weblogic等等），当然在此之前你要对程序入口做简单调整。

项目构建我们使用Maven或Gradle，这将使项目依赖、jar包管理、以及打包部署变的非常方便。

## 一.自动配置
自动配置,可以读取外部的配置文件,直接实例化 Bean.

### 1.在resources 中新建配置文件 mail.properties

```
mail.protocol=STMP
mail.host=localhost
mail.port=25

```

### 2.创建 MailConfig 读取配置文件

```java
@Component //不加这个注解的话, 使用@Autowired 就不能注入进去了
@PropertySource("classpath:mail.properties") //说明当前的配置文件路径
public class MailConfig {


    @Value("${mail.protocol}")
    private String protocol;

    @Value("${mail.host}")
    private String host;

    @Value("${mail.port}")
    private int port;

    @Override
    public String toString() {
        return "MailConfig{" +
                "protocol='" + protocol + '\'' +
                ", host='" + host + '\'' +
                ", port=" + port +
                '}';
    }
}
```

### 3.编写测试用例
测试上面的注解必须的加上.

```java
/**
 * 测试时,最好在测试类上加如下两个注解
 */
@RunWith(SpringRunner.class) //告诉Junit运行使用Spring 的单元测试支持；SpringRunner是SpringJunit4ClassRunner新的名称，只是视觉上看起来更简单了。
@SpringBootTest//该注解可以在一个测试类指定运行Spring Boot为基础的测试。
public class ApplicationTests {

    @Test
    public void contextLoads() {
    }

    @Autowired
    public ServerObj serverObj;

    @Test
    public void test1(){
        System.out.println(serverObj.toString());
    }

    @Autowired
    public MailConfig mailConfig;

    @Test
    public void test2(){
        System.out.println(mailConfig.toString());
    }

}
```


### 4.依赖于 IOC Bean 的对象
依赖于 IOC Bean 的对象,该对象也需要依赖 IOC 注解自动装载对象.如果,自己`new`,就需要自己指定配置项了.

例如:

```java
@Component
public class TestServer {

    @Autowired
    private MailConfig mailConfig;

    
    public TestServer(MailConfig mailConfig) {
        this.mailConfig = mailConfig;
    }

    public void testMail(){
        System.out.println(mailConfig.protocol);

        System.out.println("Hello world");
    }
}
```

调用时:使用 `estServer testServer = new TestServer();` 创建对象就会自动装载失败,替代了自动装载,自己手动实例化的对象.

所以最好将上面的 `@Autowired` 自动装载写到构造函数上是最好的.避免使用无参数构造函数使用.查找错误有难度啊!

```java
@Component
public class TestServer {


    private MailConfig mailConfig;

    @Autowired
    public TestServer(MailConfig mailConfig) {
        this.mailConfig = mailConfig;
    }
    

    public void testMail(){
        System.out.println(mailConfig.protocol);

        System.out.println("Hello world");
    }
}
```

## 二.单元测试
普通的不依赖于环境的单元测试是没有问题的.但是当依赖于SpringBoot 上下文中时,就需要完整的单元测试说明.

```java
/**
 * 测试时,最好在测试类上加如下两个注解
 */
@RunWith(SpringRunner.class)

@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)

public class MyTest{

    // ...

}
```

- `@RunWith(SpringRunner.class)`: 告诉Junit运行使用Spring 的单元测试支持
- `SpringRunner` 是 `SpringJunit4ClassRunner` 新的名称，只是视觉上看起来更简单了
- `@SpringBootTest` : 该注解可以在一个测试类指定运行 Spring Boot 为基础的测试.当然还可以配置一些属性.

当然不仅仅就这么简单了，SpringBoot 1.4 在单元测试还有另外一些特性，大家可以在去官方看看文档，比如还有 `@JsonTest`，`@DataJpaTest` 等。


## 参考
1. [ Spring Boot 部署与服务配置](http://blog.csdn.net/catoop/article/details/50588851)