---
title: 开源spagobi--hsqldb的数据库信息
tags: [spagobi]
---

All-In-One-SpagoBI压缩包中测试数据使用hsqldb数据库

### 1. 运行hsqldb服务器（脚本startspagobi.bat）

批处理文件startspagobi.bat

在G:\spagoBi\All-In-One-SpagoBI-5.2.0_11042016\All-In-One-SpagoBI-5.2.0\database目录下存在startspagobi.bat文件和databaseManager.bat是分别运行hsqldb服务器和客户端的批处理。

startspagobi.bat文件内容：

```
java -cp ./hsqldb1_8_0_2.jar org.hsqldb.Server -database.0 ./spagobi -dbname.0 spagobi
```

注：命令末尾的spagobi是用于数据库连接的数据库名

![](/images/open/spagobi/spagobi-hsqldb-file.png)

注：.script用来保存最终数据，hsqldb正常关闭的时候会把内存和日志文件（file.log）中的数据写入file.script。

双击运行startspagobi.bat启动hsqldb服务器

### 2. 运行hsqldb客户端（脚本databaseManager.bat）

批处理文件databaseManager.bat

文件内容

```
java -cp ../lib/hsqldb1_8_0_2.jar org.hsqldb.util.DatabaseManager
```

双击运行databaseManager.bat启动hsqldb客户端，客户端连接工具如下图

![](/images/open/spagobi/spagobi-hsqldb-connection.png)

连接url：

```
jdbc:hsqldb:hsql://localhost/spagobi
```

注：默认连接的端口号是9001
