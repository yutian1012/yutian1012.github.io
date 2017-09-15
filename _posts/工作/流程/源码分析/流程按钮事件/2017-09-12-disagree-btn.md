---
title: 流程反对按钮
tags: [bpmx]
---

流程反对功能的实现与流程执行同意是一样的，都会时流程继续执行下去，只会改变流程的节点的显示信息。与驳回不同，驳回实现的功能是流程驳回到前一个节点，由前一个节点执行人执行代办。

1）按钮定位

taskToStart.jsp页面中引入了incToolBarNode.jsp工具栏页面，查看按钮事件

```
$("#btnNotAgree").click(function(){
    setExtFormData();//设置提交表单内容
    //该功能只针对会签节点有效，不同节点无效。
    var isDirectComlete = getIsDirectComplete();
    operatorType=2;
    //反对；2普通节点的反对，6会签节点的一票否定，
    var tmp =isDirectComlete?'6':'2';
    objVoteAgree.val(tmp);
    objBack.val("0");
    completeTaskBefore();
});
//completeTaskBefore会调用该方法
function completeTask(){
    var action="${ctx}/platform/bpm/task/complete.ht";
}
```

注：普通节点不支持一票通过和一票否决，如果要想实现该功能可以使用网关分支，通过判断来实现该功能。

2）taskController类的complete方法

该方法会经过一系列的校验操作，最后执行ProcessRunService类的nextProcess方法。

3）processRunService的nextProcess方法

执行过程参考同意，执行完成后流程会继续向下执行。只是流程查看时会显示不同的状态和审批意见。

