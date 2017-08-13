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

注：调用DeploymentBuilder的deploy方法返回Deployment对象。

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

1）DeploymentBuilder对象

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

2）添加资源

内部的执行过程：会将添加的资源封装到ResourceEntity对象中，最后存到DeploymentEntity中。

方式一：添加输入流资源

DeploymentBuilder中提供了一个addInputStream方法，参数为InputStream对象。

```
// 资源输入流
InputStream is1 = new FileInputStream(new File(
    "resource/artifact/flow_inputstream1.png"));
builder.addInputStream("inputA", is1);
```

方式二：添加Classpath资源

addClasspathResource方法将classpath下的资源写入到数据库中，底层会得到当前的ClassLoader对象，调用getResourceAsStream方法将指定的classpath下的资源文件转换为InputStream，再调用addInputStream方法。

```
//添加classpath下的资源
builder.addClasspathResource("artifact/classpath.png");
```

方式三：添加字符串资源

将一些流程定义的属性或公用的变量保存起来时，可以使用addString方法。

```
//添加String资源
builder.addString("test", "this is string method");
```

```
+-----+-------+---------------------+
| ID_ | NAME_ | DEPLOY_TIME_        |
+-----+-------+---------------------+
| 1   | NULL  | 2017-08-13 22:49:55 |
+-----+-------+---------------------+

+-----+------+-------+----------------+-----------------------+------------+
| ID_ | REV_ | NAME_ | DEPLOYMENT_ID_ | BYTES_                | GENERATED_ |
+-----+------+-------+----------------+-----------------------+------------+
| 2   |    1 | test  | 1              | this is string method |          0 |
+-----+------+-------+----------------+-----------------------+------------+
```

方式四：添加压缩包资源

使用DeploymentBuilder提供的addZipInputStream方法直接部署压缩包，该方法会遍历压缩包内的全部文件，然后将这些文件转换为byte数组，写到资源表中。

```
//获取zip文件的输入流
FileInputStream fis = new FileInputStream(new File("resource/artifact/ZipInputStream.zip"));
//读取zip文件，创建ZipInputStream对象
ZipInputStream zi = new ZipInputStream(fis);
//添加Zip压缩包资源
builder.addZipInputStream(zi);
```

注：zipInputStream是jdk自带的java类

3）修改部署名称

DeploymentBuilder的name方法可以设置部署名称，会更新ACT_RE_DEPLOYMENT的NAME_字段值。

```
builder.name("crazyit");
```

4）过滤重复部署

多次调用同一个资源部署，会在数据库中生成多个Deployment记录（ACT_RE_DEPLOYMENT）。如果想避免这种情况，可以调用DeploymentBuilder的enableDuplicateFiltering方法。

```
builder.enableDuplicateFiltering();
```

该方法仅仅将DeploymentBuilder的isDuplicateFilterEnabled属性设置为true，在执行deploy方法时，如果发现该值为true，则根据部署对象（DeploymentEntity）的名称去查询最后部署的记录。如果发现最后一条部署的记录与当前部署的记录一致，则不会重复部署。

注：这里所说的一致，是指Deployment下的资源是否相同（ACT_GE_BYTEARRAY表），包括资源名称和资源内容。