---
title: 业务数据关联表BpmBusLink
tags: [bpmx]
---

1）busStatus

在BpmBusLink中存在字段busStatus，该字段用于表示该业务主键的状态，防止主键重复。如在复制表单时，会需要重新生成相应的主键信息。

注：业务数据状态(0,业务数据,1,流程运行,2,流程结束,3,手工结束)