---
title: ruisiBI多维分析源码查看
tags: [BI]
---

多维分析处理类ReportDesignController，是建立在数据建模（cube立方体）的基础上进行操作的。通过拖动建模数据的维度和kpi（度量值）来设计表格和图表，进而分析数据的关联性。

保存分析数据，通过菜单栏文件--保存功能，会将设置保存到olap_user_save表中。同时也可以通过文件--打开功能，打开保存的分析数据。

1）表格数据

行和列只能放置立方体的维度字段（dim），中间的数据展示区域只能放置度量字段（kpi）。

表格的配置json（节选）

```
{
    "name": "表格组件", 
    "id": 1, //相当于图表，图表的id值为2
    "type": "table",
    "cubeId": 2, //关联建模主键，即olap_cube_meta
    "dsid": "df3ab5d1af56002fd6be0622c3b69b23", //数据源主键
    "dsetId": "c2114b83c1099c3fbd996255493e8561", //数据集主键
    //表格中显示的观测值
    //主要从olap_kpi_list表和olap_cube_col_meta表中获取字段信息
    "kpiJson": [{
        "kpi_id": 255, //关联olap_kpi_list表的RID字段
        "kpi_name": "固定租金", //显示名称
        "col_name": "sum(gdzj)",
        "aggre": "sum", //聚合函数
        "fmt": "#,###.00", //格式信息
        "alias": "gdzj", 
        "tname": "dm_zjfx",
        "unit": "",
        "rate": null,
        "calc": 0
    }],
    "cols": [{ //设置表格的列
        "id": 17, //关联olap_dim_list表中的dim_id字段
        "dimdesc": "楼层", //维度描述
        "type": "frd", //？
        "colname": "floor_name", //列名称
        "alias": "floor_name",
        "tname": "dm_zjfx", //表名
        "iscas": "y",
        "tableName": "",
        "tableColKey": "",
        "tableColName": "",
        "dimord": "", //维度的排序
        "grouptype": "af107c90e9ba808d7ec7b79c2c136349",//维度所在组
        "valType": "String",
        "ordcol": "",
        "dateformat": "",
        "calc": 0
    }],
    "rows": [{ //设置行维度
        "id": 15,
        "dimdesc": "年度",
        "type": "year",
        "colname": "year",
        "alias": "year",
        "tname": "dm_zjfx",
        "iscas": "y",
        "tableName": "",
        "tableColKey": "",
        "tableColName": "",
        "dimord": "",
        "grouptype": "9bfe37fabf83ca5777db9ae04e9c1d38",
        "valType": "String",
        "ordcol": "",
        "dateformat": "yyyy",
        "calc": 0
    }]
}
```

注：该配置信息存放到olap_user_save表的pageinfo字段中。

### 页面的加载

bireport/ReportDesign.jsp页面，该页面定义了标题栏的菜单栏；维度字段的菜单功能，如排序、聚合等（页面元素#dimoptmenu）；度量字段的菜单功能，如计算、筛选、预警等（页面元素#kpioptmenu）。

1）页面初始化左侧树

左侧树为创建的数据模型（cube），默认加载最新的模型数据。执行reloadDatasetTree方法，该方法位于bireport.js文件中。加载数据访问CubeController类的tree方法（url路径为：model/treeCube.action），该方法直接返回tree的json数据，从而完成加载。

2）初始化参数

initparam方法位于bireport.js文件中。在设置表格和图表时可以设置页面查询参数，通过拖拽维度字段到参数控件中。并注册接收维度拖拽事件。

参数设置bireport.js文件的paramFilter方法中，定义了参数设置及加载数据的方法，设置完成参数后点击确定调用flushPage方法，会去后台加载数据。加载数据是针对表格会调用tableView方法（定义在bitable.js文件中），针对图表会调用chartview方法（定义在bichart.js文件中）。

参数的json数据（节选）

```
"params": [{ //页面参数，通过维度字段的拖拽功能设置参数
    "id": 15, ///关联olap_dim_list表中的dim_id字段
    "name": "年度",
    "type": "year",
    "colname": "year",
    "alias": "year",
    "tname": "dm_zjfx",
    "cubeId": 2,
    "valType": "String",
    "tableName": "",
    "tableColKey": "",
    "tableColName": "",
    "dimord": "",
    "grouptype": "9bfe37fabf83ca5777db9ae04e9c1d38",
    "dateformat": "yyyy",
    "dsid": "df3ab5d1af56002fd6be0622c3b69b23",
    "vals": "2013", //设置的值，可以多选
    "valStrs": "2013"
}]
```

注：该配置信息也存放到olap_user_save表的pageinfo字段中。

3）初始化组件

for循环页面设置的组件信息（表格组件和图表组件），调用addComp方法（位于bireport.js文件中），该方法会执行initDropDiv（位于bitable.js文件中）来初始化组件的拖拽监听事件。

针对表格数据，当有相应的维度拖放到组件的相应位置后，就会调用tableView方法（位于bitable.js文件），来加载显示数据。tableView方法加载数据，访问TableView.action并将参数信息传递到后台，TableController类的tableView方法来接收处理请求。