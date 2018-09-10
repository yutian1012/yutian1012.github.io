---
title: java ArrayList类
tags: [java]
---

1）基本用法

ArrayList内部使用动态数组维护集合对象，允许插入null空值，并且不是线程安全的。在多线程并发修改的时，建议使用线程安全的集合对象代替ArrayList。或者使用Collections.synchronizedList将集合对象包装成线程安全的集合。

注：由于数组的特性，末尾添加删除元素时间复杂度为O(1)，其他位置操作需要移动后续元素，因此，适用于读多写少的场景。

2）属性分析

```
//默认初始容量
private static final int DEFAULT_CAPACITY = 10;
//存放元素的数组，transient类型修饰，
transient Object[] elementData;
```

3）问题

ArrayList中使用transient修饰符修饰数组对象，那么又是如何实现序列化的？

虽然transient修饰的成员变量序列化时不会处理，但是在ArrayList中提供了writeObject和readObject方法，因此在使用ObjectOutputStream调用writeObject方法序列化对象时，会反射调用ArrayList类中提供的writeObject方法，实现ArrayList对象的序列化操作。而在writeObject方法中是会遍历数组元素，将其序列化输出。同时反序列化输入时调用readObject方法，将信息还原成ArrayList对象。