---
layout: post
title: Java 并发 synchronized
categories: [并发编程]
description: Java 并发 synchronized
keywords: 并发编程,Java
---

# Java 并发 synchronized
在 Java 编程中，经常需要用到同步，而用得最多的也许是`synchronized`关键字了，下面看看这个关键字的用法。
因为 `synchronized` 关键字涉及到锁的概念，所以先来了解一些相关的锁知识

## Java内置锁

- java的内置锁：每个java对象都可以用做一个实现同步的锁，这些锁成为内置锁。线程进入同步代码块或方法的时候会自动获得该锁，在退出同步代码块或方法时会释放该锁。获得内置锁的唯一途径就是进入这个锁的保护的同步代码块或方法。
- java内置锁是一个互斥锁，这就是意味着最多只有一个线程能够获得该锁，当线程A尝试去获得线程B持有的内置锁时，线程A必须等待或者阻塞，知道线程B释放这个锁，如果B线程不释放这个锁，那么A线程将永远等待下去

## Java的对象锁和类锁

- java的锁属于对象和calss类
- 对象锁的`synchronized`修饰方法和代码块
- 类锁的`synchronized`修饰 **静态方法和静态代码块**
- 对于同步方法块，锁是`synchonized`括号里配置的对象。

## 为什么区分对象锁和类锁
其实，类锁修饰方法和代码块的效果和对象锁是一样的，因为类锁只是一个抽象出来的概念，只是为了区别静态方法的特点，因为静态方法是所有对象实例共用的，所以对应着synchronized修饰的静态方法的锁也是唯一的，就抽象出来个类锁。

## 两个锁的区别
类锁和对象锁是两个不一样的锁，控制着不同的区域，它们是互不干扰的。同样，线程获得对象锁的同时，也可以获得该类锁，即同时获得两个锁，这是允许的。

## 降低锁粒度

- 方法修饰:必须基于该对象锁.同一个对象不能降低粒度,但可以声明多个对象
- 代码块修饰:可以多个对象降低锁粒度

```java
class FineGrainLock {
   MyMemberClass x, y;
   Object xlock = new Object(), ylock = new Object();
   public void foo() {
      synchronized(xlock) {
         //access x here
      }
      //do something here - but don't use shared resources
      synchronized(ylock) {
         //access y here
      }
   }
   public void bar() {
      synchronized(this) {
         //access both x and y here
      }
      //do something here - but don't use shared resources
   }
}
```

## 参考
1. [java synchronized关键字的用法](http://zhh9106.iteye.com/blog/2151791)
2. [Java中的锁](http://www.importnew.com/19472.html)