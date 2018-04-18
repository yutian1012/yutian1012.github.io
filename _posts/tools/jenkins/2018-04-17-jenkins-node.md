---
title: 持续集成jenkins的节点
tags: [jenkins]
---

参考：https://www.cnblogs.com/sparkdev/p/7102622.html

Jenkins是一个可扩展的持续集成引擎。主要用于持续、自动地构建、测试软件项目。

Jenkins中的slave也称为node（节点）。

1）使用场景

参考：https://www.cnblogs.com/derekchen/p/5892286.html

当我们使用多台服务器时，并且配置了tomcat或jboss集群服务，可通过jenkins的节点配置，将jenkins项目发布在不同服务器上（分布jenkins工作空间，部署项目到不同服务器的tomcat或jboss），这就形成了jenkins的分布式。节点服务器不需要安装jenkins（只需要运行一个slave节点服务），构建事件的分发由master端（jenkins主服务）来执行。

2）创建node

系统管理--管理节点