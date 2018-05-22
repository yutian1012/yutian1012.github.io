---
title: spring data jpa实现原理分析
tags: [spring]
---

当spring data在指定的包路径下查找到继承了Repository接口的类后，会自动创建相应的实现类，同时实现接口中定义的方法实现。通过自动实现方式基本CRUD和基本的业务逻辑操作都得到了解决。

那么具体过程是如何实现的呢？

1）@EnableJpaRepositories注解

首先该功能在配置类上添加了注解@EnableJpaRepositories，表示启动spring data功能。那么该注解一定是一个解析的关键注解。

