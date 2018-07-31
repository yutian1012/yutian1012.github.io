---
title: java异常体系
tags: [interview]
---

在java中，异常对象都是派生于Throwable类的一个实例。如果java内置的异常类不能够满足需求，用户还可以创建自己的异常类。

1）异常体系结构

![](/images/interview/jdk/exception/exception_struct.jpg)

所有的异常都是由Throwable类，直接子类为Error和Exception

2）Error类

Error层次结构描述了java运行时系统的内部错误和资源耗尽错误。大多数错误与代码编写者执行的操作无关，而表示代码运行时JVM（Java虚拟机）出现的问题。

注：应用程序不应该抛出这种类型的对象。

常见的Error实现类：StackOverFlowError，OutOfMemoryError

3）Exception类

Exceprion又分为RuntimeException和IOException

RuntimeException是由程序错误导致的异常；常见的RuntimeException（运行时异常)

```
IndexOutOfBoundsException(下标越界异常)
NullPointerException(空指针异常)
NumberFormatException （String转换为指定的数字类型异常）
ArithmeticException -（算术运算异常 如除数为0）
ArrayStoreException - （向数组中存放与声明类型不兼容对象异常）
SecurityException -（安全异常）
...
```

IOException表示程序本身没有问题，但由于像IO错误导致的异常。

```
IOException（其他异常）
FileNotFoundException（文件未找到异常。）
IOException（操作输入流和输出流时可能出现的异常。）
EOFException （文件已结束异常）
```

4）检查异常和非检查异常

unchecked exception（非检查异常）：包括运行时异常（RuntimeException）和派生于Error类的异常。对于运行时异常，java编译器不要求必须进行异常捕获处理或者抛出声明，由程序员自行决定。

checked exception（检查异常）：也称非运行时异常（编译期的异常）java编译器强制程序员必须进行捕获处理，对于非运行时异常如果不进行捕获或者抛出声明处理，编译都不会通过。

注：两者的主要区别是异常出现的时机，从而导致是否编译器强制需要进行捕获相应的异常。