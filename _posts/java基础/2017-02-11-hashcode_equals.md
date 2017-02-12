---
title: Hashcode和equals方法
tags: [basic]
---

java中Hashcode和equals方法

### 1. Object类默认的两个方法
1）默认eqals方法是比较内存地址是否相等

2）默认的hashCode方法则是根据内存地址产生一个整数。

### 2. Object.hashCode的通用约定
1）无论你何时实现 equals 方法，你必须同时实现 hashCode 方法

2）在应用程序的执行期间，只要对象的equals方法所用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法都必须始终如一地返回同一个整数（同一个对象）。

3）如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果（两个对象间的比较）。

4）如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中的任意一个对象的hashCode方法，则不一定要产生不同的整数结果。

### 3. IDE提醒同时设置equals和hashcode
在IDE工具中设置编译时提醒设置类实现了equals却没有实现hashcode的信息：

1）配置 Eclipse 来检测实现了 equals 方法但是没有实现 hashCode 方法的类，并显示错误。不幸的是，此选项默认是指为“忽略”。

2）Preferences > Java >Compiler > Errors/Warnings，然后用快速筛选器来搜索“hashcode”。将Ignore值设置为warning或errors就会给出相应的提醒。

### 4. hashcode
只会影响基于散列的集合（HashMap、HashSet和Hashtable）

### 5. equals和hashcode实现
通过equals方法可以判断对象是否相同，直接使用eclipse生成equals和hashcode方法即可。

代码判断相等过程：

```
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null) return false;
    if (getClass() != obj.getClass()) return false;
    Employee other = (Employee) obj;
    if (age != other.age) return false;//针对原生数据类型，非包装类类型的判断
    if (birth == null) {//日期类型
        if (other.birth != null) return false;
    } else if (!birth.equals(other.birth)) return false;
    if (name == null) {//字符串类型
        if (other.name != null) return false;
    } else if (!name.equals(other.name)) return false;
        
    return true;
}
```

### 6. 集合list的contains方法
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

### 7. 集合Set的contains方法判断
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

注：修改Employee类，实现hashcode代码值，再执行上面的代码，此时会返回true。

注2：set内部使用hashmap，而hashmap中的contains使用hashcode定位对象存放的位置，从该位置的列表中一个一个的比较待存放的对象，因此在使用set需要确保equals和hashcode都要被重写，并且满足相应约定。--过程先使用hashcode值计算出hash，再从列表中使用equals比较对象。

### 8. hashset原理
1）HashSet中的contains方法实际是调用的HashMap中的containsKey方法来判断的。

2）HashSet的数据是如何在Map中存放的，将对象e放置在map的key中，并将Object实例（new Object()）设置到map的value中,value没有什么意义。

3）HashMap中的containsKey会根据传递的key计算出该key的hash值，然后通过hash值查找对应的元素（Entry<K,V> e = table[indexFor(hash, table.length)];），如果table中找不到hashcode相应的数组元素（链表头元素），则返回null。

注：因此，在使用HashSet存储对象时，一定要实现hashcode方法。

### 9. 总结
1）首先hashCode必须是一个整数，即Integer类型的

2）其次满足一致性，即在程序的同一次执行无论调用该函数多少次都返回相同的整数。

3）另外与equas配合，如果两个对象调用equals相同那么一定拥有相同的hashcode，然而反之，如果两个对象调用equals不相等，hashcode不一定就不同。

注：相等（相同）的对象必须具有相等的哈希码（或者散列码）。如果两个对象的hashCode相同，它们并不一定相同。
