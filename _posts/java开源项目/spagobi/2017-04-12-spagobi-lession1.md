---
title: 开源spagobi--教程（一）
tags: [spagobi]
---

### 1. 介绍

SpagoBi is an open source business intelligence suite.  It consists of several engines and analytical areas.The engines includes SpagoBIBirtReportEngine, SpagoBIJPivotEngine and SpagoBIHighChartsEngine.

```
reporting using the SpagoBIBirtReportEngine,
OLAP using the SpagoBIJPivotEngine 
Charts using the SpagoBIHighChartsEngine。
```

spagoBi是一个开源的BI组件，有多个引擎和分析领域组成。

### 2. 软件

1）SpagoBI Server 

This is the actual business intelligence platform that offers all the core and analytical functionalities. It is also where we will be hosting all reports created using BIRT. 

是一个运行在tomcat（web服务器）上的web平台环境。用来部署、显示分析、图表等。

2）SpagoBI Studio

to create BIRT reports.the acronyms stand for Business Intelligence 
and Reporting Tools.

创建BIRT的报表工具

### 3. OLAP

参考：http://blog.csdn.net/zhangzheng0413/article/details/8271322/

On-Line Analytical Processing（联机分析处理）是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。 

OLAP系统则强调数据分析，强调SQL执行市场，强调磁盘I/O，强调分区等。在这样的系统中，语句的执行量不是考核标准，因为一条语句的执行时间可能会非常长，读取的数据也非常多。所以，在这样的系统中，考核的标准往往是磁盘子系统的吞吐量（带宽），如能达到多少MB/s的流量。

### 4. example实例

运行并查看D:\All-In-One-SpagoBI-5.2.0\bin\SpagoBIStartup.bat

注：如何环境变量中设置了CATALINA_HOME，需要将批处理中的跳转注释掉

```
rem if not "%CATALINA_HOME%" == "" goto gotHome
```

1）报表查看

Navigate to Root -> Examples -> Report_BIRT and click on Report with image

![](/images/open/spagobi/spagobi-example-report.png)

2）OLAP处理

Navigate to Root -> Examples ->  OLAP_Jpivot_Mondrian and click on Simple OLAP

![](/images/open/spagobi/spagobi-example-loap.png)

3）图表

Navigate to Root -> Examples ->  Charts – Highcharts.

![](/images/open/spagobi/spagobi-example-chart.png)