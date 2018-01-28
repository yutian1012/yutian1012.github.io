---
title: memcached与应用本身自带的缓存比较
tags: [memecached]
---

1）本地缓存（Local cache）

参考：http://blog.csdn.net/u011683530/article/details/51029734

### 三者的比较

memcached和服务器的local-cache（如PHP的APC、mmap文件等）相比，有什么优缺点？

1）内存大小的限制

首先，local cache面临着严重的内存限制，能够利用的内存容量受到（单台）服务器空闲内存空间的限制。

2）local cache有一点比memcached和query cache（数据库自带的查询缓存）都要好，那就是它不但可以存储任意的数据，而且没有网络存取的延迟。

3）集体失效性？

local cache缺少集体失效（group invalidation）的特性。在memcached集群中，删除或更新一个key会让所有的观察者觉察到。但是在localcache中，我们只能通知所有的服务器刷新cache（很慢，不具扩展性）或者仅仅依赖缓存超时失效机制。