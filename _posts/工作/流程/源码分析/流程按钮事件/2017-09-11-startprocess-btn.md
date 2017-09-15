---
title: 流程启动流程按钮
tags: [bpmx]
---

主要实现过程：1根据流程定义判断流程是否能正常启动；2初始化processRun对象；3处理业务表单数据，并向processRun中设置businessKey；4启动流程获取ProcessInstance对象，并向ProcessRun中设置actInstId值；5初始化执行堆栈，记录待执行的任务信息；6保存ProcessRun对象到数据库中；7记录运行期节点设置的运行表单；8发送消息。

1）按钮定位

启动流程，taskStartFlowForm.jsp页面，查看startWorkFlow方法，提交地址

```
${ctx}/platform/bpm/task/startFlow.ht
```

2）taskController类的startFlow方法

启动流程会判断是否是重草稿启动，还是直接启动。

```
ProcessCmd processCmd = BpmUtil.getProcessCmd(request);
//runId不为空，则使用草稿方式启动
Long runId = RequestUtil.getLong(request, "runId", 0L);
if (runId != 0L) {
    ProcessRun processRun = processRunService.getById(runId);
    //将流程运行对象设置到processCmd（流程命令对象中）
    processCmd.setProcessRun((IProcessRun) processRun);
}
//不管是否是使用草稿启动，都使用startProcess方式启动流程。
processRunService.startProcess(processCmd);
```

注：草稿启动和直接启动的唯一区别是ProcessCmd对象中的processRun成员变量是否为空。

3）ProcessRunService类的startProcess方法

```
// 通过流程key获取流程定义，processCmd中能够获取相应的flowkey
BpmDefinition bpmDefinition = getBpmDefinitionByCmd(processCmd);
...
//跳过第一个节点

//如果processRun为null，则说明不是通过草稿启动的流程，初始化ProcessRun对象
if (BeanUtils.isEmpty(processRun)) {
    processRun = initProcessRun(bpmDefinition, processCmd);
}

//从已完成的流程实例启动，这部分代码可以忽略，系统中基本没有使用
// 处理关联流程实例CMD对象。重点是relRunId对应的ProcessRun对象的业务主键businessKey设置到processCmd对象中
handRelRun(processCmd, processRun);

// 处理业务表单数据，将相应的关联信息设置到processRun对象上。启动流程后，处理业务表单，外部调用不触发表单处理。
//保持数据到业务表中，add还是update根据关联数据表（bpmBusLinkService.checkBusExist(businessKey)）来判断。
handForm(processCmd, processRun);

//processCmd中设置变量信息：流程标题，主流程ID，流程runId（系统生成），当前组织
//processRun中设置信息：流程标题，运行状态，组织名称，组织id，流程目标节点

//通过processCmd对象启动流程
ProcessInstance processInstance = startWorkFlow(processCmd);

//更新或添加流程实例
if (BeanUtils.isEmpty(processCmd.getProcessRun())) {
    this.add(processRun);
} else {
    this.update(processRun);
}

// 初始化执行的堆栈树，taskList是流程启动后产生的下一个节点的任务信息
executionStackService.initStack(processInstanceId, taskList, 0L);

//添加运行期表单，外部调用时不对表单做处理
//会根据BpmNodeSet对象构建BpmFormRun对象
//将当前最新的表单配置信息添加到表单运行情况。
//之后的流程表单从表单运行情况表中取值
bpmFormRunService.addFormRun(actDefId, processRun.getRunId(), processInstanceId);

//消息处理--发送通知消息
taskMessageService.notify(TaskThreadService.getNewTasks(), "", processRun.getSubject(), null, "", "");

//执行节点sql配置信息
EventUtil.publishNodeSqlEvent(actDefId, nodeId, BpmNodeSql.ACTION_SUBMIT, (Map) processCmd.getTransientVar("mainData"));

//如果不跳过第一个任务节点则添加审批历史信息
addSubmitOpinion(processRun, processCmd);
```

注：handForm方法除了会将数据保存到业务表中，还有一个主要的操作时将业务主键信息设置到processCmd和processRun对象中。

4）根据节点的设置执行前置/后置处理器

```
//在发起流程时需要跳转到的目标节点。不是指开始节点，而是由人员参与的第一个节点
String nodeId = processCmd.getStartNode();
BpmNodeSet bpmNodeSet = getStartBpmNodeSet(defId, nodeId);//获取节点的设置

// 调用前置处理器
if (!processCmd.isSkipPreHandler()) {
    invokeHandler(processCmd, bpmNodeSet, true);
}
...
// 后置处理器
if (!processCmd.isSkipAfterHandler()) {
    invokeHandler(processCmd, bpmNodeSet, false);
}
```
