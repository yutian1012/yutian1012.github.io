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

内部实现是一个数组， 数组中