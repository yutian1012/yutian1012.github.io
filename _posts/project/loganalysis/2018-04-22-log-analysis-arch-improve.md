---
title: 日志实时分析系统架构优化
tags: [project]
---

使用ELK架构，应用程序需要调用Logstash客户端jar，因此，与应用层产生了耦合。使用kafaka做一个日志的生成和消费中间层。

![](/images/project/loganalysis/ELI-improve.png)

ELK组件与应用层分隔；App应用端日志全部写入Kafka；Logstash负责从Kafaka集群去出数据。

注：实现应用层日志统一规范

1）kafka

kafka是linkedin开源的高性能消息队列组件，并不是基于jms实现的，对jms进行了部分兼容。

2）工作过程

原应用中需要在Logback配置中将日志输出到logstash中，从而进行日志持久化到Elasticsearch中，进而展示数据。

现在logback充当一个日志生产者，将日志信息写入到kafka消息中间件中，而logstash作为消费者，从消息队列中获取，然后写入到elasticsearch中。

3）部署和业务的问题

Logstash是CPU密集型操作，ES属于IO密集型操作；ES集群未对业务系统进行拆分。

解决方法：Logstash与ES单独部署（性能问题）。ES集群拆分。

![](/images/project/loganalysis/ELI-improve-cluster.png)

4）其他方面的问题

a.安全性问题

不能暴露所有日志信息，有些涉及到用户私密的信息，不能直接对外暴露。

b.跨数据中心问题

查询timeout问题

c.日志定期清理

做冷热分区。