---
title: 流程跳过第一个节点
tags: [bpmx]
---

1）设置方式

在流程启动时，可以通过设置（流程定义管理--其他参数）是否跳过第一个节点。

2）实现步骤：

启动流程（startWorkFlow方法获取ProcessInstance），流程会自动生成下一步的节点任务。
在TaskCreateListener任务创建监听中会将任务设置到ThreadLocal线程管理中（TaskThreadService.addTask((TaskEntity)delegateTask)）
在TaskCreateListener监听中同时会为任务分配执行人，从ThreadLocal线程管理中获取（taskUserAssignService.getNodeUserMap()）

3）代码逻辑

查看ProcessRunService类的startProcess方法

```
// 是否跳过第一个任务节点，当为1 时，
//启动流程后完成第一个任务。流程定义设置的其他参数设置
Short toFirstNode = bpmDefinition.getToFirstNode();
//使用本地线程变量存放设置值（是否跳过的设置）
TaskThreadService.setToFirstNode(toFirstNode);
//如果设置为跳过第一个任务节点，设置该节点在执行人
if (toFirstNode == 1) {
    List<TaskExecutor> excutorList = new ArrayList<TaskExecutor>();
    //获取执行人，即将当前用户设置为任务执行人
    excutorList.add(TaskExecutor.getTaskUser(sysUser.getUserId().toString(), sysUser.getFullname()));
    //将该节点的执行人员设置到nodeUserMapLocal对象中（本地线程变量），去掉该对象中原来的nodeId对应的执行人列表（这些值来自于该流程节点的人员设置）
    //任务分配时使用
    taskUserAssignService.addNodeUser(nodeId, excutorList);
}
...
//执行启动流程
ProcessInstance processInstance = startWorkFlow(processCmd);
...
// 获取下一步的任务并完成跳转。
if (toFirstNode == 1) {
    handJumpOverFirstNode(processInstanceId, processCmd);
}

// 更新流程历史实例为 开始标记为1。
if (toFirstNode == 1) {
    historyActivityDao.updateIsStart(Long.parseLong(processInstanceId), nodeId);
}
...
// 更新流程历史实例为 开始标记为1。
if (toFirstNode == 1) {
    historyActivityDao.updateIsStart(Long.parseLong(processInstanceId), nodeId);
}
```

4）执行跳过的任务节点

查看ProcessRunService类的handJumpOverFirstNode方法

```
//处理任务跳转
private void handJumpOverFirstNode(String processInstanceId, ProcessCmd processCmd) throws Exception {
    //从ThreadLocal线程管理中获取生成的任务对象
    List<Task> taskList = TaskThreadService.getNewTasks();
    //情况ThreadLocal线程
    TaskThreadService.clearNewTasks();

    TaskEntity taskEntity = (TaskEntity) taskList.get(0);
    //任务节点定义信息，如UserTask2
    String taskDefKey = taskEntity.getTaskDefinitionKey();

    // 填写第一步意见。将意见信息放置到processCmd对象中
    processCmd.getVariables().put(BpmConst.NODE_APPROVAL_STATUS + "_" + taskDefKey, TaskOpinion.STATUS_SUBMIT);
    processCmd.getVariables().put(BpmConst.NODE_APPROVAL_CONTENT + "_" + taskDefKey, "填写表单");
    processCmd.setVoteAgree(TaskOpinion.STATUS_SUBMIT);

    // 设置流程变量。
    setVariables(taskEntity, processCmd);

    //执行下一步任务
    skipTask(taskEntity, processCmd);
}
```

注：skipTask中会调用ProcessRunService类的nextProcess方法。

5）执行完成下一步后更新流程历史实例为开始标记

```
historyActivityDao.updateIsStart(Long.parseLong(processInstanceId), nodeId);
```

注：该类会直接修改ACT_HI_ACTINST表的ISSTART字段（历史节点表）。该表存放历史完成的活动信息

注2：ISSTART字段是hotent自己扩展的字段。