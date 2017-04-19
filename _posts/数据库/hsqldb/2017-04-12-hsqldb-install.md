---
title: hsqldb数据库安装与使用
tags: [hsqldb]
---

参考：http://blog.csdn.net/luxideyao/article/details/19834959

参考：http://www.myexception.cn/database/619858.html

### 1. 安装

下载地址：https://sourceforge.net/projects/hsqldb/files/

下载完成后解压到D:\work\installer目录下，完成安装。

注：解压后即完成了安装。

### 2. 运行

1）启动方式一，通过java命令启动

```
java -classpath lib/hsqldb.jar org.hsqldb.server.Server -database.0 data/testdb -dbname.0 testdbName
```

![](/images/database/hsqldb/hsqldb-run-server.png)

运行后会在data目录下生成相应的文件和文件夹

```
testdb.lck--标识数据库锁状态，即用来标记当前数据库是否已经被某一个hsqldb访问了，同一时间只有一个hsqldb能操作数据库文件，这样才能保证不会出现数据冲突。这个文件在hsqldb启动时自动生成，正常关闭时会自动删除。

testdb.log--运行数据库产生的log信息，它将记录每一个运行和用户操作环节。因为用到了缓存，更新的数据不会直接写入文件，而是在内存中积累一定量后，才会批量写入file.log这个日志文件。

testdb.properties--存放数据库配置，包括数据库版本，缓存，表结构设置等等

testdb.script--用来保存最终数据，hsqldb正常关闭的时候会把内存和日志文件（file.log）中的数据写入file.script，并删除日志（file.log）文件。

testdb.tmp 文件夹
```

注：默认的端口是9001

2）启动管理器

```
java -classpath lib\hsqldb.jar org.hsqldb.util.DatabaseManager
```

![](/images/database/hsqldb/hsqldb-run-manager.png)

注：type类型选择HSQL Database Engine Server，用户名为SA，密码空即可。

注2：url设置为jdbc:hsqldb:hsql://localhost/testdbName

### 3. 使用管理器测试sql

1）创建表

```
CREATE TABLE TBL_USERS(
         ID INTEGER NOT NULL PRIMARY KEY,
         FIRST_NAME VARCHAR(20),
         LAST_NAME VARCHAR (30),
         LOGIN_DATE DATE
 )
```

![](/images/database/hsqldb/hsqldb-run-sql.png)

粘贴建表语句并执行execute按钮执行sql。

2）查看创建的表

选择菜单栏中的 View -> Refresh Tree 命令，左侧栏中将显示创建的 TBL_USERS

查看testdb.log文件，就可以看到执行的sql语句被记录到了日志文件中。

3）插入数据

```
insert into tbl_users values(1,'zhang','san','2017-04-12');
```

```
select * from tbl_users;
```

再次查看日志文件，发现存在insert语句，但不存在select语句。

### 4. DatabaseManagerSwing工具管理

```
java -classpath lib\hsqldb.jar org.hsqldb.util.DatabaseManagerSwing
```

![](/images/database/hsqldb/hsqldb-run-managerswing.png)

注：操作和执行与DatabaseManager相同