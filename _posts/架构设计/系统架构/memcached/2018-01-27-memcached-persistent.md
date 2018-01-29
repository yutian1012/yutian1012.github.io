---
title: memcached的持久化方案
tags: [memecached]
---

1）使用redis替代memcached

redis支持数据持久化功能，同时也提供了内存访问数据。

读写10W/s，是性能最快的key-valueDB。支持更丰富的类型（List，Set集合），单个value的最大限制是1GB，memcached只能保存1MB。

注：不能用作海量数据的高性能读写（容量受物理内存限制）。

2）repcached

为memcached提供复制（replication）功能的patch。

3）Flared

存储到QDBM，实现异步复制和fail-over等功能

4）memagent

连接多个memcached，实现一致性hash，请求转发

5）memcacheddb

持久化到berkelyDB，主服务器可读写，辅服务器只读。

6）Tokyo Tyrant

粗糙你到Tokyo cabinet，与memcached协议兼容，能通过http访问。

### 分布式数据库（扩展阅读）

1）CouchDB持久化

2）Cassandra分布式/NoSql

3）MongoDB分别式/NOSQL

4）BigTable/Hbase

适合于数据仓库，大型数据的处理与分析。