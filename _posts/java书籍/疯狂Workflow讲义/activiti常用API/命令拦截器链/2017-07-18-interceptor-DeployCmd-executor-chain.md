---
title: 命令拦截器--DeployCmd的命令拦截器链
tags: [activiti]
---

### commandInterceptorsTxRequired对象的实例化

1）定义信息（ProcessEngineConfigurationImpl类中）

```
protected List<CommandInterceptor> commandInterceptorsTxRequired;
```

2）初始化过程

```
protected void initCommandInterceptorsTxRequired() {
    if (commandInterceptorsTxRequired==null) {
      //自定义前置命令拦截器链
      if (customPreCommandInterceptorsTxRequired!=null) {
        commandInterceptorsTxRequired = new ArrayList<CommandInterceptor>(customPreCommandInterceptorsTxRequired);
      } else {
        commandInterceptorsTxRequired = new ArrayList<CommandInterceptor>();
      }
      //默认命令拦截器链
      commandInterceptorsTxRequired.addAll(getDefaultCommandInterceptorsTxRequired());
      //自定义后置命令拦截器链
      if (customPostCommandInterceptorsTxRequired!=null) {
        commandInterceptorsTxRequired.addAll(customPostCommandInterceptorsTxRequired);
      }
      commandInterceptorsTxRequired.add(actualCommandExecutor);
    }
}
```

3）最终命令执行者

查看actualCommandExecutor是最终的命令执行者，该类的实现比较简单，就是调用command实例的execute方法

CommandExecutorImpl类的execute方法

```
public <T> T execute(Command<T> command) {
    return command.execute(Context.getCommandContext());
}
```

4）查看getDefaultCommandIntercerptorsTxRequired的实现。

```
protected abstract Collection< ? extends CommandInterceptor> getDefaultCommandInterceptorsTxRequired();
```

注：该方法是一个抽象方法，其实现应该会在子类中

StandaloneProcessEngineConfiguration中该方法的实现

```
protected Collection< ? extends CommandInterceptor> getDefaultCommandInterceptorsTxRequired() {
    List<CommandInterceptor> defaultCommandInterceptorsTxRequired = new ArrayList<CommandInterceptor>();
    defaultCommandInterceptorsTxRequired.add(new LogInterceptor());
    defaultCommandInterceptorsTxRequired.add(new CommandContextInterceptor(commandContextFactory, this));
    return defaultCommandInterceptorsTxRequired;
}
```

注：具体的命令拦截器为CommandContextInterceptor，该类的实例化获取了commandContextFactory对象。

5）CommandContextInterceptor类

CommandContextInterceptor主要作用是设置命令对象的上下文环境，待命令执行时使用，具体可查看命令对象的执行过程，如DeployCmd中的execute方法（命令接口方法）。

commandContextFactory类的实例

该类的初始化也在ProcessEngineConfigurationImpl类中。

```
protected void initCommandContextFactory() {
    if (commandContextFactory==null) {
      commandContextFactory = new CommandContextFactory();
      commandContextFactory.setProcessEngineConfiguration(this);
    }
}
```

6）查看命令链的执行过程

以RepositoryServiceImpl类的deploy方法为例查看

```
public Deployment deploy(DeploymentBuilderImpl deploymentBuilder) {
    return commandExecutor.execute(new DeployCmd<Deployment>(deploymentBuilder));
}
```

命令对象DeployCmd接收了deploymentBuilder对象作为命令的接收者。

经过上述配置的一系列的拦截链中的方法后，会到actualCommandExecutor（CommandExecutorImpl实例），拦截器的最后一个链。执行DeployCmd对象的execute方法。

7）DeployCmd对象的execute方法

```
public Deployment execute(CommandContext commandContext) {
    DeploymentEntity deployment = deploymentBuilder.getDeployment();

    deployment.setDeploymentTime(ClockUtil.getCurrentTime());

    if ( deploymentBuilder.isDuplicateFilterEnabled() ) {
      DeploymentEntity existingDeployment = Context.getCommandContext()
        .getDeploymentManager()
        .findLatestDeploymentByName(deployment.getName());
      
      if ( (existingDeployment!=null)
           && !deploymentsDiffer(deployment, existingDeployment)) {
        return existingDeployment;
      }
    }

    deployment.setNew(true);
    
    Context
      .getCommandContext()
      .getDeploymentManager()
      .insertDeployment(deployment);
    
    return deployment;
}
```