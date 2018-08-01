---
title: java 字符串相关的类
tags: [interview]
---

参考：https://www.cnblogs.com/xiaoxi/p/6036701.html

参考2：https://www.cnblogs.com/dreamroute/p/5946272.html

参考3：https://blog.csdn.net/wangtaomtk/article/details/52267548

1）String类

String类是final类，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。

String对象，那它的值就无法改变了。

String类其实是通过char数组来保存字符串的（源码分析）。

2）分析string类的常用方法

无论是sub操、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

注：String对象一旦被创建就是固定不变的了，对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象

3）字符串常量池

字符串的分配和其他对象分配一样，是需要消耗高昂的时间和空间的，而且字符串我们使用的非常多。JVM为了提高性能和减少内存的开销，在实例化字符串的时候进行了一些优化，即使用字符串常量池。

每次创建字符串常量时，JVM会首先检查字符串常量池，如果该字符串已经存在常量池中，那么就直接返回常量池中的实例引用。如果字符串不存在常量池中，就会实例化该字符串并且将其放到常量池中。

4）实例1

a、b和字面上的chenssy都是指向JVM字符串常量池中的"chenssy"对象，他们指向同一个对象。new关键字一定会产生一个对象chenssy，而这个对象是存储在堆中，与常量池没有任何关系。只有调用对象的intern方法，才会保存到常量池中。

```
String a = "chenssy";
String b = "chenssy";
String c = new String("chenssy");
```

另外，下面的代码也能看出new对象是存储在堆中的

```
String str1=new String("aaa");
String str2=new String("aaa");
System.out.println(str1==str2);//false 可以看出用new的方式是生成不同的对象 
```

5）实例2

字符串常量池与new对象的比较

```
package org.test.basedemo.type;

public class StringDemo {
    public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = "Hel" + "lo";
        String s4 = "Hel" + new String("lo");
        String s5 = new String("Hello");
        String s6 = s5.intern();
        String s7 = "H";
        String s8 = "ello";
        String s9 = s7 + s8;
                  
        System.out.println(s1 == s2);  // true
        System.out.println(s1 == s3);  // true
        System.out.println(s1 == s4);  // false
        System.out.println(s1 == s9);  // false
        System.out.println(s4 == s5);  // false
        System.out.println(s1 == s6);  // true
        
        System.out.println(System.identityHashCode(s1));//2018699554
        System.out.println(System.identityHashCode(s2));//2018699554
        System.out.println(System.identityHashCode(s4));//1311053135
        System.out.println(System.identityHashCode(s5));//118352462
        System.out.println(System.identityHashCode(s9));//1550089733
    }
}
```