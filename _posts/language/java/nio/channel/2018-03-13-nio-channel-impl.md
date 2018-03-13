---
title: nio通道常用实现类
tags: [java]
---

NIO中的一些主要Channel的实现类，即实现了java.nio.channels包的Channel接口类，这些通道实现类涵盖了UDP和TCP网络IO，以及文件IO。

1）文件通道FileChannel

实现从文件中读写数据，文件通道总是阻塞式的，因此不能被置于非阻塞模式。即文件通道不能与selector结合使用。

2）数据包通道DatagramChannel

能通过UDP读写网络中的数据，DataGram在网络中是指数据报，而数据报一般是用来封装UDP报文数据的格式数据。

3）Socket通道SocketChannel

能通过TCP读写网络中的数据，java的Socket底层依赖TCP协议实现网络数据传输的。

4）Server端的Socket通道ServerSocketChannel

可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

