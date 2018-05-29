---
title: eureka入门实例
tags: [spring]
---

场景分析：

前面我们完成了microservicecloud-provider-dept-8001和microservicecloud-consumer-dept-80两个模块的实战。服务提供者通过RestController对外提供服务，而服务消费者则使用RestTemplate来访问服务提供者提供的服务。

下面我们继续对其进行改造，使用Eureka的服务注册与发现机制，完成服务消费者与服务提供者间的服务调用。

1）实现步骤

Eureka-Server提供服务注册和发现

Service-Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到

Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务。

2）eureka server的使用

首先需要在pom中引入eureka-server的依赖，然后在主启动类上使用注解@EnableEurekaServer来完成eureka-server的使用。