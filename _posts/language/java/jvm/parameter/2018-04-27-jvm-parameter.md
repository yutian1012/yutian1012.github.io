---
title: jvm参数
tags: [jvm]
---

参考：https://blog.csdn.net/Thousa_Ho/article/details/77278656

###  参数分类

1）标准参数

功能和输出的参数都是很稳定的，在未来的JVM版本中不会改变，可以使用java -help检索出所有的标准参数。

2）X参数

X参数，非标准化参数，在未来的版本可能会改变，所有的参数都用-X开始。可以使用java -X检索。

3）XX参数

XX参数，非标准化参数，很长一段时间不会列出来，用于JVM开发的debug和调优。

### 参数设置方式

1）对于布尔类型的参数，

使用+或-来设置相应的选项，例如，

```
# +号用于激活<name>选项
-XX:+<name>

# -号用于注销<name>选项
-XX:-<name>
```

2）对于需要非布尔值的参数

对于需要非布尔值的参数，如string或者integer，先写参数的名称，后面使用=号赋值，

```
# 给<name>赋值<value>
-XX:<name>=<value>
```
