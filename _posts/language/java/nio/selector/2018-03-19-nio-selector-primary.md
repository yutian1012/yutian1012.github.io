---
title: nio选择器selector原理
tags: [java]
---

参考：http://zhhphappy.iteye.com/blog/2032893

参考：http://blog.csdn.net/wuxianglong/article/details/6612282

参考：https://www.cnblogs.com/guazi/p/6605221.html

1）理解阻塞IO的处理过程

通常在进行同步I/O操作时，如果读取数据，代码会阻塞直至有可供读取的数据。同样，写入调用将会阻塞直至数据能够写入。

传统的Server/Client模式会基于TPR（Thread per Request），服务器会为每个客户端请求建立一个线程，由该线程单独负责处理一个客户请求。这种模式带来的一个问题就是线程数量的剧增，大量的线程会增大服务器的开销。

大多数的实现为了避免这个问题，都采用了线程池模型，并设置线程池线程的最大数量，这由带来了新的问题，如果线程池中有200个线程，而有200个用户都在进行大文件下载，会导致第201个用户的请求无法及时处理，即便第201个用户只想请求一个几KB大小的页面。

2）理解无阻塞IO的处理过程

NIO中非阻塞IO采用了基于Reactor模式的工作方式，IO调用不会被阻塞，相反是使用通道注册感兴趣的特定I/O事件。如可读数据到达，新的套接字连接等等，在发生特定事件时，系统再通知我们。NIO中实现非阻塞IO的核心对象就是Selector，Selector就是注册各种IO事件地方，而且当那些事件发生时，就是这个Selector对象告诉我们所发生的事件。

![](/images/java_basic/nio/selector/selector-channel.gif)

当有读或写等任何注册的事件发生时，可以从Selector中获得相应的SelectionKey，同时从 SelectionKey中可以找到发生的事件和该事件所发生的具体的SelectableChannel，以获得客户端发送过来的数据。

3）使用过程

使用NIO中非阻塞I/O编写服务器处理程序，大体上可以分为下面三个步骤：

a.向Selector对象注册感兴趣的事件

b.从Selector中获取感兴趣的事件

c.根据不同的事件进行相应的处理