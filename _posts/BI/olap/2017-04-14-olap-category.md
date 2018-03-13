---
title: OLAP的分类
tags: [BI]
---

多维数据模型作为一种新的逻辑模型赋予了数据新的组织和存储形式，而真正体现其在分析上的优势还需要基于模型的有效的操作和处理，也就是OLAP（On-line Analytical Processing，联机分析处理）。

1）MOLAP(Multidimensional)

即基于多维数组的存储模型，也是最原始的OLAP，但需要对数据进行预处理才能形成多维结构。维的属性值被映射成多维数组的下标值或下标的范围，而总结数据作为多维数组的值存储在数组的单元中。

注：即以多维数组的方式存放分析数据

2）ROLAP(Relational)

比较常见的OLAP类型，这里介绍和讨论的也基本都是ROLAP类型，其实ROLAP是完全基于关系模型进行存放的，只是它根据分析的需要对模型的结构和组织形式进行的优化，更利于OLAP。

注：使用关系型数据库存放分析数据

3）HOLAP(Hybrid)

介于MOLAP和ROLAP的类型，我的理解是细节的数据以ROLAP的形式存放，更加方便灵活，而高度聚合的数据以MOLAP的形式展现，更适合于高效的分析处理。

注：Hybrid表示混合在一起的意思

4）其他

另外还有WOLAP(Web-based OLAP)、DOLAP(Desktop OLAP)、RTOLAP(Real-Time OLAP)，具体可以参开维基百科上的解释——OLAP。