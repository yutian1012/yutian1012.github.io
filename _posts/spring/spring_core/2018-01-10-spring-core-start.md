---
title: spring容器的启动过程
tags: [spring core]
---

1）容器的启动入口

通过ClassPathXmlApplicationContext对象的创建来查看spring容器的启动过程。

```
new ClassPathXmlApplicationContext("applicationContext.xml");
```

ClassPathXmlApplicationContext类的继承体系

![](/images/spring/core/ClassPathXmlApplicationContext-hierarchy.png)

2）AbstractApplicationContext类的refresh方法

该方法定义了spring容器的启动步骤