---
title: eureka服务注册与发现
tags: [spring]
---

参考：https://www.cnblogs.com/woshimrf/p/springclout-eureka.html

Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的。有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要像传统方式修改服务调用的配置文件了。

注：功能类似与dubbo的注册中心，如Zookeeper。

1）Eureka的架构设计

Eureka采用了C-S的设计架构，Eureka server作为服务注册功能的服务器，是服务注册中心。

Eureka包含两个组件：Eureka server和Eureka Client

2）运行原理

Eureka-Server提供服务注册服务，微服务架构中的其他微服务使用Eureka的客户端连接到Eureka-server（注册）并维持心跳连接。这样Eureka-Server中的服务注册表中将会存储所有可用服务节点的信息。系统维护人员可以通过Eureka-Server来监控系统中各个微服务是否运行正常，可通过界面直接查看到各服务节点信息。

EurekaClient是一个java客户端，用于简化微服务与Eureka-Sever的交互，客户端同时也具备一个内置的、使用轮询（round-robin）负载算法的负载均衡器。在应用启动后，将会向Eureka-Sever发送心跳（默认周期为30秒）。如果Eureka-Server在多个心跳周期内没有接收到某个节点的心跳，Eureka-Server将会冲服务注册表中将这个服务节点移除。