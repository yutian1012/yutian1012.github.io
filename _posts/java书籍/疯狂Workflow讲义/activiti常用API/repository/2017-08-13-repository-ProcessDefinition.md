---
title: activiti流程定义管理
tags: [activiti]
---

### 实体对象

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

### 部署流程定义

```
//创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
//得到流程存储服务对象
RepositoryService repositoryService = engine.getRepositoryService();
//创建DeploymentBuilder实例
DeploymentBuilder builder = repositoryService.createDeployment();
builder.addClasspathResource("bpmn/processDeploy.bpmn").deploy();
```

### 问题

问题1：revision和version的区别？

分别表示数据的版本号（用于记录的更新或保存的判断），version表示流程的版本号信息（新旧流程定义）。

```
//创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
//得到流程存储服务对象
RepositoryService repositoryService = engine.getRepositoryService();
//创建DeploymentBuilder实例
DeploymentBuilder builder = repositoryService.createDeployment();
builder.addClasspathResource("bpmn/processDeploy.bpmn").deploy();

DeploymentBuilder builder2 = repositoryService.createDeployment();
builder2.addClasspathResource("bpmn/processDeploy.bpmn").deploy();
```

注：运行后会发现数据库表act_re_procdef表中有两条流程定义信息，version分别为1和2

问题2：resourceName，diagramResourceName，hasStartFormKey字段的意义？

流程定义对象ProcessDefinition存储到act_re_procdef表中，其中Resource_name字段值为部署时指定的资源路径（bpmn/processDeploy.bpmn），并且与act_ge_bytearray表的Name_字段值相同。act_re_procdef表的DGRM_RESOURCE_NAME_与act_ge_bytearray表的name_相同。

注：一个流程定义文件部署后会对应两个资源对象（act_re_bytearray表的两条记录），分别是流程定义文件xml（资源名称resouceName）和流程图文件png图片（diagramResourceName）。

### 流程定义的中止和激活

```
// 部署流程描述文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/suspendProcessDef.bpmn")
    .deploy();
//查询流程定义实体
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();  
// 调用suspendProcessDefinitionById中止流程定义 
repositoryService.suspendProcessDefinitionById(def.getId());
// 调用activateProcessDefinitionById激活流程定义 
repositoryService.activateProcessDefinitionById(def.getId());
// 调用suspendProcessDefinitionByKey中止流程定义 
repositoryService.suspendProcessDefinitionByKey(def.getKey());
// 调用activateProcessDefinitionByKey激活流程定义 
repositoryService.activateProcessDefinitionByKey(def.getKey());
```

注：提供了根据流程定义ID和流程定义Key值的中止和激活方法。流程定义ID是指ProcessDefinition的主键（act_re_procdef表的ID_字段值）。流程定义key是指ProcessDefinition的key属性（act_re_procdef表的KEY_字段值），也是流程定义描述文件xml中的proces标签的id属性值。