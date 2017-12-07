---
title: 并发容器
tags: [architecture]
---

### 集合包装

使用包装的情况实现集合线程安全，只适用于并发量比较小的情况下。

以Map为例查看内部实现：内部使用synchronzied关键字实现，在synchronizedMap方法中，将Map包装成SynchronizedMap类，该类实现了map接口，map中的所有方法都重新使用synchronized关键字进行了限制。通过synchronized锁住SynchronizedMap实例的mutex成员变量实现互斥访问。

```
private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
    private final Map<K,V> m;     // Backing Map
    final Object      mutex;        // Object on which to synchronize
    ...
    public int size() {
        synchronized (mutex) {return m.size();}
    }
    public boolean isEmpty() {
        synchronized (mutex) {return m.isEmpty();}
    }
    public boolean containsKey(Object key) {
        synchronized (mutex) {return m.containsKey(key);}
    }
    ...
}
```

注：这种方式的缺点是，将多个线程都串行化了，get和put方法都需要互斥访问，不适合大并发量的场景。

1）HashMap类

HashMap不是一个线程安全的集合，在多线程操作情况下会出现。使用Collections的synchronizedMap对其进行包装，使其变成线程安全的集合。

```
Map m=Collections.synchronizedMap(new HashMap());
```

2）List类

同样在Collections工具类中也提供了synchronizedList方法将其包装成线程安全集合

```
List list=Collections.synchronizedList(new ArrayList());
```

3）Set类

同样在Collections工具类中也提供了synchronizedSet方法将其包装成线程安全集合

```
Set set=Collections.synchronizedSet(new HashSet());
```

### ConcurrentHashMap类

一个线程安全的HashMap类，是一个高并发下使用HashMap线程安全的解决方案。

1）HashMap的实现（线程不安全的集合类）

内部实现是一个数组，数组中存放类型为Entry。当放入一个key时，会计算key的hash值，确定出存放值的数组位置。有时候会出现hash冲突，解决方法是使用链表的方式解决的。entry对象中包含一个next引用，用来对接hash冲突产生的数据。

2）ConcurrentHashMap的实现

将一个大的hashmap切割成多个小的hashmap，如一个大的hashmap中包含16个小的hashmap，意味着这个大的hashmap能够同时接收16个线程。小的hashmap使用Segment表示，通过位移和偏移地址来划分多个Segment。

Segment类的put方法用于存放数值，Segment继承了ReentrantLock，在设置值时会先tryLock方法，试图获取锁，如果不成功会尝试多次tryLock方法（retries次），如果依然无法成功，则会lock将线程挂起，等待激活。

rehash方法会扩充hashmap的容量，同时还有可能对已有元素进行重新设置位置的操作（再次hash排列）。

注：tryLock内部使用CAS方式尝试获取锁。

### BlockingQueue类

阻塞队列，是一个线程安全的类，但不是一个高性能的并发容器。但是一个非常适合在多线程间共享数据的容器。当队列为空，如果有读数据的线程来读取数据，就会等待，直到有写线程将数据放入，并唤醒读线程。同样，如果队列满了，会阻塞写线程，读线程读出数据后，唤醒写线程继续写入。

注：BlockingQueue是一个接口

ArrayBlockingQueue是BlockingQueue接口的一个实现。内部使用数值作为队列。内部使用了ReentrantLock，Condition等并发常用控制类。

注：查看ArrayBockingQueue源码的实现。

### ConcurrentLinkedQueue类

是一个高性能的队列，在高并发的情况下，能够保证一个很好的吞吐量。

注：合理的选择BlocingQueue和ConcurrentLinkedQueue。