---
title: redis list对象的存储
tags: [redis]
---

Redis在内部使用quicklist存储列表对象。有两个配置选项可以调整列表对象的存储逻辑：

1）list-max-ziplist-size：

一个列表条目中一个内部节点的最大大小（quicklist的每个节点都是一个ziplist）。在大多数情况下，使用默认值即可。

2）list-compress-depth：列表压缩策略。

如果我们会用到Redis中列表首尾的元素，那么可以利用这个选项来获得更好的压缩比（该参数表示quicklist两端不被压缩的节点的个数，当列表很长的时候最可能被访问的数据是位于列表两端的数据，因此对这个参数精确地进行调优可以实现在压缩比和其他因素之间的平衡）。