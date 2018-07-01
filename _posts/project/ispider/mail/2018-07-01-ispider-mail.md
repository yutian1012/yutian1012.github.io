---
title: 监控报警系统
tags: [ispider]
---

用于监控爬虫节点是否宕机，通过监控信息发送邮件提醒任务。监控报警系统是一个独立的进程，需要单独启动。

1）原理

监控报警系统的开发主要依赖于zookeeper实现，监控程序对zookeeper下面的这个节点目录进行监听。

2）zookeeper相关API

使用第三方封装的API，即curator，来进行zookeeper客户端程序的开发。

3）爬虫系统zookeeper注册

ISpider类的registerZK方法完成向zookeeper中注册分布式爬虫节点。

4）监控系统运行

SpiderMonitorTask类运行监听，该类实现了Watcher接口，用于监控zookeeper目录的变化。

通过监听节点的变化，从而调用MailUtil来进行邮件的发送。

注：启动监控只需要运行该类即可。