---
title: log4j日志过滤器
tags: [log4j]
---

### 实例：设置一个按照level设置过滤器

1）LevelRangeFilter过滤器

LevelRangeFilter类，是log4j的jar包中提供的过滤器。这里会过滤输出的日志等级信息，如果等级在指定的LevelMin和LevelMax之间，则正常输出，否则不能输出。

```
<appender name="myConsole" class="org.apache.log4j.ConsoleAppender">       
    <layout class="org.apache.log4j.PatternLayout">       
        <param name="ConversionPattern"          
            value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />       
    </layout>  
    <filter class="org.apache.log4j.varia.LevelRangeFilter">
        <param name="LevelMin" value="WARN" />
        <param name="LevelMax" value="ERROR" />
    </filter>
</appender>

<root>
    <level value="debug"></level>       
    <appender-ref ref="myConsole"/>       
</root> 
```

注：filter功能只支持xml方式配置，不支持属性文件的方式配置。

2）测试代码

```
logger.error("ERROR");//过滤器处理后，能正常输出
logger.debug("debug");//过滤器处理后，不能输出debug日志信息
```

注：在代码中输出错误信息，由于error方法的输出级别为ERROR，因此经过滤器处理时，发现level处于LevelMin和LevelMax之间，因此会正常打印出日志信息。

### 源码分析

```
public class LevelRangeFilter extends Filter {
    Level levelMin;
    Level levelMax;
    ...
    //属性相关的set/get方法
    public int decide(LoggingEvent event) {
        //判断是否小于levelMin，小于则deny
        if(this.levelMin != null) {
          if (event.getLevel().isGreaterOrEqual(levelMin) == false) {
            return Filter.DENY;//-1
          }
        }
        //判断是否大于levelMax，大于则deny
        if(this.levelMax != null) {
          if (event.getLevel().toInt() > levelMax.toInt()) {
            return Filter.DENY;//-1
          }
        }

        if (acceptOnMatch) {
          return Filter.ACCEPT;//1，不会在判断后续的过滤器了
        }
        else {
          return Filter.NEUTRAL;//0，通过后续过滤器来决定
        }
  }
}
```

通过分析源代码可以发现，decide方法最终决定日志是否会被输出。