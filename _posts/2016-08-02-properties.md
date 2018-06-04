---
layout: post
title: properties 配置
categories: [Java]
description: 熟练的 Java 工具包有助于开发....................................................................................................................................................................................................
keywords: log,Java
---
# properties
`.properties` 配置是 Java 中常用的,还有另外一个常用的是 `yml`.选择很多,总有一款适合你.

## 一.配置 
Java中比较重要的类 `Properties（Java.util.Properties）`, 主要用于读取 Java 的配置文件，各种语言都有自己所支持的配置文件，配置文件中很多变量是经常改变的，这样做也是为了方便用户，让用户能够脱离程序本身去修改相关的变量设置。

像 Python 支持的配置文件是 `.ini` 文件，同样，它也有自己读取配置文件的类 ConfigParse，方便程序员或用户通过该类的方法来修改 `.ini` 配置文件。

在Java中，其配置文件常为 `.properties` 文件，格式为文本文件，文件的内容的格式是`Key=Value` 的格式，文本注释信息可以用 `#` 来注释。

## 二.Java配置类 Properties
Properties类继承自Hashtable，是线程安全的.如下：

```Java

Class Properties

java.lang.Object
    java.util.Dictionary<K,V>
        java.util.Hashtable<Object,Object>
            java.util.Properties

All Implemented Interfaces:
    Serializable, Cloneable, Map<Object,Object>
Direct Known Subclasses:
    Provider

public class Properties
extends Hashtable<Object,Object>
```

它提供了几个主要的方法：

1. `getProperty ( String key)`，用指定的键在此属性列表中搜索属性。也就是通过参数 key ，得到 key 所对应的 value.
2. `load ( InputStream inStream)`，从输入流中读取属性列表（键和元素对）。通过对指定的文件（比如说上面的 test.properties 文件）进行装载来获取该文件中的所有键 - 值对。以供 getProperty ( String key) 来搜索。
3. `setProperty ( String key, String value)`，调用 Hashtable 的方法 put 。他通过调用基类的put方法来设置 键 - 值对。
4. `store ( OutputStream out, String comments)`，以适合使用 load 方法加载到 Properties 表中的格式，将此 Properties 表中的属性列表（键和元素对）写入输出流。与 load 方法相反，该方法将键 - 值对写入到指定的文件中去。
5. `clear ()`，清除所有装载的 键 - 值对。该方法在基类中提供。

### Java获取 `.properties` 文件方法
Java 读取`.properties` 文件的方法有很多，详见：参考2. 但是最常用的还是通过java.lang.Class类的getResourceAsStream(String name)方法来实现，如下可以这样调用：作为我们写程序的，用此一种足够。

```Java
InputStream in = getClass().getResourceAsStream("资源Name");
```

或者下面这种也常用：

```Java
InputStream in = new BufferedInputStream(new FileInputStream(filepath))
```

## 三.Java读取 `.properties` 文件实例

### 1.获取Java虚拟机(JVM)系统配置文件
来,我们讲个有意思的,后去虚拟机的配置文件:

```java
import java.util.Properties;

public class ReadJVM {
    public static void main(String[] args) {
        Properties pps = System.getProperties();
        pps.list(System.out);
    }
}
```

输出太多,可以参考自己的

### 2.自定义配置文件

#### 2.1新建配置`Test.properties`文件,内容如下

```
name=JJ
#注释为`#`
#Weight=4444
Height=3333
```

#### 2.1解析,获取value值:
通过读取配置文件的绝对路径

```java
public static void main(String[] args) throws IOException {
    String ConfPath =  "E:\\WLGIT\\Sword\\src\\com\\wangl1\\Test.properties";

    Properties properties = new Properties();

    //获取文件输入流
    FileInputStream fileInputStream = new FileInputStream(ConfPath);

    //加载文件输入流
    properties.load(fileInputStream);

    //获取配置文件中key列表
    Enumeration enumeration = properties.propertyNames();

    //通过key获取相应的value
    while (enumeration.hasMoreElements()){
        String key = (String) enumeration.nextElement();
        String value = properties.getProperty(key);

        System.out.println(key + "=" + value);
    }
}
```

输出
```
name=JJ
Height=3333
```

## 四.获取路径的方式
对上面实例中,绝对路径的优化.

### 1.获取当前类所在的包：

```java
File fileB = new File(this.getClass().getResource("").getPath());

System.out.println("fileB path: " + fileB);
```

### 2.获取当前类所在的工程名：

```java
System.out.println("user.dir path: " + System.getProperty("user.dir"));
```

### 3.获取路径的一个简单的Java实现：

```java
/**
 * 获取项目的相对路径下文件的绝对路径
 *
 * @param parentDir 目标文件的父目录,例如说,工程的目录下,有lib与bin和conf目录,那么程序运行于lib or
 *                  <p>
 *                  bin,那么需要的配置文件却是conf里面,则需要找到该配置文件的绝对路径
 * @param fileName  文件名
 * @return 一个绝对路径
 */

public static String getPath(String parentDir, String fileName) {
    String path = null;
    String userdir = System.getProperty("user.dir");
    String userdirName = new File(userdir).getName();
    if (userdirName.equalsIgnoreCase("lib")|| userdirName.equalsIgnoreCase("bin")) {
        File newf = new File(userdir);
        File newp = new File(newf.getParent());
        if (fileName.trim().equals("")) {
            path = newp.getPath() + File.separator + parentDir;
        } else {
            path = newp.getPath() + File.separator + parentDir + File.separator + fileName;
        }
    } else {
        if (fileName.trim().equals("")) {
            path = userdir + File.separator + parentDir;
        } else {
            path = userdir + File.separator + parentDir + File.separator + fileName;
        }
    }
    return path;
}
```

### 4.利用反射的方式获取路径：

```java
InputStream ips1 = Enumeration.class.getClassLoader() .getResourceAsStream("cn/zhao/enumStudy/testPropertiesPath1.properties");

        

InputStream ips2 = Enumeration.class.getResourceAsStream("testPropertiesPath1.properties");

        

InputStream ips3 = Enumeration.class.getResourceAsStream("properties/testPropertiesPath2.properties");
```

## 五.程序读取外部配置
Java的main方法有个初始化入参args，将参数表示为配置文件的路径，代码如下：

```java
public static void main(String[] args) {
        loadConf(args[0]);
 }

public static void loadConf(String path) throws Exception {
         Properties props = new Properties();
         InputStream in = new FileInputStream(path);
         props.load(in);
         fromDB = props.getProperty("fromDB");
         fromDBUser = props.getProperty("fromDBUser");
         fromDBPassword = props.getProperty("fromDBPassword");
         if (StringUtils.isEmpty(fromDB)) {
             String errmsg = "fromDB or tables is null";
             logger.error(errmsg);
             throw new Exception(errmsg);
         }
 }
```

**打包**:通过 Maven 或者其他工具打包

**运行**:在linux下执行jar包引入外部配置文件的命令（window下比如进入d: 同样的道理，java -jar XXX.jar config.properties）：



## 六.参考链接
1. [Java中Properties类的操作](http://www.cnblogs.com/bakari/p/3562244.html)
2. [Java读取Properties文件的六种方法](http://blog.csdn.net/Senton/article/details/4083127)
3. [Java项目中读取properties文件，以及六种获取路径的方法](http://www.cnblogs.com/allenzhaox/p/3215776.html)