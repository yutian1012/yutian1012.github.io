---
title: Kylin, Mondrian, Saiku系统的整合
tags: [BI]
---

参考：http://blog.csdn.net/gsying1474/article/details/51655532

### 介绍

1）Apache Kylin

基于Hadoop的OLAP引擎，它的技术特点是能够充分利用Hadoop集群的MapReduce/Spark技术进行分布式的数据立方体预计算，并提供强大的高并发支持能力，能够响应标准SQL的访问请求。Kylin是MOLAP引擎，也就是说除了包括多维立方体模型，还包括多维立方体的预计算的实际数据。

2）saiku

Saiku是一款开源BI前端工具。

3）Mondrian

Mondrian是其内部使用的一个开源ROLAP引擎，ROLAP的元数据仅仅包括了多维立方体模型，而不包括实际的立方体数据，因此适合小规模数据。

4）大数据的处理

当数据量达到亿级，为了能够提供高性能的查询，Saiku可以直接连接Kylin的数据源，发送标准SQL查询语句，然后Kylin会从数据立方体中直接找到对应的结果返回给前端，速度能达到秒级或者亚秒级，非常适合大数据应用。