---
title: JDK的SPI（Service Provider Interface）原理分析
tags: [basic]
---

SPI机制提供了一个表达接口和其具体实现类之间的绑定关系的方案。具体是在JAR包的"META-INF/services/"目录下建立一个文件，文件名是接口的全限定名，文件的内容可以有多行，每行都是该接口对应的具体实现类的全限定名。

参考：http://blog.csdn.net/liuchangqing123/article/details/52304644

参考：http://www.cnblogs.com/javaee6/p/3714719.html

参考：http://blog.csdn.net/dslztx/article/details/47145729