---
title: oracle-terms
tags: [database]
---

### 1. 逻辑结构

逻辑存储结构主要描述Oracle数据库的内部存储结构，即从技术概念上描述在Oracle数据库种如何组织、管理数据。

![](/images/database/oracle/logicstruct.jpg)

从逻辑上来看. 数据库是由一个或者多个表空间等组成。一个表空间(tablespace)由一组段组成,一个段(segment)由一组区组成,一个区(extent)由一批数据库块组成,一个数据库块(block)对应一个或多个物理块

注：表空间是最大的逻辑单位，块是最小的逻辑单位。

### 2. 存储结构

物理存储结构主要描述Oracle数据库的外部存储结构,即在操作系统种如何组织、管理数据，数据库由控制文件、数据文件、重做日志文件和参数文件等操作系统文件组成。

### 3. 数据库块（block）

1）数据库块（Database Block）

是数据库使用的I/O最小单元，又称逻辑块或ORACLE块。一个数据库块对应一个或多个物理块，块的大小由参数DB_BLOCK_SIZE确定。

2）块的大小是操作系统块大小的整数倍.

操作系统块(OS block)的大小为4kb，所以Oracle Block的大小可以是4kb,8kb,16kb等等。

3）数据读取

如果块的大小为4kb，某表每行的数据是100 bytes.，如果某查询语句只返回1行数据,那么，在将数据读入到数据高速缓存时,读取的数据量是4kb而不是100 bytes。

4）数据库的组成

在Oracle中，不论数据块中存储的是表（table）、索引（index）或簇表（clustered data），其内部结构都是类似的。

![](/images/database/oracle/databaseblock.jpg)

包括：数据块头（包括标准内容和可变内容）（common and variable header），表目录区（table directory），行目录区（row directory），可用空间区（free space），行数据区（row data）。

数据块头：包含了此数据块的概要信息，例如块地址（操作系统块地址）及此数据块所属的段（segment）的类型（例如，表或索引）。

表目录区：如果一个数据表在此数据块中储存了数据行，那么数据表的信息将被记录在数据块的表目录区（table directory）中，即记录表信息。

行目录区：理解为行在块中的偏移量。此区域包含数据块中存储的数据行的信息（每个数据行片断（row piece，数据块只包含了行的一部分，行内容大） 在行数据区（row data area）中的地址）。

数据区：包含了表或索引的实际数据。一个数据行可以跨多个数据块，一个数据块也可能包含多行数据。

可用空间区：插入新数据行，或在更新数据行需要更多空间时使用该空间。

注：PCTFREE 参数用来设置一个数据块（data block）中至少需要保留（reserve）多少可用空间（百分比值），为数据块中已有数据更新时可能发生的数据量增长做准备。可以在create table时指定。

参考：http://www.jb51.net/article/31929.htm

### 4. 区（extent）

区(Extent)是数据库存储空间分配的逻辑单位，一个区由一组数据库块组成，区是由段分配的，分配的第一个区称初始区，以后分配的区称增量区。

区由物理上连续的多个块组成的。

参考：http://blog.csdn.net/dreamllover/article/details/53483129
参考2：http://www.jb51.net/article/32017.htm
参考3：http://blog.itpub.net/17203031/viewspace-682003/

### 5. 段（segment）

一个表就是一个段，如果表分区了，那么每个分区是一个段。索引也是独立的段，索引分区了，每个分区也是一个独立的段。临时段，临时表空间中存在。undo段（回滚段），在undo表空间中。lobl段（二进制大空间段）。

oracle分配空间时，是以区为分配单位的，将区分给段。

### 6. 表空间

mysql是多库结构，可以建立多个库来放置不同应用数据。而oracle是通过表空间的概念来实现的，通过不同的表空间，放置不同应用数据来方便管理。

表空间是ORACLE数据库恢复的最小单位,容纳着许多数据库实体,如表、视图、索引、聚簇、回退段和临时段等。

1）表空间

表空间由一个或多个文件组成。将表创建在表空间上，就会存储子该表空间的文件上。

表空间容量就是数据文件的容量总和，数据文件最大2G，一个8G容量的表如何存储在表空间中。在表空间中可以创建多个文件来容纳8G的数据表，而一个文件却不能容纳8G的数据量。

注：实现在表和数据文件直接建立一个中间层，

```
select * from v$tablespace;
```

2）表空间分类

系统级别的表空间包括：SYSTEM,SYSAUX,TEMP,UNDO

3）表空间对应的数据文件

```
select file_name,tablespace_name from dba_data_files;
```

### 7. 总结

![](/images/database/oracle/construct.png)

一个数据库由多个表空间组成（SYSTEM和SYSAUX必须）。一个表空间中由多个段组成（多个表），由多个数据文件存放数据；一个段由多个区组成；一个区由多个连续的oracle块组成；一个块由多个操作系统块组成。其中，数据文件由多个操作系统块组成。

### 8. 方案（schema）

1）schema方案概念

一个schema可以理解为一个用户。

Oracle数据库中不能新创建一个schema，要想创建一个schema，只能通过创建一个用户的方法解决，在创建一个用户的同时为这个用户创建一个与用户名同名的schem并作为该用户的缺省shcema。

一个用户有一个缺省的schema，其schema名就等于用户名，当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。

2）schema与表空间的区别和联系

Schema和表空间是一个层次的概念，他们都有一个很重要的特性，就是对表的独占性。Schema是表的逻辑集合，是所有应用访问表必须指定的对象（虽然一般大家都省略了，但是实际上一定是db.schema.table这种访问模式），同一张表不可能既属于这个Schema，又属于另一个Schema。表空间是表的物理集合，是所有磁盘读写必须访问的文件（大家一般也不用太管，主要是Oracle管，个性化的需求DBA管），同一张表也不可能既放在这个表空间，又放在那个表空间。

注：schema是应用程序使用时的逻辑对象。

### 9. oracle数据如何定位存储

vdisplay -v /dev/vgepm/lvdat1_32G（数据文件）


http://blog.csdn.net/sunboy8764/article/details/7343225

http://blog.sina.com.cn/s/blog_9f607ab70100yxyu.html