---
title: 流程操作
tags: [activiti]
---

RuntimeService提供的API可以进行启动流程，发送信号，中断流程和激活流程等操作。

1）发送信号

RuntimeService类的signal方法能够让流程向下执行。但与TaskService的complete方法是有区别的。如果当前流程节点为任务节点，signal方法并不会将该任务节点完成（不会将任务数据存放到历史数据中，也不会删除该节点的任务数据），而是让执行流继续往下进行。而TaskService的complete方法在完成任务后，会将相关的任务数据删除，放到历史数据表中，然后让执行流继续往下进行。

注：不管是使用singal还是complete方法，流程结束后，就会将流程相关的数据存放到相应的历史表中。

```
runtimeService.signal(execution.getId());
runtimeService.signal(execution.getId(), vars);
```

注：singal方法需要接收执行流ID作为参数，另外也可以设置相应的流程参数，singal设置的参数会作用于整个流程，而不仅仅作用于某个执行流。

注2：signal方法传递的参数是执行流，当子执行流结束后，子执行流对应的任务信息会被清除。

注3：如果某个任务节点被signal方法执行了，再被complete方法执行就会抛出异常。

2）触发信号事件

事件节点是流程中记录事件发生的流程元素。与runtimeService相关的事件触发方法包括signalEventReceived抛出信号和messageEventReceived方法抛出消息，从而触发相应的事件，使流程继续执行下去。

3）中断与激活流程

表ACT_RU_EXECUTION使用SUSPENSION_STATE_字段来保存流程的中断状态。值为1表示流程为活跃状态，值为2表示流程为中断状态。中断的是流程实例，而不会设置相关的执行流的SUSPENSION_STATE_字段值。

runtimeService提供了中断流程（suspendProcessInstanceById）和激活流程（activateProcessInstanceById）方法。

```
// 中断流程
runtimeService.suspendProcessInstanceById(pi.getId());
// 激活流程
runtimeService.activateProcessInstanceById(pi.getId());
```

注：流程中断后，仍然可以使用signal或者complete方法让流程继续执行下去。Activiti将流程是否可以往下执行的选择权交给使用人决定。

4）删除流程实例

流程实例被删除后，相关的数据将会从原来运行时的表中删除，将这部分数据存放到历史表中。包括流程实例，流程任务和流程节点等。

删除流程实例时，需要传递流程实例ID以及删除原因字符串

```
// 删除流程
runtimeService.deleteProcessInstance(pi.getId(), "abc");
```