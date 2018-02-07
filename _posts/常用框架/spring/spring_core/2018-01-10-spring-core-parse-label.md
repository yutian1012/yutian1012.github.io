---
title: spring对xml标签的解析
tags: [spring core]
---

1）标签解析的入口

通过查看spring容器的加载过程来找到标签的解析入口

```
ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
```