---
title: memcached内存管理机制
tags: [memecached]
---

参考：https://www.cnblogs.com/UnGeek/p/5136410.html

1）malloc内存管理机制

malloc的全称为memory allocation，即动态内存分配。早起的memcached内存管理方式是通过malloc分配内存，并通过free来回收内存。这种方式容易产生内存碎片，进而导致操作系统对内存的管理效率下降。最坏的情况下，会导致操作系统比memcached进程本身都慢。因此，引入了slab allocator来管理memcached内存。

2）slab内存管理机制

现在的memcached利用slab allocation机制来分配和管理内存。

a.按照内存预先规定的大小，将内存分成若干个slab，大小默认是1MB。在每个slab内存区中再建立若干大小相等的chunk（不同slab的chunk大小是不同的，同一个slab中的chunk大小是相同的）。在存储数据时，将对象存入到适合的chunk中，避免大量重复的初始化和清理，减轻了内存管理器负担。这些内存块不会释放，可以重复利用。

注：slab在很多地方也称为page

b.可以启动时使用-f来调整slab间的chunk大小的差异。

c.分配内存空间时不是一次性的就将所有的内存都划分到不同的slab组中（slab class）。而是在需要的时候才会去分配一个page（slab大小，即1MB），放到这个slab组中。并且该组中的chunk大小都是相同的。

d.存储对象时，会找到最佳大小的匹配chunk进行分配。

注：memcached中的slab最大是1MB，因此不能存储超过1MB的数据

### memcached的检测过期与删除机制

1）memcached懒惰检测对象过去机制

memcached使用的是懒惰检测对象过期策略，自己不会监控存入的key/value键值对是否过期，而是在获取key值时查看记录的时间戳，检测其是否过期。这种策略不会再过期检测上浪费cpu资源。

不主动监测item对象是否过期，而是在get时才会检查item对象是否过期以及是否应该删除。同时，memcached也不会主动释放过期对象的内存空间。如果为添加的数据设定了过期时间或内存缓存满了，在数据过期后，客户端不能通过key取出它的值，而其存储空间将被重新利用（存放数据时直接覆盖这部分过期内存空间）。

2）memcached懒惰删除对象机制

当删除item对象时，一般不释放内存空间，而是作标记删除，将指针放入slot回收插槽，下次分配的时候直接使用。

memcached在分配空间时，会优先使用已经过期的key/value对空间。当分配的内存空间占满时，memcached就会使用LRU算法来分配空间（最近最少被使用的键值对），将其空间分配给新的key/value对。

在某些情况下（完整缓存），如果不想使用LRU算法，那么可以同-M参数来启动memcached服务器。这样memcached在内存耗尽时，会返回一个错误信息，表示系统分配内存少了，需要手动调整memcached分配的内存空间。

注：对于过期的数据，即使在内存满的情况下也不会被回收，而是采用LRU算法替换数据。可以考虑，失效的数据一定是很久没有被访问了，因为只要被访问了发现失效就很快会被替换掉，因此失效数据可以在一定程度上通过LRU来解决。

注2：删除的对象会放入回收插槽后，可以当作为分配的空间来使用，即此时缓存空间未满。

3）总结

a.不主动检测item对象是否过期，而是在get时才会检查item对象是否过期以及是否应该删除。

b.当删除item对象时，一般不释放内存空间，而是作标记删除，将指针放入slot回收插槽，下次分配的时候直接使用。

c.当内存空间满的时候，才会根据LRU算法把最近最少使用的item对象删除

d.数据存入可以设定过期时间，但是数据过期后不会被理解删除，而是在get时才会检查item对象是否过期以及是否应该删除。

e.如果不希望系统使用LRU算法清除数据，可以使用-M参数启动memcached服务。

### slab内存分配

1）slab内存结构

slab内存结构是一个二维数组链表，一次申请内存的最小单位是一个page（一个slab）。

![](/images/architecture/memcached/slab.png)

2）slab的分配实例

![](/images/architecture/memcached/slab-assign.png)

3）slab分配的数据信息

![](/images/architecture/memcached/slab-data.png)

### slab为item分配内存过程

这里的item指存储在hash表中的数据信息。

1）快速定位slab classid

计算item的key+value+suffix+32（假设值为90byte），判断值是否大于1MB，如果大于则无法存储直接丢弃。如果不大于则取最小适应的chunk对应的slab-class。如有48，96，120，则存储90byte的数据会选择96，即选择96的slab组存储数据。

2）按顺序寻找可以的chunk（上一个确定出slab-classId，在该slab组中继续查找）

a.检查slot回收空间：

检查slab回收空间slot中是否有剩余chunk（delete时标记到slot，get时检查到过期对象标记到slot），如果有则直接分配。

b.检查page中是否还剩余可以空间：

通过end_page_ptr指针检查page中是否有剩余chunk，如果有则直接分配。

c.检查memory是否可再分配

检查内存是否还有剩余，即memcached服务器指定分配的内存大小是否已满，如果未满，则开辟新的slab（分配新page），连接到slab-classid组中，然后再分配chunk。

d.替换数据

slab组内使用LRU扫描Item双向链表，将最近最少使用的chunk替换掉。

### 内存调优

调优的目的是提供内存的利用率，减少内存浪费。在并发量大的情况下，需要对业务数据进行分离。

调优的核心是提高内存命中率。

调优方法：-f参数（factor增长因子，默认1.25）；-n参数（设置chunk初始值，一般使用默认）。