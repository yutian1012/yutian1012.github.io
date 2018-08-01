---
title: 输出内容地址
tags: [interview]
---

1）对象内存地址的输出

使用System.identityHashCode方法输出对象的内存地址

```
package org.test.basedemo.other;

public class AddressDemo {
    
    public static void main(String[] args) {
        AddressDemo demo=new AddressDemo();
        
        System.out.println(demo);
        System.out.println(demo.hashCode());
        System.out.println(System.identityHashCode(demo));
    }
}
```

输出结果

```
org.test.basedemo.other.AddressDemo@7852e922
2018699554
2018699554
```

2）输出内容分析

其中对象的输出@后跟着的是内存地址，这里是使用的16进制的输出形式。7852e922转换成十进制值为：2018699554，与identityHashCode方法输出的值相同。

3）使用场景

可以用该方法输出字符串常量池中字符串的地址，从而验证是否是同一个对象。

```
String s1 = "Hello";
String s2 = "Hel" + "lo";
String s3 = "Hel" + new String("lo");

System.out.println(s1 == s2);  // true
System.out.println(s1 == s3);  // false

System.out.println(System.identityHashCode(s1));//2018699554
System.out.println(System.identityHashCode(s2));//2018699554
System.out.println(System.identityHashCode(s3));//1311053135
```

注：从内存输出上就能看出是否是同一个对象引用