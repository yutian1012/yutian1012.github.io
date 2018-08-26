---
title: redis 字符串对象的存储
tags: [redis]
---

字符串在Redis内部是如何进行编码的问题。

Redis使用了三种不同的编码方式来存储字符串对象，并会根据每个字符串值自动决定所使用的编码方式：

1）int：用于能够使用64位有符号整数表示的字符串

```
set a 12345

object encoding a
# 输出int
```

注：能表示java的一个长整型long

2）embstr：短字符串

用于长度小于或等于44字节（在Redis3.x中曾经是39字节）的字符串；

```
set a xxxx

object encoding a
# 输出embstr
```

注：这种类型的编码在内存使用和性能方面更有效率。

3）raw：用于长度大于44字节的字符串。

```
set a xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

object encoding a
# 输出raw
```

4）使用Object命令或查看

可以使用OBJECT命令来查看与键相关联的Redis值对象的内部编码方式