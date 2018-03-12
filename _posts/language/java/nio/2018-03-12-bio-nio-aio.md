---
title: JAVA网络IO（BIO，NIO与AIO）
tags: [java]
---

参考：http://blog.csdn.net/anxpp/article/details/51512200

参考：https://www.zhihu.com/question/40930889/answer/146567853

1）关系

io最老；nio是1.4的特性；aio是1.7的特性。

2）场景

a.传统IO

传统IO是面向流的，是就用多少拿多少。是阻塞的。数据量不大的或者不在意阻塞的时可以用。