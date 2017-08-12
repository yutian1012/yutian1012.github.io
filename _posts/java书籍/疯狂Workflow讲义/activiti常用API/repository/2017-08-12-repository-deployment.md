---
title: activiti流程文件部署
tags: [activiti]
---

### 流程资源文件

1）流程文件

主要包括流程描述文件、流程图等。Activiti如果需要对这些资源进行操作（包括添加、删除和查询等），可以使用RepositoryService提供的API来实现。

2）ACT_GE_BYTEARRAY表的存储内容

流程文件数据将会被保存在ACT_GE_BYTEARRAY表中，对应的实体为ResourceEntity。

另外，该表还会保存用户的图片数据，对应的实体为ByteArrayEntity。

两者的区别？

ResourceEntity会设置ACT_GE_BYTEARRAY表的GENERATED_字段值，而ByteArrayEntity保存时，该值为null。

### Deployment对象

1）接口

Deployment是一个接口，表示ACT_RE_DEPLOYMENT表的数据。实现类为DeploymentEntity。

```
protected String id;//对应ACT_RE_DEPLOYMENT表的ID_列
protected String name;//对应ACT_RE_DEPLOYMENT表的NAME_列
protected Date deploymentTime;//对应ACT_RE_DEPLOYMENT表的DEPLOY_TIME_列
```

### DeploymentEntity和ResourceEntity的关系

1）实体与表

ResourceEntity与DeploymentEntity均为持久化对象。一个ResourceEntity实例表示一条ACT_GE_BYTEARRAY表的数据，一个DeploymentEntity实例表示一条ACT_RE_DEPLOYMENT表的数据。

2）多对一的关系

ResourceEntity与DeploymentEntity为多对一的关系，级一个DeploymentEntity下可以有多个ResourceEntity。在DeploymentEntity中维护一个Map对象来保存该部署的全部资源文件。

3）addInputStream的操作过程

DeploymentBuilder中的addInputStream方法，会将参数InputStream对象设置到ResourceEntity对象中，最后保存到DeploymentEntity中。每个DeploymentBuilder实例中均会维护一个DeploymentEntity实例，调用多次addInputStream会生成多个ResourceEntity对象，而与这些对象相关的只有一个DeploymentEntity。

4）数据存储的对应关系

DeploymentBuilder调用deploy方法后会将DeploymentEntity及它的全部ResourceEntity（map集合中维护关系）写到数据库中。ACT_GE_BYTEARRAY表的DEPLOYMENT_ID_值与ACT_RE_DEPLOYMENT表的主键相关联。

### 流程资源的部署

2）DeploymentBuilder对象

对流程的部署可以使用DeploymentBuilder对象完成，该对象用于从指定资源中加载（addClasspathResource等方法），并部署到系统中（deploy方法）。

获取DeploymentBuilder对象

```
//创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
//得到流程存储服务对象
RepositoryService repositoryService = engine.getRepositoryService();
//创建DeploymentBuilder实例
DeploymentBuilder builder = repositoryService.createDeployment();
```

3）添加输入流资源

DeploymentBuilder中提供了一个addInputStream方法，参数为InputStream对象。