---
title: eureka的自我保护
tags: [spring]
---

1）eureka自我保护

```
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

某时刻某一个微服务不可用力，eureka不会立刻清理，依旧会对给微服务的信息进行保护。

2）原理

默认情况下，如果eureka-server在一定时间内没有接收到某个微服务实例的心跳，eureka-sever将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与eureka-sever之间无法正常通行，以上的行为可能变得非常危险，因为微服务本身其实是健康的，次数本不该注销这个微服务。

eureka通过自我保护模式来解决这个问题，当eureka-sever节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，eureka-server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就不会注销任何微服务）。当网络故障恢复后，eureka-server节点会自动退出自我保护模式。

在自我保护模式中，eureka-server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该eureka-server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。

*一句话讲：好死不如赖活着。*

3）禁用自我保护机制（不推荐）

修改microservicecloud-eureka-7001服务注册中心模块配置文件yml，在eureka:server节点下配置enable-self-preservation属性

```
eureka:
 server: 
  enable-self-preservation: false
```