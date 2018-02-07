---
title: 流程数据查询
tags: [activiti]
---

activiti为流程实例和流程执行流提供了相应的query查询对象。得到Query对象，然后设置条件，最后调用list、singleResult等方法返回查询结果，也可以使用orderByXXX方法对象查询结果进行排序。

1）执行流查询

RuntimeService的createExecuteQuery方法可以得到一个ExecutionQuery对象，通过该对象可以根据执行流的相关数据查询执行流，包括执行流参数，执行流的活动状态，执行流信号事件和执行流消息事件等。

```
//通过流程定义Key查询执行流
List<Execution> exes = runtimeService.createExecutionQuery()
    .processDefinitionKey("testProcess").list();
//通过bussinesskey查询
exes = runtimeService.createExecutionQuery()
    .processInstanceBusinessKey("businessKey1").list();
//通过消息事件名称查询执行流
exes = runtimeService.createExecutionQuery()
    .messageEventSubscriptionName("messageName").list();
//通过节点id属性（activityId）查询当前的执行流
Execution exe = runtimeService.createExecutionQuery()
    .activityId("messageintermediatecatchevent1")
    .processInstanceId(pi1.getId()).singleResult();
//通过信号事件名称查询执行流
exes = runtimeService.createExecutionQuery().signalEventSubscriptionName("signalName").list();
// 根据参数查询执行流
exes = runtimeService.createExecutionQuery().variableValueEquals("name", "crazyit").list();
exes = runtimeService.createExecutionQuery().variableValueGreaterThan("days", 5).list();
```

注：业务主键businessKey只有流程实例才有，因此查询的是流程实例对应的执行流。

2）流程实例查询

RuntimeService中可以通过createProcessInstanceQuery方法获取ProcessInstanceQuery实例，然后通过设置相关查询参数实现查询。

```
//通过流程定义Key查询
List<ProcessInstance> pis = runtimeService.createProcessInstanceQuery()
    .processDefinitionKey("testProcess").list();
//查询活跃的流程实例，查询SUSPENSION_STATE_字段来判断是否中断
pis = runtimeService.createProcessInstanceQuery().active().list();
//通过businessKey查询流程实例
pis = runtimeService.createProcessInstanceQuery()
    .processInstanceBusinessKey("key2").list();
//根据流程实例ID查询
ProcessInstance pi=runtimeService.createProcessInstanceQuery()
    .processInstanceId(pi1.getId());
// 根据多个流程实例ID查询
Set<String> ids = new HashSet<String>();
ids.add(pi1.getId());
ids.add(pi2.getId());
pis = runtimeService.createProcessInstanceQuery().processInstanceIds(ids).list();
```