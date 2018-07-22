---
title: 使用dubbo搭建微服务架构-服务消费者@Reference注解属性
tags: [architecture]
---

Dubbo的@Reference注解可以用于生成远程服务代理

1）服务相关的配置

version，服务版本，与服务提供者的版本一致

group，服务分组，当接口有多个实现时，可以用分组区分，必须和服务提供方一致

cache，以调用参数为key，缓存返回结果，可选值为：lru，threadlocal，jcache等

2）连接相关的配置

timeout，服务方法调用的超时时间（毫秒）

retries，远程服务调用重试次数，不包括第一次调用，不需要重试设为0

connections，设置提供者的最大连接数

loadbalance，负载均衡策略，可选值为random（随机），roundrobin（轮询），leastactive（最少活跃调用。

async，是否异步执行，不可靠，只是忽略返回值，不阻塞执行线程。

3）初始化相关的配置

init，是否等到有服务注入或引用该实例时再初始化

4）协议相关的配置

protocol，只调用指定协议的服务提供方法，其他协议忽略。