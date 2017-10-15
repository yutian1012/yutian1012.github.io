---
title: log4j中LoggerFactory接口
tags: [log4j]
---

LoggerFactory接口只提供了一个方法

```
public Logger makeNewLoggerInstance(String name);
```

实现类DefaultCategoryFactory

```
public Logger makeNewLoggerInstance(String name) {
    return new Logger(name);
} 
```

注：该类只是new出Logger对象，并返回，不做任何处理。对Logger进一步的处理是交由Hierarchy类来处理的，如设置父Logger，放置到缓存集合中等。