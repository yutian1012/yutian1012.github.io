---
title: UnsupportedClassVersionError错误
tags: [basic]
---

java.lang.UnsupportedClassVersionError，版本不一致的错误信息。即编译版本高，运行版本低。即低版本的运行环境无法支撑高版本编译的class文件。

### 1. 错误发生

运行最新的hsqldb.jar时出现错误

```
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/hsqldb/server/Server : Unsupported major.minor version 52.0
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(Unknown Source)
        at java.security.SecureClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.access$100(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
```

注：最后一句Unsupported major.minor version 52.0（版本52.0）

### 2. 分析

使用解压缩软件解压hsqldb.jar文件，查看META-INF/MANIFEST.MF文件，文件最后面的两行可以看出使用的

```
Created-By: 1.8.0_121 (Oracle Corporation)
Specification-Version: 2.4.0
```

注：可以看出是java 1.8版本编译的

### 3. 通过class文件查看

od或者hexdump工具，这里使用notepad++的十六进制插件查看

```
od -x hello.class
或者
hexdump hello.class
```

![](/images/java_basic/exception/class-version.png)

注：前四个字节是Java class的特殊符号，叫做magic字段，用来告诉JVM这是个class文件，之后的两个字节是minor版本号，再之后的两个字节是major版本号。

```
JDK1.5      31（16进制）  49（十进制）

JDK1.6      32（16进制）  50（十进制）

JDK1.7      33（16进制）  51（十进制）

JDK8        34（16进制）  52（十进制）
```

因此，根据错误version 52.0，所以可以确定使用的jdk是1.8版本的。