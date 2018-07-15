---
title: zookeeper客户端连接
tags: [zookeeper]
---

1）zookeeper客户端工具

位于安装路径下的bin目录中提供了相应的工具类zkCli.cmd，使用该工具可以直接连接，并用命令行的形式与服务进行交互

2）程序引入jar包进行连接

```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.12</version>
    <type>pom</type>
</dependency>
```