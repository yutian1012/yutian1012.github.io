---
title: saiku在mysql中展示foodmart实例
tags: [BI]
---

foodmart数据库迁移

系统中只提供了h2的数据库建表文件，需要先生成相应的h2数据库，然后运行命令导出成csv格式的文件，然后再将csv格式的文件导入到mysql中实现数据库的迁移。

1）安装hsqldb数据库

下载安装的版本为hsqldb-2.4.0，在安装目录下启动hsqldb数据库

```
# 进入到saiku-server的测试数据库运行环境中
cd D:\work\installer\saiku-server\data
# 运行bin下的批处理文件，启动hsqldb数据库
java -classpath D:/work/installer/hsqldb-2.4.0/hsqldb/lib/hsqldb.jar org.hsqldb.server.Server -database.0 file:foodmart -dbname.0 foodmart


# 运行客户端连接服务器，默认连接端口是9001
%HSQLDB%\bin\runManagerSwing.bat
```

![](/images/BI/product/saiku/hsqldb-connection.png)