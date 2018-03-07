---
title: JNI数据结构对应关系
tags: [jni]
---

参考：http://blog.csdn.net/linzhuowei0775/article/details/49965877

Java编程语言中的数据类型怎样对映到本地编程语言C和C++中的数据类型。实际上，大多数程序都需要传递参数给本地方法，或者从本地方法接受结果。

在java与c进行交互时一定只能通过JNI定义的类型来实现交互，除此之外可以考虑使用进程通信。

注：数据类型的对应关系可以通过JNI规范（oracle官网）来具体查看。

### 基本类型的映射

java的8种原生类型与JNI中声明的类型的对应关系，以及JNI的类型与C语言中数据类型的对应关系。

注：将JNI类型作为一个中间过渡类型数据。

![](/images/middleware/jni/cygwin/jni-data-type.png)

### 引用类型的映射

所有的JNI引用都是类型jobject。

1）JNI jstring类型
