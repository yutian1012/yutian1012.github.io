---
title: 请假流程设计与实现
tags: [bpmx]
---

1）场景

发起人：填写请假天数，请假日期，联系电话，备注原因后提交。

审批人：若请假1天，则直接审批通过；若请假大于1天，则与人力部门共同审批（并行）；汇总审批结果，所有任务节点都通过才认为审批通过，否则审批不通过；

2）场景2

在场景1的基础上增加功能，当请假流程通过后发起一个销假流程；

发起人：点击待办，销假表单的前几个字段引用请假流程的字段信息（不允许改写）；填写销假日期，延误原因，附件字段信息后提交。

审批人：部门经理直接审批。