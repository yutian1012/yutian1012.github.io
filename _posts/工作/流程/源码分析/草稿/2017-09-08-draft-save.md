---
title: 流程中保存草稿
tags: [bpmx]
---

1）页面组件

流程编辑页面：taskStartFlowForm.jsp，流程工具页面：incToolBarStart.jsp。

```
<div class="group"><a class="link save isDraft"><span></span>保存草稿</a></div>
```

保存草稿的js位于taskStartFlowForm.jsp文件中

```
//保存表单
$("a.save").click(function(){
    saveForm(this);
});

function saveForm(obj){
    isStartFlow=false;
    var  action="";
    //判断控件是否包含isDraft的class属性来执行不同的逻辑
    if($(obj).hasClass('isDraft')){
        action="${ctx}/platform/bpm/task/saveForm.ht";
    }else{
        action="${ctx}/platform/bpm/task/saveData.ht";
    }
    submitForm(action,"a.save");
}
```

2）TaskController类的saveForm方法保存草稿

```
//将请求参数封装成ProcessCmd对象
ProcessCmd processCmd = BpmUtil.getProcessCmd(request);
//保存草稿
processRunService.saveForm(processCmd);
```

3）ProcessRunService类的saveForm方法

```
public void saveForm(ProcessCmd processCmd) throws Exception {
    ...
    //初始化对象
    ProcessRun processRun = initProcessRun(bpmDefinition, processCmd);
    ...
    // 调用前置处理器
    if (!processCmd.isSkipPreHandler()) {
        invokeHandler(processCmd, bpmNodeSet, true);
    }
    ...
    //保存processRun对象到数据库
    this.add(processRun);
    // 调用前置处理器
    if (!processCmd.isSkipPreHandler()) {
        invokeHandler(processCmd, bpmNodeSet, true);
    }

    //会调用流程节点的sql设置，并执行相应的sql命令。
    EventUtil.publishNodeSqlEvent(actDefId, nodeId, BpmNodeSql.ACTION_SAVE, (Map) processCmd.getTransientVar("mainData"));

    ...
    //记录运行日志
    bpmRunLogService.addRunLog(processRun.getRunId(), BpmRunLog.OPERATOR_TYPE_SAVEFORM, memo);
}
```

注：该方法会保存业务数据，并保存bpmBusLink业务数据关联信息，已经存储ProcessRun对象，并且会调用流程节点设置的前置和后置处理方法，以及节点设置的sql信息。