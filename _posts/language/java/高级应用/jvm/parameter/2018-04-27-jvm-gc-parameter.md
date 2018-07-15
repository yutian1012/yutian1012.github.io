---
title: jvm垃圾回收参数设置
tags: [jvm]
---

1）-XX:+UseSerialGC

串行收集器，单线程，独占式垃圾回收。在JVM-Client模式下，是默认的垃圾回收器。

Hot Spot虚拟机

```
#  指定新生代和老年代都使用串行收集器
-XX:+UseSerialGC
```

2）-XX:UseParNewGC

Hot Spot虚拟机

```
# 新生代使用并行收集器，老年代使用串行收集器
-XX:UseParNewGC
```

3）-XX:+UseParallelGC

并行收集器

Hot Spot虚拟机

```
# 新生代使用并行收集器，老年代使用串行收集器
-XX:+UseParallelGC
```

4）-XX:+ParallelGCThreads

并行收集器的线程数量可以使用-XX:ParallelGCThreads设置

5）-XX:+UseAdaptiveSizePolicy

-XX:+UseAdaptiveSizePolicy设置新生代并行回收器的自适应性。

6）-XX:+UseParallelOldGC

Hot Spot虚拟机

```
# 新生代和老年代都使用并行收集器
-XX:+UseParallelOldGC
```

7）-XX:+UseConcMarkSweepGC

CMS收集器

Hot Spot虚拟机

```
# 新生代使用并行收集器，老年代使用CMS收集器
-XX:+UseConcMarkSweepGC
```

8）-XX:CMSInitinatinOccupancyFaction

-XX:CMSInitinatinOccupancyFaction参数指定老年代使用率达到多少时，进行一次CMS垃圾回收，会产生内存碎片，垃圾收集完成以后，会进行一次独立式的碎片整理。