---
title: bi中多维数据模型
tags: [BI]
---

参考：https://wenku.baidu.com/view/723a6cb50b4e767f5bcfce1e.html

多维数据模型将数据看作数据立方体形式，满足用户从多角度多层次进行数据查询和分析的需要而建立起来的基于事实（度量）和维的数据库模型。其数据组织采用多维结构文件进行数据存储，并有索引及相应的元数据管理文件与数据相对应。

### 多维数据模型的相关概念

1）粒度（Granularity）

粒度是指数据仓库中数据单元的详细程度和级别。数据越详细，级别就越低，粒度就越小。数据综合度越高，级别就越高，粒度就越大。例如，地址数据中"北京市"，比"北京市海淀区"的粒度小。因为"北京市海淀区"数据综合度高。可以将粒度小的数据拆分成粒度大的数据，如将"北京市海淀区"拆分成"北京市"和"海淀区"。

2）维度（Dimension）

维度（简称维）是指人们观察事物的特定的角度，概念上类似于关系表的属性。如企业常常关系产品销售数据随着时间推移而变化的请况，这是从时间的角度来观察产品的销售，即时间维。

3）维属性和维成员

参考：http://blog.csdn.net/kk185800961/article/details/8456347

维属性：一个维是通过一组属性来描述的，如时间维包含年份、季度、月份和日期等属性。这里的年份、季度等称为时间维的维属性。

维成员：维的一个取值称为该维的一个维成员（维的一个记录值）。如果一个维是多层次的，那么该维的维成员是在不同维层次的取值组合。如一个时间维具有年份、季度、月份、日期四个层次。分别在四个层次各取一个值，就得到时间维的一个维成员（即某年某季度某月某日）。

4）维层次

同一维度可以存在细节程度不同的各个值，可以将粒度大的值映射到粒度小的值，这样构成维层次或概念分层。如时间维，可以从年、季度、月份、日期来描述，那么"年度--季度--月份--日期"就是维层次。