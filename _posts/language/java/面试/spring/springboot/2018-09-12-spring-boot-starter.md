---
title: spring boot starter的原理
tags: [interview]
---

参考：https://blog.csdn.net/flygoa/article/details/68484439

SpringBoot最大的特点就是提供了很多默认的配置，Spring4.x提供了基于条件来配置Bean的能力，SpringBoot就是通过这一原理来实现的。 

注：条件配置的使用@Conditional

1）常见的条件注解

```
@ConditionalOnClass(HelloService.class)//条件注解，当类路径下有指定的类的条件下

@ConditionalOnProperty
...

```

2）注册配置

在src/main/resource下新建META-INFO/spring.factories文件。

注：该文件可在spring-boot-1.5.1.RELEASE\spring-boot-autoconfigure\src\main\resources\META-INF下找到。 

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration
```