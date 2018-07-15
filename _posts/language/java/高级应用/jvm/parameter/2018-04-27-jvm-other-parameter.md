---
title: jvm其他参数设置
tags: [jvm]
---

1）JIT编译参数

a.-XX:CompileThreshold

-XX:CompileThreshold指通过JIT编译器，将方法编译成机器码的触发阀值。可以理解为调用方法的次数，例如调1000次，将方法编译为机器码，那么下次方法再次调用时就可以直接使用编译的机器码运行了。

b.-XX:+PrintCompilation

通过设置-XX:+PrintCompilation，可以简单的输出一些关于从字节码转化成本地代码的编译过程。

c.-XX:+CITime

通过设置-XX:+CITime，可以在JVM关闭时得到各种编译的统计信息。

2）快照参数

a.-XX:+PrintGCDetails

-XX:+PrintGCDetails能够打印GC详细信息。

b.-XX:+HeapDumpOnOutOfMemoryError

-XX:+HeapDumpOnOutOfMemoryError参数，JVM会在遇到OutOfMemoryError时拍摄一个“堆转储快照”，并将其保存在一个文件中。

c.-XX:HeapDumpPath

-XX:HeapDumpPath将堆转储快照存储到文件的路径配置。