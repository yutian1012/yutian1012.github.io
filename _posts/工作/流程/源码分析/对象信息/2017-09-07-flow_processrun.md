---
title: 流程实例扩展对象ProcessRun
tags: [bpmx]
---

### ProcessRun对象内容

1）流程表单信息：

FormDefId（表单的定义ID）；
BusinessUrl（外部表单路径）；
TableName（业务数据主表名）；
PkName（主表的主键名称）；
DsAlias（数据源别名）；
businessKey（业务主键）。

2）用户信息：

Creator（创建人）；
CreatorId（创建人ID）；
Createtime（创建时间）

3）流程定义（bpmDefinition）信息：

actDefId（流程定义ID，来自Activiti的流程定义ID，act_re_procdef表）；
DefId（流程定义ID）；
subject（流程标题）；
defKey（流程定义Key）；
typeId（流程定义的分类ID）；

注：这部分信息可以在流程定义设置中获取

4）流程实例信息：

status（流程实例状态）；
isFormal（否正式流程实例）；
runId（启动流程可以从外部获取runId，需要在流程变量集合中设置startRunId变量）；
RelRunId（关联的流程实例ID，从请求封装的cmd对象中获取）；
GlobalFlowNo（流程实例的全局流水号）。

注：isFormal是根据BpmDefinition对象的状态来决定的（包括测试和运行，流程定义其他设置节点）。
