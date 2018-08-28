---
title: invokeDynamic字节码指令
tags: [jdk]
---

参考：https://blog.csdn.net/zxhoo/article/details/38387141

1）实例代码

```
package org.sjq.test.demo.lambda;
/**
 * 参考：https://blog.csdn.net/zxhoo/article/details/38387141
 */
public class InvokeDynamicDemo {
    public static void main(String[] args) {
        Runnable x = () -> {
            //System.out.println("Hello, World!"); 
        };
    }
}
```

2）反编译代码查看

```
# 定位到class类所在的目录，而不是包开始位置处
javap -p -c InvokeDynamicDemo.class > E:/InvokeDynamicDemo.txt
```

查看生成的main方法的字节码

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V --方法描述，字符串数组，返回值为V
    flags: ACC_PUBLIC, ACC_STATIC  -- 方法的访问权限，public和static
    Code:  -- 代码执行体
      stack=1, locals=2, args_size=1 
         0: invokedynamic #19,  0             // InvokeDynamic #0:run:()Ljava/lang/Runnable;
         5: astore_1
         6: return
      LineNumberTable:
        line 7: 0
        line 10: 6
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  args   [Ljava/lang/String;
            6       1     1     x   Ljava/lang/Runnable;
```