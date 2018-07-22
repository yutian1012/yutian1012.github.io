---
title: dubbo注册中心zookeeper
tags: [architecture]
---

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务。zookeeper是分布式协调服务，用于解决分布式数据的一致性问题。

1）安装并启动zookeeper

从官网下载zookeeper的tar.gz压缩包，解压即可使用

注：需要jdk环境的支持

启动zookeeper服务，双击安装目录下/bin/zkServer.cmd文件启动服务

使用客户端工具连接，安装目录下/bin/zkCli.cmd

```
# 运行
zkCli.cmd  -timeout 5000  -r  -server  localhost:2181

# 退出节点
quit
```

2）dubbo配置zookeeper

这里以spring-boot方式配置zookeeper，在application.yml中配置信息

```
spring:
 dubbo:
  application:
   name: consumer
  registry:
   address: zookeeper://127.0.0.1:2181
  scan: org.sjq.dubbo.client
```

注：指定application的应用名称，address指定zookeeper地址，scan指定扫描出来的包。会对@Service和@Reference注解的实体使用相应的dubbo处理。