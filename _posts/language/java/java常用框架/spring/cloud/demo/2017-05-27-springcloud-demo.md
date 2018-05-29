---
title: spring cloud案例
tags: [spring]
---

springboot+springmvc+mybatis实现微服务，以Dept部门模块做一个微服务通用案例。consumer消费者（client）通过restful调用Provider提供者（Server）提供的服务。

1）项目组织

springclouddemo(整体父工程)

microservicecloud-api(封装的整体entity/接口/公共配置等)

microservicecloud-provider-dept-8001(微服务落地的服务提供者)

microservicecloud-consumer-dept-80（微服务调用的客户端）