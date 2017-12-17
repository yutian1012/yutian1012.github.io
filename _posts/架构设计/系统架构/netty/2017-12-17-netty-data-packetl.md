---
title: Netty 自定义数据包协议
tags: [architecture]
---

### 协议包的错误现象

1）粘包现象

客户端发送了两条命令，而服务器端收到的确是两个命令组合起来的一条命令。

```
//客户端发送两条命令
give me a coffee
give me a tea
//服务器端收到一条命令，造成命令不可解读
give me a coffeegive me a tea
```

2）分包现象

```
//客户端发送两条命令
give me a coffee
give me a tea
//服务器端收到两条命令，但不能正常解析
give me
 a coffeegive me a tea
```

注：粘包和分包出现的原因，是因为客户端和服务器端没有一个稳定的数据结构。

3）解决粘包和分包策略

采用分割符的方式，在命令后添加分割符，如以分号分隔

```
//客户端发送两条命令
give me a coffee;
give me a tea;
```

采用长度+数据的方式，在数据之前跟上规定位的字节表示命令长度

```
//客户端发送两条命令
16give me a coffee
13give me a tea
```

注：长度字段的位数是固定的。

### 自定义数据包结构

将数据包的格式定义为：包头（4字节）+模块号（2字节）+命令号（2字节）+长度（4字节，描述数据部分字节长度）+数据。

