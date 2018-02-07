---
title: activiti流程图部署
tags: [activiti]
---

### 流程图部署

1）添加流程图资源

使用addClasspathResource方法将流程描述文件和流程图部署到数据库中。最终会在文件数据表ACT_GE_BYTEARRAY中记录两条记录，一条为流程描述文件数据xml，另外一条为流程图片的文件数据。

注：部署流程定义文件时，如果不提供流程图文件的addClasspathResource方法调用，会自动根据BPMN规范的xml流程定义文件生成流程图，并保存到数据库中。

2）流程图的匹配规则

如果显示的部署多个流程图，流程定义如何实现最佳的匹配呢（DGRM_RESOURCE_NAME_字段存储什么值）？

```
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/diagram.bpmn")
    .addClasspathResource("bpmn/diagram.png")
    .addClasspathResource("bpmn/diagram.vacationProcess.png").deploy();
//查询流程定义实体
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 输出结果为 bpmn/diagram.vacationProcess.png
System.out.println(def.getDiagramResourceName());
```

匹配规则：

优先查找与流程描述文件xml同名并且后缀为流程ID的图片文件。

如果根据第一条规则查找不到相应的流程文件，那么与流程描述文件xml同名的图片文件。

解析：

这里传递了两个图片bpmn/diagram.png和bpmn/diagram.vacationProcess.png。

最终会匹配bpmn/diagram.vacationProcess.png图片文件

打开bpmn/diagram.bpmn可以发现xml中的流程ID值为：vacationProcess

```
<process id="vacationProcess" name="vacation">
```

因此，这里最终匹配到的流程图为bpmn/diagram.vacationProcess.png

3）查询流程部署图名称

通过ProcessDefinitionQuery查询对象来查询得到流程定义数据。

```
//查询流程定义实体，这里根据deployemnt的ID来查询
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 输出结果为 bpmn/diagram.vacationProcess.png
System.out.println(def.getDiagramResourceName());
```

查看数据库表act_re_procdef表记录可以发现DGRM_RESOURCE_NAME_字段值为bpmn/diagram.vacationProcess.png