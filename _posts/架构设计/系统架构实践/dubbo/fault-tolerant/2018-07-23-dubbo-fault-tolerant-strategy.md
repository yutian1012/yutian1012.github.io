---
title: 使用dubbo搭建微服务架构-集群容错策略
tags: [architecture]
---

1）failover：

失败自动切换，当出现失败时，重试其他服务器（默认）。可以通过在服务提供方的@Service或消费方的@Reference注解中设置retries参数来指定重试次数（不含第一次）

注：通常用于读操作，但重试会带来更长的延迟

2）failfast：

快速失败，只发起一次调用，失败立即报错。

注：通常用于非幂等性的写操作，比如新增记录。

3）failsafe：

失败安全，出现异常时，直接忽略。

注：通常用于写入审计日志等操作

4）failback：

失败自动恢复，后台记录失败请求，定时重发

注：通常用于消息通知操作。

5）forking：

并行调用多个服务器，只要一个成功即返回

注：通常用于实时性要求比较高的读操作，但需要浪费更多服务资源

6）broadcast：

广播调用所有提供者，逐个调用，任意一台报错则报错。

注：通常用于通知所有提供者更新缓存或日志等本地资源信息。