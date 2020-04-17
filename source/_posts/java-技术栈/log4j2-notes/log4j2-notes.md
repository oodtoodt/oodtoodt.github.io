---
title: log4j2-notes
date: 2020-04-14 15:48:47
tags:
---

http://logging.apache.org/log4j/2.x/manual/appenders.html
http://www.slf4j.org/manual.html
https://maven.apache.org/pom.html#Properties

<!--more-->

## 知识点
log4j2有一套寻找配置文件的流程。

### 节点分析
根节点Configuration有两个属性:status和monitorinterval,有两个子节点:Appenders和Loggers
+ Appender定义了输出的格式
+ 这些logger通过name进行区分，来对不同的logger配置不同的输出

status指定log4j本身的打印日志的级别
monitorinterval指定log4j自动重新配置的检测间隔时间

#### Appenders
常见的有三种子节点:Console、RollingFile、File
##### console
用来定义输出到控制台的Appender.
name
target 
PatternLayout输出格式，不设置默认为:%m%n.
##### flie
用来定义输出到指定位置的文件的Appender.
name
fileName
PatternLayout
##### RollingFile
用来定义超过指定大小自动删除旧的创建新的的Appender.
name
fileName
PatternLayout
filePattern
Policies指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.

TimeBasedTriggeringPolicy:Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am.

SizeBasedTriggeringPolicy:Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.

DefaultRolloverStrategy:用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性)。

#### Loggers
常见的有两种:Root和Logger.
##### Root
用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出

level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.打印往高处打

AppenderRef：Root的子节点，用来指定该日志输出到哪个Appender.

##### Logger
level
name
AppenderRef


### 名称继承
用`.`继承，存在向下的继承关系。
additivity是 子Logger 是否继承 父Logger 的 输出源（appender）的标志位。具体说，默认情况下子Logger会继承父Logger的appender，也就是说子Logger会在父Logger的appender里输出。若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出。
所以经过一通探究，这里到底在说什么呢？在说到底应该如何调用。一般我们用log都是跟踪类的，所以我们写log4j2的时候就可以跟踪相应的类（package和类名等等）


### syslog
空



## 先来简单例子
其实就是缺省形态
```xml
 <?xml version="1.0" encoding="UTF-8"?>  
<configuration status="OFF">  
    <appenders>  
        <Console name="Console" target="SYSTEM_OUT">  
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>  
        </Console>  
    </appenders>  
    <loggers>  
        <root level="error">  
            <appender-ref ref="Console"/>  
        </root>  
    </loggers>  
</configuration>
```

## 复杂例子
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
    Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出。 
-->
<!--
    monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数。
-->
<configuration status="error" monitorInterval=”30″>
    <!--先定义所有的appender-->
    <appenders>
        <!--这个输出控制台的配置-->
        <Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
            <!--这个都知道是输出日志的格式-->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </Console>
        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用-->
        <File name="log" fileName="log/test.log" append="false">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File> 
        <!-- 这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFile" fileName="logs/app.log"
                    filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
            <SizeBasedTriggeringPolicy size="50MB"/>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--建立一个默认的root的logger-->
        <root level="trace">
            <appender-ref ref="RollingFile"/>
            <appender-ref ref="Console"/>
        </root> 
    </loggers>
</configuration> 
```

## json
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration status="OFF">  
  <appenders>  
    <Console name="Console" target="SYSTEM_OUT">  
      <JsonLayout/> <!--使用json格式输出-->
    </Console>  
  </appenders>  
  <loggers>  
    <root level="trace">  
      <appender-ref ref="Console"/>  
    </root>  
  </loggers>  
</configuration>
```
>config - 插件配置。
locationInfo - 如果为“true”，则在生成的 JSON 中包含位置信息。
properties - 如果为“true”，则在生成的 JSON 中包含线程上下文映射。
propertiesAsList - 如果为 true，则将线程上下文映射包括为映射条目对象的列表，其中每个条目具有“key”属性（其值为键）和“value”属性（其值为值）。默认为 false，在这种情况下，线程上下文映射包含为键值对的简单映射。
complete - 如果为“true”，则包括 JSON 页眉和页脚，以及记录之间的逗号。
compact - 如果为“true”，则不使用行尾和缩进，默认为“false”。
eventEol - 如果为“true”，则在每个日志事件后强制执行 EOL（即使compact为“true”），默认为“false”。即使在紧凑模式下，这也允许一行甚至每一行。
headerPattern- 标题模式，默认为"[“如果是 null。
footerPattern- 标题模式，默认为”]"如果是 null。
charset- 要使用的字符集if null，使用“UTF-8”。
includeStacktrace - 如果为“true”，则包括生成的 JSON 中任何Throwable的堆栈跟踪，默认为“true”。
————————————————
版权声明：本文为CSDN博主「caolist」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/caolist/article/details/89330735


## 意犹未尽啊
那么就把slf4j的也放过来吧

slf4j = Simple Logging Facade for Java
就是做在日志上层的抽象类，底层具体用什么日志工具不用关心，用一个适配层把api转换成具体工具的接口，这样就可以移植了。

**slf4j只是一个日志标准，并不是日志系统的具体实现**。理解这句话非常重要，slf4j只做两件事情：

+ 提供日志接口
+ 提供获取具体日志对象的方法

slf4j的作用：只要所有代码都使用门面对象slf4j，我们就不需要关心其具体实现，最终所有地方使用一种具体实现即可，更换、维护都非常方便。

```java
import java.io.IOException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory; 
/**
* @author: ketao Date: 14-5-3 Time: 上午1:03 
* @version: \$Id$ 
*/ 
public class LogTest { 
    private static final Logger logger = LoggerFactory.getLogger(LogTest.class); 
    public static void main(String[] args) { 
        logger.info("纯字符串信息的info级别日志"); 
        logger.info("一个参数:{}的info级别日志", "agr1"); 
        logger.info("二个参数:agrs1:{};agrs2:{}的info级别日志", "args1", "args2"); // 下面两种方式都可以，一般使用上面一种就可以了 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", "args1", "args2", "args3"); 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", new Object[] { "args1", "args2", "args3" }); 
        logger.info("======================异常相关===================================="); // 测试异常相关日志 
        logger.info("抛出异常,e:", new IOException("测试抛出IO异常信息")); 
        logger.info("二个参数:agrs1:{};agrs2:{}的info级别日志", "args1", new IOException("测试抛出IO异常信息")); 
        logger.info("二个参数:agrs1:{};agrs2:{}的info级别日志", "args1", "args2", new IOException("测试抛出IO异常信息")); // 下面两种方式都可以，一般使用上面一种就可以了 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", "args1", "args2", new IOException("测试抛出IO异常信息")); 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", "args1", "args2", "agrs3", new IOException( "测试抛出IO异常信息")); 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", 
            new Object[] { "args1", "args2", "args3", 
                new IOException("测试抛出IO异常信息") 
            }
        ); 
        logger.info("三个参数:agrs1:{};agrs2:{};args3:{} 的info级别日志", 
            new Object[] { 
                "args1", "args2", 
                new IOException("测试抛出IO异常信息") 
            }
        ); 
    } 
}
```
throwable 异常信息单独作为一个参数输入，因此，如果把异常信息作为{}占位符中的字符串，则会调用其对应toString方法，而无法打印异常堆栈信息。

atTrace(), atDebug(), atInfo(), atWarn() and atError() methods是快速的Logging API

## log4j2的官方文档体系
### usage
最好声明为静态
关于name...
in the cloud
### Using Log4j 2 in Web Applications
You must take particular care when using Log4j or any other logging framework within a Java EE web application

### Lookups
查找系统提供了一种在任意位置向Log4j配置添加值的方法。 
#### Context Map Lookup

#### Date Lookup
这就是最常见的SimpleDateFormat的问题了，会查找这东西的，指匹配日期等

#### 环境查找
#### java查找
#### ...
还有很多查找。当你看到不认识的XX:`${XX:..}`，不要犹豫，先来查文档吧。

### Appenders
Appender负责将LogEvents传递到其目的地。

### Layouts
将logEvent格式化来满足日志事件的需求，比如json
### Filters
对日志事件评估，以确定是否或如何发布它们

### Asynchronous Loggers for Low-Latency Logging
低延迟日志记录的异步记录器
### Garbage-free Steady State Logging
无垃圾稳态记录

### 最后
我觉得没什么用，串个大概到时候查吧。
主要是太多了，而且很多都是空白的知识领域

## maven-POM.Reference.basics
### Maven Coordinates

### Relationships
#### dependecies
#### Inheritance
#### Aggregation
### properites
属性是了解POM基础知识的最后一个要素。Maven属性是值占位符，如Ant中的属性。它们的值可以通过使用符号${X}在POM中的任何位置访问，其中X是属性。


1. env.X：使用“env”来定义变量。将返回shell的环境变量。例如，${env.PATH}包含PATH环境变量。

注意：虽然环境变量本身在Windows上不区分大小写，但查找属性区分大小写。换句话说，虽然Windows shell为`％PATH％`和`％Path％`返回相同的值，但Maven区分`${env.PATH}`和`${env.Path}`。对于Maven 2.1.0，为了可靠性，环境变量的名称被归一化为所有大写。

2. `project.x：POM`中的点（.）记号路径将包含相应元素的值。例如：可以通过`${project.version}`访问`<project><version>1.0</version></project>`。

3. `settings.x：settings.xml`中的点（.）标注路径将包含相应的元素的值。例如：`<settings><offline>false</offline></ settings>`可通过`${settings.offline}`访问。

4. Java系统属性：可通过java.lang.System.getProperties()访问的所有属性都可用作POM属性，如${java.home}。

5. x：在POM中的`<properties />`元素中设置。`<properties><someVar>value</someVar></properties>`的值可以用作`${someVar}`。