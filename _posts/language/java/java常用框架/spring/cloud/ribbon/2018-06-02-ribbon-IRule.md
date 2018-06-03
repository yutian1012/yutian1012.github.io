---
title: spring cloud ribbon核心组件
tags: [spring]
---

1）IRule接口

IRule是Ribbon的负载均衡算法接口，能够根据特定的算法从服务列表中选取一个要访问的服务。

2）常见的负载均衡算法

a.RoundRobinRule 轮询

b.RandomRule 随机

c.AvailabilityFilteringRule：

过滤掉那些因为一直连接失败的被标记为circuit-tripped的后端server，并过滤掉那些高并发的的后端server（active connections 超过配置的阈值）。使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就是检查status里记录的各个server的运行状态.

d.WeightedResponseTimeRule

根据响应时间分配一个weight(权重)，响应时间越长，weight越小，被选中的可能性越低。

e.RetryRule

对选定的负载均衡策略机上重试机制，在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server.

f.BestAvaliableRule

选择一个最小的并发请求的server，逐个考察Server，如果Server被tripped了，则忽略。

g.ZoneAvoidanceRule

复合判断server所在区域的性能和server的可用性选择server.