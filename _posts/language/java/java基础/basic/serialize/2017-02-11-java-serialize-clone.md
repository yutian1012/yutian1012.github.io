---
title: java序列化与克隆
tags: [basic]
---

*利用序列化来做深拷贝*

有时为了方便构造对象，可以通过序列化的方式实现对象的克隆操作。

1）场景

java对象的clone操作有时是比较麻烦的，需要实现cloneable接口，并重新clone方法。但对于有些第三方封装好的类，无法实现重新代码，继承Cloneable接口，那么如何实现深拷贝呢？

针对这种情况可以考虑利用序列化来实现对象的深拷贝，从而达到对象的复制。

*注：必须确定被拷贝对象的父对象是否能够满足反序列化操作。*

2）默认的约定

*对于java的实体类一般都需要实现Serializable接口。*

有些框架要求实体类必须实现Serializable接口，这方面框架的父类中实现了该接口，那么我们编写的代码继承了框架提供的类，自然就实现了Serializable接口。

