---
title: nio选择器selector
tags: [java]
---

参考：https://www.2cto.com/kf/201610/556476.html

参考：http://www.importnew.com/22623.html

参考：http://ifeve.com/selectors/

1）Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

注：Selector一般是用于网络连接，如SocketChannel，ServerSocketChannel类的处理，是属于非阻塞模型下运行的通道，而FileChannel不具备非阻塞。

2）为什么使用Selector

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道（Selector线程）。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

传统的Socket变成，服务器端accept一个socket连接，服务器端都会启动相应的一个Socket线程来监听，并服务这个socket，直到客户端的socket主动端口或服务器超时，才会回收socket。回收的socket可以直接销毁也可以放入到一个线程池中，供下一个Socket客户端连接使用。

3）基本使用

首先创建一个Selector；然后将channel注册到Selector上，注册时同时要指定通道注册的事件类型（Connect，Accept，Read，Write），即该通道会触发哪种类型的事件；

注：与Selector一起使用时，Channel必须处于非阻塞模式下，这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注2：理解事件是谁触发的，事件是由通道触发的。通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

注3：事件类型定义在了Selector的常量中，SelectionKey.OP_CONNECT，SelectionKey.OP_ACCEPT，SelectionKey.OP_READ，SelectionKey.OP_WRITE。