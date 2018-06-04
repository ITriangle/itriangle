---
layout: post
title: SpringBoot 异步任务
categories: [SpringBoot]
description: Listener 有助于完成其他的功能
keywords: SpringBoot,Java
---

# SpringBoot 异步任务
SpringBoot对常用的功能进行了封装,比如定时任务和异步任务.用注解的方式简化了创建的流程.


## SpringBoot 异步任务
异步任务是相对于同步任务调用的,实质是开启多线程任务执行.异步任务同样使用两个注解: `@EnableAsync` 和 `@Async`

- `@EnableAsync` :添加在主类上,使能异步配置
- `@Async` :添加在异步任务上,自定义异步任务, @Async所修饰的函数不要定义为static类型，这样异步调用不会生效

## 同步执行任务

### 1.同步执行任务样例 `doTaskOne`、`doTaskTwo`、`doTaskThree`

```java
@Component
public class Task {
    public static Random random =new Random();
    public void doTaskOne() throws Exception {
        System.out.println("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    }
    public void doTaskTwo() throws Exception {
        System.out.println("开始做任务二");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
    }
    public void doTaskThree() throws Exception {
        System.out.println("开始做任务三");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
    }
}
```

### 2.测试三个任务

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class ApplicationTests {
    @Autowired
    private Task task;
    @Test
    public void test() throws Exception {
        task.doTaskOne();
        task.doTaskTwo();
        task.doTaskThree();
    }
}
```

### 3.输出
`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数顺序的执行完成。

```
开始做任务一
完成任务一，耗时：5226毫秒
开始做任务二
完成任务二，耗时：5937毫秒
开始做任务三
完成任务三，耗时：8163毫秒
```


## 异步任务
在 `doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数加上 `@Async` 注解.

### 1.测试
此时可以反复执行单元测试，您可能会遇到各种不同的结果，比如：

- 没有任何任务相关的输出
- 有部分任务相关的输出
- 乱序的任务相关的输出

原因是`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数已经是异步执行了。主程序在异步调用之后，主程序并不会理会这三个函数是否执行完成了，由于主程序在没有其他需要执行的内容，就自动结束了，导致了不完整或是没有输出任务相关内容的情况。

### 2.等待异步任务完成

为了让`doTaskOne`、`doTaskTwo`、`doTaskThree`能正常结束，假设我们需要统计一下三个任务并发执行共耗时多少，这就需要等到上述三个函数都完成调动之后记录时间，并计算结果。

那么我们如何判断上述三个异步调用是否已经执行完成呢？我们需要使用`Future<T>`来返回异步调用的结果，就像如下方式改造`doTaskOne`函数：

```java
@Async
public Future<String> doTaskOne() throws Exception {
    System.out.println("开始做任务一");
    long start = System.currentTimeMillis();
    Thread.sleep(random.nextInt(10000));
    long end = System.currentTimeMillis();
    System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    return new AsyncResult<>("任务一完成");
}
```

按照如上方式改造一下其他两个异步函数之后，下面我们改造一下测试用例，让测试在等待完成三个异步调用之后来做一些其他事情。

```java
@Test
public void test() throws Exception {
    long start = System.currentTimeMillis();
    Future<String> task1 = task.doTaskOne();
    Future<String> task2 = task.doTaskTwo();
    Future<String> task3 = task.doTaskThree();
    while(true) {
        if(task1.isDone() && task2.isDone() && task3.isDone()) {
            // 三个任务都调用完成，退出循环等待
            break;
        }
        Thread.sleep(1000);
    }
    long end = System.currentTimeMillis();
    System.out.println("任务全部完成，总耗时：" + (end - start) + "毫秒");
}
```

单元测试的改变:

- 在测试用例一开始记录开始时间
- 在调用三个异步函数的时候，返回`Future<String>`类型的结果对象
- 在调用完三个异步函数之后，开启一个循环，根据返回的`Future<String>`对象来判断三个异步函数是否都结束了。若都结束，就结束循环；若没有都结束，就等1秒后再判断。
- 跳出循环之后，根据结束时间 - 开始时间，计算出三个任务并发执行的总耗时。

输出:

```
开始做任务一
开始做任务二
开始做任务三
完成任务三，耗时：5661毫秒
完成任务二，耗时：55毫秒
完成任务一，耗时：2749毫秒
任务全部完成，总耗时：8365毫秒
```

## 参考
1. [Spring Boot中使用@Async实现异步调用](http://blog.didispace.com/springbootasync/)