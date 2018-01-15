---
title: csv数据
tags: [BI]
---

### CSV

1）定义

逗号分隔值（Comma-Separated-Values，CSV，有时也称为字符分隔值），分隔字符也可以不是逗号。该文件以纯文本形式存储表格数据。

注：编码可以使用ASCII和unicode（utf-8）等编码格式

2）数据存储形式（类似表数据）

CSV文件由任意数目的记录（行）组成，记录间以某种换行符分隔；每条记录由字段（列）组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符。

注：文本中每行表示一条记录，字段间使用逗号或制表符等分隔

注2：数据时可以包含列标题

3）应用

最广泛的应用是在程序之间转移表格数据，如可以将数据库的信息导出成cvs数据，cvs格式的数据可以使用excel等工具打开。这样就完成了数据的转移，即从数据库文件到excel文件的转移。

另外，cvs数据可以在多个数据库应用间转移数据，如mongodb可以通过cvs将数据转移到mysql，反过来也是可以的。

4）编辑工具：

作为纯文本文件建议使用note等文本编辑工具打开。

5）约束条件

第一：纯文本，使用某个字符集，（比如ASCII、Unicode、EBCDIC或GB2312）；

第二：由记录组成（典型的是每行一条记录），使用换行符分隔；

第三：每条记录被分隔符分隔为字段（典型分隔符有逗号、分号或制表符等）；

第四：每条记录都有同样的字段序列（字段的顺序和字段的数量）。

第五：开头是不留空，以行为单位；

注：一行数据不跨行，并且无空行

### 实例数据

数据库表导出和导入cvs数据

1）数据库表

数据库mysql中user表信息

```
CREATE TABLE `test1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=latin1;
```

表内容：

```
INSERT INTO `test`.`test1` (`id`, `name`, `age`) VALUES ('1', 'zhangsan', '23');
INSERT INTO `test`.`test1` (`id`, `name`, `age`) VALUES ('2', 'lisi', '22');
```

2）导出数据

导出时可以设置文本限定符，是否包含列标题等。

```
select * from test1 into outfile 'C:/Users/dell2/Desktop/test1.csv' fields terminated by ',' optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';
```

代码分析

```
select * from test1 
into outfile 'C:/Users/dell2/Desktop/test1.csv' -- 指定输出文件的位置
fields terminated by ',' -- 字段间以,号分隔
optionally enclosed by '"' -- 字段用"号括起，指定文本限定符
escaped by '"' -- 字段中使用的转义符为"
lines terminated by '\r\n'; -- 行以\r\n结束，指定记录分隔符，即回车换行
```

输出的数据为（打开test1.csv文件）：

```
1,"zhangsan",23
2,"lisi",22
```

3）导入csv文件

```
delete from test1;
# 导入命令
load data infile 'C:/Users/dell2/Desktop/test1.csv' into table test1 fields terminated by ',' optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';
```