---
title: jvm编译模式
tags: [jvm]
---

编译方式参数：-Xint,-Xcomp和-Xmixed

a.-Xint 指令

int是interpretation的简称，翻译解释的意思。意味着强制JVM执行所有的字节码（class文件），这会降低运行速度（10倍左右）

b.-Xcomp 指令

comp是Compile的简称，编译的意思。这里的编译与我们理解的java文件编译成class文件不同，是更底层的编译，是将class文件编译为本地代码。意味着JVM在第一次使用时会把所有的字节码编译成本地代码，从而带来最大程度的优化。

注：虽然比-Xint的效率要高，但是它没有让JVM启动JIT编译器的全部功能，JIT编译器一般会在运行时创建方法使用文件，然后一步步的优化每个方法，因此该指令还是会造成一定的效率衰减。使用JIT即时编译效率更高。

-Xmixed 指令

JVM在运行时可以动态的把字节码编译成本地代码，默认开启了混合模式，因此无需显示的指定。

实例比较

```
public class ModelTest {
    public static void main(String[] args) {
        /*run in -Xint 强制执行所有字节码 Average time: 3271*/
       /*run in -Xcomp 第一次使用时把字节码转换成本地文件 Average time: 1874*/
       /*run in -Xmixed JVM默认支持的模式 无需设置  Average time: 1250*/
       long startTime = System.currentTimeMillis();
       Map<String, Object> map = new HashMap<>();
       for (int i = 0; i < 1000000; i++) {
           map.put(String.valueOf(i), i);
       }
       System.out.println("Average time: " + (System.currentTimeMillis() - startTime));
   }
}
```

运行测试

```
# 测试-Xint运行模式
java -Xint ModelTest
Average time: 2561

# 测试-Xcomp运行模式
java -Xcomp ModelTest
Average time: 1971

# 测试-Xmixed运行模式
java -Xmixed ModelTest
Average time: 1789
```
