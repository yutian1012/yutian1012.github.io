---
title: activeMQ的持久化
tags: [java_middleware]
---

参考：http://blog.csdn.net/terrymanu/article/details/37566783

消息持久性的原理很简单，就是在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件、数据库等，然后试图将消息发送给接收者，发送成功则将消息从存储中删除，失败则继续尝试。消息中心启动以后首先要检查指定的存储位置，如果有未发送成功的消息，则需要把消息发送出去。

ActiveMQ持久化方式：AMQ、KahaDB、JDBC、LevelDB。

配置持久化的方式，都是修改%ACTIVEMQ_HOME%conf/acticvemq.xml文件。

