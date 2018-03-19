---
title: saiku迁移到mysql数据库下
tags: [BI]
---

参考：https://www.cnblogs.com/Jason-Xiang/p/6360065.html

1）mysql中创建saiku数据库

```
CREATE DATABASE IF NOT EXISTS saiku DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

2）修改web.xml中的配置

修改名为db.url,db.user,db.password的初始化数据，该配置信息会应用到org.saiku.database.Database类的init方法中用来初始化数据库。

```
<!-- 配置mysql的访问地址 -->
<context-param>
    <param-name>db.url</param-name>
    <param-value>jdbc:mysql://localhost:3306/saiku</param-value>
</context-param>
<!-- 配置访问mysql的用户 -->
<context-param>
    <param-name>db.user</param-name>
    <param-value>sa</param-value>
</context-param>
<!-- 配置访问mysql用户的密码 -->
<context-param>
    <param-name>db.password</param-name>
    <param-value>root</param-value>
</context-param>
```

3）修改saiku-bean.xml

注释h2database，添加mysqlDatabase

```
<!--
<bean id="h2database" class="org.saiku.database.Database" init-method="init">
    <property name="datasourceManager" ref="repositoryDsManager"/>
</bean>
-->
<bean id="mysqlDatabase" class="org.saiku.database.Database" init-method="init">
    <property name="datasourceManager" ref="repositoryDsManager"/>
</bean>

<!-- 修改licenseBean中的databaseManager属性的引用 -->
<bean id="licenseBean" class="org.saiku.web.rest.resources.License">
    <property name="databaseManager" ref="mysqlDatabase"/>
    <property name="licenseUtils" ref="licenseUtils"/>
    <property name="userService" ref="userServiceBean"/>
</bean>
```

修改属性文件的加载配置文件

```
<!-- 加载saiku-beans.properties文件，而不是saiku-beans-test.properties -->
<context:property-placeholder
    location="/WEB-INF/saiku-beans.properties"
    ignore-resource-not-found="true"
    ignore-unresolvable="true"
    order ="0" />
```

4）修改saiku-beans.properties文件

修改userdao.driverclass，userdao.url，userdao.username，userdao.password的值

```
userdao.driverclass=com.mysql.jdbc.Driver
userdao.url=jdbc:mysql://localhost:3306/saiku
userdao.username=root
userdao.password=root
```

5）修改applicationContext-spring-security-jdbc.properties文件

修改jdbcauth.driver，jdbcauth.url，jdbcauth.username，jdbcauth.password的值

```
jdbcauth.driver=com.mysql.jdbc.Driver
jdbcauth.url=jdbc:mysql://localhost:3306/saiku
jdbcauth.username=root
jdbcauth.password=root
```

6）修改类Database

类文件：org.saiku.database.Database，修改init方法，注释掉loadFoodmart和loadEarthquakes方法，并测试运行。

错误信息

```
Caused by: java.sql.SQLException: No suitable driver found for jdbc:mysql://localhost:3306/saiku
```

主要原因是使用的DataSource类是h2数据库的，因此不能加载到相应的驱动。

解决方案：

可简单将DataSource类的JdbcDataSource切换成BasicDataSource。然后修改不兼容的sql，sql修改可根据错误提示进行进一步修改。

```
//private JdbcDataSource ds;
private BasicDataSource ds;
```

7）创建一个Mysql数据源的DataSource

由于上面的问题，DataSource，建表语句等都存在问题，要兼容mysql数据库。可重写一个针对Mysql数据库的初始化类，来重新加载并配置数据，实现mysql数据库的迁移。

注：系统提供的DataSource类当作是h2数据库的配置类，仿造该类实现一个Mysql的DataSource类。