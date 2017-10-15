---
title: log4j中LoggerRepository接口
tags: [log4j]
---

重点分析Hierarchy类，该类是核心。

参考：http://www.blogjava.net/DLevin/archive/2012/07/10/382678.html

LoggerRepository从字面上理解，它是一个Logger的容器，它会创建并缓存Logger实例，从而具有相同名字的Logger实例不会多次创建，以提高性能。

实现类

![](/images/java_structure/log/log4j/LoggerRepository.png)

1）实现类NOPLoggerRepository

LoggerRepository接口下存在Hierarchy和NOPLoggerRepository两个实现类。其中NOPLoggerRepository是为了防止类重新加载时LogManager.repositorySelector出现空指针异常而引入的。而该类中所有的方法基本都是空实现。

LogManager类的getLoggerRepository方法，该方法判断，如果repositorySelector为null时，则构造出一个NOPLoggerRepository做为Logger的容器，并从该容器中获取Logger实例。

```
static public LoggerRepository getLoggerRepository() {
    if (repositorySelector == null) {
        repositorySelector = new DefaultRepositorySelector(new NOPLoggerRepository());
    ...
```

NOPLoggerRepository类的getLogger方法，返回NOPLogger对象，该对象中日志的记录方法都是空方法，即不进行任何日志的记录操作。

```
public Logger getLogger(final String name) {
    return new NOPLogger(this, name);
}
```

注：NOP表示No-operation，即不进行任何处理操作。

2）Hierarchy实现类（实际中使用该类）

Hierarchy中用一个Hashtable来存储所有Logger实例，它以CategoryKey作为key，Logger作为value，其中CategoryKey是对Logger中Name字符串的封装，有利于加快hashmap的查找效率。这里把CategoryKey当做字符串key即可。

```
private Vector listeners;//注册HierarchyEventListener监听
Hashtable ht;//存放Logger实例的集合
Logger root;//根日志对象
```

注：ht用来存放Logger，调用getLogger时，name做为key，生成的Logger对象做为value，放入集合中。