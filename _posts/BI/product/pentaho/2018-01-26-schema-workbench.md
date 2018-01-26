---
title: Schema Workbench 开发mdx
tags: [BI]
---

参考1：https://www.cnblogs.com/liqiu/p/5202708.html
参考2：http://blog.csdn.net/neweastsun/article/details/43490663

1）工具下载地址

地址：https://community.hds.com/docs/DOC-1009931-downloads

pentaho提供了3种设计工具：Aggregation Designer，Schema Workbench和Metadata Editor。

下载压缩包，并解压widows下运行workbench.bat批处理文件启动该工具。

2）官方文档

参考：https://help.pentaho.com/Documentation/8.0/Products/Schema_Workbench

3）初始化数据

在mysql中创建数据库和表，并存入一些测试数据。

```
create database testmdx;
show databases;
use testmdx;

-- 执行sql语句
CREATE TABLE sale
(
  saleid integer NOT NULL,
  proid integer,
  cusid integer,
  unitprice double precision,
  num integer,
  CONSTRAINT sale_pkey PRIMARY KEY (saleid)
);

CREATE TABLE customer
(
  cusid integer NOT NULL,
  gender character(1),
  CONSTRAINT customer_pkey PRIMARY KEY (cusid)
);
CREATE TABLE product
(
  proid integer NOT NULL,
  protypeid integer,
  proname character varying(32),
  CONSTRAINT product_pkey PRIMARY KEY (proid)
);
CREATE TABLE producttype
(
  protypeid integer NOT NULL,
  protypename character varying(32),
  CONSTRAINT producttype_pkey PRIMARY KEY (protypeid)
);

-- 存入测试数据
insert into Customer(cusId,gender) values(1,'F');
insert into Customer(cusId,gender) values(2,'M');
insert into Customer(cusId,gender) values(3,'M');
insert into Customer(cusId,gender) values(4,'F');

insert into producttype(proTypeId,proTypeName)values(1,'电器');
insert into producttype(proTypeId,proTypeName)values(2,'数码');
insert into producttype(proTypeId,proTypeName)values(3,'家具');

insert into product(proId,proTypeId,proName)values(1,1,'洗衣机');
insert into product(proId,proTypeId,proName)values(2,1,'电视机');
insert into product(proId,proTypeId,proName)values(3,2,'mp3');
insert into product(proId,proTypeId,proName)values(4,2,'mp4');
insert into product(proId,proTypeId,proName) values(5,2,'数码相机');
insert into product(proId,proTypeId,proName)values(6,3,'椅子');
insert into product(proId,proTypeId,proName)values(7,3,'桌子');

insert into sale(saleId,proId,cusId,unitPrice,num)values(1,1,1,340.34,2);
insert into sale(saleId,proId,cusId,unitPrice,num)values(2,1,2,140.34,1);
insert into sale(saleId,proId,cusId,unitPrice,num)values(3,2,3,240.34,3);
insert into sale(saleId,proId,cusId,unitPrice,num)values(4,3,4,540.34,4);
insert into sale(saleId,proId,cusId,unitPrice,num)values(5,4,1,80.34,5);
insert into sale(saleId,proId,cusId,unitPrice,num)values(6,5,2,90.34,26);
insert into sale(saleId,proId,cusId,unitPrice,num)values(7,6,3,140.34,7);
insert into sale(saleId,proId,cusId,unitPrice,num)values(8,7,4,640.34,28);
insert into sale(saleId,proId,cusId,unitPrice,num)values(9,6,1,140.34,29);
insert into sale(saleId,proId,cusId,unitPrice,num)values(10,7,2,740.34,29);
insert into sale(saleId,proId,cusId,unitPrice,num)values(11,5,3,30.34,28);
insert into sale(saleId,proId,cusId,unitPrice,num)values(12,4,4,1240.34,72);
insert into sale(saleId,proId,cusId,unitPrice,num)values(13,3,1,314.34,27);
insert into sale(saleId,proId,cusId,unitPrice,num)values(14,3,2,45.34,27);
```

4）schema workbench中配置数据源

将mysql的连接驱动jar包放入schema-workbench\drivers下，重启工具。