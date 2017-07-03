---
title: mysql中group_concat函数
tags: [database]
---

参考：http://blog.csdn.net/hi_kevin/article/details/8473719

### 准备测试环境

1）建表语句

```
DROP TABLE IF EXISTS `test1`;
CREATE TABLE `test1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

2）数据插入

```
INSERT INTO `test1` VALUES ('1', 'zhangsan', '23');
INSERT INTO `test1` VALUES ('2', 'lisi', '22');

INSERT INTO test1(NAME,AGE) SELECT NAME,AGE FROM test1;
```

注:这里的主键是自动增长的,因此不需要设置值.

### 用法

1）指定分隔符

分隔符可以自定义，使用SEPARATOR来指定

```
select group_concat(age) from test1 where name='zhangsan';

select group_concat(age separator '!!') from test1 where name='zhangsan';
```

注:默认使用逗号分隔

2）使用时一般结合group by语句

```
select name,group_concat(age separator '!!') from test1 group by name;
```

### 常见问题

1)长度陷阱

用group_concat连接字段的时候是有长度限制的，并不是有多少连多少。使用group_concat_max_len系统变量，可以设置最大长度。

```
set group_concat_max_len = 5
```

注：再次执行查询发现长度变成5，多出的字段都被截掉了。

查看系统目前支持的长度

```
show variables like 'group_concat%';
```