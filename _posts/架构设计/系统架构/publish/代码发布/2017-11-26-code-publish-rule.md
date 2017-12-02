---
title: 上线规则制度
tags: [architecture]
---

### 上下规则制度

1）大于2天的项目需要架构组参与设计评审

2）有数据库变动的项目需要DBA参与设计评审

3）项目负责制，有负责人随时发email周知

4）有数据库变更的上线，需要签上线单的同时，附加数据库变更申请单，DBA签字方可在正式库执行。

![](images/architecture/svn/code-publish/data-change-apply.png)

### 上线申请单


申请单包括：产品上线申请单，例行上线申请单，新产品上线申请单，紧急上线申请单等。

申请单中主要包括项目名称，上线时间，所属部门。包括各种签字：技术，产品，运营等。最后交由开发和运营来执行。需严格的流程和监督进行审核，方可上线。

### 系统发布流程

![](images/architecture/svn/code-publish/publish-flow.jpg)