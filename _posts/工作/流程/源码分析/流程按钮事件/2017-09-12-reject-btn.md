---
title: 流程驳回按钮
tags: [bpmx]
---

1）按钮定位

taskToStart.jsp页面中引入了incToolBarNode.jsp工具栏页面，查看按钮事件

```
$("#btnReject").click(function(){
        setExtFormData();//封装表单数据
        
        operatorType=4;//按钮的操作类型，js中定义使用
        objVoteAgree.val(3);//$("#voteAgree")
        objBack.val(1);//$("#back");
        completeTaskBefore(); 
})
//completeTaskBefore会调用该方法
function completeTask(){
    var action="${ctx}/platform/bpm/task/complete.ht";
}
```

注：与同意按钮执行了相同的方法。

2）taskController类的complete方法

该方法会经过一系列的校验操作，最后执行ProcessRunService类的nextProcess方法。

3）processRunService的nextProcess方法

ProcessCmd中获取的驳回信息中包含startNode节点，即流程开始的第一个任务节点（不是startEvent节点）。该值是如何设置的？

在taskToStart.jsp中的隐藏域中存在startNode控件，该值是在taskController的toStart方法中查询到的，并在渲染taskToStart.jsp页面时设置到控件上的。即，在代办中执行任务时，startNode控件的默认值为toBackNodeId（）。

```
// 获取任务对象
TaskEntity taskEntity = getTaskEntByCmd(processCmd);
String taskStatus = taskEntity.getDescription();//-1表示正在审批

//ProcessRun对象
processRun = dao.getByActInstanceId(new Long(instanceId));

// 处理在线表单的业务数据
handFormData(processCmd, bpmNodeSet, processRun, parentNodeId);

//流程回退处理的前置处理，查找该任务节点回退的堆栈树父节点
//该方法内部会先获取父节点（循环遍历向上查找）
//向processCmd中设置destTask（nodeId信息）
//向线程中设置上一步节点的执行人。
ExecutionStack parentStack = backPrepare(processCmd, taskEntity, taskToken);

// 是否仅完成当前任务，方法内部调用bpmService类的transTo方法
//会清除当前节点的向后箭头，并创建一个到目标节点的连线（actviti中只有存在相应的方向箭头才能实现跳转）
//执行taskService类的complete方法完成任务
completeTask(processCmd, taskEntity, bpmNodeSet.getIsJumpForDef());

//任务回退时，弹出历史执行记录的堆栈树节点
if (processCmd.isBack() > 0 && parentStack != null) {
    //删除前一个节点的后续节点，即将后续节点出栈
    executionStackService.pop(parentStack);
} 

//处理消息

//发布事件，执行nodesql或启动子流程等
```

注：ExecutionStatck对象的parentId字段记录上一个堆栈对象的主键。

4）驳回到发起人

该功能与驳回的主要区别是在查找返回节点（下一步的目标节点）时的处理不同，以及执行堆栈的出栈节点的处理。具体查看processRunService类的backPrepare方法

```
 if(processCmd.isBack() == BpmConst.TASK_BACK_TOSTART) {
    // 获取发起节点。
    parentStack = executionStackService.getLastestStack(instanceId, backToNodeId, null);
    if (parentStack != null) {
        processCmd.setDestTask(parentStack.getNodeId());
        taskUserAssignService.addNodeUser(parentStack.getNodeId(), parentStack.getAssignees());
    }
}
```

注：parentStack直接根据ProcessCmd对象的startNode值查询出第一个用户任务节点。

弹出堆栈时，需要根据用户任务的第一个节点找到所有的后续节点，并将这些节点都清除。具体实现查看ExecutionStackService类的pop方法

```
public void pop(ExecutionStack parentStack){
    //删除当前堆栈。
    List<ExecutionStack> list=findChildsByParentId(parentStack);
    for(ExecutionStack stack:list){
        dao.delById(stack.getStackId());
    }
}
```

注：findChildsByParentId方法内部进行了递归操作，不断找出其后继节点，放置到list集合中，然后再从堆栈中删除。