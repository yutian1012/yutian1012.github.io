---
title: java序列化与serialVersionUID
tags: [basic]
---


*serialVersionUID是序列化与反序列化条件*

1）限制条件

虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化ID是否一致。

```
private static final long serialVersionUID = 1L
```

虽然两个类的功能代码完全一致，但是序列化ID不同，他们无法相互序列化和反序列化。

*注：如果没有特殊需求，就是用默认的1L，这样可以确保代码一致时反序列化成功。*

注2：有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。

2）修改serialVersionUID使其无法反序列化

将代码反序列化，然后改变serialVersionUID查看是否能够反序列化。

错误信息：

```
java.io.InvalidClassException: ... local class incompatible: stream classdesc serialVersionUID = 1, local class serialVersionUID = 2
```