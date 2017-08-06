---
title: 命令拦截器--DeployCmd的执行过程总结
tags: [activiti]
---

DeploymentBuilderImpl类的deploy方法的执行最终会commandExecutor来执行

```
commandExecutor.execute(new DeployCmd<Deployment>(deploymentBuilder));
```

1）commandExecutor对象

RepositoryServiceImpl类成员，实际实现是ProcessEngineConfigurationImpl类的成员变量commandExecutorTxRequired（CommandExecutor类实例），是在初始化RepositoryService时注入的引用。而commandExecutorTxRequired的初始化是通过ProcessEngineConfigurationImpl类的commandInterceptorsTxRequired拦截器链的第一个拦截器（因为CommandInterceptor也实现了CommandExecutor接口）。

因此，实际的commandExecutor.execute方法应查看commandInterceptorsTxRequired拦截器链的执行。

2）拦截器链的执行

通过ProcessEngineConfigurationImpl类的源码可以查看到commandInterceptorsTxRequired拦截器链的初始化过程，包括自定义拦截器（customPreCommandInterceptorsTxRequired）默认命令拦截器（getDefaultCommandInterceptorsTxRequired），自定义后缀拦截器（customPostCommandInterceptorsTxRequired），以及最终的actualCommandExecutor命令执行拦截器，其中actualCommandExecutor也是实际的命令执行者。

```
protected void initCommandInterceptorsTxRequired() {
    if (commandInterceptorsTxRequired==null) {
      if (customPreCommandInterceptorsTxRequired!=null) {
        commandInterceptorsTxRequired = new ArrayList<CommandInterceptor>(customPreCommandInterceptorsTxRequired);
      } else {
        commandInterceptorsTxRequired = new ArrayList<CommandInterceptor>();
      }
      commandInterceptorsTxRequired.addAll(getDefaultCommandInterceptorsTxRequired());
      if (customPostCommandInterceptorsTxRequired!=null) {
        commandInterceptorsTxRequired.addAll(customPostCommandInterceptorsTxRequired);
      }
      commandInterceptorsTxRequired.add(actualCommandExecutor);
    }
}

protected void initActualCommandExecutor() {
    actualCommandExecutor = new CommandExecutorImpl();
}
```

注：拦截器前方的对象都会调用next.execute(command)将命令对象传递到拦截器链下的一个连接器进行执行，直到遍历完成整个拦截器。因此最终会教育actualCommandExecutor来结束拦截器链的执行。

可以看到命令拦截器增强了命令对象的执行，在命令执行过程中如记录日志（LogInterceptor）等信息。

3）actualCommandExecutor执行命令

```
public class CommandExecutorImpl extends CommandInterceptor {
  public <T> T execute(Command<T> command) {
    return command.execute(Context.getCommandContext());
  }
}
```

注：Context.getCommandContext()从线程堆栈中获取命令上下文环境对象。如果是部署命令，那么这里将调用DeployCmd类的execute方法。

4）DeployCmd的执行

```
public Deployment execute(CommandContext commandContext) {
    //加载部署的资源xml文件，并封装为DeploymentEntity对象
    DeploymentEntity deployment = deploymentBuilder.getDeployment();

    deployment.setDeploymentTime(ClockUtil.getCurrentTime());

    //判断是否过滤重复流程（流程没有任何变动是否仍然部署的问题）
    if ( deploymentBuilder.isDuplicateFilterEnabled() ) {
      //获取库中最新的部署对象
      DeploymentEntity existingDeployment = Context
        .getCommandContext()
        .getDeploymentManager()
        .findLatestDeploymentByName(deployment.getName());
      
      //在库中查询到，并且如果对象内容相同，则直接返回
      if ( (existingDeployment!=null)
           && !deploymentsDiffer(deployment, existingDeployment)) {
        return existingDeployment;
      }
    }

    //部署新流程
    deployment.setNew(true);
    
    //通过Context上下对象，获取操作数据库的Manager，
    //然后将部署信息保存到数据库中
    Context
      .getCommandContext()
      .getDeploymentManager()
      .insertDeployment(deployment);
    
    return deployment;
}
```