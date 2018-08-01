---
title: String、StringBuffer、StringBuilder的区别
tags: [interview]
---

1）可变与不可变：

String是不可变字符串对象，StringBuilder和StringBuffer是可变字符串对象（其内部的字符数组长度可变）。

2）是否多线程安全：

String中的对象是不可变的，也就可以理解为常量，显然线程安全。

StringBuffer与StringBuilder中的方法和功能完全是等价的，只是StringBuffer中的方法大都采用了synchronized关键字进行修饰，因此是线程安全的，而StringBuilder没有这个修饰，可以被认为是非线程安全的。

3）String、StringBuilder、StringBuffer三者的执行效率：

```
StringBuilder > StringBuffer > String 
```

当然这个是相对的，不一定在所有情况下都是这样。

```
String str = "hello"+ "world"
StringBuilder st  = new StringBuilder().append("hello").append("world")
```

注：string常量的定义方式就比stringBuilder构建的方式效率高，主要是由编译器的优化。

4）应当根据不同的情况来进行选择使用：

当字符串相加操作或者改动较少的情况下，建议使用 String str="hello"这种形式；

当字符串相加操作较多的情况下，建议使用StringBuilder

当字符串相加操作较多的并且采用了多线程的情况，则使用StringBuffer