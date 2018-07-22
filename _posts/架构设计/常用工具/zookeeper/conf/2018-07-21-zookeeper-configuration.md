---
title: zookeeper常用配置
tags: [zookeeper]
---

1）zookeeper的配置文件

zookeeper配置文件位于解压目录下的/conf目录中，文件名为zoo.cfg

2）常用配置

```

# 服务器之间或客户端与服务器之间维持心跳的时间间隔
tickTime=2000

# 配置在集群中与其他zookeeper的连接最大心跳时间间隔数
initLimit=10

# 标识leader和follower之间发送消息时请求和应答的时间长度，规定了在此期间最长不能超过多少个心跳数
syncLimit=5

# 保存数据的目录
dataDir=E:/work/installer/zookeeper-3.4.12/data

# 客户端连接服务器的端口，zookeeper会监听这个端口，接受客户端的访问请求
clientPort=2181

```

3）集群配置

集群配置格式为：server.A:B:C:D

注：A表示服务器编号，B表示该服务器的IP地址，C和D是两个TCP端口号，分别用于仲裁和leader选举。

```
server.1:192.168.1.22:2222:2223
server.2:192.168.1.23:2222:2223
```

注：示例表示当前的zookeeper与ip为22和23的zookeeper组成集群。