---
title: DataSource的种类
tags: [datasource]
---

参考：http://www.cnblogs.com/zy19930408/p/4883799.html

1）方式一：直接使用Mysql提供的jar包访问数据库

在lib下引入mysql-connector-java-5.1.27.jar

2）方式二：使用数据库连接池--dbcp

在lib下引入commons-dbcp.jar和mysql-connector-java-5.1.27.jar文件

补充：commons-dbcp.jar与commons-pool.jar的关系？

3）方式三：使用数据库连接池--c3p0

在lib下引入c3p0.jar和mysql-connector-java-5.1.27.jar文件

补充：c3p0.jar与mchange-commons.jar的关系？

4）方式四：使用数据库连接池--druid

在lib下引入druid.jar和mysql-connector-java-5.1.27.jar文件

5）方式五：jndi方式访问数据库