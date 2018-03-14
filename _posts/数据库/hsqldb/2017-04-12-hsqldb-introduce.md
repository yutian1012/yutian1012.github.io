---
title: hsqldb数据库介绍
tags: [hsqldb]
---

HSQLDB（HyperSQL DataBase）是一个开放源代码的JAVA数据库，其具有标准的SQL语法和JAVA接口。

hsqldb的默认用户是sa密码为空。

### 运行模式

在HSQLDB中，有三种比较常用模式：服务器模式，In-Process模式，数据库。

1）服务器模式

应用程序（客户端）通过Hsqldb的JDBC驱动连接服务器。在服务器模式中，服务器在运行的时候可以被指定为最多10个数据库。

根据客户端和服务器之间通信协议的不同，Server模式可以分为以下三种：

a.Hsqldb Server

这种模式是首选的也是最快的。它采用HSQLDB专有的通信协议。启动服务器需要编写批处理命令。Hsqldb提供的所有工具都能以java class归档文件(也就是jar)的标准方式运行。

运行命令启动

```
java -cp ../lib/hsqldb.jar org.hsqldb.Server -database.0 mydb -dbname.0 demoDB
```

注：hsqldb.jar中提供了运行hsqldb的类。

问题：[-database.0 ]、 [dbname.0]为什么在后面加[0]

服务模式运行的时候可以指定10个数据库，如有多个数据库，则继续写命令行参数-database.1 aa -dbname.1 aa -database.2 bb-dbname.2 bb ... ...

注：启动服务器的命令启动了带有一个（默认为一个数据库）数据库的服务器，这个数据库是一个名为"mydb.*"文件，这些文件就是mydb.Properties、mydb.script、mydb.log等文件。其中demoDB是mydb的别名，可在连接数据库时使用。

b.Hsqldb Web Server

这种模式只能用在通过HTTP协议访问数据库服务器主机，采用这种模式唯一的原因是客户端或服务器端的防火墙对数据库对网络连接强加了限制（这种模式不推荐被使用）。

```
java -cp ../lib/hsqldb.jar org.hsqldb.WebServer -database.0 mydb -dbname.0 demoDB
```

c.Hsqldb Servlet

这种模式和Web Server一样都采用HTTP协议，当如Tomcat或Resin等servlet引擎（或应用服务器）提供数据库的访问时，可以使用这种模式。但是Servlet模式不能脱离servlet引擎独立启动。为了提供数据库的连接，必须将HSQLDB.jar中的hsqlServlet类放置在应用服务器的相应位置。

Servlet模式只能启动一个单独的数据库

注：Web Server和Servlet模式都只能在客户端通过JDBC驱动来访问

2）In-Process模式

In-Process模式又称Standalone模式。这种模式下，数据库引擎作为应用程序的一部分在同一个JVM中运行。对于一些应用程序来说，这种模式因为数据不用转换和通过网络的传送而使得速度更快一些。

主要的缺点就是默认的不能从应用程序外连接到数据库。所以当应用程序正在运行的时候，你不能使用类似于Database Manager的外部工具来查看数据库的内容。

在1.8.0版本中，你可以从同一个JVM的一个线程里面来运行一个服务器实例，从而可以提供外部连接来访问你的In-Process数据库。

注：推荐的使用In-Process模式方式是：开发的时候为数据库使用一个HSQLDB服务器实例，然后在部属的时候转换到In-Process内模式。

3）数据库模式（内存模式memory-only）

Memory-Only数据库不是持久化的而是全部在随机访问的内存中。因为没有任何信息写在磁盘上。这种模式通过mem:协议的方式来指定。

注意事项：当一个服务器实例启动或者建立一个in-process数据库连接的时候，如果指定的路径没有数据库存在，那么就会创建一个新的空的数据库。这个特点的副作用就是让那些新用户产生疑惑。在指定连接已存在的数据库路径的时候，如果出现了什么错误的话，就会建立一个指向新数据库的连接。为了解决这个问题，你可以指定一个连接属性ifexists=true只允许和已存在的数据库建立连接而避免创建新的数据库，如果数据库不存在的话，getConnection()方法将会抛出异常。

### 运行

内存（Memory-Only）模式和进行（In-Process）模式都是通过JDBC连接实现的，不能启动单独的数据库服务，即不能被外部管理工具查看。

服务器模式通过java命令启动，是以守护进程的方式启动的，因此可以被管理工具查看和使用。