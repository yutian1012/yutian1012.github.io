---
title: 使用dubbo搭建微服务架构-负载均衡实现
tags: [architecture]
---

通过将服务提供方部署2个来实现负载均衡。

1）服务提供者的maven项目

```
<artifactId>dubbo-service-demo</artifactId>
```

2）在@Service上配置负载均衡策略

在dubbo-service-demo的helloImpl类的@Service上配置负载均衡策略为roundrobin，即轮询策略。

4）启动测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务dubbo-service-demo提供方法实现注册（运行DubboServiceApplication类）

注：DubboServiceApplication多次运行该类的main方法，即可开启多个服务提供者实例

![](/images/architecture/application/dubbo/dubbo-admin/dubbo-multi-service.png)

启动网关服务

注：使用测试代码（运行InvokeServiceTest类）无法正常看到轮询修改