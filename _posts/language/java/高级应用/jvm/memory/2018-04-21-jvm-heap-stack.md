---
title: jvm堆和栈的区别
tags: [interview]
---

参考：https://iamjohnnyzhuang.github.io/java/2016/07/12/Java%E5%A0%86%E5%92%8C%E6%A0%88%E7%9C%8B%E8%BF%99%E7%AF%87%E5%B0%B1%E5%A4%9F.html

1）功能不同

栈内存用来存储局部变量和方法调用。

而堆内存用来存储Java中的对象。无论是成员变量，局部变量，还是类变量，它们指向的对象都存储在堆内存中。

2）共享性不同

栈内存是线程私有的。

堆内存是所有线程共有的。

3）异常错误不同

如果栈内存或者堆内存不足都会抛出异常。

栈空间不足：java.lang.StackOverFlowError。

堆空间不足：java.lang.OutOfMemoryError。

4）空间大小

栈的空间大小远远小于堆的。
