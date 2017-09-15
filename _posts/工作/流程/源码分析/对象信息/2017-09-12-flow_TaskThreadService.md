---
title: 流程任务执行线程绑定服务类
tags: [bpmx]
---

1）对象信息

extSubProcess（ThreadLocal<List<String>>外部子流程调用，存放子流程实例id列表）

newTasksLocal（ThreadLocal<List<Task>>，用于任务完成后，收集产生的新任务）