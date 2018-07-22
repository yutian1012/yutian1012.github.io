---
title: 使用dubbo搭建微服务架构-负载均衡实现方式
tags: [architecture]
---

实现方式（4种）

1）在服务提供方配置

通过注解@Service的loadBalance属性可配置负载均衡策略

2）在调用端配置

通过注解@Reference的loadBalance属性可配置负载均衡策略

3）使用dubbo-admin图像化界面配置

dubbo-admin配置负载均衡位于服务治理目录下的负载均衡页面。

新增配置并提供服务名称，方法名称，负载均衡策略，保存后便可完成配置

![](/images/architecture/application/dubbo/dubbo-admin/dubbo-loadBalance.png)

4）通过自定义策略实现

通过实现com.alibaba.dubbo.rpc.cluster.LoadBalance接口扩展自定义策略。

5）配置策略的优先级

当同时配置了不同的策略时，dubbo-admin > @Reference > @Service