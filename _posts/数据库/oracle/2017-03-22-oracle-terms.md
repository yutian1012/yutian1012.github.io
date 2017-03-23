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

参考：http://blog.csdn.net/dreamllover/article/details/53483129
参考2：http://www.jb51.net/article/32017.htm
参考3：http://blog.itpub.net/17203031/viewspace-682003/


http://blog.csdn.net/sunboy8764/article/details/7343225

http://blog.sina.com.cn/s/blog_9f607ab70100yxyu.html