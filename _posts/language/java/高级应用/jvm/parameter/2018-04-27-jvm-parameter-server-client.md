---
title: jvm运行方式
tags: [jvm]
---

java的运行模式参数：-server 和 -client

两种类型的HotSpot JVM，即客户端和服务器端虚拟机。从JDK5开始，当应用启动时会检测当前的运行环境是否是服务器，如果是服务器就使用Server-JVM这是为了提升性能。一般来说Server-JVM启动比Client-JVM慢，原因是使用的是重量级的虚拟机，但是内部进行了很多优化，而Client-JVM使用的是轻量级的JVM，当服务稳定运行后还是Server-JVM的速度更快一些。

1）server

默认为堆提供了一个更大的空间和并行的垃圾收集器，并且在运行时可以更大程度的优化代码。

2）client

客户端虚拟机有较小的默认堆内存，可以缩短JVM启动的时间和占用更少的内存，客户端的JVM只有在32位操作系统中才有。

3）运行时设置参数

运行test类，该类中提供了一个main方法。

```
# server模式运行
java -server Test

# client模式运行
Java -client Test
```