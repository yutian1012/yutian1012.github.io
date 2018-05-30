---
title: spring cloud介绍
tags: [spring]
---

*这里spring cloud笔记主要是学习尚硅谷视频的总结。*

测试实例地址：https://github.com/yutian1012/springclouddemo.git

springcloud是分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶。

![](/images/spring/springcloud/base/springcloud-arch.png)

1）服务维度

分布式系统的微服务架构从服务的维度可分为：服务治理，服务注册，服务调用，服务负载，服务监控等。每个维度都有相应的解决方案和框架，而springcloud都提供了一套相应维度的解决方案，因此，它是多个框架解决方案的集合体。这些组件除了netFlix的开源组件做高度抽象封装外，还有一些选项中立的开源组件。

2）springcloud与springboot的关系

springboot关注的是微观，一个个微服务的实现。而springcloud关注的是宏观，整合多个springboot微服务，实现对外提供服务。

springcloud依赖springboot，而springboot可作为单独的服务被部署。即springboot可以离开springcloud独立使用开发项目，但springcloud离不开springboot，属于依赖关系。

a.springboot实现微服务(微观)

springcloud利用springboot的开发便利性巧妙地简化了分布式系统基础设施的开发。springboot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟，经得起实际考验的服务框架组合起来。通过springboot风格进行再封装。屏蔽掉了复杂的配置和实现原理，最终给开发者提供了一套简单易懂，易部署和易维护的分布式系统开发工具包。

b.springcloud协调微服务（宏观）

springcloud是关注全局的微服务协调治理框架，它将springboot开发的一个个单体微服务整合并管理起来。为各个微服务之间提供配置管理，服务发现，断路器，路由，微代理，事件总线，全局锁，决策竞选，分布式会话等等集成服务。

3）相关参考文献

官网：http://projects.spring.io/spring-cloud/

中文网：https://springcloud.cc/spring-cloud-dalston.html

社区：http://springcloud.cn