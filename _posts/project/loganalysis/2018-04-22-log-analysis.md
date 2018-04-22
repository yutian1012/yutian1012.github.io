---
title: 日志实时分析系统
tags: [project]
---

日志分析

1）日志

日志数据是结构化数据+时间戳，作用是为了排查问题，信息检索，监控等。

2）传统日志分析思路

通过cat/grep.regex等工具查看日志文件。

注：缺点是手工操作，效率低，易出问题。延时高，无可视化效果。

3）实时日志分析系统

a.实现日志索引、检索、分类、排序，提供可视化界面。

b.实现监控，分析重要日志。

c.实现不同数据规模的横向扩展。

4）设计思路（ELK）

a.Elastic Search存储、索引日志

基于lucene全文检索系统的存储引擎，作为日志的存放组件。

b.logstash收集应用日志

理解为收集器，日志信息无法直接存储到ElasticSearch中，这需要通过logstash将日志转换成ElasticSearch库的数据结构。

应用程序中可以直接调用相应的jar包（mvn仓库中就存在），将应用日志数据保存到ElasticSearch中。该jar包相当于Logstash的客户端。

c.Kibana提供可视化界面

实现从Elastic search中获取数据进行展示。

5）系统的架构实现

a.ELK组件安装及配置，

b.APP应用程序使用Logback产生日志，

c.kibana可视化界面。

![](/images/project/loganalysis/ELK.png)