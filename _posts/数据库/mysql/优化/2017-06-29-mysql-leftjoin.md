---
title: mysql中leftjoin的优化
tags: [database]
---

使用mysql版本5.5.29

参考：http://www.cnblogs.com/zedosu/p/6555981.html

left join的主要使用场合是当从记录中没有与主记录关联的信息时，数据也能被显示出来，即以主记录为基准。

### 场景

两张表关联查询，使用left join关联，表A是主记录，表B是与表A关联的记录，并且表A与表B之间的关系为一对多的关系。A表大概有900条记录，表B大概有5000条记录。记录量不是很大，但是有left join关联查询却比较慢。

```
SELECT DISTINCT
    a.id,b.name,GROUP_CONCAT(DISTINCT b.name)
FROM
    A a
    LEFT JOIN B b ON a.id = b.aid
WHERE
    app.pid = 1
GROUP BY A.ID
LIMIT 20;
```

### left join的意义

1）防止主记录变少

left join是为了解决主表记录不变少的查询，因为如何使用直接连接的方式查询的话，A（主）表中不存在与B（从）表中对应的记录，会导致A表中查询的记录变少。

2）leftjoin与分组统计函数

不使用GROUP BY并且不使用GROUP_CONCAT函数会使记录变多

```
SELECT DISTINCT
    a.id,b.name
FROM
    A a
    LEFT JOIN B b ON a.id = b.aid
WHERE
    app.pid = 1
LIMIT 20;
```

注：显然这种方式对于分页系统来说存在数据量与主表数据量不匹配

使用GROUP BY但不使用GROUP_CONCAT会是查询的子表记录不全，因为B表中可能存在多条与A相关联的记录。如果使用GROUPBY语句会导致显示的B表中的name字段信息缺失（只会显示第一个相关的值）。

```
SELECT DISTINCT
    a.id,b.name
FROM
    A a
    LEFT JOIN B b ON a.id = b.aid
WHERE
    app.pid = 1
GROUP BY A.ID
LIMIT 20;
```

### 解决方案

1）方案一

将left join改为直连。

```
SELECT DISTINCT
    a.id,b.name,GROUP_CONCAT(DISTINCT b.name)
FROM
    A a , B b
WHERE
    a.id = b.aid and app.pid = 1
GROUP BY A.ID
LIMIT 20;
```

注：将条件放到where中能够加速检索，显然这种方式容易导致记录数据量变少，而不是以主表A的数据量为基准（因为关联B中可能不存在与A表对应的记录）。

注2：使用B表字段做查询条件，并传递了查询条件的值时，那么left join和join没有太大区别（查询字段中不出现B表中的字段，即B表字段不做投影），但效率却完全不同。

2）方案二

执行2步查询，第一步查询获取A表相关的主键信息，只需要返回主表的主键记录。使用DISTINCT关键字可以时记录数量不会增多。在使用B（从）表中的字段作为查询条件时在追加left join关键字，否则查询时不使用left join语句。

查询获取A表的主键

```
SELECT DISTINCT
    a.id
FROM
    A a
    //先判断查询条件中是否有B从表中的字段，如果有则添加上left join语句
    LEFT JOIN B b ON a.id = b.aid
WHERE
    app.pid = 1 ....其他条件
LIMIT 20;
```

通过A表中的主键信息获取

```
SELECT DISTINCT
    a.id,b.name,GROUP_CONCAT(DISTINCT b.name)
FROM
    A a
    LEFT JOIN B b ON a.id = b.aid
WHERE
    app.pid = 1 and app.id in (...)
GROUP BY A.ID
LIMIT 20;
```

注：与最初代码的区别是添加了条件app.id，即缩小了待查询的结果集，这种方式也能够很快的返回数据。

3）方案三

升级mysql版本，发现当前版本使用的是5.5.29，在使用mysql版本5.6.23时节省了一半的时间。

### 补充

1）不能使用直连方式：

因为可能会是记录数量变少，从表B中不存在主表A关联的记录，因此导致A记录不能显示。

2）不能去掉GROUP_CONCAT

去掉GROUP_CONCAT而不是使用GROUP BY会导致记录数量变多，而使用GROUP BY又会导致从表相关记录信息不全。
