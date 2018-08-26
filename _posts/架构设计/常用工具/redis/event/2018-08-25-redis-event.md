---
title: redis的事件模型
tags: [redis]
---

参考：Redis 4.x Cookbook 中文版 

Redis以其高性能而闻名，它最大程度地利用了单线程、非阻塞的I/O模型来快速地处理请求。

下面通过源码的方式来分析redis的事件模型机制。

1）c代码的集成开发环境

使用eclipse c++作为c代码的集成开发环境。

2）异步事件库

Redis包含了一个简单但功能强大的异步事件库，称为ae。该库中封装了不同操作系统的polling机制（即非阻塞I/O相关的机制），如epoll、kqueue和select等。

对polling机制的理解（以Linux中的epoll为例）

首先，我们调用epoll_create通知操作系统内核我们要使用epoll。

然后，调用epoll_ctl把文件描述符（FD）和所关注的事件类型传给内核。

之后，调用epoll_wait等待通过epoll_ctl所设置的文件描述符上发生所关注的事件。

当文件描述符被更新时，内核会向应用程序发送通知。我们唯一需要做的就是为我们所这些关注的这些事件创建事件处理函数/回调函数。