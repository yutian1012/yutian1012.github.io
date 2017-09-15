---
title: 流程沟通按钮
tags: [bpmx]
---

1）按钮定位

taskToStart.jsp页面中引入了incToolBarNode.jsp工具栏页面，查看按钮事件

```
//显示沟通意见。
function showTaskCommunication(conf){
    //表单数据
    var obj = {data:CustomForm.getData()};
    //判断任务是否已经处理完成了，并设置回调
    isTaskEnd(function(){
        if(!conf) conf={};
        //输入子页面
        var url=__ctx + "/platform/bpm/task/toStartCommunicate.ht?taskId=${task.id}";
        ...
    })
}
```

2）查看taskToStartCommunicate.jsp页面

在弹出框中点击沟通按钮

```
//参数设置，cmpIds沟通人员信息，opinion沟通内容，informType提醒消息类型，taskId任务ID，formData业务表单信息
var params= {cmpIds:cmpIds.val(),
     cmpNames:cmpNames.val(),
     opinion:taskOpinion,
     informType:informType,
     taskId:taskId,
     formData:formData};
var url="${ctx}/platform/bpm/task/toStartCommunication.ht";
```