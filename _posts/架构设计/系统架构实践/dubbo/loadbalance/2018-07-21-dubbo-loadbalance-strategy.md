---
title: 使用dubbo搭建微服务架构-负载均衡策略
tags: [architecture]
---

负载均衡策略

1）random loadBalance 随机

按权重设置随机概率，默认策略。在一个截面上的碰撞率高，但调用量越大分布越均匀。

2）RoundRobin LoadBalance 轮询

按公约后的权重设置轮询比率。存在慢的提供者累积请求的问题。

3）LeastActive LoadBalance 最少活跃数

相同活跃数的随机，活跃数指调用前后计数差（差值为处理时间）。使慢的提供者收到更少的请求，因为越慢的提供者的调用前后计数差会越大。

4）ConsistentHash LoadBalance 一致性Hash

相同参数的请求总发给同一个提供者，当某一台提供者崩溃时，原本发往该提供者的请求，基于虚拟节点被平摊给其他提供者，不会引起剧烈变动。