---
title: JAVA网络IO（BIO，NIO与AIO）
tags: [java]
---

参考：http://blog.csdn.net/anxpp/article/details/51512200

参考：https://www.zhihu.com/question/40930889/answer/146567853

1）关系

传统IO；NIO是1.4的特性；AIO是1.7的特性。

注：传统的IO包已经使用NIO重新实现过，即使我们不显式的使用NIO编程，也能从中受益。

2）三者的比较

a.传统IO

传统IO是面向流的，是就用多少拿多少，是阻塞的。数据量不大的或者不在意阻塞的时可以用。

b.NIO

nio里有Buffer类作为缓冲区，Channel（通道）相当于io里的steam的抽象，Selector是nio提供的管理多个Channel的工具。nio出现也是因为io渐渐成为一些程序速度的瓶颈。

c.AIO

AIO加了一个异步的特性。当我们要拿数据花费时间太长的时候，我们可以考虑使用异步的io。异步就是可以理解为，让io先处理着，线程先去干别的事情了，等IO处理完了通知我一下。AIO提供的事件处理接口CompletionHandler，定义了回调函数，这些函数在IO完成后会被自动的调用。