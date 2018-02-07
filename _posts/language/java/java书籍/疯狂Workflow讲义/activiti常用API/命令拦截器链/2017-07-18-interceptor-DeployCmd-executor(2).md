---
title: 命令拦截器--DeployCmd的命令执行者
tags: [activiti]
---

### commandExecutor何时被注入的

1）RepositoryServiceImpl类继承了ServiceImpl，在ServiceImpl类中定义了commandExecutor属性，并提供了set和get方法。

```
protected CommandExecutor commandExecutor;
```

注：因此RepositoryServiceImpl能够很轻松的访问到该对象。

2）具体的注入实例还需要进一步查看RepositoryService的获取，我们是通过ProcessEngine类的getRepositoryService方法获取的。

查看ProcessEngineImpl（ProcessEngine的具体实例对象）类内部的RepositoryService

```
this.repositoryService = processEngineConfiguration.getRepositoryService();
```

3）查看ProcessEngineConfigurationImpl（ProcessEngineConfiguration的具体实例对象）类内部的repositoryService的实例化

```
protected RepositoryService repositoryService = new RepositoryServiceImpl();
...
protected void initServices() {
    initService(repositoryService);//初始化repositoryService
    initService(runtimeService);
    initService(historyService);
    initService(identityService);
    initService(taskService);
    initService(formService);
    initService(managementService);
}
```

向repositoryService中注入命令的执行者

```
protected void initService(Object service) {
    if (service instanceof ServiceImpl) {
      ((ServiceImpl)service).setCommandExecutor(commandExecutorTxRequired);
    }
}
```

注：可以看到commandExecutor的实际为CommandExecutorTxRequired对象。发现注入commandExecutor是发生在ProcessEngineConfigurationImpl类的initService方法中，并把该对象的属性commandExecutorTxRequired注入到各个service中。

### 查看commandExecutor命令执行者的实现

查看commandExecutorTxRequired对象的实例化过程，发现commandExecutorTxRequired实现了CommandExecutor接口。

首先，ProcessEngineConfigurationImpl类调用buildProcessEngine方法

```
public ProcessEngine buildProcessEngine() {
    init();
    return new ProcessEngineImpl(this);
}
```

init方法中会调用initCommandExecutors，该方法中会初始化各种命令执行者

```
protected void initCommandExecutors() {
    initActualCommandExecutor();
    initCommandInterceptorsTxRequired();
    initCommandExecutorTxRequired();
    initCommandInterceptorsTxRequiresNew();
    initCommandExecutorTxRequiresNew();
}
```

具体的实现过程查看initCommandExecutorTxRequired方法

```
protected void initCommandExecutorTxRequired() {
    if (commandExecutorTxRequired==null) {
      commandExecutorTxRequired = initInterceptorChain(commandInterceptorsTxRequired);
    }
}
```

命令执行者拦截器链，CommandInterceptor也实现了CommandExecutor这个接口

```
protected CommandInterceptor initInterceptorChain(List<CommandInterceptor> chain) {
    if (chain==null || chain.isEmpty()) {
      throw new ActivitiException("invalid command interceptor chain configuration: "+chain);
    }
    for (int i = 0; i < chain.size()-1; i++) {
      chain.get(i).setNext( chain.get(i+1) );
    }
    return chain.get(0);
}
```

注：可以看到初始化commandExecutorTxRequired是将命令拦截器链的第一个CommandInterceptor对象返回给commandExecutorTxRequired。即注入各个service中的命令执行者，是拦截器链的第一个对象。