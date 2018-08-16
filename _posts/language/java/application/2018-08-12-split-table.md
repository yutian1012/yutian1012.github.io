---
title: 分库分表
tags: [java]
---

1）数据库读写分离

为什么要做读写分离，主要目的是为了减少主库的负载（减少主库的读压力）。从库用来读取，主库用来写入。

2）分库分表

如果多个应用都访问一个主库（访问量大），并且主库的数据量也比较大时，为了防止主库宕机，需要进行分库分表操作。

3）业务分库

将一个库分成：交易，商品，和用户等库。另外，交易数据库业务数据量一般是比较大的，随着业务的增长，可能需要对这部分数据进行分表操作。

4）分表类型

垂直分表：

users:id\userid\sex\password\adr\tel\code\age...

将users表将字段拆分成2个表user-base和user-info表

user-base:id\userid\password

user-info:id\userid\sex\adr\tel\code\age...

水平分表：

表结构不变，将表的数据按照某种规则存储到不同的数据表中。

策略：

a. hash，通过一个字段进行hash，定位到不同的表

b. range范围，如将不同日期的数据放置到不同的数据表中

c. list，预定义集合

5）hash实践分库分表

创建数据库

```
create database tl01;
create database tl02;
create database tl03;
create database tl04;
create database tl05;
create database tl06;
```

创建User表（每个库中都要相应的表）

```
create table user(
    id bigint(20) not null auto_increment,
    username varchar(255) default null,
    age int(11) default null,
    phone bigint(20) default null,
    email varchar(255) default null,
    primary key(id)
) engine=InnodDB Default charset=utf8;
```

创建类User，映射数据库字段，并使用mybatis创建相应的map文件。

在properties文件中创建相应数据库的连接

```
jdbc1.url=jdbc:mysql://localhost:3306/tl01?characterEncoding=UTF-8
jdbc1.username=root
jdbc1.password=root

jdbc2.url=jdbc:mysql://localhost:3306/tl02?characterEncoding=UTF-8
...

```

在spring的配置文件中定义相应的数据源

```
<bean id="datasource1" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    ...
</bean>

<bean id="datasource2" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    ...
</bean>

...

```

创建HashFunction类，用来进行hash路由，即根据数据的键找到存放的库位置。

```
pulic class HashFunction{
    public int user(String username){
        int result=Math.abs(username.hashCode()%1024);//防止小于0
        if(0<=result&&result<=255){
            return 1;
        }else if(256<=result&&result<=511){
            ...
        }
    }
}
```

6）存在问题

分布式全局唯一性ID，跨库事务的问题，sql改写（查询）。

如果要查看当前的订单，需要查询多个数据库才能得到

数据异构问题。