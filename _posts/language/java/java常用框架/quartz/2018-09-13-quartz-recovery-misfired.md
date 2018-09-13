---
title: Quartz recovery 及misfired机制的源码分析
tags: [quartz]
---

参考：https://blog.csdn.net/gklifg/article/details/27089605?utm_source=tuicool&utm_medium=referral

1）理解Quartz对异常及崩溃后处理机制

quartz作为成熟的任务调度系统对系统的异常及崩溃后处理机制有很好的设计，以保证整个调度过程是一个逻辑闭环，任何阶段出现的问题都可以通过框架中的机制尽最大限度的弥补，并将系统的状态引向正轨。

主要处理两大类异常：misfired（哑火），fail-over（故障转移）

注：对于具体的任务Job的异常处理是由程序的执行逻辑来处理的，并不在Quartz的处理机制中。

2）misfired（哑火）理解

哑火顾名思义，就是quartz在应该触发（fire）trigger的时候未能及时将其触发。

Quartz是如何检查这种机制的？

调度器需要每隔一段时间（15s~60s）查看一次各trigger的nextfiretime，检查出否有tirgger的下一次触发落在了当前时间之前足够长的时间，在这里系统设定了一个60s的域（misfireThreshold），当一个trigger的下一次触发时间早于当前时间60s之外时，调度器判定该触发器misfired。在发现有触发器哑火之后启动相应的流程回复trigger至正常状态。

注：可以把该线程比作调度器监控线程

调度器监控线程是在调度器初始化时与主调度线程类quartzSchedulerThread同时开启的一个线程类MisfireHandler中进行的。