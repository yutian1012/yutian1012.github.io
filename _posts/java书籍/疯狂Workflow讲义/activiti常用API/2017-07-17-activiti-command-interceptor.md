---
title: 命令拦截器
tags: [activiti]
---

参考：http://www.cnblogs.com/yg_zhang/p/3676804.html

外界对activiti流程中的各个实例进行的操作，实际可以被看作是对数据进行相应的操作。当对数据执行操作时，activiti都会调用相应的拦截器来处理（责任链模式）。

activiti的每个操作都会发出相应的命令，会被拦截器链执行。以RepositoryService部署流程，底层会调用层次为

1）部署流程

```
// 创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
// 得到流程存储服务组件
RepositoryService repositoryService = engine.getRepositoryService();
...
// 部署流程文件
repositoryService.createDeployment().addClasspathResource("bpmn/First.bpmn").deploy();
```

2）deploy方法执行

即DeploymentBuilderImpll类的deploy方法

```
public Deployment deploy() {
    return repositoryService.deploy(this);
}
```

注：可以看到使用的是repositoryService的deploy方法。

3）RepositoryServiceImpl类的deploy方法的执行

```
public Deployment deploy(DeploymentBuilderImpl deploymentBuilder) {
    return commandExecutor.execute(new DeployCmd<Deployment>(deploymentBuilder));
}
```

注：构建了DeployCmd对象（命令对象）

4）commandExecutor何时被注入的

RepositoryServiceImpl类继承了ServiceImpl，在ServiceImpl类中定义了commandExecutor属性，并提供了set和get方法。

```
protected CommandExecutor commandExecutor;
```