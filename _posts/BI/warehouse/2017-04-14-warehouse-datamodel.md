---
title: 多维数据模型
tags: [BI]
---

DB的模型是ER模型，遵从范式化设计原则，而DW的数据模型是雪花型结构或者星型结构，用的是面向主题，面向问题的设计思路，所以DB和DW的模型结构不同，需要进行转换。

通过多维数据模型的数据展示、查询和获取。通过数据仓库可以根据不同的数据需求（面向主题）建立起各类多维模型，并组成数据集市开放给不同的用户群体使用，也就是根据需求定制的各类数据商品摆放在数据集市中供不同的数据消费者进行采购。

参考：http://blog.csdn.net/ecjtuxuan/article/details/6273983

1）定义

多维数据模型是为了满足用户从多角度多层次进行数据查询和分析的需要而建立起来的基于事实和维度（维表、事实表）的数据库模型，其基本的应用是为了实现OLAP（Online Analytical Processing）。

多维数据模型是数据仓库的一大特点，也是数据仓库应用和实现的一个重要的方面，通过在数据的组织和存储上的优化，使其更适用于分析型的数据查询和获取。

### RLOAP数据模型

在多维分析的商业智能解决方案中，根据事实表（度量表）和维度表的关系，又可将常见的模型分为星型模型和雪花型模型。在设计逻辑型数据的模型的时候，就应考虑数据是按照星型模型还是雪花型模型进行组织。

1）星型模型（推荐）

当所有维表都直接连接到事实表上时，整个图解就像星星一样，故将该模型称为星型模型。

星型架构是一种非正规化的结构，多维数据集的每一个维度都直接与事实表相连接，不存在渐变维度，所以数据有一定的冗余，如在地域维度表中，存在国家A省B的城市C以及国家A省B的城市D两条记录，那么国家A和省B的信息分别存储了两次，即存在冗余。

![](/images/BI/data/startmodel.jpg)

2）雪花型模型

当有一个或多个维表没有直接连接到事实表上，而是通过其他维表连接到事实表上时，其图解就像多个雪花连接在一起，故称雪花模型。

雪花模型是对星型模 型的扩展。它对星型模型的维表进一步层次化，原有的各维表可能被扩展为小的事实表，形成一些局部的层次区域，这些被分解的表都连接到主维度表而不是事实表。如，将地域维表又分解为国家，省份，城市等维表。它的优点是 : 通过最大限度地减少数据存储量以及联合较小的维表来改善查询性能。雪花型结构去除了数据冗余。

![](/images/BI/data/snowmodel.jpg)

3）两模型的比较

星型模型因为数据的冗余所以很多统计查询不需要做外部的连接，因此一般情况下效率比雪花型模型要高。星型结构不用考虑很多正规化的因素，设计与实现 都比较简单。 雪花型模型由于去除了冗余，有些统计就需要通过表的联接才能产生，所以效率不一定有星型模型高。正规化也是一种比较复杂的过程，相应的数据库结构设计、数 据的ETL、以及后期的维护都要复杂一些。因此在冗余可以接受的前提下，实际运用中星型模型使用更多，也更有效率。

### 多维数据模型的优缺点

1）优点：

多维数据模型最大的优点就是其基于分析优化的数据组织和存储模式。举个简单的例子，电子商务网站的操作数据库中记录的可能是某个时间点，某个用户购买了某个商品，并寄送到某个具体的地址的这种记录的集合，于是我们无法马上获取2010年的7月份到底有多少用户购买了商品，或者2010年的7月份有多少的浙江省用户购买了商品？但是在基于多维模型的基础上，此类查询就变得简单了，只要在时间维上将数据聚合到2010年的7月份，同时在地域维上将数据聚合到浙江省的粒度就可以实现，这个就是OLAP的概念。

2）缺点：

多维模型的缺点就是与关系模型相比其灵活性不够，一旦模型构建就很难进行更改。

比如一个订单的事实，其中用户可能购买了多种商品，包括了时间、用户维和商品数量、总价等度量。如果我们进而需要区分订单中包含了哪些商品，对于关系模型而言，我们只需要另外再建一张表记录订单号和商品的对应关系即可，但在多维模型里面一旦事实表构建起来后，我们无法将事实表中的一条订单记录再进行拆分，于是无法建立以一个新的维度——产品维，只能另外再建个以产品为主题的事实表。

所以，在建立多维模型之前，我们一般会根据需求首先详细的设计模型，应该包含哪些维和度量，应该让数据保持在哪个粒度上才能满足用户的分析需求。