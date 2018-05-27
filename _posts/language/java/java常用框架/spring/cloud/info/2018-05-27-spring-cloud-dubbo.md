---
title: spring cloud与dubbo的比较
tags: [spring]
---

目前成熟的互联网架构=分布式+服务器治理Dubbo

*dubbo的定位始终是一款RPC框架，而springcloud的目标是微服务架构下的一站式解决方案。*

1）springcloud与dubbo的对比

a.稳定性方面

springcloud与dubbo对比相当于品牌机与组装机的区别。很明显，springcloud的功能比dubbo更强大，涵盖面更广。使用dubbo构建的微服务架构就像组装电脑，各环节的选择自由度很高，但最终结果很可能因为内存质量不行造成系统不稳定。而springcloud就像品牌机，各个组件做了大量的兼容性测试，保证了机器拥有更高的稳定性。

注：dubbo构建的微服务需要对相关的技术有很深的了解，防止出现问题

b.通信的区别

springcloud与dubbo的最大区别在于，springcloud抛弃了dubbo的RPC通信，采用的是基于http的Restful方式进行通信。Restful方式牺牲了性能的依赖。但Restful相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。

c.技术的支持

![](/images/spring/springcloud/base/springcloud-dubbo.png)

d.社区支持和更新力度

springcloud的活跃度远远高于dubbo