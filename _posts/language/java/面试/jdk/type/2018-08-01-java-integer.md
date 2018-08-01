---
title: java Integer类
tags: [interview]
---

参考：https://blog.csdn.net/login_sonata/article/details/71001851

1）Integer和int（元素类型与包装类型）

Java是面向对象的编程语言，一切都是对象，但是为了编程的方便还是引入了基本数据类型。

为了能够将这些基本数据类型当成对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class），int的包装类就是Integer。

注：从Java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。

2）基本类型

原始类型：boolean，char，byte，short，int，long，float，double

包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

注：Java中的基本数据类型只有8个，除了基本类型（primitivetype），剩下的都是引用类型（referencetype）。

3）区别

Ingeter是int的包装类，int的初值为0，Ingeter的初值为null。

4）自动装箱/拆箱

```
# 自动装箱
Integer i = 127;//编译时被翻译成：Integer i = Integer.valueOf(127);

# 自动拆箱
int i = 128;
Integer i2 = 128;
System.out.println(i == i2); //Integer会自动拆箱为int，所以为true
```

5）缓存问题

valueOf()函数了，这个函数对于-128到127之间的数，会进行缓存，下次使用时，就会直接从缓存中取，就不会new了。

```
Integer i4 = 127;
Integer i5 = 127;
System.out.println(i4 == i5);//true
Integer i6 = 128;
Integer i7 = 128;
System.out.println(i6 == i7);//false
```

Integer的new对象与valueOf方法装箱的对象是不同的（位于不同位置）

```
# new与非new不会相同
Integer i8 = new Integer(127);
System.out.println(i5 == i8); //false
```

6）总结

无论如何，Integer与new Integer不会相等。不会经历拆箱过程，new出来的对象存放在堆，而非new的Integer常量则在常量池（在方法区），他们的内存地址不一样，所以为false。

两个都是new出来的,都为false。还是内存地址不一样。

int和Integer(无论new否)比，都为true，因为会把Integer自动拆箱为int再去比。

两个都是非new出来的Integer，如果数在-128到127之间，则是true,否则为false。因为java在编译使用valueOf()函数会对-128到127之间的数进行缓存。