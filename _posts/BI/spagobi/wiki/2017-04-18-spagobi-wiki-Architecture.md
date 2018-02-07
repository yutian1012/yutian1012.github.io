---
title: 开源spagobi--结构组件
tags: [spagobi]
---

介绍spagobi的系统组件，每个模块的意义，解析数据引擎等。

参考：http://wiki.spagobi.org/xwiki/bin/view/spagobi_server/analytical_angines

中文：http://www.zhimengzhe.com/shujuku/other/210255.html

![](/images/open/spagobi/wiki/Architettura.png)

### 1. Analytical Model

分析模型：是SpagoBI服务器的核心，涵盖分析需求的整个范围，对每一个分析领域提供多种解决方案。

1）报表（Reporting），

实现结构化的报表，并将其导出为多种格式

2）OLAP，

允许通过灵活的和友好的OLAP引擎对数据进行多维分析

3）图表（Charts），

允许开发单个随时可以使用的图形和交互式仪表盘控件

4）KPI，创建，管理，查看和浏览KPI业务模型

5）管理驾驶舱（Interactive cockpits），

整合几个分析文件（报表或图表等）到一个单一的视图，以提高交互性，并让数据使用起来更加直观

6）即席报表（Ad-hoc reporting），

用户可以自由创建多页的分析报告

7）地理位置分析（Location Intelligence），

地理位置和业务数据之间的建立联系，让数据呈现更加直观有效；

8）自助分析（Free Inquiry (Driven Data Selection)），

通过QBE引擎，用完全图形化和基于web的界面，让你查看和分析数据更加容易和直观，并可以将结果保存下来，以备将来使用；

9）数据挖掘（Data Mining），

从大量数据中挖掘出有价值的信息，以辅助企业经营决策

10）实时仪表板和控制台（Real-time dashboards and console），

允许生产实时监控控制台

11）协作（Collaboration），

自动创建有组织的报告卷宗，并提交您的审核意见和反馈需要注意的事项

12）办公自动化（Office Automation），

用于上传和管理个人的Office文档

13）ETL，

允许将数据加载到数据仓库中，并对其进行管理

14）移动BI（Mobile），

通过常见的移动设备访问您的决策支持系统外部流程，

15）管理定制的流程（External processes），

在后台运行并可以设置预定的时间开始运行

16）主数据管理（Master Data Management），

拥有对数据库的回写功能

17）社交网络分析（Network analysis），

它允许用户基于社交网络进行可视化分析 
