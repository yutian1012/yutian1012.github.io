---
title: log4j中LogManager
tags: [log4j]
---

LogManager是研究log4j日志的入口类。

1）使用log4j进行日志操作

```
import org.apache.log4j.Logger;
...
private static Logger logger=Logger.getLogger(Test.class);
```

注：获取Logger的方式是通过Logger.getLogger方法实现的，内部其实是调用LogManager.getLogger(name)方法实现的。因此LogManager是研究源码运行的入口类。

2）LogManager类的getLogger方法

```
// Delegate the actual manufacturing of the logger to the logger repository.
public static Logger getLogger(final String name) {
    return getLoggerRepository().getLogger(name);
  }
```

实际是委托给LoggerRepository来获取Logger对象，LoggerRepository是一个Logger的容器，它会创建并缓存Logger实例。