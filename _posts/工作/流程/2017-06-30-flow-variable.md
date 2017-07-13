---
title: 流程中的流程变量
tags: [bpmx]
---

### 页面脚本中选择流程变量

![](/images/work/bpmx/flowvariableselect.png)

页面bpmDefinitionShowScript.jsp页面中获取流程变量的标签。

```
<f:flowVar defId="${defId}" controlName="selFlowVar" parentActDefId="${parentActDefId}"></f:flowVar>
```

标签定义位置：HtTag.tld文件，标签处理类

```
com.hotent.platform.tag.FlowVarsTag
....
List<BpmFormField> fieldList=bpmFormFieldService.getFlowVarByFlowDefId(defId);
List<BpmDefVar> bpmdefVars= bpmDefVarService.getVarsByFlowDefId(defId);
```

注：流程变量主要由两部分地方定义和获取，第一部分流程变量是在自定义表字段时定义的，保存在bpm_form_field表中，另一部分定义在流程定义中--变量管理中设置。

### 流程中设置流程变量

ProcessRunService类的setVariables方法会将processCmd对象中的变量值设置到流程变量中。

```
Map<String, Object> vars = processCmd.getVariables();
if (BeanUtils.isNotEmpty(vars)) {
    for (Iterator<Entry<String, Object>> it = vars.entrySet().iterator(); it.hasNext();) {
        Entry<String, Object> obj = it.next();
        runtimeService.setVariable(entity.getProcessInstanceId(), obj.getKey(), obj.getValue());
        //taskService.setVariable(taskId, obj.getKey(), obj.getValue());
    }
}
```

注：会调用org.activiti.engine.RuntimeService的setVariable设置流程变量。

### 流程变量无法在流程中获取

在启动流程/执行流程时，脚本中的方法参数无法获取到流程变量的值。以人员选择脚本为例，再运行时脚本方法参数是通过流程变量获取的，而在运行时参数值为null，并且代码中也未找到设置流程变量的地方。

修改可以借助setVariables方法，即调用runtimeService类的setVariables设置流程变量来实现，即扩展BpmUtil类的getProcessCmd方法，将相应的流程变量先查询出来，再设置到ProcessCmd类的variables属性中（map集合）。