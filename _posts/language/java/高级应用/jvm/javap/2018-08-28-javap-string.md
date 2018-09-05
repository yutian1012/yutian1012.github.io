---
title: javap查看底层对字符串的优化操作
tags: [jdk]
---

参考：https://blog.csdn.net/xieyuooo/article/details/17452383

1）实例代码

```
package org.test.basedemo.javap;

public class JavapStringDemo {
    public static void main(String[] args) {
        String a="a"+"b"+1;
        String b="ab1";
        System.out.println(a==b);//true
    }
}
```

2）反编译class文件

```
# 定位到class文件目录下，而不是包目录下
javap -v JavapStringDemo.class > E:/javapStringDemo.txt
```

3）查看文件中的常量池

```
Constant pool:
   #1 = Class              #2             // org/test/basedemo/javap/JavapStringDemo
   #2 = Utf8               org/test/basedemo/javap/JavapStringDemo
   #3 = Class              #4             // java/lang/Object
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Methodref          #3.#9          // java/lang/Object."<init>":()V
   #9 = NameAndType        #5:#6          // "<init>":()V
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lorg/test/basedemo/javap/JavapStringDemo;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = String             #17            // ab1
  #17 = Utf8               ab1
  ....
```

常量池部分：#数字开头，这个数字是顺序递增的，通常把它叫做常量池的入口位置。当程序中需要使用到常量池的时候，就会在程序的对应位置记录下入口位置的标识符。

常量池内容：常量内容会分为很多种。在每个常量池项最前面的1个字节，来标志常量池的类型（我们看到的Method、String等等都是经过映射转换后得到的，字节码中本身只会有1个字节来存放）。找到类型后，接下来就是内容，内容可以是直接存放在这个常量池的入口中，也可能由其它的一个或多个常量池域组合而成。

注：常量池的条目内容分为2部分，第一部分表示类型，第二部分表示内容

代码中调用常量池数据

```
执行代码（截取的main方法的输出指令）
0: ldc           #16 
-----------------------------
常量池信息
#16 = String             #17            // ab1
#17 = Utf8               ab1

分析：
ldc表示将常量加载到操作数栈的指令，#16即指定了操作数的常量池入口，可查看常量池

常量池#16处的常量类型为String，内容为#17（即内容引用了#17处的数据）
常量池#17处的常量类型为utf8编码的字符串，字符串的内容为ab1。
```



查看生成的main方法

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: ldc           #16                 // String ab1
         2: astore_1
         3: ldc           #16                 // String ab1
         5: astore_2
         6: getstatic     #18                 // Field java/lang/System.out:Ljava/io/PrintStream;
         9: aload_1
        10: aload_2
        11: if_acmpne     18
        14: iconst_1
        15: goto          19
        18: iconst_0
        19: invokevirtual #24                 // Method java/io/PrintStream.println:(Z)V
        22: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  args   [Ljava/lang/String;
            3      20     1     a   Ljava/lang/String;
            6      17     2     b   Ljava/lang/String;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 18
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, int ]
}
```