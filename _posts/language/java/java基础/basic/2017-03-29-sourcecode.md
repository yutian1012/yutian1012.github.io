---
title: java源码查看--sun开头的源码查看
tags: [basic]
---

### 以sun开头的java源代码查看

以查看sun.reflect.annotation.AnnotationInvocationHandler为例

1）该类位于rt.jar包中

![](/images/java_basic/object/sourceAnnotationInvocationHandler.png)

2）下载源码

http://download.java.net/openjdk/jdk7/

http://hg.openjdk.java.net/jdk7/jdk7/jdk/

3）解压后搜索到AnnotationInvocationHandler类的位置

```
D:\迅雷下载\openjdk\jdk\src\share\classes\sun\reflect\annotation
```

4）打包成zip，并在myeclipse中添加源码关联

目录定位到D:\迅雷下载\openjdk\jdk\src\share\classes下，将该文件夹下的内容打成压缩包openjdk-share.zip文件。

在myeclipse中添加相应的关联，即可查看。

![](/images/java_basic/object/attachsource.png)

注：jdk中的存在src.zip，但该压缩包中很多java类的源码并没有提供，因此必须查看openjdk，才能看到某些类的源代码。