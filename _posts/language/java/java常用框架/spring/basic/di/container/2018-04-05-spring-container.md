---
title: spring容器
tags: [spring]
---

在基于spring的应用中，你的应用对象生存于Spring容器中。Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期。

通过bean的相关配置（如xml方式配置），spring可以知道如何创建，组装这些对象。

1）容器是Spring的核心

Spring使用DI管理构成应用的组件，它会创建相互协作的组件之间的关联，这会使这些组件对象简单干净，更易于理解，更易于重用并且更易于进行单元测试。

2）容器分类

a.Bean工厂

Bean工厂（如BeanFactory）是最简单的容器，提供最基本的DI支持

b.应用上下文

应用上下文（ApplicationContext接口），基于BeanFactory构建，并提供应用框架级别的服务，如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

注：ApplicationContext比较复杂，它不但继承了BeanFactory的大部分属性，还继承其它可扩展接口，扩展的了许多高级的属性

3）应用上下文

a.AnnotationConfigApplicationContext类

从一个或多个基于Java的配置类中加载Spring应用上下文。

b.AnnotationConfigWebApplicationContext类

从一个或多个基于Java的配置类中加载Spring web应用上下文

c.ClassPathXmlApplicationContext类

从类路径下的一个或多个XML配置文件中加载上下文定义，把上下文的定义文件作为类资源。

d.FileSystemXmlApplicationContext类

从文件系统下的一个或多个XML配置文件中加载上下文定义。

e.XmlWebApplicationContext类

从Web应用下的一个或多个XML配置文件中加载上下文定义。

注：多个XML配置文件需要结合通配符（如星号*）指定配置文件。