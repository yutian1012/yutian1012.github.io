---
title: mysql中leftjoin优化的一个实例
tags: [database]
---

在实际工作中有一个连接查询，A主表，B从表（是一对多的关系）。

```
select a.*,group_concat(b.name) from A a left join B b on a.id=b.aid
```

只是一个简单的查询语句，A中的记录大概有1000条，B中的记录大概有5000条，查询却使用了将近5s的时间，非常不正常。

1）为B表中的aid字段添加索引

```
alter table B add index aid_index(aid);
```

再次查询，发现效率几乎没有任何提升

2）使用Explain分析sql执行

```
explain select a.*,group_concat(b.name) from A a left join B b on a.id=b.aid
```

发现在扫描B表时type为All，并没有使用到索引。

3）观察表结构

发现A表中的id使用的是bigint类型，而B表中的aid使用的却是varchar类型，两者类型不相同，导致查询时不能使用索引。

修改B表的aid字段的字段类型为bigint

```
alter table B modify column aid bigint(20);
```

再次执行查询，发现查询速度变得只有0.1s左右。而且explain发现B表的扫描type变成ref了。