---
title: 流程实例
tags: [activiti]
---

1）流程实例与执行流

在Activiti中，启动一个流程后，会创建一个流程实例（ProcessInstance），每个流程实例至少会有一个执行流（Execution）。

当流程实例没有流程分支时，一般情况下只会存在一个执行流（这个执行流就是流程实例），假设流程出现了并行网关，那么activti会产生多个执行流。

![](/images/book/workflow/activiti/execution/multiexecution.png)

注：在执行到并行网关时会产生出3个执行流，一个为原来的流程执行流（即流程实例），而其余两个为第一个执行流的子执行流。

注：区分子执行流和子流程。

2）流程实例和执行流对象（ProcessInstance和Execution）

ProcessInstance是一个接口，一个ProcessInstance实例表示一个流程实例，而ProcessInstance实际上是执行流（Execution）的子接口。因此，可以理解为流程实例也是执行流。

流程实例/执行流将数据保存到act_ru_execution表中。实体对象为ExecutionEntity，该类实现了ProcessInstance接口（即实现了Execution接口）。

3）数据对象

```
protected String id = null;//主键
protected int revision = 1;//数据的修订版本好
protected String businessKey;//流程实例的业务主键，只有ProcessInstance实例才有
//父执行流ID，一般情况下，流程实例与执行流的数据为同一条数据。当出现并行网关，多个执行流时，该字段值指向流程实例的ID
protected String parentId;
protected String processDefinitionId;//流程定义ID，流程实例和执行流都存储该字段
protected String superExecutionId;//父执行流的ID，如果该执行流是子流程的一部分
protected String activityId;//当前执行流的动作，一般为流程节点的名称。
//指示该执行流状态是否活跃，如果当前的执行流分为两个执行流，则当前的执行流被标识为非活跃状态，而两个子执行流则为活跃状态。
protected boolean isActive = true;
protected boolean isScope = true;//是否在执行流的范围内
protected boolean isConcurrent = false;//执行流是否并行
protected boolean isEnded = false;//执行流是否结束
protected boolean isEventScope = false;//是否在事件范围内
//流程中断状态，1为活跃，2为中断
protected int suspensionState = SuspensionState.ACTIVE.getStateCode();
protected int cachedEntityState;//流程相关实体的缓存状态。
```

4）ProcessInstance流程实例的产生

```
ProcessInstance pi = runtimeService.startProcessInstanceById(pd.getId());
pi=runtimeService.startProcessInstanceByKey("vacationRequest");
```

注：根据流程定义的Id和key等方式可以启动流程，并产生流程实例对象。