---
title: jvm java虚拟机栈-线程状态
tags: [jvm]
---

通过jstack命令来分析线程的情况的话，首先要知道线程都有哪些状态:

1）NEW（未启动）

不会出现在Dump中。

2）RUNNABLE（运行中）

在虚拟机内执行的，运行中状态，可能里面还能看到locked字样，表明它获得了某把锁。

3）BLOCKED（阻塞）

受阻塞并等待监视器锁，被某个锁(synchronizers)給block住了。

4）WATING（等待）

无限期等待另一个线程执行特定操作，等待某个condition或monitor发生，一般停留在park(), wait(), sleep(),join() 等语句里。

5）TIMED_WATING（有时限的等待）

有时限的等待另一个线程的特定操作，和WAITING的区别是wait() 等语句加上了时间限制wait(timeout)。

6）TERMINATED（终止）

线程终止，已退出的。
