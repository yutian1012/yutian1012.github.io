---
title: 尽量避免使用JNI
tags: [jni]
---

当你开始着手准备一个使用JNI的项目时，请确认是否还有替代方案。应用程序使用JNI会带来一些副作用。

1）避免使用JNI的时候

a.通过网络交换

JAVA程序和本地程序使用TCP/IP或者IPC（Inter-Process Communication，进程间通信）进行交互。

b.分布式对象技术

JAVA程序可以使用分布式对象技术，如JAVA IDL API（Interface Definition Language）即使有CORBA。

注：这些方案的共同点是，JAVA和C处于不同的线程，或者不同的机器上。这样，当本地程序崩溃时，不会影响到JAVA程序。