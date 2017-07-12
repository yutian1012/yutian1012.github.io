---
title: 执行大量sql插入操作后服务不再响应
tags: [work]
---

web开发的模块中有一个导入模块，该导入模块可能会批量导入上万条数据。在执行过程中经常出现系统不再响应的问题。

### 问题排除过程

1）判断tomcat是否还能够访问

系统部署到了tomcat服务器，当应用不再响应时，访问tomcat下的其他项目，发现能够正常访问，说明tomcat服务可以正常接收请求。

```
http://localhost:8080
```

如：访问tomcat的ROOT应用（tomcat本身提供的），能够正常访问。

2）判断应用是否还能继续处理

由于应用使用了spring-security作为安全处理框架，因此，在访问时进行一系列的认证和授权操作。发现访问功能页面不能正常方法（包括登录页面）。

访问应用的静态文件，如js等文件，而这些文件也没有被spring-security框架进行拦截，js文件能够正常访问

spring-security的配置文件，不对js等静态文件进行拦截配置

```
<security:http pattern="/js/**" security="none" />
```

说明应用本身还是在运行。

3）查看数据库表是否锁死，已经数据库连接是否用尽

整个导入伴随大量的sql插入操作，因此与数据库有很大的关系。

第一步：mysql连接

```
mysql -uroot -p

show processlist;//查看处理的mysql进程的state是否有lock的情况
```

![](/images/work/problem/showprocesslist.png)

注：state状态列没有出现lock等字样的行，并且可以看到应用的数据库连接（应用连接池中存在的10个连接）与数据库服务器端的进程对应关系。

第二步：向批量导入的表中手动插入一条数据，查看是否表被锁住

批量操作过程，会向A表中不断插入数据，因此A表可能会被锁住，造成系统不响应。

```
insert into A (id) values(1);

delete from A where id=1;
```

注：测试能够插入，说明该表没有被锁住。

第三步：查看数据库连接

```
show VARIABLES like '%connect%';//能够看到数据库的最大连接--max_connections

show status like '%thread%';//查看到当前的连接数--Threads_connected-47

show full processlist;//根据返回的记录数量也能得出当前的连接数-47
```

4）应用与数据库连接的模块（数据库连接池）

经过上面的问题排除，感觉问题很可能出现在数据库连接池上。

系统中配置了数据源状态查询监控工具Druid，具体配置不在详细描述（web.xml）

```
http://localhost:8080/xxx/druid/index.html--查看web.xml中配置的servlet
```

