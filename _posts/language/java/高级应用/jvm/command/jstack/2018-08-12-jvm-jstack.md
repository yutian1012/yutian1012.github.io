---
title: jvm java虚拟机栈
tags: [jvm]
---

jstack是java虚拟机自带的一种堆栈跟踪工具。

1）主要分为两个功能： 

a. 针对活着的进程做本地的或远程的线程dump； 

jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合。

生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

b. 针对core文件做线程dump（JVM崩溃时生成的文件）。

如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。

2）命令

```
jstack [ option ] pid
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP
```

参数信息

```
-h | -help：打印帮助信息

-l：长列表，打印关于锁的附加信息，会使得JVM停顿得长久得多，一般情况不建议使用。

-m：打印java和native（c或c++框架）的所有栈信息，可以打印JVM的堆栈，显示上Native的栈帧。一般应用排查不需要使用。

-F：当jstack命令没有响应的时候强制打印栈信息
```

3）常用工具

a. jdk安装目录的bin下的jstack命令

b. IBM Thread and Monitor Dump Analyze for Java

下载地址：https://www.ibm.com/developerworks/community/groups/service/html/communitystart?communityUuid=2245aa39-fa5c-4475-b891-14c205f7333c

注：一个小巧的Jar包，能方便的按状态，线程名称，线程停留的函数排序，快速浏览。

c. Spotify提供的Web版在线分析工具，将dump文件内容粘贴到在线分析工具即可

使用地址：http://spotify.github.io/threaddump-analyzer/