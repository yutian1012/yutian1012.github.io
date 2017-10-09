---
title: log4j日志additivity特性
tags: [log4j]
---

### 理解子logger和父logger

在定义logger时，log4j.rootLogger是所有logger的父logger。

1）下面定义两个新的logger

```
log4j.logger.com.test = info, stdout
log4j.logger.com.test.a = info, stdout
```

从定义上可以看出：第一个logger值为com.test，第二个logger值为com.test.a，从代码的包组织信息中，com.test.a是com.test的子文件夹，因此定义出的第二个logger就是第一个logger的子logger。而这两个logger都是rootLogger的子logger。

2）理解additivity（附加）

如果在com.test.a包下有一个类，调用logger.info方法输出日志信息，并且配置文件中该logger的level也是info，因此，会打印出info信息。同时会将log信息向其父logger传递，如果父logger的level大于等于info，则也会打印。即additivity决定是否将日志信息向父logger传递。

```
package com.test.a;

import org.apache.log4j.Logger;

public class App 
{
    private static Logger logger=Logger.getLogger(App.class);
    public static void main( String[] args )
    {
        if(logger.isInfoEnabled()) {
            logger.info("info");
        }
    }
}
```

注：日志会打印出2遍

```
public class App 
{
    private static Logger logger=Logger.getLogger("com.test");
    public static void main( String[] args )
    {
        if(logger.isInfoEnabled()) {
            logger.info("info");
        }
    }
}
```

注：此时会只会输出1遍，注意这里调用getLogger传递的字符串

### additivity属性设置

additivity设置为false表示不向父logger传递日志信息。

1）属性文件配置方式

log4j.properties文件

```
log4j.logger.com.test = info, stdout
log4j.logger.com.test.a = info, stdout
结果会导致test.a下的所有日志会在stdout输出两次。即会向父日志com.test中也输出
```

屏蔽方法

```
log4j.additivity.com.test.a=false
```

2）xml配置方式

log4j.xml文件

```
<logger name="com.test">
    <level value = "debug" />
    <appender-ref ref="appender-3" />
</logger>
<logger name="com.test.a" additivity="false">
    <!--指定类的日志级别，会影响指定类日志发出信息的成功与否-->
    <level value = "debug" />
    <appender-ref ref="appender-3" />
</logger>
```