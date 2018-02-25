---
title: Hashcode和equals方法
tags: [basic]
---

hashcode和equals会影响基于散列的集合HashMap、HashSet和Hashtable。

注：散列值即hash值，是一个整型数值。

### Object类默认的equals和hashCode方法

1）默认equals方法是比较内存地址是否相等

2）默认的hashCode方法则是根据内存地址产生一个整数。

### hashCode的通用约定

1）equals和hashCode方法必须同时实现

无论你何时实现equals方法，你必须同时实现hashCode方法

2）hashCode和equals使用相同的成员变量做限制

在应用程序的执行期间，只要对象的equals方法所用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法都必须始终如一地返回同一个整数（同一个对象）。

3）equals与hashCode的一致性

如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果（两个对象间的比较）。

如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中的任意一个对象的hashCode方法，则不一定要产生不同的整数结果。

### equals和hashcode实现

通过equals方法可以判断对象是否相同，直接使用eclipse生成equals和hashcode方法即可。

1）Employee类

```
public class Employee {
    private String name;
    private int age;
    private Date birth;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Date getBirth() {
        return birth;
    }
    public void setBirth(Date birth) {
        this.birth = birth;
    }
}
```

2）equals方法

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

2）hashcode方法

```
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + age;
    result = prime * result + ((birth == null) ? 0 : birth.hashCode());
    result = prime * result + ((name == null) ? 0 : name.hashCode());
    return result;
}
```

注：可以看到hashcode值的计算涉及到变量与在equals方法比较时使用的变量是相关的。即equals需要通过哪些成员变量来判断，对应的hashcode就需要用哪些成员变量来计算散列值。