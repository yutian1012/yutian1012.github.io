---
title: mysql的bin-log日志
tags: [database]
---

### bin-log日志相关操作

参考：http://www.cnblogs.com/it-cen/p/5234345.html

关键点：bin-log的日志恢复需要与数据库备份一起使用，否则bin-log的备份恢复操作是很困难的。

1. bin-log日志服务

二进制日志，记录对数据发生或潜在发生更改的SQL语句，并以二进制的形式保存在磁盘中。作用，可以用来查看数据库的变更历史（具体的时间点所有的SQL操作）、数据库增量备份和恢复（增量备份和基于时间点的恢复）、MySQL的复制（主主数据库的复制、主从数据库的复制）。

1）查看日志服务是否启动：

```
show global variables like 'log_bin';
```

![](/images/database/mysql/logbin.png)

2）开启日志服务：修改my.ini配置文件

注：如果在windows的安装目录中有my-default.ini文件，没有my.ini文件。复制一份my-default.ini文件，重命名为my.ini。

将log_bin前的#去掉，并设置日志的输出位置

```
log_bin=D:\Program Files\mysql-5.6.23-winx64\log\mysql-bin
```

注：这里指定了bin-log的存储位置，以及文件前缀mysql-bin，默认存放位置为数据库文件所在目录下。

重新启动mysql，再次运行查看。

2. bin-log日志的使用

1）常用命令

```
flush logs; 会多一个最新的bin-log日志
show master status; 查看最后一个bin-log日志的相关信息
reset master; 清空所有的bin-log日志
```

注：每次重新启动mysql或flush logs都会创建一个新的bin-log日志文件，用来写入数据库的变化信息（记录数据库的操作信息）。

![](/images/database/mysql/bin-log.png)

3. 操作数据库，查看bin-log的变化

1）在test数据库中创建一个数据表

```
create table test1 (id int,name varchar(32),age int);
```

2）插入测试数据

```
insert into test1 values(1,'zhangsan',23);
```

3）查看bin-log的变化

```
show master status;
```

![](/images/database/mysql/bin-log-position.png)

注：发现日志文件的位置发生了变化。

4）打开bin-log文件查看

```
mysqlbinlog --no-defaults ../log/mysql-bin.000002
```

注：该命令位于mysql安装目录的bin目录下。

![](/images/database/mysql/bin-log-analysis.png)

5）利用bin-log恢复数据库操作

原理：将bin-log中记录的操作再次执行

首先删除数据表test1中的内容，然后在将其进行恢复

```
delete from test1;
```

利用定位进行恢复，首先查看bin-log日志信息，定位操作位置

```
mysqlbinlog --no-defaults ../log/mysql-bin.000002
```

![](/images/database/mysql/bin-log-recovery.png)

注：可以看出delete语句开始的位置为471，将数据库恢复到该位置即可。

--stop-position = "100" --start-position = "50"
根据开始位置或者结束位置来恢复自己想恢复的参数

--stop-date= "2016-03-02 12:00:00" --start-date= "2016-03-02 11:55:00" 根据开始日期时间或者结束位置来恢复自己想恢复的参数

执行恢复命令：

```
mysqlbinlog --database=test --stop-position=471 ../log/mysql-bin.000002 | mysql -uroot -p
```

错误信息：

```
ERROR 1050 (42S01) at line 26: Table 'test1' already exists
```

注：可以发现使用bin-log恢复数据的过程是将bin-log中的操作再次执行一遍。因为我们在操作时只删除了数据表中的信息，并没有删除数据表。因此，在执行时当执行到bin-log中的create table test1时，会出现该错误。

执行恢复命令：

```
mysqlbinlog --database=test --start-position=246 --stop-position=471 ../log/mysql-bin.000002 | mysql -uroot -p
```

注：执行insert语句是从246位置开始的，在471位置结束，因此，只会取出这两个位置之间的操作进行执行。

恢复数据库后，再次查看bin-log信息

![](/images/database/mysql/bin-log-recovery2.png)

发现：bin-log日志中的信息是在原来的基础上继续添加执行操作。

问题：如果数据的插入操作来源于不同的时间段，并且可能中间夹杂着其他表的数据操作，那么如何使用bin-log进行恢复？

这种数据的恢复是非常麻烦的，甚至会造成数据不能恢复完全的情况。因此，数据库的备份是必须的。一般的操作是，结合备份数据，并从备份点开始恢复数据。

4. bin-log恢复数据库的实际操作

参考：http://www.cnblogs.com/kevingrace/p/5904800.html

1）数据恢复思路

首先：利用全备的sql文件中记录的CHANGE MASTER语句，binlog文件及其位置点信息，找出binlog文件中增量的那部分。

然后：用mysqlbinlog命令将上述的binlog文件导出为sql文件，并剔除其中的drop语句。

最后：通过全备文件和增量binlog文件的导出sql文件，就可以恢复到完整的数据。

2）情况bin-log日志信息（在全新的基础上做测试）

3）创建数据表

```
create table customers(
id int not null auto_increment,
name char(20) not null,
age int not null,
primary key(id)
)engine=InnoDB;
```

4）插入数据

```
insert into customers values(1,"wangbo","24");
insert into customers values(2,"guohui","22");
insert into customers values(3,"zhangheng","27");
```

5）备份数据库

```
linux系统
mysqldump -uroot -p -B -F -R -x --master-data=2 test|gzip >/opt/backup/test_$(date +%F).sql.gz
注：执行时并伴随着linux的shell命令

windows系统

mysqldump -uroot -p -B -F -R -x --master-data=2 test > ../backup/test_0331.sql
```

参数信息

```
-B：指定数据库
-F：刷新日志，会重新创建一个binlog日志文件
-R：备份存储过程等
-x：锁表
--master-data：在备份语句里添加CHANGE MASTER语句以及binlog文件及位置点信息
```

问题：--master-data参数的作用？

会在备份的sql文件中添加信息：

```
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=120;
```

注：实际运行时将-F参数去掉了，因此生成的log日志信息为mysql-bin.000001

6）再次添加数据

```
insert into customers values(4,"liupeng","21");
insert into customers values(5,"xiaoda","31");
insert into customers values(6,"fuaiai","26");
```

7）模拟误删除操作

```
drop database test;
```

8）恢复数据

通过查看备份sql中的CHANGE MASTER信息确定出备份的待还原的position信息

```
linux命令：
grep CHANGE xxxback.sql 

windows中查询test_0331.sql中该字段的信息
```

内容为：

```
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=120;
```

查看bin-log信息

```
mysqlbinlog --no-defaults ../log/mysql-bin.000001
```

![](/images/database/mysql/bin-log-recovery3.png)

恢复数据

```
从bin-log中导出sql语句
mysqlbinlog --start-position=120 --stop-position=808 -d test ../log/mysql-bin.000001 > ../backup/001bin.sql

从备份文件中先恢复数据库
mysql -uroot -p < ../backup/test_0331.sql

执行查询语句，发现只有3条数据，也就是备份前的数据，备份后的插入数据没有
select * from customers;

执行bin-log中导出的sql，更新备份后的数据
mysql -uroot -p test < ../backup/001bin.sql

再次执行查询可以完整的看到6条数据。
```

在执行bin-log时应先将bin-log移动到其他位置，然后再使用恢复命令。

8）补充

这里在到处备份后的sql语句时是有点问题的，理论上来将会将备份后的所有操作都导出来，然后删掉误操作的语句。保证剩余的语句都能正确执行。因为，这里在drop database后没有再进行其他操作（实际工作中可能还会进行其他操作），不要忘了把其他操作执行的sql语句也更新到库中，否则就会造成数据库数据丢失问题。

注：恢复过程中应先禁用mysql的访问（可以通过控制远程连接的方式禁用访问）



