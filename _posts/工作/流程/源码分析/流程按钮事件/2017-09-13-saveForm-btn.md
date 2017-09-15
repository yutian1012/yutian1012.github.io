---
title: 流程保存表单
tags: [bpmx]
---

该功能最主要的工作是完成表单数据的保存，至于流程意见的处理这里显得有些多余。保持表单后下次再办理的话，是在已修改的表单上来操作的。

1）按钮定位

taskToStart.jsp页面中引入了incToolBarNode.jsp工具栏页面，查看按钮事件

```
$("#btnSave").click(function(){
    setExtFormData();//设置表单
    ...
    saveData();
})
function saveData(){
    var action="${ctx}/platform/bpm/task/saveData.ht";
}
```

2）taskController类的saveData方法

```
//从请求中获取ProcessCmd命令对象
ProcessCmd processCmd = BpmUtil.getProcessCmd(request);
//如果runId不空，则查询ProcessRun对象
ProcessRun processRun = processRunService.getById(runId);
//将ProcessRun对象设置到processCmd中
processCmd.setProcessRun((IProcessRun) processRun);
//执行保存数据操作
processRunService.saveData(processCmd);
```

3）ProcessRunService类的saveData方法

```
public void saveData(ProcessCmd processCmd) throws Exception {
    String taskId = processCmd.getTaskId();
    ProcessRun processRun = null;
    BpmNodeSet bpmNodeSet = null;
    //判断taskId是否为空，如果为空则表示保存表单按钮时在开始节点被触发的
    //开始节点没有该按钮，开始节点只有保存草稿
    if (StringUtil.isEmpty(taskId)) {
        processRun = (ProcessRun) processCmd.getProcessRun();
        //获取开始节点
        bpmNodeSet = getStartBpmNodeSet(processRun, "");
    } else {
        TaskEntity task = bpmService.getTask(taskId);
        String actInstId = task.getProcessInstanceId();
        String actDefId = task.getProcessDefinitionId();
        String nodeId = task.getTaskDefinitionKey();
        processRun = getByActInstanceId(Long.parseLong(actInstId));
        //更新或添加意见（有点多余）
        String opinion = processCmd.getVoteContent();
        saveOpinion(task, opinion);
        //获取父流程
        String parentActDefId = (String) taskService.getVariableLocal(taskId, BpmConst.FLOW_PARENT_ACTDEFID);
        //获取节点设置信息，从而决定是否设置了前置处理器
        if (StringUtil.isEmpty(parentActDefId)) {
            bpmNodeSet = bpmNodeSetService.getByActDefIdNodeId(actDefId, nodeId);
        } else {
            bpmNodeSet = bpmNodeSetService.getByActDefIdNodeId(actDefId, nodeId, parentActDefId);
        }
        // 在流程审批时，更新流程变量
        setVariables(task, processCmd);
    }
    // 处理业务表单
    if (!processCmd.isInvokeExternal()) {
        handlerFormData(processCmd, processRun, bpmNodeSet.getNodeId());
    }
    // 调用前置处理器
    if (!processCmd.isSkipPreHandler()) {
        invokeHandler(processCmd, bpmNodeSet, true);
    }
}
```

4）保存表单问题

一个任务同时又多个候选人，不同候选人都可以保存表单，其他人查看到的数据是已经保存了的数据。这样会造成数据冲突。

解决方案，保存表单后，相当于用户认领了任务，其他用户不能再办理。