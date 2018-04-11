---
title: spring的Environment对象
tags: [spring]
---

参考：https://blog.csdn.net/andy_zhang2007/article/details/78511815?locationNum=5&fps=1

Environment这个接口，是对spring所处的环境的概念性建模。这个接口定义在包 org.springframework.core.env中。

1）该接口对应用程序运行环境的两个关键方面进行了建模 :

a.profile

profile是一组Bean定义(Bean definition)的逻辑分组(logical group)。

profile，被赋予一个命名，就是这个profile的名字。也可以将该名称看着逻辑分组的名称。只有当一个profile处于active状态时，它对应的逻辑上组织在一起的这些Bean定义才会被注册到容器中。

Bean添加到profile可以通过XML定义方式或者annotation注解方式。Environment对于profile所扮演的角色是用来指定哪些profile是当前活跃的/缺省活跃的。

b.property属性

一个应用的属性有很多来源: 属性文件(properties files)，JVM系统属性，系统环境变量，JNDI，servlet上下文参数，临时属性对象等。Environment对于property所扮演的角色是提供给使用者一个方便的服务接口用于配置属性源，或从属性源中获取属性。

注：绝大多数情况下，bean都不需要直接访问Environment对象，而是通过类似@Value注解的方式把属性值注入进来。

2）如何使用Environment对象

容器(ApplicationContext)所管理的bean如果想直接使用Environment对象访问profile状态或者获取属性，可以有两种方式。

a.实现EnvironmentAware 接口

b.@Inject 或者 @Autowired 一个 Environment对象