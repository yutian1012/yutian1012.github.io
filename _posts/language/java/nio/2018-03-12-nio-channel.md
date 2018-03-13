---
title: nio通道
tags: [java]
---

参考：http://www.importnew.com/22623.html

1）介绍

通道是一种途径，借助该途径，可以用最小的总开销来访问操作系统本身的IO服务。缓冲区则是通道内部用来发送和接收数据的端点。通道channel充当连接IO服务的导管。

通道Channel用于在字节缓冲区和位于通道另一侧的实体（通常是一个文件或套接字）之间有效地传输数据。

![](/images/java_basic/nio/channel/channel.png)

2）通道单向或双向传递数据

通道可以是单向或者双向的。ReadableByteChannel和WritableByteChannel接口。

一个channel类可能实现定义read方法的ReadableByteChannel接口，而另一个channel类也许实现WritableByteChannel接口以提供write方法。实现这两种接口其中之一的类都是单向的，只能在一个方向上传输数据。如果一个类同时实现这两个接口，那么它是双向的，可以双向传输数据。

每一个file或socket通道都实现了这两个接口。从类定义的角度而言，这意味着全部file和socket通道对象都是双向的。

3）通道阻塞和非阻塞模式

非阻塞模式的通道永远不会让调用的线程休眠。请求的操作要么立即完成，要么返回一个结果表明未进行任何操作。只有面向流的（stream-oriented）的通道，如sockets和pipes才能使用非阻塞模式。

4）通道会受I/O服务的特征限制

通道会连接一个特定I/O服务且通道实例（channel instance）的性能受它所连接的I/O服务的特征限制，记住这很重要。

一个连接到只读文件的Channel实例不能进行写操作，即使该实例所属的类可能有write方法。基于此，程序员需要知道通道是如何打开的，避免试图尝试一个底层IO服务不允许的操作。

File和socket通道都实现了ReadableByteChannel和WritableByteChannel接口，因此这两个通道都具有读和写的特性。这对于sockets不是问题，因为它们一直都是双向的。不过对于files却是个问题了。一个文件可以在不同的时候以不同的权限打开。从FileInputStream对象的getChannel方法获取的FileChannel对象是只读的，不过从接口声明的角度来看却是双向的，因为FileChannel实现ByteChannel接口。在这样一个通道上调用write方法将抛出未经检查的NonWritableChannelException异常，因为FileInputStream对象总是以read-only的权限打开文件。

*注：使用通道时要注意通道连接的IO服务对象是否支持写入，防止发生异常。*