---
title: 使用dubbo搭建微服务架构-集群容错
tags: [architecture]
---

参考：https://blog.csdn.net/bjo2008cn/article/details/54411051

为了提高系统可用性，将服务提供方法部署成多个，从而组成服务集群。在消费方调用服务失败或超时，dubbo可以配置多种策略重新调用集群中的其他服务提供方。

配置方式

1）注解配置

在服务提供方的@Service或消费方的@Reference注解中配置cluster参数。

2）配置文件配置

或者用application.properties中的spring.dubbo.service.cluster参数来指定容错策略。