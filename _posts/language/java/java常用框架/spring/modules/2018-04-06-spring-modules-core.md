---
title: spring核心模块
tags: [spring]
---

spring核心容器

容器是spring框架最核心的部分，包括的jar文件有：Beans，Core，Context，Expression，Context Support等jar包。

它管理着Spring应用中bean的创建、配置和管理。在该模块中，包括了Springbean工厂，它为Spring提供了DI的功能。基于bean工厂，我们还会发现有多种Spring应用上下文的实现，每一种都提供了配置Spring的不同方式。除了bean工厂和应用上下文，该模块也提供了许多企业服务（context的扩展模块），例如E-mail、JNDI访问、EJB集成和调度。所有的Spring模块都构建于核心容器之上。当你配置应用时，其实你隐式地使用了这些类。

1）spring-core.jar（核心工具包）

这个jar文件包含Spring框架基本的核心工具类。Spring其它组件要都要使用到这个包里的类，是其它组件的基本核心，当然你也可以在自己的应用系统中使用这些工具类。

外部依赖cglib等（可查看源码的spring-core.gradle文件）。

2）spring-beans.jar（依赖注入）

这个jar文件是所有应用都要用到的，它包含访问配置文件、创建和管理bean以及进行Inversion of Control/Dependency Injection（即IoC/DI）操作相关的所有类。

如果应用只需基本的IoC/DI 支持，引入spring-core.jar 及spring-beans.jar 文件就可以了。

外部依赖spring-core

3）spring-context.jar（spring容器相关）

这个jar文件为Spring核心提供了大量扩展。可以找到使用Spring ApplicationContext特性时所需的全部类，JDNI所需的全部类，instrumentation组件以及校验Validation方面的相关类。

外部依赖spring-beans，spring-aop，spring-core，spring-expression等

4）spring-context-support.jar（容器的扩展支持）

这个jar文件包含支持UI模版（Velocity，FreeMarker，JasperReports），邮件服务，脚本服务(JRuby)，缓存Cache（EHCache），任务计划Scheduling（uartz）方面的类。

外部依赖spring-context, （针对相关的服务，需要依赖的jar包也不同spring-jdbc, Velocity, FreeMarker, JasperReports, BSH, Groovy, JRuby, Quartz, EHCache）。

5）spring-Expression.jar

提供Spring表达式语言，能够以一种强大和简介的方式将值装配到bean属性和构造器参数中。支持使用bean的ID来引用bean；调用方法和访问对象的属性；对值进行算数、关系和逻辑运算；正则表达式匹配；集合操作。

依赖spring-core。