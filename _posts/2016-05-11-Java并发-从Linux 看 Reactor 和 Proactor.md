---
layout: post
title: Java 并发 从 Linux 看 Reactor 和 Proactor模型
categories: [并发编程]
description: 并发框架
keywords: 并发编程,Java
---

# Reactor 和 Proactor 模型
随着网络设计模式的兴起，Reactor和Proactor事件处理模式应运而生。同步I/O模型通常用于实现Reactor模式，异步I/O模型则用于实现Proactor模式。

## Reactor模式
Reactor 是这样一种模式，它要求主线程（I/O处理单元）只负责监听文件描述上是否有事件发生，有的话就立即将该事件通知工作线程（逻辑单元）。除此之外，主线程不做任何其他实质性的工作。读写数据，接受新的连接，以及处理客户请求均在工作线程中完成。

使用同步I/O模型（以epoll_wait为例）实现的Reactor模式的工作流程是：

1. 主线程往epoll内核时间表中注册socket上的读就绪事件。
2. 主线程调用epoll_wait等待socket上有数据可读。
3. 当socket上有数据可读时，epoll_wait通知主线程。主线程则将socket可读事件放入请求队列。
4. 睡眠在请求队列上的某个工作线程被唤醒，它从socket读取数据，并处理客户请求，然后往epoll内核事件表中注册该socket上的写就绪事件。
5. 主线程调用epoll_wait等待socket可写。
6. 当socket可写时，epoll_wait通知主线程。主线程则将socket可写事件放入请求队列。
7. 睡眠在请求队列上的某一个工作线程被唤醒，它往socket上写入服务器处理客户请求的结果。
<center>![](http://onm5tc1df.bkt.clouddn.com/%E5%BA%95%E5%B1%82%E6%A8%A1%E5%9E%8BReactor.png)</center>

## Proactor模式
与Reactor模式不同，Proactor模式将所有I/O操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。剥离了I/O操作.

使用异步I/O模型（以aio_read和aio_write为例）实现的Proactor模式的工作流程是：

1. 主线程调用aio_read函数向内核注册socket上的读完事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（可以通过信号）
2. 主线程继续处理其他逻辑。
3. 当socket上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，已通知应用程序数据已经可以用了。
4. 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用aio_write函数向内核注册socket上的写完事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序
5. 主线程继续处理其他逻辑。
6. 当用户缓冲区的数据被写入socket之后，内核将向应用程序发送一个信号，已通知应用程序数据已经发送完毕。
7. 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭socket。
<center>![](http://onm5tc1df.bkt.clouddn.com/%E5%BA%95%E5%B1%82%E6%A8%A1%E5%9E%8BProactor.png)</center>

## 参考
1. Linux 服务器编程