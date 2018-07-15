---
title: memcached适用场景
tags: [memecached]
---

参考：https://www.cnblogs.com/jiaosq/p/5812250.html

### 适用memcached的业务场景

1）减轻数据负载压力

如果网站包含了访问量很大的动态网页，因而数据库的负载将会很高。由于大部分数据库请求都是读操作，那么memcached可以显著地减小数据库负载。

2）减轻cpu的负载

如果数据库服务器的负载比较低但CPU使用率很高，这时可以缓存计算好的结果（ computed objects ）和渲染后的网页模板（enderred templates）。

3）存储session

利用memcached可以缓存session数据、临时数据以减少数据库操作。

4）缓存一些很小但是被频繁访问的文件。

### 不适用memcached的业务场景

1）缓存对象的大小大于1MB

Memcached本身就不是为了处理庞大的多媒体（large-media）和巨大的二进制块（streaming-huge-blobs）而设计的。

默认编译下Memcache只支持1M的Value。事实上由于存在序列化反序列化的过程，所以从实践的角度来说也不建 议把非常大的数据保存在Memcache中。Memcache适合面向输出的内容缓存，而不是面向处理的数据缓存，也就是不太适合把大块数据放进去拿出来处理之后再放回去，而是适合拿出来就直接给输出了或是拿出来不需要处理直接使用。

2）key的长度大于250字符

memcached对key的长度有限制，不能大于250字节。

3）虚拟主机不让运行memcached服务

如果应用本身托管在低端的虚拟私有服务器上，像vmware, xen这类虚拟化技术并不适合运行memcached。Memcached需要接管和控制大块的内存，如果memcached管理的内存被OS或hypervisor交换出去，memcached的性能将大打折扣。

注：虚拟机的内存不能完全由memcached掌控

4）应用运行在不安全的环境中

Memcached未提供任何安全策略，仅仅通过telnet就可以访问到memcached。如果应用运行在共享的系统上，需要着重考虑安全问题。

5）业务需要持久化

业务本身很重要需要进行持久化操作，而memcached是内存数据库，一旦断电内存被清理了，数据也就丢失了。

6）业务数据的修改更新频繁

业务数据的修改更新如果很频繁，那么缓存的数据也就没有意义了。