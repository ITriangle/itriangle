---
layout: post
title: Java 并发 Reactor模型
categories: [并发编程]
description: 并发框架
keywords: 并发编程,Java
---

#

## C/S架构可以抽象为如下模型：
<center>![](http://onm5tc1df.bkt.clouddn.com/client-server.png)</center>

1. C就是Client(客户端).也可为B,则是Browser(浏览器)作为客户端
2. S就是Server(服务器)：服务器管理某种资源，并且通过操作这种资源来为它的客户端提供某种服务

## Socket
Socket是网络连接描述符,是所有网络通信协议的基础.下面是Server 监听 Socket 基本流程图:

<center>![](http://onm5tc1df.bkt.clouddn.com/socket.png)</center>

1. 服务端Socket会bind到指定的端口上，Listen客户端的”插入”
2. 客户端Socket会Connect到服务端
3. 当服务端Accept到客户端连接后
4. 就可以进行发送与接收消息了
5. 通信完成后即可Close

## IO

### 对于IO来说，我们听得比较多的是:

- BIO:阻塞IO
- NIO:非阻塞IO
- 同步IO
- 异步IO

### 以及其组合:

- 同步阻塞IO
- 同步非阻塞IO
- 异步阻塞IO
- 异步非阻塞IO

### 那么什么是阻塞IO、非阻塞IO、同步IO、异步IO呢？
一个IO操作其实分成了两个步骤：发起IO请求和实际的IO操作

- 阻塞IO和非阻塞IO的区别在于第一步：发起IO请求是否会被阻塞，如果阻塞直到完成那么就是传统的阻塞IO;如果不阻塞，那么就是非阻塞IO
- 同步IO和异步IO的区别就在于第二个步骤是否阻塞，如果实际的IO读写阻塞请求进程，那么就是同步IO，因此阻塞IO、非阻塞IO、IO复用、信号驱动IO都是同步IO;如果不阻塞，而是操作系统帮你做完IO操作再将结果返回给你，那么就是异步IO

### 举例说明
比如你家网络断了，你打电话去中国电信报修！

1. 你拨号-----客户端连接服务器
2. 电话通了-----连接建立
3. 你说：“我家网断了,帮我修下”-----发送消息

IO请求处理:对应下面两种处理情况:

-  说完你就在那里等-----那么就是阻塞IO
-  如果正好你有事，你放下带电话，然后处理其他事情了，过一会你来问下，修好了没-----那就是非阻塞IO

IO操作处理:对应下面两种处理情况:

-  如果客服说：“马上帮你处理，你稍等”-----同步IO
-  如果客服说：“马上帮你处理，好了通知你”，然后挂了电话-----异步IO

## BIO

### 客户端代码

```java
//Bind,Connect
Socket client = new Socket("127.0.0.1",7777);    
//读写
PrintWriter pw = new PrintWriter(client.getOutputStream());
BufferedReader br=
        new BufferedReader(new InputStreamReader(System.in));  
pw.write(br.readLine());  
//Close
pw.close();  
br.close();
```

### 服务端代码

```java
Socket socket;  
//Bind,Listen
ServerSocket ss = new ServerSocket(7777);  
while (true) {  
    //Accept
    socket = ss.accept();  
    //一般新建一个线程执行读写
    BufferedReader br = new BufferedReader(
            new InputStreamReader(socket  .getInputStream()));  
    System.out.println("you input is : " + br.readLine());  
}
```

上面的代码可以说是学习Java的Socket的入门级代码了,可以看出服务端在等待IO的请求到来,客户端在等待IO的应答.模型图如下所示：

<center>![](http://onm5tc1df.bkt.clouddn.com/bio.png)</center>


### BIO优缺点

- 优点:模型简单,编码简单
- 缺点:性能瓶颈低

优缺点很明显。这里主要说下缺点：主要瓶颈在线程上。每个连接都会建立一个线程。虽然线程消耗比进程小，但是一台机器实际上能建立的有效线程有限，以Java来说，1.5以后，一个线程大致消耗1M内存！且随着线程数量的增加，CPU切换线程上下文的消耗也随之增加，在高过某个阀值后，继续增加线程，性能不增反降！而同样因为一个连接就新建一个线程，所以编码模型很简单！

就性能瓶颈这一点，就确定了BIO并不适合进行高性能服务器的开发！像Tomcat这样的Web服务器，从7开始就从BIO改成了NIO，来提高服务器性能！


## NIO

### NIO客户端代码(连接)

```java
//获取socket通道
SocketChannel channel = SocketChannel.open();        
channel.configureBlocking(false);
//获得通道管理器
selector=Selector.open();        
channel.connect(new InetSocketAddress(serverIp, port));
//为该通道注册SelectionKey.OP_CONNECT事件
channel.register(selector, SelectionKey.OP_CONNECT);
```

### NIO客户端代码(监听)

```java
while(true){
    //选择注册过的io操作的事件(第一次为SelectionKey.OP_CONNECT)
   selector.select();
   while(SelectionKey key : selector.selectedKeys()){
       if(key.isConnectable()){
           SocketChannel channel=(SocketChannel)key.channel();
           if(channel.isConnectionPending()){
               channel.finishConnect();//如果正在连接，则完成连接
           }
           channel.register(selector, SelectionKey.OP_READ);
       }else if(key.isReadable()){ //有可读数据事件。
           SocketChannel channel = (SocketChannel)key.channel();
           ByteBuffer buffer = ByteBuffer.allocate(10);
           channel.read(buffer);
           byte[] data = buffer.array();
           String message = new String(data);
           System.out.println("recevie message from server:, size:"
               + buffer.position() + " msg: " + message);
       }
   }
}
```

NIO模型示例如下：这里的Channel就是Socket文件描述符的替代.

1. Acceptor注册Selector，监听accept事件
2. 当客户端连接后，触发accept事件
3. 服务器构建对应的Channel，并在其上注册Selector，监听读写事件
4. 当发生读写事件后，进行相应的读写处理

<center>![](http://onm5tc1df.bkt.clouddn.com/nio.png)</center>

### NIO优缺点

- 优点:性能瓶颈高
- 缺点:模型复杂,编码复杂,需处理半包问题

NIO的优缺点和BIO就完全相反了!性能高，不用一个连接就建一个线程，可以一个线程处理所有的连接！相应的，编码就复杂很多，从上面的代码就可以明显体会到了。还有一个问题，由于是非阻塞的，应用无法知道什么时候消息读完了，就存在了半包问题！

<center>![](http://onm5tc1df.bkt.clouddn.com/%E5%8D%8A%E5%8C%85%E9%97%AE%E9%A2%98.png)</center>

我们知道TCP/IP在发送消息的时候，可能会拆包(如上图)！这就导致接收端无法知道什么时候收到的数据是一个完整的数据。例如:发送端分别发送了ABC,DEF,GHI三条信息，发送时被拆成了AB,CDRFG,H,I这四个包进行发送，接受端如何将其进行还原呢？在BIO模型中，当读不到数据后会阻塞，而NIO中不会!所以需要自行进行处理!例如，以换行符作为判断依据，或者定长消息发生，或者自定义协议！

NIO虽然性能高，但是编码复杂，且需要处理半包问题！为了方便的进行NIO开发，就有了Reactor模型!


## Reactor模型
Reactor模型，就是同步接收请求,将请求放到了一个队列中，通过异步线程池对其进行消费！

### Reactor中的组件

- Reactor:Reactor是IO事件的派发者。
- Acceptor:Acceptor接受client连接，建立对应client的Handler，并向Reactor注册此Handler。
- Handler:和一个client通讯的实体，按这样的过程实现业务的处理。一般在基本的Handler基础上还会有更进一步的层次划分， 用来抽象诸如decode，process和encoder这些过程。比如对Web Server而言，decode通常是HTTP请求的解析， process的过程会进一步涉及到Listener和Servlet的调用。业务逻辑的处理在Reactor模式里被分散的IO事件所打破， 所以Handler需要有适当的机制在所需的信息还不全（读到一半）的时候保存上下文，并在下一次IO事件到来的时候（另一半可读了）能继续中断的处理。为了简化设计，Handler通常被设计成状态机，按GoF的state pattern来实现。

对应上面的NIO代码来看:

- Reactor：相当于有分发功能的Selector
- Acceptor：NIO中建立连接的那个判断分支
- Handler：消息读写处理并返回等操作类

### Reactor 线程模型
Reactor从线程池和Reactor的选择上可以细分为如下几种：

#### Reactor单线程模型

<center>![](http://onm5tc1df.bkt.clouddn.com/reactor1.png)</center>

这个模型和上面的NIO流程很类似，只是将消息相关处理独立到了Handler中去了！

虽然上面说到NIO一个线程就可以支持所有的IO处理。但是瓶颈也是显而易见的！我们看一个客户端的情况，如果这个客户端多次进行请求，如果在Handler中的处理速度较慢，那么后续的客户端请求都会被积压，导致响应变慢！所以引入了Reactor多线程模型!

#### Reactor多线程模型

<center>![](http://onm5tc1df.bkt.clouddn.com/reactor2.png)</center>

Reactor多线程模型就是将Handler中的IO操作和非IO操作分开，操作IO的线程称为IO线程，非IO操作的线程称为工作线程!这样的话，客户端的请求会直接被丢到线程池中，客户端发送请求就不会堵塞！

但是当用户进一步增加的时候，Reactor会出现瓶颈！因为Reactor既要处理IO操作请求，又要响应连接请求！为了分担Reactor的负担，所以引入了主从Reactor模型!

#### 主从Reactor模型

<center>![](http://onm5tc1df.bkt.clouddn.com/reactor3.png)</center>

主Reactor用于响应连接请求，从Reactor用于处理IO操作请求！

## Netty
Netty是一个高性能NIO框架，其是对Reactor模型的一个实现！

### Netty客户端代码

```java
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    Bootstrap b = new Bootstrap();
    b.group(workerGroup);
    b.channel(NioSocketChannel.class);
    b.option(ChannelOption.SO_KEEPALIVE, true);
    b.handler(new ChannelInitializer<SocketChannel>() {
        @Override
        public void initChannel(SocketChannel ch) throws Exception {
            ch.pipeline().addLast(new TimeClientHandler());
        }
    });
    ChannelFuture f = b.connect(host, port).sync();
    f.channel().closeFuture().sync();
} finally {
    workerGroup.shutdownGracefully();
}
```

### Netty Client Handler

```java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        try {
            long currentTimeMillis =
                (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
                Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### Netty服务端代码

```java
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
     .channel(NioServerSocketChannel.class)
     .childHandler(new ChannelInitializer<SocketChannel>() {
         @Override
         public void initChannel(SocketChannel ch) throws Exception {
             ch.pipeline().addLast(new TimeServerHandler());
         }
     })
     .option(ChannelOption.SO_BACKLOG, 128)  
     .childOption(ChannelOption.SO_KEEPALIVE, true);
    // Bind and start to accept incoming connections.
    ChannelFuture f = b.bind(port).sync();
    f.channel().closeFuture().sync();
} finally {
    workerGroup.shutdownGracefully();
    bossGroup.shutdownGracefully();
}
```

### Netty Server Handler

```java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(final ChannelHandlerContext ctx) {
        final ByteBuf time = ctx.alloc().buffer(4);
        time.writeInt((int)
            (System.currentTimeMillis() / 1000L + 2208988800L));
        final ChannelFuture f = ctx.writeAndFlush(time);
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        });
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
        Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

我们从Netty服务器代码来看，与Reactor模型进行对应！

- EventLoopGroup就相当于是Reactor，bossGroup对应主Reactor,workerGroup对应从Reactor
- TimeServerHandler就是Handler
- child开头的方法配置的是客户端channel，非child开头的方法配置的是服务端channel