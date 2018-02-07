---
title: 开源spagobi--quick start
tags: [spagobi]
---

参考：http://wiki.spagobi.org/xwiki/bin/view/Main/QuickStart

### 1. spagobi模块

![](/images/open/spagobi/wiki/SpagoBI-modules.png)

1）server

使用一个wb应用，可以运行在tomcat，jboss等应用服务器上；需要数据库的支持（如mysql数据库，下载mysql-dbscripts-5.2.0_24032016.zip并创建库）；

2）Meta

是一个eclipse插件，用于管理元数据。

3）Studio

是一个eclipse插件，用于开发者创建分析文档（chart, report, dashboard）。

4）SDK

实现server与其他应用交换的api（collection of web services, tags and JavaScript API），同时实现外部工具与server的交互等工作。

### 2. 安装 spagoBI server

1）数据库配置

修改tomcat/conf下的server.xml文件；

JDBC driver放置到tomcat/lib下；

修改目录下的webapps\SpagoBI\WEB-INF\classeshibernate.cfg.xml, quartz.properties and jbpm.cfg.xml文件。

2）数据仓库资源配置

与数据库配置相似，以All-In-One-SpagoBI-5.2.0_11042016.zip为例

修改tomcat/conf下的server.xml文件；

```
<Resource name="jdbc/foodmart" auth="Container"
    type="javax.sql.DataSource" driverClassName="org.hsqldb.jdbcDriver"
    url="jdbc:hsqldb:hsql://localhost:9001/foodmart"
    username="sa" password="" maxActive="20" maxIdle="10"
    maxWait="-1" validationQuery="select 1 from INFORMATION_SCHEMA.SYSTEM_USERS"
    removeAbandoned="true" removeAbandonedTimeout="3600"/>
         
<Resource name="jdbc/bam" auth="Container"
    type="javax.sql.DataSource" driverClassName="org.hsqldb.jdbcDriver"
    url="jdbc:hsqldb:file:${catalina.base}/database/bam"
    username="sa" password="" maxActive="20" maxIdle="10"
    maxWait="-1"/>
```

JDBC driver放置到tomcat/lib下；

修改所有的应用上下文的META-INF\context.xml文件，添加配置的jndi资源信息。

### 3. spagobi meta使用

下载提供的压缩包SpagoBIMeta_5.2.0_win64_12042016.zip，解压后运行SpagoBI.exe文件启动。

1）创建business model

运行D:\All-In-One-SpagoBI-5.2.0\database下的MetaData数据库

```
java -cp ./hsqldb1_8_0_2.jar org.hsqldb.Server -database.0 ./MetaData -dbname.0 metadata
```

建立连接，并配置相应的连接属性，测试连接。

![](/images/open/spagobi/wiki/meta_connection.png)

创建business model（SpagoBI视图下点击工具栏的spagobi按钮），设置项目名称为test

![](/images/open/spagobi/wiki/meta_bussiness_model.png)

新建model，右击test下的bussiness models-- new model

![](/images/open/spagobi/wiki/meta_new_model.png)

选择待创建business model的物理表

![](/images/open/spagobi/wiki/meta_new_model_selecttable.png)

设置查询字段，查询条件和分组，通过下方的edit/result标签页切换查看查询结果

![](/images/open/spagobi/wiki/meta_new_query.png)

2）发布查询和模型到server

