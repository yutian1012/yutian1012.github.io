---
title: jvm内存参数设置
tags: [jvm]
---

### java Heap堆

堆空间是用来存储java对象实例的空间，被所有线程共享。

1）空间分配

堆空间分为：老年代和新生代（通过-XX:NewRatio参数设置比例），新生代又分三个区（Eden，s0，s1，可通过-XX:SurvivorRatio参数设置）

2）堆调优参数

a.-Xms

-Xms设置堆的最小值，堆内存越大，就越不容易FullGC。同样越小，垃圾回收频率越高。而垃圾回收总时间越大，吞吐量越低。因此，频繁的垃圾回收会降低系统的吞吐量。

b.-Xmx

-Xmx设置堆的最大值

c.-Xmn

-Xmn设置新生代的大小。

注：没有参数设置老年代空间大小，老年代大小可由堆空间大小减去新生代大小来确定。

d.-XX:NewSize

-XX:NewSize设置新生代的大小。

e.-XX:NewRatio

-XX:NewRatio设置老年代和新生代的比例。

f.-XX:SurvivorRatio

-XX:SurvivorRatio设置新生代中的eden与survivor区的比例。如果该参数调大，那么s0和s1空间会变小。

g.-XX:TargetSurvivorRatio

-XX:TargetSurvivorRatio设置survivor区的可使用率，达到此值，被传送到老年代。

### java方法区

java方法区，所有线程共享，也将该区域称为持久代，或永久代。

1）参数设置

a.-XX:MaxPermSize

-XX:MaxPermSize参数设置最大的持久代大小

b.-XX:PermSize

-XX:PermSize设置最小的持久代大小

### 线程栈

线程栈，是线程私有区域。jvm中堆的内存越大，线程栈的内存越大，能创建的线程数就越小。

1）参数设置

a.-Xss

-Xss设置线程栈的大小。