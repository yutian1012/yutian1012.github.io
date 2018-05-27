---
title: 微服务技术栈
tags: [spring]
---

1）前期技术（微服务开发使用的技术）

springMVC+spring/springboot+mybatis+maven+git...

2）相关核心技术（微服务框架搭建的核心技术）

Eureka服务注册与发现；Ribbon负载均衡；Feign负载均衡；Hystrix断路器；zuul路由网关，以及spring cloud config配置中心等。

注：了解spring cloud与dubbo的区别，eureka与zookeeper的区别。

### 不同维度下的相关落地技术

分布式系统的微服务架构从服务的维度可分为：服务治理，服务注册，服务调用，服务负载，服务监控等。

1）服务开发

落地技术：spring boot,spring,spring mvc

2）服务配置与管理

Netflix公司的Archaius，阿里的Diamond等。

3）服务注册与发现

Eureka,Consul,Zookeeper等

4）服务调用

Restful，PRC，gRPC等

5）服务熔断器

Hystrix，Envoy等

6）负载均衡

Ribbon，Nginx等

7）服务接口调用

Feign等

8）消息队列

Kafka，RabbitMQ，ActiveMQ等

9）服务配置中心管理

Spring cloud config，chef等

10）服务路由（API网关）

Zuul等

11）服务监控

Zabbix，Nagios，Metrics，Spectator等

12）全链路追踪

Zipkin，Brave，Dapper等

12）服务部署

Docker，OpenStack，Kubernetes等

13）数据流操作开发包

Spring cloud Stream（封装与Redis，Rabbit，Kafka等发送接收消息）

14）事件消息总线

Spring Cloud Bus