---
layout: post
title: SpringBoot Listener
categories: [SpringBoot]
description: Listener 有助于完成其他的功能
keywords: SpringBoot,Java
---

# SpringBoot Listener
SpringBoot 在启动过程中增加事件监听机制，为用户功能拓展提供极大的便利。

**支持的事件类型四种**:

- `ApplicationStartedEvent` : spring boot启动开始时执行的事件,在该事件中可以获取到`SpringApplication`对象，可做一些执行前的设置.
- `ApplicationEnvironmentPreparedEvent`:对应 `Enviroment` 已经准备完毕，但此时上下文 `context` 还没有创建.在该监听中获取到`ConfigurableEnvironment`后可以对配置信息做操作，例如：修改默认的配置信息，增加额外的配置信息等等
- `ApplicationPreparedEvent`:上下文`context`创建完成，但此时`spring`中的`bean`是没有完全加载完成的,在获取完上下文后，可以将上下文传递出去做一些额外的操作。在该监听器中是无法获取自定义bean并进行操作的。
- `ApplicationFailedEvent`:启动异常时执行事件,在异常发生时，最好是添加虚拟机对应的钩子进行资源的回收与释放，能友善的处理异常信息

**实现监听步骤**:

1. 监听类实现 `ApplicationListener` 接口 
2. 将监听类添加到 `SpringApplication` 实例


# 实例
这里还是参考了,Spring 中的 `ContextRefreshedEvent` 监听事件,SpringBoot 也是支持的.上下文`context`创建完成,自定义bean也可以被自动装载.

## 1.实现 `ApplicationListener`

```java
public class ApplicationStartup implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        ApplicationConf applicationConf = contextRefreshedEvent.getApplicationContext().getBean(ApplicationConf.class);

        File dir = new File(applicationConf.dataDir);
        if (!dir.exists()){
            dir.mkdir();
        }
    }
}
```

## 2.添加监听器

### 1.直接在配置文件中添加,监听器类

```java
# 配置启动监听器
context.listener.classes=com.seentech.conf.ApplicationStartup
```

### 2.在执行入口处添加

```java
SpringApplication springApplication =new SpringApplication(Application.class);
springApplication.addListeners(new ApplicationStartup());
springApplication.run(args);
```

相比之下,第一种方式更加的灵活.

## 参考
1. [spring boot实战(第二篇)事件监听](http://blog.csdn.net/liaokailin/article/details/48186331)
2. [在Spring Boot启动后执行指定代码](https://www.huangyunkun.com/2015/01/01/run-code-after-spring-boot-started/)