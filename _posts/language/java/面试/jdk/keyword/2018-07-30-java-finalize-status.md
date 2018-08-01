---
title: finalize的执行过程(待确认)
tags: [interview]
---

参考：https://www.cnblogs.com/Smina/p/7189427.html

参考：https://www.cnblogs.com/smilesmile/p/3849122.html

1）对象finalize方法的执行流程

当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象复活。

2）对象的两种状态

针对对象是否可达，以及是否执行了finalize方法，可确定出该对象的状态信息。

两类状态空间：

一是终结状态空间（针对对象是否执行了finalize方法的状态） 

```
F = {unfinalized, finalizable, finalized}；
三种状态：不能执行finalize方法；可以执行finalize方法；已经执行finalize方法；

unfinalized: 新建对象会先进入此状态，GC并未准备执行其finalize方法，因为该对象是可达的

finalizable: 表示GC可对该对象执行finalize方法，GC已检测到该对象不可达。但此时并没有执行finalize方法，即finalizeable表示该方法的finalize可被执行。

finalized: 表示GC已经对该对象执行过finalize方法
```

二是可达状态空间（针对对象是否可达）

```
R = {reachable, finalizer-reachable, unreachable}
三种状态：对象可达；执行finalize方法后重新可达（对象再生）；不可达；

reachable: 表示GC Roots引用可达

finalizer-reachable（f-reachable）：表示除了reachable外，从F-Queue可以通过引用到达的对象。

unreachable：对象不可通过上面两种途径可达
```

3）对象状态转换

![](/images/interview/jdk/finalize/finalize-status.gif)

状态转换说明：

```
1 首先，所有的对象都是从Reachable+Unfinalized走向死亡之路的。 

2 当从当前活动集到对象不可达时，对象可以从Reachable状态变到F-Reachable或者Unreachable状态。 

3 当对象为非Reachable+Unfinalized时，GC会把它移入F-Queue，状态变为F-Reachable+Finalizable。 

4 好了，关键的来了，任何时候，GC都可以从F-Queue中拿到一个Finalizable的对象，标记它为Finalized，然后执行它的finalize方法，由于该对象在这个线程中又可达了，于是该对象变成Reachable了（并且Finalized）。而finalize方法执行时，又有可能把其它的F-Reachable的对象变为一个Reachable的，这个叫做对象再生。 

5 当一个对象在Unreachable+Unfinalized时，如果该对象使用的是默认的Object的finalize，或者虽然重写了，但是新的实现什么也不干。为了性能，GC可以把该对象之间变到Reclaimed状态直接销毁，而不用加入到F-Queue等待GC做进一步处理。 

6 从状态图看出，不管怎么折腾，任意一个对象的finalize只至多执行一次，一旦对象变为Finalized，就怎么也不会在回到F-Queue去了。当然没有机会再执行finalize了。 

7 当对象处于Unreachable+Finalized时，该对象离真正的死亡不远了。GC可以安全的回收该对象的内存了。进入Reclaimed。 
```