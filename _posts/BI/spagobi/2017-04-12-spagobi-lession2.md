---
title: 开源spagobi--教程（二）
tags: [spagobi]
---

### 1. OLAP的处理过程

1）measure和dimesion

Online Analytical Processing (OLAP) enables one to analyze different dimensions of multidimensional data.  It enables one to analyze data from different perspectives.

OLAP能够从不同的维度，不同视角分析数据。

Consider sales data as an example. One might be interested in analyzing sales data in terms of the date when the sale occurred, the region the sales occurred, the store the sales occurred .  The sales amount we are analyzing is called a measure. The way we analyze the measure (sales amount)  is called a dimension. Therefore the sales date is one dimension of looking at the sales; the store where the sales occurred is another dimension of looking at the sales.  We can therefore look at the sales data by date, by store e.t.c

以销售数据为例，你可能对销售数据中的销售日期，销售地区，销售金额等感兴趣。销售金额作为销售的衡量字段（measure），销售日期，销售地区可以作为一个维度（dimension）来查看（筛选）和分析销售额数据。

2）fact table和dimetion table以及star schema

Fact table和dimension table这是数据仓库的两个概念，是数据仓库的两种类型表。从保存数据的角度来说，本质上没区别，都是表。

区别：

Fact表用来存fact数据，就是一些可以计量的数据和可加性数据，数据数量，金额等。

dimension table用来存描述性的数据，用来描述fact的数据，如区域，销售代表，产品等。

star schema 就是一个fact表有多个维表（dimension table）关联。 

3）实例

实例一：

统计运行系统出现异常时的内存使用情况，衡量字段（关注的点）是内存使用量，维度是出现异常的时间。即表的event_date和used_memory字段。

![](/images/open/spagobi/spagobi-example-olap-table.png)

注：这种方式dimetion data 存储在fact table中

实例二：

以电信公司为例，查看star schema结果，即fact table和dimetion table的设计

![](/images/open/spagobi/spagobi-examp-fact_table.png)

it is easy to answer questions like how many mobile phone subscribers were 
activated in the last quarter or how many subscribers are post paid or prepaid. 

### 2. OLAP Cube

把多维模型以三维的方式为代表进行展现和描述，一方面是出于更方便地解释和描述，同时也是给思维成像和想象的空间；另一方面是为了与传统关系型数据库的二维表区别开来，于是就有了数据立方体的叫法。

### 3. OLAP

参考：http://blog.csdn.net/mylifewang/article/details/7372449

SpagoBI 的OLAP使用的是开源项目mondrian，一个用Java写成的OLAP（在线分析性处理）引擎，它用MDX语言实现查询，从关系数据库(RDBMS)中读取数据，然后经过JPivot用多维的方式对结果进行展示。

要使用JPivot，书写SCHEMA是关键，这个是把关系表转换为维度表的配置文件，具体的写法可以参照mondrian的说明文档，写好SCHEMA文件，修改 webapps/SpagoBIJPivotEngine/WEB-INF/classes/engine-config.xml 文件，添加SCHEMA文件位置，然后重启tomacat服务。

再通过biadmin登录SpagoBI，打开Documents development 添加OLAP document，通过Template build 查看修改配置。





