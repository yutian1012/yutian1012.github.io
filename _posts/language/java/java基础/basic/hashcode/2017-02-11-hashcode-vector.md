---
title: 集合中hash散列与equals方法
tags: [basic]
---

### 集合list的contains方法

查看内部源代码可以发现list中是通过数组中的对象分别调用equals方法来比较是否包含对象的。(查看ArrayList源代码）

```
public static boolean compareCollection(){
    Date date=new Date();
    List<Employee> list=new ArrayList<Employee>();
    Employee employee=new Employee("zhangsan", 22, date, true);
    list.add(employee);
    Employee employee2=new Employee("zhangsan", 22, date, true);
    return list.contains(employee2);//返回true
}
```

注：这里Employee已经生成了相应的equals和hashcode方法了

### 集合Set的contains方法判断

1）Set内部是使用HashMap来存储数据的。

2）代码：Employee没有重写hashcode方法

```
public static boolean compareCollectionSet(){
    Date date=new Date();
    Set<Employee> set=new HashSet<Employee>();
    Employee employee=new Employee("zhangsan", 22, date, true);
    set.add(employee);
    Employee employee2=new Employee("zhangsan", 22, date, true);
    return set.contains(employee2);//返回false
}
```

3）修改Employee类

实现hashcode代码值，并且同时重写equals方法，再执行上面的代码，此时会返回true。

### HashMap中对象的hashCode方法

hashMap中的contains使用hashcode定位对象存放的位置，从该位置的列表中一个一个的比较待存放的对象（调用equals方法比较对象），因此在使用时需要确保存储对象的equals和hashcode都要被重写，并且满足相应约定。

注：具体过程先使用hashcode值计算出hash，再从列表中使用equals比较对象。

### hashset原理

1）判断对象是否存在

HashSet中的contains方法实际是调用的HashMap中的containsKey方法来判断的。

2）数据存放

HashSet的数据是如何在Map中存放的，将对象e放置在map的key中，并将Object实例（new Object()）设置到map的value中,value没有什么意义。

3）containsKey方法

HashMap中的containsKey会根据传递的key计算出该key的hash值，然后通过hash值查找对应的元素，如果table中找不到hashcode相应的数组元素（链表头元素），则返回null。

```
（Entry<K,V> e = table[indexFor(hash, table.length)];）
```

注：因此，在使用HashSet存储对象时，对象一定要实现hashcode方法和equals方法。