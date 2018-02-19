---
title: 数据仓库--维（Dimension）和立方（Cube）
tags: [BI]
---

参考：http://webdataanalysis.net/web-data-warehouse/dimension-and-cube/

### 1. 维（Dimension）

维是用于从不同角度描述事物特征的，一般维都会有多层（Level），每个Level都会包含一些共有的或特有的属性（Attribute）。

1）维度表的设计

以时间维为例，时间维一般会包含年、季、月、日这几个Level（级别），每个Level一般都会有ID、NAME、DESCRIPTION这几个公共属性，这几个公共属性不仅适用于时间维，也同样表现在其它各种不同类型的维。

![](/images/other/data-analysis/Dimension.png)

其中ID一般被视为代理主键（Agent），它只被用于作为唯一性标志，并且是多维模型中关联关系的代理者，在业务层面并不具有任何意义；NAME一般是业务主键（Business），在业务层面限制唯一性，一般作为数据装载（Load）时的关联键；而DESCRIPTION则记录了详细描述信息，在多维展示和分析时我们都会选择使用DESCRIPTION来表述具体含义。这3个属性一般是所有Level都会共用的，而比如用于描述星期几的属性weekid可能只会用于"日期"这层，因为年月都不具备这一信息。所以图中我将Attributes放到了一个层面上，就如同是不同的Level从底层的多个Attributes中选取自身所需的属性，Attributes层是包含着各个Level的共有和特有属性的集合。

2）OLAP中维的hierarchy

OLAP需要基于有层级的自上而下的钻取，或者自下而上地聚合。所以每一个维必须有Hierarchy，至少有一个默认的，当然可以有多个。

![](/images/other/data-analysis/Hierarchy.png)

Hierarchy，表示维里面的Level的自上而下的树形结构关系，也就是上层的每一个成员（Member）都会包含下层的0个或多个成员，也就是树的分支节点。这里需要注意的是每个Hierarchy树的根节点一般都设置成所有成员的汇总（Total），当该维未被OLAP中使用时，默认显示的就是该维上的汇总节点，也就是该维所有数据的聚合（或者说该维未被用于细分）。Hierarchy中的每一层都会包含若干个成员（Member）。

以时间维为例，假设我们建的是2006-2015这样一个时间跨度的时间维，那么最高层节点仅有一个Total的成员，包含了所有这10年的时间，而年的那层Level中包含2006、2007…2015这10个成员，每一年又包含了4个季度成员，每个季度包含3个月份成员……这样似乎顺理成章多了，我们就可以基于Hierarchy做一些OLAP操作了。