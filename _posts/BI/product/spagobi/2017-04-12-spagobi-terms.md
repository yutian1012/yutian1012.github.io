---
title: 开源spagobi--名词
tags: [spagobi]
---

### 1. OLAP

OLAP（On-Line Analytical Processing）联机分析处理。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。OLAP的目标是满足决策支持或者满足在多维环境下特定的查询和报表需求,它的技术核心是"维"这个概念。

### 2. OLTP

OLTP（on-line transaction processing）联机事务处理。OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。

### 3. JPIVOT

JPivot是一个OLAP的客户端（http://jpivot.sourceforge.net/），使用XML+XSL来展现OLAP的数据，可以任意选择LOAP server和OLAP client的组合。JPivot也支持MSSQL Server的OLAP数据源。

### 4. Mondrian

Mondrian是一个基于Java语言的开源OLAP引擎，它通过MDX语句执行查询，从关系型数据库RDBMS中读取数据，以多维度的形式展示查询结果。

### 5. KPI

KPA（Key Process Area)意为关键过程领域，这些关键过程领域指出了企业需要集中力量改进和解决问题的过程。

### 6. Ad-hoc

Ad-Hoc（点对点）模式，即席报表。就是允许用户（注意是终端用户）自行设计的一种报表，比如自己添加行列，汇总，排序等。

### 7. Palo

一个MLOAP的实现，产品相对成熟。

### 8. weka

WEKA作为一个公开的数据挖掘工作平台，集合了大量能承担数据挖掘任务的机器学习算法，包括对数据进行预处理，分类，回归、聚类、关联规则以及在新的交互式界面上的可视化。

参考：http://blog.csdn.net/yangliuy/article/details/7589306

### 9. talend

talend是一种ETL工具，提供了数据集成，数据分析以及清洗，以及主数据管理。

参考：http://www.docin.com/p-635429817.html

### 10. MDX

MDX（multi-dimensional expressions多维表达式）是一种语法，支持多维对象与数据定义和操作。用途是使对多维的数据的访问更为简单和直观。

主要概念：维度（Dimension），级别（Levels），成员（Members），度量值（Measure）


