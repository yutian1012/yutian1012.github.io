---
title: 流程反馈按钮--针对非会签节点
tags: [bpmx]
---

实现功能：存储表单；存储意见；删除加签任务；更新加签任务的上一个任务状态；提醒消息；添加act活动历史；删除流转记录bpmProTransTo对象。

1）按钮定位

taskToStart.jsp页面中引入了incTaskTransTo.jsp工具栏页面，查看按钮事件

```
function openOpinionDialog(){
    ...
    //
    var obj = {data:CustomForm.getData()};
    var url=__ctx + "/platform/bpm/task/transToOpinionDialog.ht?taskId=${task.id}";
    ...
    url=url.getNewUrl();
    ...   
}
```

2）意见反馈表单页面

taskTransToOpinionDialog.jsp页面，执行同意和反对功能

```
function save(isAgree){
    var rtn=$("#frmComm").form().valid();
    if(!rtn) return;
    var content=$("#opinion").val();//提交的意见信息
    var informType=$.getChkValue("informType");//通知类型
    
    var params= {isAgree:isAgree,opinion:content,informType:informType,taskId:taskId,formData:formData};

    var url=__ctx +"/platform/bpm/task/saveOpinion.ht";
    ...
}
```

注：同意时执行save方法传递的isAgree为true，反对时为false。

3）查看taskController类的saveOpinion方法

该方法用来保存沟通/流转意见。

```
public String saveOpinion(HttpServletRequest request, HttpServletResponse response, TaskOpinion taskOpinion) throws Exception {
    String informType = RequestUtil.getString(request, "informType");
    boolean isAgree = RequestUtil.getBoolean(request, "isAgree");
    ProcessCmd taskCmd = BpmUtil.getProcessCmd(request);
    taskCmd.setVoteContent(taskOpinion.getOpinion());//设置意见
    ...

    //处理流转操作
    processRunService.handTransTo(taskOpinion, informType, isAgree, taskCmd);
}
```

注：页面传递的参数opinion，taskId都封装到了TaskOpinion对象中了；isAgree和informType都是单独获取的；formData信息封装到了taskCmd中。

4）processRunService类的handTransTo方法

```
...
//设置taskOpinion字段信息
taskOpinion.setActDefId(taskEntity.getProcessDefinitionId());
...

// 向原执行人发送任务完成提醒消息
Map<Long, Long> usrIdTaskIds = new HashMap<Long, Long>();
//加签任务的parentTaskId字段会记录提起加签的任务主键
ProcessTask parentTask = getByTaskId(Long.valueOf(taskEntity.getParentTaskId()));
//向集合中添加发送人和相关的taskId（提起加签的任务）
usrIdTaskIds.put(Long.valueOf(parentTask.getAssignee()), Long.valueOf(taskEntity.getParentTaskId()));
//发送通知
notifyCommu(processRun.getSubject(), usrIdTaskIds, informType, sysUser, taskOpinion.getOpinion(), sysTemplateType);

//处理业务数据
BpmFormData bpmFormData = handlerFormData(taskCmd, processRun, taskEntity.getTaskDefinitionKey());

// 保存反馈信息，添加意见 这里面会删除加签节点对应task的数据，同时删除该加签节点生成的沟通任务
saveOpinion(taskEntity, taskOpinion);

// 设置沟通人员或流转人员的状态为已反馈
commuReceiverService.setCommuReceiverStatusToFeedBack(taskEntity, sysUser);

// 添加已办历史，向act_hi_actinst表中添加记录--存放历史所有完成的活动
addActivityHistory(taskEntity);

//获取流转记录--bpm_pro_transto表记录
BpmProTransTo bpmProTransTo = bpmProTransToService.getByTaskId(Long.valueOf(parentTaskId));
//处理流转信息，更新上一个任务的状态description值为38（从流转中变为流转）
//对于非会签任务，需要根据上一个任务的主键删除相关其余加签任务节点
//删除流转记录bpmProTransTo
handlerTransTo(taskCmd, bpmProTransTo, sysUser, parentTaskId, isAgree);
...

//记录日志
bpmRunLogService.addRunLog(runId, BpmRunLog.OPERATOR_TYPE_ADDOPINION, memo);
```