---
title: 数据仓库特点
tags: [BI]
---

1）面向主题

数据仓库是面向主题的（而操作型数据库的数据组织面向事务处理任务），即数据仓库中的数据是按照一定的主题域进行组织。主题是指用户使用数据仓库进行决策时所关心的重点方面，一个主题通常与多个操作型信息系统相关。主题是与传统数据库的面向应用相对应的，是一个抽象概念，是在较高层次上将企业信息系统中的数据综合、归类并进行分析利用的抽象。每一个主题对应一个宏观的分析领域。数据仓库排除对于决策无用的数据，提供特定主题的简明视图。

2）数据来源

数据仓库是集成的（多个数据源的集成），数据仓库的数据有来自于分散的操作型数据，将所需数据从原来的数据中抽取出来，进行加工与集成，统一与综合之后才能进入数据仓库；

3）查询为主

数据仓库是不可更新的，数据仓库主要是为决策分析提供数据，所涉及的操作主要是数据的查询。

4）随时间变化

数据仓库是随时间而变化的，传统的关系数据库系统比较适合处理格式化的数据，能够较好的满足商业商务处理的需求。稳定的数据以只读格式保存，且不随时间改变。

5）汇总的，大容量的，非规范化的

操作性数据映射成决策可用的格式。数据集合通常都非常大。Dw数据可以是而且经常是冗余的。