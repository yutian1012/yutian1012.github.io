---
title: 简单的日志实例
tags: [log4j]
---

场景：在开发过程中进程引入spring，作为开发轻量级应用的框架，从而简化代码的编写。项目中包括spring的jar包中都会使用log日志信息来记录和输出执行过程。有时候需要显示程序的详细执行信息，便于后期的错误排查。基于以上的原因采用引入log4j作为日志框架，以最快的方式实现日志的记录。

目标：以最简单最快速的情况下搭建日志记录功能。

### 引入log4j（实现）

1）需要将log4j的jar包放在到classpath中，或者使用maven的方式添加依赖

```
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

2）添加log4j的配置信息，放置到classpath下

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration PUBLIC "-//log4j/log4j Configuration//EN" "log4j.dtd">
<log4j:configuration>
    <!--输出到控制台-->
    <appender name="consoleAppender" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
        </layout>
    </appender>

    <root>
        <level value="ALL"/>
        <appender-ref ref="consoleAppender" />
    </root>
</log4j:configuration>
```

### 分析

1）明确spring日志的使用

在业务中为了方便查看执行过程，经常需要打印出日志信息，这里可以使用spring的方式来实现业务代码打印日志功能。

Spring日志的输出代码查看：

```
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

//创建Logger实例，用来操作日志信息
protected final Log logger = LogFactory.getLog(getClass());

//记录日志，可根据设置的日志级别进行日志的输出
//debug信息的输出
if (logger.isDebugEnabled()) {
    logger.debug("Initializing filter '" + filterConfig.getFilterName() + "'");
}

//错误信息的输出
catch (BeansException ex) {
    String msg = "Failed to set bean properties on filter '";
    logger.error(msg, ex);
    throw new NestedServletException(msg, ex);
}
```

注：spring的jar包中依赖commons-logging-1.2.jar实现日志信息的输出

2）commons-logging包中日志类

log信息的输出：trace，debug，info，warn，error等方法，用来输出不同的日志信息

3）commons-logging包如何加载log4j的实现类

获取Log日志对象

```
public static Log getLog(Class clazz) throws LogConfigurationException {
    //这里获取的LogFactory为LogFactoryImpl
    return getFactory().getInstance(clazz);
}
```

LogFactoryImpl内部会间接调用discoverLogImplementation方法（查看所有的Log实现）

```
//待发现的class类
private static final String[] classesToDiscover = {
    LOGGING_IMPL_LOG4J_LOGGER,//org.apache.commons.logging.impl.Log4JLogger
    "org.apache.commons.logging.impl.Jdk14Logger",
    "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
    "org.apache.commons.logging.impl.SimpleLog"
};

private Log discoverLogImplementation(String logCategory)
    throws LogConfigurationException {
    ...
    //classesToDiscover数组中的第一个元素就是Log4j，创建Logger时，默认先从Log4j开始创建，如果创建失败，在循环创建数组中的其他log对象，直到找到一个可以使用的Log。如classpath下存在Log4j的jar包则能正确创建成功并返回Log。
    for(int i=0; i<classesToDiscover.length && result == null; ++i) {
        result = createLogFromClass(classesToDiscover[i], logCategory, true);
    }
}
```

注：commons-logging包中的Log4JLogger是一个适配器类，该类中包含了log4j.jar包中的org.apache.log4j.Logger，如果classpath中没有找到会抛出异常。

注2：commons-logging包中不包含log4j的实现，但却引入了log4j包中的类，因此可以断定在commons-logging打包时一定依赖了log4j，但不把log4j打进包中。

4）log4j创建Logger日志

Log4JLogger（适配器类）调用Logger.getLogger创建Log4j的实例日志对象

```
import org.apache.log4j.Logger;

private transient volatile Logger logger = null;

public Logger getLogger() {
    ...
    logger = result = Logger.getLogger(name);
    ...
}
```

Log4j包中Logger.getLogger方法的实现

```
static public Logger getLogger(String name) {
    return LogManager.getLogger(name);
}
```

注：LogManager会读取log4j.xml作为配置文件，初始化日志控件后，返回Logger对象。