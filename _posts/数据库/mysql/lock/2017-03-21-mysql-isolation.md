---
title: mysql隔离级别
tags: [database]
---

参考：http://blog.csdn.net/jiao_fuyou/article/details/16368827

场景：使用异步方式在mysql数据库中执行大量的sql操作，应用系统不响应问题。

连接上mysql，查看事务级别

```
select @@tx_isolation;

+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```