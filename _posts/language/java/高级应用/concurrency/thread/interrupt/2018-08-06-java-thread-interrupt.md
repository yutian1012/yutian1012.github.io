---
title: java中的线程中断说明
tags: [concurrency]
---

理解中断，中断是指当前正在运行的线程暂时挂起（先不执行）。等到时机成熟了再执行，或者中断后就直接将线程销毁。

在线程中断过程中，线程占有的锁资源（计算机资源）必须要释放，否则会造成死锁。如果线程占用的锁不会被共享，当然是可以不是释放的。

1）官方文档描述

```
Interrupts this thread.
Unless the current thread is interrupting itself, which is always permitted, the checkAccess method of this thread is invoked, which may cause a SecurityException to be thrown.

If this thread is blocked in an invocation of the wait(), wait(long), or wait(long, int) methods of the Object class, or of the join(), join(long), join(long, int), sleep(long), or sleep(long, int), methods of this class, then its interrupt status will be cleared and it will receive an InterruptedException.

If this thread is blocked in an I/O operation upon an interruptible channel then the channel will be closed, the thread's interrupt status will be set, and the thread will receive a ClosedByInterruptException.

If this thread is blocked in a Selector then the thread's interrupt status will be set and it will return immediately from the selection operation, possibly with a non-zero value, just as if the selector's wakeup method were invoked.

If none of the previous conditions hold then this thread's interrupt status will be set.

Interrupting a thread that is not alive need not have any effect.
```

2）中文翻译

```
interrupt()的作用是中断本线程。
本线程中断自己是被允许的；其它线程调用本线程的interrupt()方法时，会通过checkAccess()检查权限。这有可能抛出SecurityException异常。

如果本线程是处于阻塞状态：调用线程的wait(), wait(long)或wait(long, int)会让它进入等待(阻塞)状态，或者调用线程的join(), join(long), join(long, int), sleep(long), sleep(long, int)也会让它进入阻塞状态。若线程在阻塞状态时，调用了它的interrupt()方法，那么它的“中断状态”会被清除并且会收到一个InterruptedException异常。例如，线程通过wait()进入阻塞状态，此时通过interrupt()中断该线程；调用interrupt()会立即将线程的中断标记设为“true”，但是由于线程处于阻塞状态，所以该“中断标记”会立即被清除为“false”，同时，会产生一个InterruptedException的异常。

如果线程被阻塞在一个Selector选择器中，那么通过interrupt()中断它时；线程的中断标记会被设置为true，并且它会立即从选择操作中返回。

如果不属于前面所说的情况，那么通过interrupt()中断线程时，它的中断标记会被设置为“true”

中断一个“已终止的线程”不会产生任何操作。
```

3）总结

线程处于运行态和阻塞态时中断的处理方式是不同的。当线程处于运行态时，通过interrupt方法可设置中断标志位，程序可通过中断标志位判断是否中断线程的执行。而当线程处于阻塞状态时，中断标志位是无法成功设置的，因此不能通过中断标志位判断是否中断线程的执行，而只能通过InterruptedException来中断线程的执行。

具体应用，查看interrupt最佳实践