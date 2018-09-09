---
title: java Collection接口
tags: [java]
---

1）Collection接口

集合（也可称为容器），是用于处理一组对象的数据结构。

2）相关方法

Collection接口是集合类的顶层接口，定义了集合中常用的方法

```
int size();//返回集合中的元素个数，最大值Integer.MAX_VALUE
boolean contains(Object o);//判断集合中是否包含某个元素，涉及到equals方法的比较
Iterator<E> iterator();//获取变量集合元素的迭代器
Object[] toArray();//将集合对象转换成数组对象
<T> T[] toArray(T[] a);//将集合对象转换成数组对象
boolean add(E e);//向集合中添加元素
boolean remove(Object o);//从集合中移除某个元素
boolean containsAll(Collection<?> c);//判断是否包含集合的所有元素
boolean addAll(Collection<? extends E> c);//将集合中的所有元素添加到该集合中
boolean removeAll(Collection<?> c);//从集合中移除所有指定集合的元素
boolean retainAll(Collection<?> c);//保留指定集合元素
void clear();//清空集合元素
```

3）jdk8后新加入的方法

在jdk8后添加了一些default方法，这样所有的子类或子接口都可以使用该default方法

```
//从集合中移除所有符合给定Predicate的元素
default boolean removeIf(Predicate<? super E> filter){...}

//spliterator是一个分割迭代器
default Spliterator<E> spliterator(){...}

//集合元素做stream流的源，构造出Stream对象，内部使用spliterator方法
default Stream<E> stream() {...}

//使用并行流的方式构造出stream对象
default Stream<E> parallelStream() {...}
```