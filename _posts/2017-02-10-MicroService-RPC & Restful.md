---
layout: post
title: RPC & Restful
categories: [MicroService]
description: 
keywords: RPC,MicroService,Java
---

# RPC & Restful
在 MicroService 中,Restful 作为一种轻量级的通信方式是非常可取的.但,时常也面临带宽的限制.所以,RPC 和 Restful 可以作为黄金搭档.

接下来,先讲解一下RPC.


## 一.RPC
接下来,先用一个实例说明什么是 RPC.还是说明下,RPC（Remote Procedure Call Protocol）----远程过程调用协议,它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

### 1.服务的暴露和引用类

```java
public class RpcFramework {

    /**
     * 暴露服务
     *
     * @param service 服务实现
     * @param port    服务端口
     * @throws Exception
     */
    public static void export(final Object service, int port) throws Exception {

        if (service == null)
            throw new IllegalArgumentException("service instance == null");
        if (port <= 0 || port > 65535)
            throw new IllegalArgumentException("Invalid port " + port);

        System.out.println("Export service " + service.getClass().getName() + " on port " + port);
        ServerSocket server = new ServerSocket(port);

        for (; ; ) {
            try {
                final Socket socket = server.accept();

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            try {
                                ObjectInputStream input = new ObjectInputStream(socket.getInputStream());
                                try {
                                    String methodName = input.readUTF();
                                    Class<?>[] parameterTypes = (Class<?>[]) input.readObject();
                                    Object[] arguments = (Object[]) input.readObject();
                                    ObjectOutputStream output = new ObjectOutputStream(socket.getOutputStream());
                                    try {
                                        Method method = service.getClass().getMethod(methodName, parameterTypes);
                                        Object result = method.invoke(service, arguments);
                                        output.writeObject(result);
                                    } catch (Throwable t) {
                                        output.writeObject(t);
                                    } finally {
                                        output.close();
                                    }
                                } finally {
                                    input.close();
                                }
                            } finally {
                                socket.close();
                            }
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 引用服务
     *
     * @param <T>            接口泛型
     * @param interfaceClass 接口类型
     * @param host           服务器主机名
     * @param port           服务器端口
     * @return 远程服务
     * @throws Exception
     */
    @SuppressWarnings("unchecked")
    public static <T> T refer(final Class<T> interfaceClass, final String host, final int port) throws Exception {

        if (interfaceClass == null)
            throw new IllegalArgumentException("Interface class == null");
        if (!interfaceClass.isInterface())
            throw new IllegalArgumentException("The " + interfaceClass.getName() + " must be interface class!");
        if (host == null || host.length() == 0)
            throw new IllegalArgumentException("Host == null!");
        if (port <= 0 || port > 65535)
            throw new IllegalArgumentException("Invalid port " + port);
        System.out.println("Get remote service " + interfaceClass.getName() + " from server " + host + ":" + port);

        return (T) Proxy.newProxyInstance(interfaceClass.getClassLoader(), new Class<?>[]{interfaceClass}, new InvocationHandler() {
            public Object invoke(Object proxy, Method method, Object[] arguments) throws Throwable {
                Socket socket = new Socket(host, port);
                try {
                    ObjectOutputStream output = new ObjectOutputStream(socket.getOutputStream());
                    try {
                        output.writeUTF(method.getName());
                        output.writeObject(method.getParameterTypes());
                        output.writeObject(arguments);
                        InputStream is = socket.getInputStream();
                        ObjectInputStream input = new ObjectInputStream(is);
                        try {
                            Object result = input.readObject();
                            if (result instanceof Throwable) {
                                throw (Throwable) result;
                            }
                            return result;
                        } finally {
                            input.close();
                        }
                    } finally {
                        output.close();
                    }
                } finally {
                    socket.close();
                }
            }
        });

    }

}
```

### 2.定义服务接口 

```java
public interface HelloService {

    String hello(String name);

}
```

### 3.实现服务

```java
public class HelloServiceImpl implements HelloService {  
  
    public String hello(String name) {  
        return "Hello " + name;  
    }  
  
}  
```

### 4.暴露服务

```java
public class RpcProvider {  
  
    public static void main(String[] args) throws Exception {  
        HelloService service = new HelloServiceImpl();  
        RpcFramework.export(service, 1234);  
    }  
  
}
```

### 5.引用服务

```java
public class RpcConsumer {  
      
    public static void main(String[] args) throws Exception {  
        HelloService service = RpcFramework.refer(HelloService.class, "127.0.0.1", 1234);  
        for (int i = 0; i < Integer.MAX_VALUE; i ++) {  
            String hello = service.hello("World" + i);  
            System.out.println(hello);  
            Thread.sleep(1000);  
        }  
    }  
      
}
```

### 6.分析说明
RPC 相对的是本地服务调用,最简单的举例就是调用系统`API`.想想,我们在调用本地服务的时候,都是通过函数名,接着传入参数,然后等待返回值.

那么,你如果像理解本地服务调用一样,去简单的理解 RPC 也是可以的.里面就增加了两个概念:

1. 通过网络传输参数和返回值,甚至函数名也是网络传输的.
2. 传输的数据经过序列化和反序列化

其实,增加的两个点是很好理解的.RPC 相对于 本地服务.序列化/反序列化则是考量了传输格式.如果理解 Restful 中常用的 JSON 格式也是可以理解的.

但,增加了两点,就增加了实现和控制的复杂度.上面的实例是没有实现序列化和反序列化的.


## 参考
1. [RPC框架几行代码就够了](http://javatar.iteye.com/blog/1123915?page=2#comments)