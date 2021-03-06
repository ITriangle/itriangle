---
layout: post
title: SpringBoot 定时任务和异步任务
categories: [SpringBoot]
description: Listener 有助于完成其他的功能
keywords: SpringBoot,Java
---

# SpringBoot 定时任务
SpringBoot对常用的功能进行了封装,比如定时任务和异步任务.用注解的方式简化了创建的流程.


## SpringBoot 定时任务
定时任务需要使用到: `@EnableScheduling` 和 `@Scheduled` 注解.

- `@EnableScheduling` :添加在主类上,使能定时任务的配置
- `@Scheduled` :添加在定时任务上,参数为时间

SpringBoot 的封装确实做得很好,有时间可以详细阅读以下 [几种任务调度的 Java 实现方法与比较](https://www.ibm.com/developerworks/cn/java/j-lo-taskschedule/).读完后会发现,SpringBoot的定时任务的优雅和不俗.

简单的定时任务样例:

```java
@Component
public class ScheduledTasks {
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        System.out.println("现在时间：" + dateFormat.format(new Date()));
    }
}
```

### 1.详细说明 `@Scheduled` 参数对应的时间配置

```java
@Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行
@Scheduled(fixedDelay = 5000) //上一次执行完毕时间点之后5秒再执行
@Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
@Scheduled(cron="*/5 * * * * *") //通过cron表达式定义规则,每五秒一次
```

### 2.回顾下:cron表达式定义规则

#### 2.1.Cron表达式时间字段

位置|时间域名|允许值|允许的特殊字符
---|---|---|---
1|秒|0-59|, - * /
2|分钟|0-59|, - * /
3|小时|0-23|, - * /
4|日期|1-31|, - * ? / L W C
5|月份|1-12|, - * /
6|星期|1-7|, - * ? / L C #
7|年(可选)|空值1970-2099|, - * /

#### 2.2.Cron表达式的时间字段除允许设置数值外，还可使用一些特殊的字符，提供列表、范围、通配符等功能，细说如下：

- `*`：可用在所有字段中，表示对应时间域的每一个时刻，例如，*在分钟字段时，表示“每分钟”；

- `?`：该字符只在日期和星期字段中使用，它通常指定为“无意义的值”，相当于点位符；

- `-`：表达一个范围，如在小时字段中使用“10-12”，则表示从10到12点，即10,11,12；

- `,`：表达一个列表值，如在星期字段中使用“MON,WED,FRI”，则表示星期一，星期三和星期五；

- `/`：x/y表达一个等步长序列，x为起始值，y为增量步长值。如在分钟字段中使用0/15，则表示为0,15,30和45秒，而5/15在分钟字段中表示5,20,35,50，你也可以使用*/y，它等同于0/y；

- `L`：该字符只在日期和星期字段中使用，代表“Last”的意思，但它在两个字段中意思不同。L在日期字段中，表示这个月份的最后一天，如一月的31号，非闰年二月的28号；如果L用在星期中，则表示星期六，等同于7。但是，如果L出现在星期字段里，而且在前面有一个数值X，则表示“这个月的最后X天”，例如，6L表示该月的最后星期五；

- `W`：该字符只能出现在日期字段里，是对前导日期的修饰，表示离该日期最近的工作日。例如15W表示离该月15号最近的工作日，如果该月15号是星期六，则匹配14号星期五；如果15日是星期日，则匹配16号星期一；如果15号是星期二，那结果就是15号星期二。但必须注意关联的匹配日期不能够跨月，如你指定1W，如果1号是星期六，结果匹配的是3号星期一，而非上个月最后的那天。W字符串只能指定单一日期，而不能指定日期范围；

- `LW`：在日期字段可以组合使用LW，它的意思是当月的最后一个工作日；

- `#`：该字符只能在星期字段中使用，表示当月某个工作日。如6#3表示当月的第三个星期五(6表示星期五，#3表示当前的第三个)，而4#5表示当月的第五个星期三，假设当月没有第五个星期三，忽略不触发；

- `C`：该字符只在日期和星期字段中使用，代表“Calendar”的意思。它的意思是计划所关联的日期，如果日期没有被关联，则相当于日历中所有日期。例如5C在日期字段中就相当于日历5日以后的第一天。1C在星期字段中相当于星期日后的第一天。

#### 2.3.Cron表示式示例

表示式|说明
---|---
"0 0 12 * * ? "|每天12点运行
"0 15 10 ? * *"|每天10:15运行
"0 15 10 * * ?"|每天10:15运行
"0 15 10 * * ? *"|每天10:15运行
"0 15 10 * * ? 2008"|在2008年的每天10：15运行
"0 * 14 * * ?"|每天14点到15点之间每分钟运行一次，开始于14:00，结束于14:59。
"0 0/5 14 * * ?"|每天14点到15点每5分钟运行一次，开始于14:00，结束于14:55。
"0 0/5 14,18 * * ?"|每天14点到15点每5分钟运行一次，此外每天18点到19点每5钟也运行一次。
"0 0-5 14 * * ?"|每天14:00点到14:05，每分钟运行一次。
"0 10,44 14 ? 3 WED"|3月每周三的14:10分到14:44，每分钟运行一次。
"0 15 10 ? * MON-FRI"|每周一，二，三，四，五的10:15分运行。
"0 15 10 15 * ?"|每月15日10:15分运行。
"0 15 10 L * ?"|每月最后一天10:15分运行。
"0 15 10 ? * 6L"|每月最后一个星期五10:15分运行。
"0 15 10 ? * 6L 2007-2009"|在2007,2008,2009年每个月的最后一个星期五的10:15分运行。
"0 15 10 ? * 6#3"|每月第三个星期五的10:15分运行。

## 参考
1. [Scheduling Tasks](http://spring.io/guides/gs/scheduling-tasks/)
