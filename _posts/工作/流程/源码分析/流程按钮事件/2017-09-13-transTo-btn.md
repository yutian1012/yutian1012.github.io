---
title: 流程加签
tags: [bpmx]
---

流程加签功能其实是实现流程流转，流程流转结束后可以直接提交也可以返回。

实现过程：

更新任务状态为流转中（39）；删除沟通任务；新建并保存BpmProTransTo对象；产生流转任务（Task），并设置task的description字段为流转（38）；添加任务意见信息（加签时设置的备注信息）；保存消息接收人信息（CommuReceiver对象，用来了解任务的已读和未读信息）；发生通知；

1）按钮定位

taskToStart.jsp页面中引入了incToolBarNode.jsp工具栏页面，查看按钮事件

```
//显示流转意见
function showTaskTransTo(conf) {
    //获取表单数据
    var obj = {data:CustomForm.getData()};
    isTaskEnd(function(){
        if(!conf) conf={};
        var url=__ctx + "/platform/bpm/task/toTransTo.ht?taskId=${task.id}&curUserId=${curUserId}";
        
        url=url.getNewUrl();
        
        DialogUtil.open({
            height:500,
            width: 550,
            title : '显示流转意见',
            url: url, 
            isResize: true,
            //自定义参数
            obj: obj,
            sucCall:function(rtn){
                if(rtn=="ok"){
                    handJumpOrClose();
                }
            }
        });
    })
}
//判断任务是否结束了，如果没有结束则可以设置加签操作
function isTaskEnd(callBack){
    var url="${ctx}/platform/bpm/task/isTaskExsit.ht";
    var params={taskId:"${task.id}"};
    
    $.post(url,params,function(responseText){
        var obj=new com.hotent.form.ResultMessage(responseText);            
        if(obj.isSuccess()){
            callBack.apply(this);
        }
        else{
            $.ligerDialog.warn("这个任务已经完成或被终止了!",'提示');
        }
    });
}
```

2）加签操作

访问路径

```
/platform/bpm/task/toTransTo.ht?taskId=${task.id}&curUserId=${curUserId}"
```

taskController类的toTransTo方法获取了系统中支持的消息发送方式，而没有使用传递的taskId和curUserId字段。这些字段是在taskToTransTo.jsp页面中使用的。

```
var taskId=${param.taskId};
var curUserId = ${param.curUserId};
```

页面设置：

页面设置添加的用户；添加节点的类型（会签或者非会签，会签要求添加的人员都要审批）；流转结束后的处理（结束后返回或者直接提交任务）；提醒方式等。

保持设置：

```
${ctx}/platform/bpm/task/toStartTransTo.ht
```

3）TaskController类的toStartTransTo方法

```
//获取流程设置参数
String cmpIds = request.getParameter("cmpIds");//添加人员ID
String taskId = request.getParameter("taskId");//任务ID
String opinion = request.getParameter("opinion");//意见信息--备注
String informType = RequestUtil.getString(request, "informType");//提醒消息方式
String transType = request.getParameter("transType");//流转类型，会签/非会签
String action = request.getParameter("action");//流程结束后的操作

//保存流转信息
processRunService.saveTransTo(taskEntity, opinion, informType, cmpIds, transType, action, processRun);

//保持业务数据
processRunService.handlerFormData(taskCmd, processRun, taskEntity.getTaskDefinitionKey());
```

注：taskService直接调用saveTask方法不会执行任务创建监听（TaskCreateListener），任务监听的设置是在xml中节点下配置的，这里直接创建的任务并没有任何的监听设置。

```
<userTask id="task1" name="填写请假流程">
    <documentation/>
    <extensionElements>
        <activiti:taskListener event="create" delegateExpression="${taskCreateListener}"/>
        <activiti:taskListener event="assignment" delegateExpression="${taskAssignListener}"/>
        <activiti:taskListener event="complete" delegateExpression="${taskCompleteListener}"/>
        <ht:order>0</ht:order>
    </extensionElements>
</userTask>
```

4）ProcessRunService类的saveTransTo方法

```
// 修改初始任务状态
taskDao.updateTaskDescription(TaskOpinion.STATUS_TRANSTO_ING.toString(), taskId);
// 删除沟通任务
taskDao.delCommuTaskByParentTaskId(taskId);

// 保存流转对象
Long id = UniqueIdUtil.genId();
BpmProTransTo bpmProTransTo = new BpmProTransTo();
...
bpmProTransToService.add(bpmProTransTo);

// 产生流转任务
Map<Long, Long> usrIdTaskIds = bpmService.genTransToTask(taskEntity, aryUsers);

// 增加流程意见
TaskOpinion taskOpinion = new TaskOpinion();
...
taskOpinionDao.add(taskOpinion);

// 保存接收人
commuReceiverService.saveReceiver(opinionId, usrIdTaskIds, sysUser);
// 发送通知。
notifyCommu(processRun.getSubject(), usrIdTaskIds, informType, sysUser, opinion, SysTemplate.USE_TYPE_TRANSTO);
```