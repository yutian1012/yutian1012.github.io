---
title: spring模块instrumentation
tags: [spring]
---

instrumentation中文意思为：使用/装设仪器

该模块提供了为JVM添加代理（agent）的功能。具体来讲，它为Tomcat提供了一个织入代理，能够为Tomcat传递类文件，就像这些文件是被类加载器加载的一样。

该模块包含的jar包：instrument，instrument-tomcat等。

1）spring-instrument.jar

对服务器的代理接口

2）spring-instrument-tomcat.jar

对Tomcat的连接池的集成