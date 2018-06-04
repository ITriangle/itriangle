---
layout: post
title: 动态代理
categories: [MicroService,Java]
description: 动态代理通过继承和多态实现对类进行的了扩展
keywords: MicroService,Java
---

# 动态代理
学习Spring不可能错过:IOC 和 AOP.AOP 实现了面向对象的编程,让类更加专注于自己的业务,体现了高内聚,是个不可多得的功能.

## 一.动态代理 Java 的 API

### 1.InvocationHandler 接口
官方说明:

> InvocationHandler is the interface implemented by the invocation handler of a proxy instance. 

> Each proxy instance has an associated invocation handler. When a method is invoked on a proxy instance, the method invocation is encoded and dispatched to the invoke method of its invocation handler.

每一个动态代理类都必须要实现`InvocationHandler`这个接口，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由`InvocationHandler`这个接口的 `invoke` 方法来进行调用。我们来看看InvocationHandler这个接口的唯一一个方法 `invoke` 方法.

### 2.invoke 方法

```java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable
```

参数说明:

1. **proxy:**　　指代我们所代理的那个真实对象
2. **method:**　　指代的是我们所要调用真实对象的某个方法的Method对象
3. **args:**　　指代的是调用真实对象某个方法时接受的参数

### 3.Proxy 类
官方说明:

> Proxy provides static methods for creating dynamic proxy classes and instances, and it is also the superclass of all dynamic proxy classes created by those methods. 

`Proxy` 这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 `newProxyInstance` 这个方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)  throws IllegalArgumentException
```

说明:

1. loader:　　一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载
2. interfaces:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了
3. h:　　一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上
4. 返回值:`Returns an instance of a proxy class for the specified interfaces that dispatches method invocations to the specified invocation handler.`返回调用指定接口方法的实例.

## 二.实例

### 1.自定义接口

```java
public interface TestHello {

    void hello(String name);

    void sayGood();

}
```

### 2.实现自定接口类

```java
public class TestHelloImpl implements TestHello{

    @Override
    public void hello(String name) {
        System.out.println("hello " + name);
    }

    @Override
    public void sayGood() {
        System.out.println("you are the best!");
    }
}
```

### 3.自定义代理类,实现 InvocationHandler

```java
public class MyProxy implements InvocationHandler {

    //这个就是我们要代理的真实对象
    private Object proxy;

    //构造方法，给我们要代理的真实对象赋初值
    public MyProxy(Object proxy) {
        this.proxy = proxy;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //在代理真实对象前我们可以添加一些自己的操作
        System.out.println("Do something before proxy!");

        //当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        method.invoke(this.proxy, args);

        //在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("Do something after proxy!");

        return null;
    }

}
```

### 4.测试客户端

```java
public class Client {

    public static void main(String[] args) {

        //我们要代理的真实对象
        TestHello reaTh = new TestHelloImpl();

        //我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
        MyProxy mp = new MyProxy(reaTh);

        /*
         * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
         * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
         * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
         * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
         */
        TestHello proxyTh = (TestHello) Proxy.newProxyInstance(mp.getClass().getClassLoader(), reaTh.getClass().getInterfaces(), mp);

        proxyTh.hello("Jerry");

        proxyTh.sayGood();

        System.out.println(proxyTh.getClass().getName());
    }
}
```

输出

```
Do something before proxy!
hello Jerry
Do something after proxy!
Do something before proxy!
you are the best!
Do something after proxy!
com.sun.proxy.$Proxy0
```

### 5.输出结果分析:

`com.sun.proxy.$Proxy0` 由 `System.out.println(proxyTh.getClass().getName());` 打印.这个就是代理类的类名.

这里可能会有疑惑:

可能我以为返回的这个代理对象会是`TestHello`类型的对象，或者是`InvocationHandler`的对象，结果却不是，首先我们解释一下为什么我们这里可以将其转化为`TestHello`类型的对象？

原因就是在`newProxyInstance`这个方法的第二个参数上，我们给这个代理对象提供了一组什么接口，那么我这个代理对象就会实现了这组接口，这个时候我们当然可以将这个代理对象强制类型转化为这组接口中的任意一个，因为这里的接口是`TestHello`类型，所以就可以将其转化为`TestHello`类型了。

同时我们一定要记住，通过 Proxy.newProxyInstance 创建的代理对象是在jvm运行时动态生成的一个对象，它并不是我们的InvocationHandler类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象，并且命名方式都是这样的形式，以$开头，proxy为中，最后一个数字表示对象的标号。

```
Do something before proxy!

Do something after proxy!
```

上面的输出也就证明了当我通过代理对象来调用方法的时候，起实际就是委托由其关联到的 handler 对象的invoke方法中来调用，并不是自己来真实调用，而是通过代理的方式来调用的。这就是我们的 Java 动态代理机制



## 参考
1. [java的动态代理机制详解](http://www.cnblogs.com/xiaoluo501395377/p/3383130.html)