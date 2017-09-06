---
title: repositoryService数据查询与删除
tags: [activiti]
---

DeploymentBuilder可以将部署相关的文件、流程描述文件和流程图等保存到数据库中。这些相关的部署资源都是可以查询获取的。

1）获取部署资源

getResourceAsStream方法需要提供部署ID和资源名称，返回资源的输入流对象。该方法实际上是到ACT_GE_BYTEARRAY表中查询DEPLOYMENT_ID_与NAME_字段与该方法的参数值相符的数据。

```
// 部署一份txt文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("artifact/GetResource.txt").deploy();
// 查询资源文件
InputStream is = repositoryService.getResourceAsStream(dep.getId(), 
    "artifact/GetResource.txt");
```

注：addClasspathResource方法会以文件的路径作为该资源数据的名称。

2）查询流程文件

部署露出描述文件xml时，Activiti会喜爱那个ACT_GE_BYTEARRAY表中写入该文件的内容，即先将其看作一份普通的文件进行解析和保存，然后在解析它的文件内容，并生成相应的流程定义数据。

getProcessModel方法，只需要提供流程定义的ID，即可返回流程文件的InputStream实例。首先根据流程定义ID到ACT_RE_PROCDEF表中获取RESOURCE_NAME_和DEPLOYMENT_ID_字段，然后再根据这两个字段信息到ACT_GE_BYTEARRAY表中获取流程定义资源。

```
//查询流程定义实体
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 查询资源文件
InputStream is = repositoryService.getProcessModel(def.getId());
```

注：获取流程定义的描述文件xml。

3）查询流程图

部署流程时，会自动根据流程定义文件xml生成流程图，如果在部署流程定义文件时同时提供了流程图（流程图规则），则不会自动生成流程图，而是采用部署提供的流程图资源。

getProcesssDiagram方法返回该流程图的输入流实例。传递流程定义ID作为参数，首先从ACT_RE_PROCDEF表中获取DGRM_RESOURCE_NAME_和DEPLOYMENT_ID_字段，然后根据这两个字段的值到ACT_GE_BYTEARRAY表中获取流程图资源。

```
// 部署一份流程文件与相应的流程图文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/getProcessDiagram.bpmn")
    .addClasspathResource("bpmn/getProcessDiagram.png").deploy();
// 查询流程定义
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 查询资源文件
InputStream is = repositoryService.getProcessDiagram(def.getId());
// 将输入流转换为图片对象  
BufferedImage image = ImageIO.read(is);
// 保存为图片文件
File file = new File("resource/artifact/result.png");
if (!file.exists()) file.createNewFile();
FileOutputStream fos = new FileOutputStream(file);
ImageIO.write(image, "png", fos);
fos.close();
is.close();
```

注：javax.imageio.ImageIO类，由jdk提供

4）查询部署资源名称

getDeploymentResourceNames方法，查询一次部署所涉及的全部文件，需要该方法传入部署ID。

```
// 部署一份流程文件与相应的流程图文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/GetResourceNames.bpmn")
    .addClasspathResource("bpmn/GetResourceNames.png").deploy();
// 查询资源文件名称集合
List<String> names = repositoryService.getDeploymentResourceNames(dep.getId());
for (String name : names) {
    System.out.println(name);
}
```

5）删除部署资源

deleteDeployment方法可以删除部署数据，可以通过传递是否级联删除参数确定是否删除流程定义相关的流程实例数据。

```
// 创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
// 得到流程存储服务对象
RepositoryService repositoryService = engine.getRepositoryService();
// 部署一份流程文件与相应的流程图文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/deleteDeployment.bpmn")
    .deploy();
// 查询流程定义
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 开始流程
engine.getRuntimeService().startProcessInstanceById(def.getId());
try {
    // 删除部署数据失败，此时将会抛出异常，由于cascade为false
    repositoryService.deleteDeployment(dep.getId());
} catch (Exception e) {
    System.out.println("删除失败，流程开始，没有设置cascade为true");
}
System.out.println("============分隔线");
// 成功删除部署数据
repositoryService.deleteDeployment(dep.getId(), true);
```

注：级联删除会删除流程实例数据（ProcessInstance），其中流程实例数据也包括流程任务（TASK）与流程实例的历史数据。

注2：设置级联删除为false时，如果存在流程实例数据，则删除流程定义是会抛出异常。

6）DeploymentQuery对象

查询流程部署对象，包括相应的查询和排序方法。

```
//deploymentName方法
Deployment depBQuery = repositoryService.createDeploymentQuery()
    .deploymentName("b").singleResult();
//deploymentNameLike, 模糊查询，结果集为2
List<Deployment> depCQuery = repositoryService.createDeploymentQuery()
    .deploymentNameLike("%b%").list();
```

7）ProcessDefinitionquery对象

该对象主要用于查询流程定义数据。

```
ProcessDefinition def = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
```

使用该对象的startableByUser方法，传递userId，获取用户有权限启动的流程定义集合。