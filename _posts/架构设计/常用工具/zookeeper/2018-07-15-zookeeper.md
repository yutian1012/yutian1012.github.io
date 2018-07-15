---
title: zookeeper介绍
tags: [zookeeper]
---

参考：https://www.cnblogs.com/felixzh/p/5869212.html

参考：https://blog.csdn.net/u010392705/article/details/69663626

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务。zookeeper是分布式协调服务，用于解决分布式数据的一致性问题。

1）zookeeper提供了什么？

文件系统和通知机制

2）使用zookeeper实现什么样的功能？

命名服务；配置管理；集群管理；分布式锁；队列管理

3）zookeeper分客户端和服务端

在集群中zookeeper的主和从都是服务端，客户端是连接zookeeper的微服务等应用。

4）数据模型

zookeeper的数据模型是一棵树，树的节点就是znode（包括持久节点和临时节点），znode可以保存信息。大多数的zookeeper开发就是和数据节点打交道，对数据节点的增删改查。