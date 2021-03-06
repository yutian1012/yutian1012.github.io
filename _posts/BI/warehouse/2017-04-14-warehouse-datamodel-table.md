---
title: 事实表和维度表
tags: [BI]
---

### 维度表和事实表的理解

事实表就是你要关注的内容；维度表就是你观察的角度，是从哪个角度去观察这个内容的。

参考：https://www.cnblogs.com/sdjnzqr/p/3829670.html

1）维度表的理解

维度表示对数据进行分析时所用的一个量，如要分析产品销售情况，可以选择按类别来进行分析，或按区域来分析，而类别或区域就是一个维度。另外每个维度还可以有子维度（称为属性），例如类别可以有子类型，产品名等属性。

一个维是通过一组属性来描述的，如时间维包含年份、季度、月份和日期等属性，这里的年份、季度等称为时间维的维属性。

维度表可以看作是用户来分析数据的窗口，纬度表中包含事实数据表中事实记录的特性，有些特性提供描述性信息，有些特性指定如何汇总事实数据表数据，以便为分析者提供有用的信息，维度表包含帮助汇总数据的特性的层次结构。例如，包含产品信息的维度表通常包含将产品分为食品、饮料、非消费品等若干类的层次结构，这些产品中的每一类进一步多次细分，直到各产品达到最低级别。

2）事实表的理解

度量是数据仓库中的信息单元，通常是数值型数据并具有可加性（如销售量）。事实表的建立是针对已经发生的事实的，是历史数据的存档，也就是说是不应该修改的。

事实表也称为度量表，是数据聚合后依据维度生成的结果表。

3）维度表与事实表间的关系

事实数据表不应该包含描述性的信息，也不应该包含除数字度量字段及使事实与纬度表中对应项的相关索引字段之外的任何数据。

一般来说，一个事实数据表都要和一个或多个纬度表相关联，用户在利用事实数据表创建多维数据集时，可以使用一个或多个维度表。

### 事实表和维表补充

事实表是用来记录具体事件的，包含了每个事件的具体要素，以及具体发生的事情；维表则是对事实表中事件的要素的描述信息。比如一个事件会包含时间、地点、人物、事件，事实表记录了整个事件的信息，但对时间、地点和人物等要素只记录了一些关键标记，比如事件的主角叫“Michael”，那么Michael到底“长什么样”，就需要到相应的维表里面去查询“Michael”的具体描述信息了。基于事实表和维表就可以构建出多种多维模型，包括星形模型、雪花模型和星座模型。

![](/images/other/data-analysis/Star-Schemas.png)

事实表里面主要包含两方面的信息：维和度量，维的具体描述信息记录在维表，事实表中的维属性只是一个关联到维表的键，并不记录具体信息；度量一般都会记录事件的相应数值，比如这里的产品的销售数量、销售额等。维表中的信息一般是可以分层的，比如时间维的年月日、地域维的省市县等，这类分层的信息就是为了满足事实表中的度量可以在不同的粒度上完成聚合，比如2010年商品的销售额，来自上海市的销售额等。

注意：维表的信息更新频率不高或者保持相对的稳定，例如一个已经建立的十年的时间维在短期是不需要更新的，地域维也是；但是事实表中的数据会不断地更新或增加，因为事件一直在不断地发生，用户在不断地购买商品、接受服务。