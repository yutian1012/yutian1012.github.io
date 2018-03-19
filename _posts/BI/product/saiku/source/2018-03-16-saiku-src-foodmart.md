---
title: saiku在mysql中展示foodmart实例
tags: [BI]
---

foodmart数据库迁移

系统中只提供了h2的数据库建表文件，需要先生成相应的h2数据库，然后运行命令导出成csv格式的文件，然后再将csv格式的文件导入到mysql中实现数据库的迁移。

1）安装h2数据库

安装完成h2数据库后，运行bin目录下的h2.bat文件，启动数据库。在web页面连接foodmart数据库。

saiku的foodmart数据库文件：D:\work\installer\saiku-server\data\foodmart.mv.db

![](/images/BI/product/saiku/h2-connection.png)

2）导出数据库文件

```
java -cp D:/work/installer/h2/bin/h2-1.4.197.jar org.h2.tools.Script -url jdbc:h2:D:/work/installer/saiku-server/data/foodmart -user sa -script test.zip -options compression zip
```

注：这种方法导出的sql文件也只能在h2数据库中使用，无法在mysql中直接导入。