---
title: java序列化接口
tags: [basic]
---

java实现序列化必须实现Serializable接口。

1）Serializable接口

该接口只是一个象征性的接口而已。继承这个接口说明这个对象可以被序列化了，没有其他的目的。

注：只有序列化的对象才可以存储在存储设备上。为了对象的序列化而需要继承该接口。

2）常见类都可序列化

java中常见的几个类（如：Interger、String等），都实现了serializable接口。

3）传递性

序列化类的所有子类本身都是可序列化的。