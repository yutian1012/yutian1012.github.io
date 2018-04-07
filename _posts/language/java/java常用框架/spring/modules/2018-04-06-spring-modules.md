---
title: spring模块
tags: [spring]
---

参考：http://www.cnblogs.com/leehongee/archive/2012/10/01/2709541.html

参考：https://blog.csdn.net/woshisap/article/details/7105255

1）spring的常用jar文件

spring框架由20个不同的模块组成，可以通过查看源码的组织目录

![](/images/spring/modules/spring-jar.png)

2）模块分类（6大部分）

![](/images/spring/modules/spring-modules.png)

a.spring核心容器

核心容器管理着Spring应用中bean的创建、配置和管理（DI）。

有多种Spring应用上下文的实现，每一种都提供了配置Spring的不同方式，除了bean工厂和应用上下文。

提供了许多企业服务（context的扩展模块），例如E-mail、JNDI访问、EJB集成和调度。

SPEL表达式语言的支持。

注：所有的Spring模块都构建于核心容器之上。当你配置应用时，其实你隐式地使用了这些类。

b.Spring的AOP模块

在AOP模块中，Spring对面向切面编程提供了丰富的支持。与DI一样，AOP可以帮助应用对象解耦。借助于AOP，可以将遍布系统的关注点（例如事务和安全）从它们所应用的对象中解耦出来。

c.数据访问与集成

简化jdbc样板代码调用，对数据库操作封装异常错误；

提供了丰富的ORM和事务支持；

JMS抽象封装处理；

java对象与xml间的转换等功能；

d.Web与远程调用

在该模块中，使用mvc将界面逻辑与应用逻辑分离，自身提供了mvc的实现，同时能够与多种MVC框架进行集成；

提供了多种构建与其他应用交互的远程调用方案，如RMI（RemoteMethodInvocation）、Hessian、Burlap、JAX-WS，以及自带的HTTPinvoker。

e.instrumentation（使用/装设仪器）

该模块提供了为JVM添加代理（agent）的功能。具体来讲，它为Tomcat提供了一个织入代理，能够为Tomcat传递类文件，就像这些文件是被类加载器加载的一样。

f.测试

该模块，Spring为使用JNDI、Servlet和Portlet编写单元测试提供了一系列的mock对象实现。对于集成测试，该模块为加载Spring应用上下文中的bean集合以及与Spring上下文中的bean进行交互提供了支持。