---
title: jvm如何使用方法区中的信息
tags: [jvm]
---

实例分析

1）代码，创建文件Volcano.java

```
class Lava { 
    private int speed = 5; // 5 kilometers per hour 
        void flow() { 
    } 
}

class Volcano { 
    public static void main(String[] args) { 
        Lava lava = new Lava(); 
        lava.flow(); 
    } 
}
```

类编译的字节码信息

```
# 编译文件
javac Volcano.java
# 反编译class类，-c参数对代码进行反汇编，并将汇编结果重定向到Volcano.txt文件
javap -c Volcano.class > Volcano.txt
```

反汇编文件查看

```
Compiled from "Volcano.java"
class Volcano {
  Volcano();
    Code:
       0: aload_0
       1: invokespecial #1  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2    // class Lava
       3: dup
       4: invokespecial #3    // Method Lava."<init>":()V
       7: astore_1
       8: aload_1
       9: invokevirtual #4    // Method Lava.flow:()V
      12: return
}
```

2）执行过程

编译完成后，运行java类时，使用java命令，并将Volcano当参数传入

a.运行入口类

```
java Volcano
```

以参数方式把“Volcano”传给了jvm。有了这个名字，jvm找到了这个类文件（Volcano.class）并读入，它从类文件提取了类型信息并放在了方法区中。

b.执行main方法

通过解析存在方法区中的字节码，jvm激活了main方法。在执行时，jvm保持了一个指向当前类（Volcano）常量池的指针。

注：jvm在还没有加载Lava类的时候就已经开始执行了。正像大多数的jvm一样，不会等所有类都加载了以后才开始执行，它只会在需要的时候才加载。

c.main的第一条指令

告知jvm为类在常量池第一项的类分配足够的内存。jvm使用指向Volcano常量池的指针找到第一项，发现是一个对Lava类的符号引用，然后它就检查方法区看lava是否已经被加载了。这个符号引用仅仅是类lava的完整有效名”lava“。

这里我们看到为了jvm能尽快从一个名称找到一个类，一个良好的数据结构是多么重要。这里jvm的实现者可以采用各种方法，如hash表，查找树等等。同样的算法可以用于Class类的forName()的实现。

d.加载Lava类

当jvm发现还没有加载过一个称为”Lava”的类，它就开始查找并加载类文件”Lava.class”。它从类文件中抽取类型信息并放在了方法区中。jvm于是以一个直接指向方法区lava类的指针替换了常量池第一项的符号引用。以后就可以用这个指针快速的找到lava类了。

注：这个替换过程称为常量池解析(constant pool resolution)。

e.初始化Lava对象

jvm为新的lava对象分配空间了，这次，jvm仍然需要方法区中的信息。它使用指向lava数据的指针(volcano常量池第一项的指针)找到一个lava对象究竟需要多少空间。jvm总能够从存储在方法区中的类型信息知道某类型对象需要的空间。一旦jvm知道了一个Lava对象所要的空间，它就在堆上分配这个空间并把这个实例的变量speed初始化为缺省值0。假如lava的父对象也有实例变量，则也会初始化。

当把新生成的lava对象的引用压到栈中，第一条指令也结束了。下面的指令利用这个引用激活java代码把speed变量设为初始值

f.调用Lava对象的方法

下一条指令会用这个引用激活Lava对象的flow()方法。