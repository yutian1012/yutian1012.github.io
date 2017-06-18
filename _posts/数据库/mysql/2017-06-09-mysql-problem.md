---
title: mysql遇到的问题
tags: [database]
---

### msvcp100.dll丢失

参考：http://jingyan.baidu.com/article/fea4511a037e69f7bb9125f8.html

1）在安装完成mysql后，尝试启动mysql时报错。

![](/images/database/mysql/msvcp100.png)

注：原因是缺少c++的运行库

2）安装C++运行库

首先：打开浏览器，在地址栏输入"www.microsoft.com/zh-cn"回车打开微软中国官方网站，并在搜索框中输入VC++。

![](/images/database/mysql/vc++lib.png)

3）下载并安装软件

![](/images/database/mysql/vc++software.png)

注：使用户能够在未安装 Visual C++ 2010 的计算机上运行使用 Visual C++ 开发的应用程序。

4）再次运行启动mysql，可以成功启动。

### Cannot create PoolableConnectionFactory

发生场景：在为用户安装测试环境时，配置完成系统后，启动并访问系统。此时出现了访问系统有时可以正常访问，有时不能正常访问的情况。

错误信息：

```
Caused by: org.apache.commons.dbcp.SQLNestedException: Cannot create PoolableConnectionFactory (Communications link failure The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.)
    at org.apache.commons.dbcp.BasicDataSource.createPoolableConnectionFactory(BasicDataSource.java:1549)
    at org.apache.commons.dbcp.BasicDataSource.createDataSource(BasicDataSource.java:1388)
    at org.apache.commons.dbcp.BasicDataSource.getConnection(BasicDataSource.java:1044)
    at org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource.getConnection(AbstractRoutingDataSource.java:148)
    at org.springframework.jdbc.datasource.DataSourceTransactionManager.doBegin(DataSourceTransactionManager.java:202)
```

原因是多数据源配置问题，系统中mysql的datasource配置了动态数据源，而多个数据源可以实现在系统中配置并初始化（多个数据源的配置信息存放在mysql数据库中）。

```
<!-- 主数据源/默认数据源 -->
<bean id="dataSource_Default" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
  <!-- 基本属性 url、user、password -->
  <property name="url" value="${jdbc.url}" />
  <property name="username" value="${jdbc.username}" />
  <property name="password" value="${jdbc.password}" />

  <!-- 配置初始化大小、最小、最大 -->
  <property name="initialSize" value="${db.minimumConnectionCount}" />
  <property name="minIdle" value="${db.minimumConnectionCount}" /> 
  <property name="maxActive" value="${db.maximumConnectionCount}" />

  <!-- 配置获取连接等待超时的时间 -->
  <property name="maxWait" value="60000" />

  <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
  <property name="timeBetweenEvictionRunsMillis" value="60000" />

  <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
  <property name="minEvictableIdleTimeMillis" value="300000" />

  <property name="validationQuery" value="select * from ACT_GE_PROPERTY" />
  <property name="testWhileIdle" value="true" />
  <property name="testOnBorrow" value="false" />
  <property name="testOnReturn" value="false" />

  <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
  <property name="poolPreparedStatements" value="true" />
  <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />

  <!-- 配置监控统计拦截的filters -->
  <property name="filters" value="stat" /> 
  <!-- 连接泄漏监测 -->
  <property name="removeAbandoned" value="true" /> <!-- 打开removeAbandoned功能 -->
  <property name="removeAbandonedTimeout" value="28800" /> <!-- 28800秒，也就是8小时，一个连接超过8小时会自动删除这个连接 -->
  <property name="logAbandoned" value="true" /> <!-- 关闭abanded连接时输出错误日志 -->
  
</bean>

<!-- 数据源导入拦截bean，实现接口ApplicationListener<ContextRefreshedEvent>动态监听上下文刷新时间，并向动态数据源中配置数据源信息-->
<bean id="dataSourceInitListener" class="com.hotent.platform.event.listener.DataSourceInitListener"></bean>

<!-- 动态数据源实现类，该类继承了AbstractRoutingDataSource，管理并配置多个数据源-->
<bean id="dataSource" class="com.hotent.core.db.datasource.DynamicDataSource">
    <property name="targetDataSources"  >
        <map>
            <entry key="dataSource_Default" value-ref="dataSource_Default" />
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="dataSource_Default" />
</bean>

<!-- 事务中的数据源配置，使用了动态数据源获取最终数据源 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

修改，找到数据库中配置数据源的表，更改数据源的配置信息（表sys_data_source）。

### bin-log日志问题

```
Error updating database.  Cause: java.sql.SQLException: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.
```

如果开启了bin-log日志，配置Read Uncommited或者Read Commited会报错。

原因：MySQL默认的binlog_format是STATEMENT，而在READ COMMITTED或READ UNCOMMITTED隔离级别下，innodb只能使用的binlog_format是ROW。

修改my.ini文件

```
binlog_format=mixed
```

或者会话中修改

```
SET GLOBAL binlog_format=MIXED
```

注：当前会话中有效

参考：http://blog.csdn.net/javadhh/article/details/70241700

### 数据库编码字符集设置

1）查看字符集

```
show variables like '%colla%';
```

![](/images/database/mysql/mysql-character-collation.png)

2）查看编码

```
show variables like '%character%';
```

![](/images/database/mysql/mysql-character-show.png)

3）my.ini配置文件内容

```
[mysql]

default_character_set=utf8

[mysqld]

character_set_server=utf8
default-storage-engine=INNODB
max_connections=300

innodb_buffer_pool_size = 128M

basedir = D:\Program Files\mysql-5.6.23-winx64
datadir = D:\Program Files\mysql-5.6.23-winx64\data
port = 3306

binlog_format=mixed

log_bin=D:\Program Files\mysql-5.6.23-winx64\log\mysql-bin
```

4）系统部署字符编码修改问题

一次将系统部署到用户本地，在部署时遇到了字符编码怎么都改不过来，原因却是非常无语。用户本地的系统默认不显示文件的扩展名，而在创建mysql配置文件时，手动创建了my.ini文件，启动mysql后并不会读取该文件。因为该文件实际上的文件全称为my.ini.ini。因此，在实际部署的时候，要确认用户系统的环境，问题可能不是技术的问题，而仅仅是细节的问题。