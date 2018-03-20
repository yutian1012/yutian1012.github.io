---
title: 数据仓库的基本架构
tags: [BI]
---

参考：http://webdataanalysis.net/web-data-warehouse/data-warehouse-frame/

数据仓库本身既不生产数据也不消费数据，只是作为一个中间平台集成化地存储数据；数据仓库实现的难度在于整体架构的构建及ETL的设计，这也是日常管理维护中的重头；而数据仓库的真正价值体现在于基于其的数据应用上，如果没有有效的数据应用也就失去了构建数据仓库的意义。

1）数据源

数据源是数据仓库系统的基础，是整个系统的数据源泉。通常包括企业内部信息和外部信息。内部信息包括存放于RDBMS中的各种业务处理数据和各类文档数据。外部信息包括各类法律法规、市场信息和竞争对手的信息等等；

2）数据的存储与管理

是整个数据仓库系统的核心。数据仓库的真正关键是数据的存储和管理。针对现有各业务系统的数据，进行抽取、清理，并有效集成（ETL），按照主题进行组织存储。

3）OLAP服务器

对分析需要的数据进行有效集成，按多维模型予以组织，以便进行多角度、多层次的分析，并发现趋势。其具体实现可以分为：ROLAP（关系型在线分析处理）、MOLAP（多维在线分析处理）和HOLAP（混合型线上分析处理）。ROLAP基本数据和聚合数据均存放在RDBMS之中；MOLAP基本数据和聚合数据均存放于多维数据库中；HOLAP基本数据存放于RDBMS之中，聚合数据存放于多维数据库中。

4）前端工具

主要包括各种报表工具、查询工具、数据分析工具、数据挖掘工具以数据挖掘及各种基于数据仓库或数据集市的应用开发工具。其中数据分析工具主要针对OLAP服务器，报表工具、数据挖掘工具主要针对数据仓库。

![](/images/other/data-analysis/warehouse-system.jpg)