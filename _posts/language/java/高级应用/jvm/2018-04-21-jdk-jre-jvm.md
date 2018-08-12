---
title: JDK，JVM，JRE的理解
tags: [interview]
---

参考：http://www.hollischuang.com/archives/78

1）如何理解一处编写处处运行。

Java语言是跨平台运行的，其实就是不同的操作系统，使用不同的JVM映射规则，让其与操作系统无关，完成了跨平台性。JVM有自己完善的硬件架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。JVM的主要工作是解释自己的指令集（即字节码）并映射到本地的CPU的指令集或OS的系统调用。

注：这就是为么不同的平台会有不同的jdk安装，windows系统jdk，linux系统openjdk。

2）JDK，JVM，JRE的理解

JDK是整个Java的核心，包括了Java运行环境JRE、Java工具和Java基础类库（狭义的jdk，即java api）。

JRE是运行JAVA程序所必须的环境的集合，包含JVM标准实现及Java核心类库。

JVM是整个java实现跨平台的最核心的部分，能够运行以Java语言写的程序。

注：JDK大于JRE，JRE大于JVM。

a.JDK(Java Development Kit) 

JDK是Java语言的软件开发工具包(SDK)，如编译器和调试器。

b.JRE（Java Runtime Environment），

JRE是Java运行环境，包含JVM标准实现及Java核心类库。JRE是Java运行环境，并不是一个开发环境。

c.JVM(Java Virtual Machine)

JVM是Java虚拟机的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

JRE是JVM的环境，一个应用程序就是一个JVM实例。一个机器上一般同时只有一个JRE环境，但是可以同时有多个JVM实例

在JDK的安装目录下有一个jre目录，里面有两个文件夹bin和lib，在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和lib合起来就称为jre。

![](/images/java_basic/interview/jvm/jdk-jre-jvm.png)

注：写代码需要JDK，运行代码要JRE，JRE包含JVM。