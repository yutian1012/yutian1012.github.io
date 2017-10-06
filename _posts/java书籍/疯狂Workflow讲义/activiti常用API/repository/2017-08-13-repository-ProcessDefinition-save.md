---
title: activiti流程定义的保存
tags: [activiti]
---

ProcessDefinition对象的产生是从Deployment部署的资源信息中解析出来，并保持到数据库中的。

### 流程定义文件的部署

流程定义的部署，使用流程定义工具绘制出流程定义文件（xml）后，是通过DeploymentBuilder进行部署的。

```
//创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
//得到流程存储服务对象
RepositoryService repositoryService = engine.getRepositoryService();
//创建DeploymentBuilder实例
DeploymentBuilder builder = repositoryService.createDeployment();
builder.addClasspathResource("bpmn/processDeploy.bpmn").deploy();
```

注：可以看到是RepositoryService服务通过Deployment方式进行部署的。

2）DeploymentEntity与ProcessDefinitionEntity是什么关系

流程定义信息会保存到ACT_RE_PROCDEF表中，对应的实体是ProcessDefinitionEntity。

### 流程定义信息的保存

1）addClasspathResource方法将资源设置到集合中

DeploymentEntity类中有一个resources集合，用来存储与Deployment相关的资源信息。

```
protected Map<String, ResourceEntity> resources;
```

addClasspathResource方法会将流程定义文件xml读入资源集合中。

2）调用deploy方法部署

DeploymentBuilderImpl类的deploy方法

```
public Deployment deploy() {
    return repositoryService.deploy(this);
}
```

RepositoryServiceImpl类的deploy方法

```
public Deployment deploy(DeploymentBuilderImpl deploymentBuilder) {
    return commandExecutor.execute(new DeployCmd<Deployment>(deploymentBuilder));
}
```

注：可以看到将DeployCmd对象设置到命令执行中，部署的过程是在DeployCmd命令对象中实现的。

3）DeployCmd命令对象

```
public Deployment execute(CommandContext commandContext) {
    ...
    //保存Deployment对象到act_re_deployment表中
    Context.getCommandContext().getDeploymentManager()
        .insertDeployment(deployment);
    ...
```

DeploymentManager类的insertDeployment方法

```
public void insertDeployment(DeploymentEntity deployment) {
    getDbSqlSession().insert(deployment);
    
    //将流程部署的资源（流程定义xml）保存到act_ge_bytearray表中
    for (ResourceEntity resource : deployment.getResources().values()) {
      resource.setDeploymentId(deployment.getId());
      getResourceManager().insertResource(resource);
    }
    
    Context
      .getProcessEngineConfiguration()
      .getDeploymentCache()
      .deploy(deployment);
}
```

DeploymentCache类的deploy方法

```
protected List<Deployer> deployers;
...
public void deploy(DeploymentEntity deployment) {
    for (Deployer deployer: deployers) {
      deployer.deploy(deployment);
    }
}
```

4）deployers集合的初始化

ProcessEngineConfigurationImpl类的initDeployers方法初始化

```
protected void initDeployers() {
    if (this.deployers==null) {
      this.deployers = new ArrayList<Deployer>();
      //自定义前置部署类
      if (customPreDeployers!=null) {
        this.deployers.addAll(customPreDeployers);
      }
      //设置默认的部署类
      this.deployers.addAll(getDefaultDeployers());
      //自定义后置部署类
      if (customPostDeployers!=null) {
        this.deployers.addAll(customPostDeployers);
      }
    }

    if (deploymentCache==null) {
      List<Deployer> deployers = new ArrayList<Deployer>();
      if (customPreDeployers!=null) {
        deployers.addAll(customPreDeployers);
      }
      deployers.addAll(getDefaultDeployers());
      if (customPostDeployers!=null) {
        deployers.addAll(customPostDeployers);
      }

      deploymentCache = new DeploymentCache();
      deploymentCache.setDeployers(deployers);
    }
}
```

ProcessEngineConfigurationImpl类的getDefaultDeployers方法

```
protected Collection< ? extends Deployer> getDefaultDeployers() {
    List<Deployer> defaultDeployers = new ArrayList<Deployer>();
    //内部实际使用的部署类
    BpmnDeployer bpmnDeployer = new BpmnDeployer();
    bpmnDeployer.setExpressionManager(expressionManager);
    bpmnDeployer.setIdGenerator(idGenerator);

    //流程定义文件的解析器
    BpmnParser bpmnParser = new BpmnParser(expressionManager);
    
    //设置解析监听
    if(preParseListeners != null) {
      bpmnParser.getParseListeners().addAll(preParseListeners);
    }
    //默认监听信息
    bpmnParser.getParseListeners().addAll(getDefaultBPMNParseListeners());
    if(postParseListeners != null) {
      bpmnParser.getParseListeners().addAll(postParseListeners);
    }
    //将解析器设置到部署对象上
    bpmnDeployer.setBpmnParser(bpmnParser);
    
    defaultDeployers.add(bpmnDeployer);
    return defaultDeployers;
}
```

因此，DeploymentEntity的部署是由BpmnDeployer类的deploy方法来实现的

5）BpmnDeployer类的depoly方法

```
public void deploy(DeploymentEntity deployment) {
    List<ProcessDefinitionEntity> processDefinitions = new ArrayList<ProcessDefinitionEntity>();
    Map<String, ResourceEntity> resources = deployment.getResources();

    //循环遍历deployment中的资源集合
    for (String resourceName : resources.keySet()) {
        //判断资源是否为bpmn流程定义文件
      if (isBpmnResource(resourceName)) {
        ...
        //构造流程文件的解析器BpmnParse
        BpmnParse bpmnParse = bpmnParser.createParse().sourceInputStream(inputStream).deployment(deployment).name(resourceName);
        //执行解析器
        bpmnParse.execute();
        //循环变量解析器解析得到的processDefinitions对象集合
        for (ProcessDefinitionEntity processDefinition: 
            bpmnParse.getProcessDefinitions()) {
            //设置ProcessDefinition的属性信息
            processDefinition.setResourceName(resourceName);
            
            //流程文件对应的流程图
            String diagramResourceName = getDiagramResourceForProcess(resourceName, processDefinition.getKey(), resources);

            //判断是否是新部署
            if(deployment.isNew()) {
                if (Context.getProcessEngineConfiguration().isCreateDiagramOnDeploy() && diagramResourceName==null && processDefinition.isGraphicalNotationDefined()) {
                  try {
                      //图片资源字节数组
                      byte[] diagramBytes = IoUtil.readInputStream(ProcessDiagramGenerator.generatePngDiagram(processDefinition), null);
                      diagramResourceName = getProcessImageResourceName(resourceName, processDefinition.getKey(), "png");
                      //保存流程图资源到数据库act_ge_bytearray表中
                      createResource(diagramResourceName, diagramBytes, deployment);
                  } catch (Throwable t) { // if anything goes wrong, we don't store the image (the process will still be executable).
                    LOG.log(Level.WARNING, "Error while generating process diagram, image will not be stored in repository", t);
                  }
                } 
              }

            ...
            processDefinition.setDiagramResourceName(diagramResourceName);
            //流程定义信息设置的到集合中
            processDefinitions.add(processDefinition);
            ...
        }
      }
    }
    //保存ProcessDefinition信息
    for (ProcessDefinitionEntity processDefinition : processDefinitions) {
      if (deployment.isNew()) {
         //补充信息
         processDefinition.setVersion(processDefinitionVersion);
         processDefinition.setDeploymentId(deployment.getId());
         ...
         //保存信息到数据库act_re_procdef表中
         dbSqlSession.insert(processDefinition);
         deploymentCache.addProcessDefinition(processDefinition);
         addAuthorizations(processDefinition);
      }
  }
}
```

### 数据保存

部署流程后会生成多条数据

1）Deploment对象保存到act_re_deployment表中

2）部署的流程定义文件xml保存到act_ge_bytearray表中。

3）解析流程定义文件时会生成流程图资源信息保存到act_ge_bytearray表中。

4）解析流程定义文件xml生成ProcessDefinition对象，保存到表act_re_procdef表中