---
title: activiti流程定义管理
tags: [activiti]
---

1）实体对象

ProcessDefinition是一个接口，一个ProcessDefinition实例表示一条流程定义数据。实体类为ProcessDefinitionEntity，对应的数据表为ACT_RE_PROCDEF

ProcessDefinitionEntity中定义的属性

```
//流程定义文件xml中的process节点的id值，对应ACT_RE_PROCDEF表的KEY_字段
//<process id="xxxx"...>
protected String key;
//该条数据的版本号，对应REV_字段
protected int revision = 1;
//流程定义的版本号，ACT_RE_PROCDEF表的VERSION_字段
protected int version;
//解析流程描述文件xml根节点targetNamespaces属性值，ACT_RE_PROCDEF表的CATEGORY_字段
protected String category;
//表示该流程定义是由哪次流程部署产生的，ACT_RE_PROCDEF表的DEPLOYMENT_ID_字段，与ACT_RE_DEPLOYMENT表的主键对应（不是外键关系）
protected String deploymentId;
//流程描述文件对应的资源名称，对应ACT_RE_PROCDEF表的RESOURCE_NAME_字段
protected String resourceName;
//流程对应的流程图文件名称，ACT_RE_PROCDEF表的DGRM_RESOURCE_NAME_
protected String diagramResourceName;
//流程的开始事件是否存在，流程定义文件xml中的activiti:formKey的定义，对应的字段为HAS_START_FORM_KEY_。
protected boolean hasStartFormKey;
//流程状态，激活状态为1，终端状态值为2，对应表ACT_RE_PROCDEF的SUSPENSIONG_STATE字段
protected int suspensionState = SuspensionState.ACTIVE.getStateCode();
```

ProcessElementImpl类中定义的属性

```
//流程定义数据的主键，值为流程定义的KEY_字段+流程定义的VERSION_+ID生成器生成的ID值，三个值之间是有冒号分隔，对应ACT_RE_PROCDEF表的ID_字段
protected String id;
```

ProcessDefinitionImpl类中定义的属性

```
//流程定义文件xml中process节点的name属性，对应ACT_RE_PROCDEF表的NAME_字段
protected String name;
```

问题1：revision和version的区别？

分别表示数据的版本号（用于记录的更新或保存的判断），version表示流程的版本号信息（新旧流程定义）。

问题2：resourceName，diagramResourceName，hasStartFormKey字段的意义？