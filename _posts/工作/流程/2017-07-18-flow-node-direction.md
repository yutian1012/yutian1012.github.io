---
title: 流程节点跳转规则的设置
tags: [bpmx]
---

流程定义--节点设置--跳转规则设置

![](/images/work/bpmx/set/nodedirectionrule.png)

注：节点设置完成后，会将节点的跳转规则设置到bpm_node_rule表中。

### 执行过程

流程执行同意操作，调用taskController类的complete方法，进而调用processRunService类的nextProcess。

1）定位跳转位置

processRunService类的nextProcess，并且在后置处理器前，调用ProcessRunService类的completeTask方法（私有方法），进而调用JumpService类的evaluate方法，该方法中会执行流程节点设置的跳转规则。

2）定义跳转规则中流程变量的设置位置

processRunService类的nextProcess方法调用handFormData，进而调用FormDataUtil类的parseJson方法。FormDataUtil类的handleMain方法中会调用getVariables方法，获取并设置流程变量（从数据库中加载流程变量的设置信息），设置到bpmFormData.setVariables(variables)中。调用processCmd.putVariables(bpmFormData.getVariables())进一步将流程变量设置到ProcessCmd对象中。

将流程变量设置到流程中，ProcessRunService类的setVariables方法会将流程变量设置的流程中。

```
/**
 * 设置流程变量。
 * @param taskId
 * @param processCmd
 */
private void setVariables(TaskEntity entity, ProcessCmd processCmd) {
    if (BeanUtils.isEmpty(entity))
        return;
    String taskId = entity.getId();
    Map<String, Object> vars = processCmd.getVariables();
    if (BeanUtils.isNotEmpty(vars)) {
        for (Iterator<Entry<String, Object>> it = vars.entrySet().iterator(); it.hasNext();) {
            Entry<String, Object> obj = it.next();
            runtimeService.setVariable(entity.getProcessInstanceId(), obj.getKey(), obj.getValue());
        }
    }
    Map<String, Object> formVars = taskService.getVariables(taskId);
    formVars.put(BpmConst.IS_EXTERNAL_CALL, processCmd.isInvokeExternal());
    formVars.put(BpmConst.FLOW_RUN_SUBJECT, processCmd.getSubject());
    // 设置线程变量。
    TaskThreadService.setVariables(formVars);
    TaskThreadService.putVariables(processCmd.getVariables());
}
```

注：由runtimeService类调用setVaribale方法，设置到act_ru_variable表中。

