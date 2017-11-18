---
title: 流程表单设置
tags: [bpmx]
---

1、表单标题的显示

表单标题显示的是流程的描述信息，是在流程设计界面设计流程时填写的流程描述。

2、上传控件中在表单中隐藏后不校验

```
$("[name$='proposalSearchAttachment']")
    .closest("div[name='div_attachment_container']").hide();
$("[name$='proposalSearchAttachment']").removeAttr("validatable");
```

注：通过移除validatable属性，防止上传控件进行校验。

3、流程启动页面

```
/platform/bpm/task/startFlow.ht?actDefId=flow_proposal:1:10000017700342
```

4、流程子表数据

在配置流程子表时，一定要给子表设置主键，如果不设置主键，只有外键，并且外键作为关联时的主键时，则保存流程子表数据时，只会保存一条数据。