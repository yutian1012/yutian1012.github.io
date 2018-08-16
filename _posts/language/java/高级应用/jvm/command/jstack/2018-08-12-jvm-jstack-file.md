---
title: jvm java虚拟机栈-jstack生成文件
tags: [jvm]
---

参考：https://blog.csdn.net/rachel_luo/article/details/8920596

1）程序运行代码

```
package org.sjq.thread.demo.stack;

public class ThreadStackFile implements Runnable{
    @Override
    public void run() {
        while(true) {
            //防止程序退出
        }
    }
    public static void main(String[] args) {
        Thread t=new Thread("workThread");
        t.start();
        
    }
}
```

2）生成stack

```
jstack.exe 6264 > e:/jstack_file.txt
```

3）文件内容

生成的文件内容共可以拆解成5个部分

```
# 第一部分: Full thread dump identifier，可以看到dump时间，虚拟机的信息。

2018-08-12 11:51:49
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):

# 第二部分: Java EE middleware, third party & custom application Threads

"DestroyJavaVM" #11 prio=5 os_prio=0 tid=0x00000000024e8000 nid=0x3ec waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"workThread" #10 prio=5 os_prio=0 tid=0x0000000016306000 nid=0x1828 runnable [0x00000000170bf000]
   java.lang.Thread.State: RUNNABLE
    at org.sjq.thread.demo.stack.ThreadStackFile.run(ThreadStackFile.java:6)
    at java.lang.Thread.run(Unknown Source)

# 第三部分：HotSpot VM Thread，JVM管理的内部线程，这些内部线程都是来执行native操作的

"Service Thread" #9 daemon prio=9 os_prio=0 tid=0x00000000162a9000 nid=0x1a1c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #8 daemon prio=9 os_prio=2 tid=0x000000001621d800 nid=0xa4c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=2 tid=0x000000001621c800 nid=0x8a8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=2 tid=0x000000001620c000 nid=0x19d0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001620a800 nid=0x16a0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x00000000150a0000 nid=0xb04 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000001508a800 nid=0x6c8 in Object.wait() [0x00000000160cf000]
   java.lang.Thread.State: WAITING (on object monitor)
    at java.lang.Object.wait(Native Method)
    - waiting on <0x00000000ec108ec8> (a java.lang.ref.ReferenceQueue$Lock)
    at java.lang.ref.ReferenceQueue.remove(Unknown Source)
    - locked <0x00000000ec108ec8> (a java.lang.ref.ReferenceQueue$Lock)
    at java.lang.ref.ReferenceQueue.remove(Unknown Source)
    at java.lang.ref.Finalizer$FinalizerThread.run(Unknown Source)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000000015043000 nid=0x370 in Object.wait() [0x00000000161ff000]
   java.lang.Thread.State: WAITING (on object monitor)
    at java.lang.Object.wait(Native Method)
    - waiting on <0x00000000ec106b68> (a java.lang.ref.Reference$Lock)
    at java.lang.Object.wait(Unknown Source)
    at java.lang.ref.Reference.tryHandlePending(Unknown Source)
    - locked <0x00000000ec106b68> (a java.lang.ref.Reference$Lock)
    at java.lang.ref.Reference$ReferenceHandler.run(Unknown Source)

"VM Thread" os_prio=2 tid=0x000000001503b800 nid=0x1bc4 runnable 

# 第四部分： HotSpot GC Thread， 当虚拟机打开了parallelGC，这些线程就会定期地GC清理。

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00000000024fd800 nid=0x1964 runnable 

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00000000024ff000 nid=0x48c runnable 

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x0000000002500800 nid=0x123c runnable 

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x0000000002502000 nid=0x1648 runnable 

"VM Periodic Task Thread" os_prio=2 tid=0x00000000162f1800 nid=0x18d0 waiting on condition 

# 第五部分：JNI global references count
JNI global references: 6
```

4）相关线程的分析

a. DestroyJavaVM线程 

在main函数结束后，虚拟机会自动启动一个DestroyJavaVM线程，该线程会等待所有user thread 线程结束后退出（即，只剩下daemon 线程和DestroyJavaVM线程自己，整个虚拟机就退出，此时daemon线程被终止），因此，如果不希望程序退出，只要创建一个非daemon的子线程，让线程不停的sleep即可。

b. C1 CompilerThread2线程

用来调用JITing，实时编译装卸class

c. C2 CompilerThread0线程

C2 Compiler是JVM在server模式下字节码编译器，-XX:CICompilerCount=threads参数能够指定启动的线程数量（默认值为2）。

d. Attach Listener线程

jvm提供一种jvm进程间通信的能力，能让一个进程传命令给另外一个进程，并让它执行内部的一些操作，比如说我们为了让另外一个jvm进程把线程dump出来，那么我们跑了一个jstack的进程，然后传了个pid的参数，告诉它要哪个进程进行线程dump，既然是两个进程，那肯定涉及到进程间通信，以及传输协议的定义，比如要执行什么操作，传了什么参数等。

注：jstack的普通模式就是通过AttachListener连接上去的

e. Signal Dispatcher线程

该线程用于处理操作系统发送给的JVM信号

f. Finalizer线程

其优先级为8，主要用于在垃圾收集前，调用对象的finalize()方法

g. Reference Handler线程

JVM在创建main线程后就创建Reference Handler线程，其优先级最高，为10，它主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题。

h. GC task thread线程

用于垃圾回收的线程，通过参数-XX:ParallelGCThreads设置并行收集器的线程数

i. VM Periodic Task Thread线程

该线程是JVM周期性任务调度的线程，它由WatcherThread创建，是一个单例对象。该线程在JVM内使用得比较频繁，比如：定期的内存监控、JVM运行状况监控。