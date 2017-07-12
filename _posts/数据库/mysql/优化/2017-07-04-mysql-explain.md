---
title: mysql中explain分析查询效率
tags: [database]
---

环境准备，建表语句

```
CREATE TABLE `person` (
  `id` int(11) NOT NULL,
  `age` int(3) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `birthday` date DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

参考：http://www.2cto.com/database/201508/428707.html

### explain的type

explain执行计划中type字段分为以下几种：ALL，INDEX，RANGE，REF，EQ_REF，CONST,SYSTEM，NULL（8种）。

注：从左至右，性能从最差到最好

1）type = ALL，全表扫描，MYSQL扫描全表来找到匹配的行。

```
EXPLAIN select * from person;
```

![](/images/database/mysql/explain/typeAll.png)

2）type = index，索引全扫描，MYSQL遍历整个索引来查找匹配的行。

在name上创建索引，并执行查询

```
ALATER TABLE PERSON ADD INDEX name_index (name);
EXPLAIN select name from person;
```

![](/images/database/mysql/explain/typeindex.png)

注：这里查询获取的字段为name索引字段

3）type = range ，索引范围扫描

```
常见于<、<=、>、>=、between等操作符
```

```
EXPLAIN select * from person where age >20 and age <100;
```

![](/images/database/mysql/explain/typerangewithoutindex.png)

添加索引后再次运行

```
alter table person add index age_index (age);
EXPLAIN select * from person where age >20 and age <100;
```

![](/images/database/mysql/explain/typerangewithindex.png)

4）type = ref，where条件中使用非唯一索引作为查询字段

用非唯一性索引或者唯一索引的前缀扫描，返回匹配某个单独值的记录行。

```
EXPLAIN select * from person where name='zz';
```

![](/images/database/mysql/explain/typeref.png)

注：name字段已经设置过索引了，这里出现在了where条件中

5）type = eq_ref，相对于ref来说就是使用的是唯一索引，一般用在关联表中

对于每个索引键值，只有唯一的一条匹配记录

新建表person_info

```
CREATE TABLE `person_info` (
  `id` int(6) NOT NULL,
  `address` varchar(255) DEFAULT NULL,
  `telphone` varchar(20) DEFAULT NULL,
  `perid` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

注：perid与person表的主键关联

```
EXPLAIN select * from person p,person_info pe where p.id=pe.perid;
```

![](/images/database/mysql/explain/typeeq_ref.png)

注：在联表查询中使用primary key或者unique key作为关联条件

6）type = const/system，单表中最多只有一条匹配行

添加一个唯一键进行查询

```
alter table person add column idcard varchar(18) comment '身份证号';
alter table person add unique idcard_un_index (idcard);
```

```
EXPLAIN select * from person where idcard='155555';
```

![](/images/database/mysql/explain/typeconst.png)


为perid字段添加外键，关联查询

```
ALTER TABLE person_info add FOREIGN KEY person_fk (perid) references person(id);
```

```
EXPLAIN select * from person p,person_info pe where p.id=pe.perid and p.id=1;
```

![](/images/database/mysql/explain/typeconst2.png)

7）type = NULL，MYSQL不用访问表或者索引就直接能到结果

```
explain select 1 from dual
```

![](/images/database/mysql/explain/typeNull.png)