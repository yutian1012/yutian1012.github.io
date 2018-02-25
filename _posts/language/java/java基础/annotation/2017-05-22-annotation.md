---
title: java注解
tags: [annotation]
---

参考：http://www.cnblogs.com/skywang12345/p/3344137.html

参考：http://www.cnblogs.com/peida/archive/2013/04/26/3038503.html

JDK5.0中提供了注解的功能，允许开发者定义和使用自己的注解类型。该功能由一个定义注解类型的语法和描述一个注解声明的语法，读取注解的API，一个使用注解修饰的class文件和一个注解处理工具组成。

Annotation并不直接影响代码的语义，但是他可以被看做是程序的工具或者类库。它会反过来对正在运行的程序语义有所影响。

Annotation可以冲源文件、class文件或者在运行时通过反射机制多种方式被读取。

### 注解的作用

Annotation 是一个辅助类，它在Junit、Struts、Spring等工具框架中被广泛使用

1）编译检查

Annotation具有“让编译器进行编译检查的作用”。

例如，@SuppressWarnings, @Deprecated和@Override都具有编译检查作用。

2）在反射中使用Annotation

在反射的Class, Method, Field等函数中，有许多于Annotation相关的接口。

3）根据Annotation生成帮助文档

通过给Annotation注解加上@Documented标签，能使该Annotation标签出现在javadoc中。

4）能够帮忙查看查看代码

通过@Override, @Deprecated等，我们能很方便的了解程序的大致结构

5）应用在aop中

可以通过定义注解，并将注解设置到相应的方法上，那么通过拦截该注解达到执行方法aop的功能。

### 注解的语法

@interface用来声明Annotation，@Documented用来表示该Annotation是否会出现在javadoc中， @Target用来指定Annotation的类型，@Retention用来指定Annotation的策略

a.定义

使用@interface定义注解时，意味着它实现了java.lang.annotation.Annotation接口，即该注解就是一个Annotation。

b.定义注解修饰

可使用@Documented，@Target，@Retention来规范注解的使用范围和作用域。

