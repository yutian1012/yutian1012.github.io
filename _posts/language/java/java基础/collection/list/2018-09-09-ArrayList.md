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

