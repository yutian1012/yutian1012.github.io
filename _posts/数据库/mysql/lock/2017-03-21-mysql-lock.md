---
title: mysql锁
tags: [database]
---

参考：http://blog.csdn.net/xifeijian/article/details/20313977

1）查询表级锁争用情况

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺。

```
show status like 'table%';

+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 2258  |
| Table_locks_waited    | 0     |
+-----------------------+-------+
```

注：Table_locks_immediate表示立即释放表锁数，Table_locks_waited表示需要等待的表锁数，如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。

2）查看连接的进程

```
show full processlist;
```

注：查看是否存在正在执行的慢SQL记录线程。
