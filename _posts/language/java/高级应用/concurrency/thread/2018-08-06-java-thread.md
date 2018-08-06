---
title: java中的线程
tags: [concurrency]
---

线程是进程内的执行单元。java中的线程会直接映射到操作系统的线程上。

1）java中线程状态的切换

![](/images/architecture/concurrency/thread-status.jpg)

2）线程的调用：

Thread的start方法是在当前系统上创建一个线程，并等待cpu的调度执行。

Thread的run方法不能真正开启一个线程，而是在原线程的基础上运行，因此不要使用run方法。

Thread的stop方法，不推荐使用，它会释放所有的monitor（锁）。很可能导致数据不一致的问题。
