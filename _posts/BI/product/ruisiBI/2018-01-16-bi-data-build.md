---
title: ruisiBI数据建模源码查看
tags: [BI]
---

通过查看源代码可以发现，前后端的交互都是以json的形式来实现的。

1）数据建模功能处理类ModelController

访问http://localhost:8080/rsbi/model/ModelIndex.action

直接跳转到ModelIndex.jsp页面，该页面使用ajax的方式加载数据。

2）数据源处理类DataSourceController

该类用于完成数据源的配置信息，支持数据库连接方式和JNDI方式配置数据源。

注：将数据保存到etl_datasource表中

3）数据集处理类DatasetController

用于通过从数据源中选择数据表（支持多表关联），并设置相关联信息，从而构造出数据集。

注：将数据保存到etl_dataset表中，配置信息存储在etl_dataset表的cfg字段中，以json的形式存储。

表关联配置信息：

```
{
    "master": "dm_zzjb",//主表
    "joininfo": [{  //设置的管理信息
        "col": "month_id", //主表的关联字段
        "ref": "dm_zsb",  //关联表名称
        "refKey": "month",  //关联表的关联字段
        "jtype": "all"  //关联类型，包括全连接，左连接和右连接
    }],
    "name": "测试关联", //dataset新建记录名称
     //数据源主键字段，关联etl_datasource表
    "dsid": "df3ab5d1af56002fd6be0622c3b69b23",
    //数据集主键字段
    "dsetId": "416af45a39f8b49690bd1a5e0a72bad0",
    "cols": [{  //列字段配置信息
        "idx": 1,  //idx为1表示主表字段
        "name": "year_id", //字段名
        "type": "String",  //字段类型
        "dispName": "年度",  //显示名称
        "length": null,  //字段长度
        "tname": "dm_zzjb",  //字段所在表名
        "isshow": true,  //是否显示
        "expression": ""  //表达式，针对动态字段设置。
    },
    ... //其他字段
    ],
    "dynamic": [{  //动态字段设置
        "name": "test_zzj",  //字段名称
        "tname": "dm_zjfx",  //表名
        "type": "Double",
        "expression": "gdzj + cczj "
    }]
}
```

注：动态字段求和操作只能在单表中操作（不能跨表），且只能是在主表中操作。

4）立方体（cube）

在数据集的基础上对数据进行维度和独立的划分操作。处理类为CubeController

保存/修改的数据结构

```
{
    "cubeName": "租金分析", //cube对象名
    "desc": "", //描述信息
    "dsetId": "c2114b83c1099c3fbd996255493e8561",//数据集主键
    "cubeId": 2, //主键
    "dims": [{ //维度字段信息
        "name": "年度", //维度名称
        "type": "year", //维度类型：包括，年、季度、月、日、省份、地市
        "col": "year", //列名称
        "tname": "dm_zjfx", //表名
        "alias": "year",    //别名
        "vtype": "String", //字段类型
        "colTable": "", //？
        "colkey": "", //维度Key字段 ？
        "coltext": "", //维度text字段 ？
        "ordcol": "", //排序列
        "dimord": "", //排序方式，正序和倒序
        //维度格式：主要是针对日期类型的格式设置，与数据库中存储的值要对应
        "dateformat": "yyyy",
        "calc": 0, //
        "targetId": 15,
        "groupName": "日期",//新建分组，分组下必须要存在维度信息否则不会保存
        "groupId": "9bfe37fabf83ca5777db9ae04e9c1d38" //js前端生成
    },
    ...
    ]
    ,
    "kpis": [{ //度量字段信息
        "name": "固定租金",  //度量字段名
        "col": "gdzj",  //列名称
        "tname": "dm_zjfx", //表名
        "alias": "gdzj", //别名
        "fmt": "", //格式化操作：包括，整数，小数（保留位），百分比
        "unit": "", //度量单位设置
        "aggre": "sum", //计算方式，包括：sum，avg，count，max，min
        "kpinote": "", //度量解释信息
        "calc": 0, //？
        "calcKpi": 0, //？
        "targetId": 255 //？
    },
    ...
    ],
    //删除字段信息，包括删除维度字段和度量字段
    "delObj": [{
        "type": "dim",
        "id": 19
    }]
}
```

注1：只有维度可以设置group分组，并将信息保存到olap_dim_group表中

注2：维度信息的group_type关联分组，另外编辑维度弹框的设置信息都会保存到olap_dim_list表中

注3：度量信息不能设置分组，另外编辑度量弹框的设置信息都会保存到olap_kpi_list表中

注4：维度和度量相关的字段信息，包括列名、表名等信息会保存到olap_cube_col_meta表中

注5：立方体的建模记录会保存到olap_cube_meta表中，并且其他表中都包含一个cube_id字段与该主表进行关联。

### 页面操作

对于数据建模功能来讲，其所有的操作都是通过js完成的，包括页面标签信息，也是通过js生成并设置到document来展示的。

以设置数据集为例查看其实现过程：

1）点击左侧树查看数据集列表

执行model-dset.js文件的initdsetTable方法，该方法会获取数据库中存储和配置的是数据集列表。

2）新增操作

点击新增按钮，执行model-dset.js文件的newdset方法。保存时将配置信息转存json字符串。

```
//提交数据
//其中JSON.stringify(transform)方法是将配置信息对象transform序列化为字符串
data: {cfg:JSON.stringify(transform), priTable: transform.master, name:transform.name,dsid:transform.dsid, dsetId:transform.dsetId}
```