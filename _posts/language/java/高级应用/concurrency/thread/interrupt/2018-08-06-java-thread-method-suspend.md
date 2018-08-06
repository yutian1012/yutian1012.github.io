---
title: java中线程的suspend方法
tags: [concurrency]
---

参考：https://blog.csdn.net/xingjiarong/article/details/47984659

suspend被弃用的原因是因为它会造成死锁。suspend方法和stop方法不一样，它不会破换对象和强制释放锁，相反它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。

suspend方法是将线程阻塞挂起，而不能控制线程获取锁的释放操作。