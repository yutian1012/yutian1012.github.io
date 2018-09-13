---
title: java ArrayList类的add方法分析
tags: [java]
---

ArrayList内部使用动态数组的方式维护集合元素，动态指的是能够扩展的数组。

注：这里的扩展指的是可以重新申请更大的空间，将原数据复制到新空间中

1）向ArrayList对象中添加元素的方法包括

```
public boolean add(E e);//末尾添加元素
public void add(int index, E element)//指定位置添加元素，会存在元素移动操作

//批量添加元素到集合中，待添加的元素是E的子类
public boolean addAll(Collection<? extends E> c);
public boolean addAll(int index, Collection<? extends E> c);
```

2）add方法代码实现

```
public boolean add(E e) {
    //确认数组容量是否能够容纳size+1的大小，size为目前集合的元素数量
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```