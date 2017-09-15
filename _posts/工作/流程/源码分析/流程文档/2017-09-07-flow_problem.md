---
title: 遇到的问题
tags: [bpmx]
---

1）流程执行堆栈树的作用（ExecutionStack）？

2）流程意见（TaskOpinion），为什么不用activiti支持的comment？

3）流程节点状态（BpmProStatus），为什么要扩展这个状态？

4）为什么要使用HistoryActivityDao直接操作ACT_HI_ACTINST表？

TaskCompleteListener类的setActHisAssignee方法。

5）流程运行日志（bpmRunLog）

6）外部表单的作用？

流程节点设置表单时，设置外部表单。

7）是否外部调用？

processCmd对象的invokeExternal字段。

8）条件同步路径？

当节点为条件同步时，可以在条件分支节点上设置条件同步路径，具体应用场景？

9）代理和转办代理有什么区别

ProcessRunService类的handleAgentTaskExe方法。转办代理对象（BpmTaskExe）