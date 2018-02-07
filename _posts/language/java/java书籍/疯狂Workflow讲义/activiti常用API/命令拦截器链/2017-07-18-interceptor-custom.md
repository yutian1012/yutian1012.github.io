---
title: 流程配置设置自定义拦截器
tags: [activiti]
---

Activiti拦截器是结合了命令模式和责任链模式实现的。可以在实现自定义配置类时，编写自己的拦截器。

### 配置自定义拦截器

1）创建拦截器

第一个拦截器

```
/**
 * 拦截器实现A
 * 
 */
public class InterceptorA extends CommandInterceptor {

    public <T> T execute(Command<T> command) {
        // 输出字符串和命令
        System.out.println("this is interceptor A "+ command.getClass().getName());
        // 然后让责任链中的下一请求处理者处理命令
        return next.execute(command);
    }
}
```

第二个拦截器

```
public class InterceptorB extends CommandInterceptor {

    @Override
    public <T> T execute(Command<T> command) {
        // 输出字符串和命令
        System.out.println("this is interceptor B "+ command.getClass().getName());
        // 然后让责任链中的下一请求处理者处理命令
        return next.execute(command);
    }
}
```

注：next就是拦截器中的下一个请求处理者。

2）创建自定义配置类

```
/**
 * 自定义配置类
 */
public class CustomConfiguration extends ProcessEngineConfigurationImpl {

    protected Collection<? extends CommandInterceptor> getDefaultCommandInterceptorsTxRequired() {
        // 创建一个拦截器集合
        List<CommandInterceptor> defaultCommandInterceptorsTxRequired = new ArrayList<CommandInterceptor>();
        // 添加自定义拦截器A
        defaultCommandInterceptorsTxRequired.add(new InterceptorA());
        // 添加系统的拦截器
        defaultCommandInterceptorsTxRequired.add(new CommandContextInterceptor(commandContextFactory, this));
        return defaultCommandInterceptorsTxRequired;
    }

    protected Collection<? extends CommandInterceptor> getDefaultCommandInterceptorsTxRequiresNew() {
        // 创建一个拦截器集合
        List<CommandInterceptor> defaultCommandInterceptorsTxRequired = new ArrayList<CommandInterceptor>();
        // 添加自定义拦截器
        defaultCommandInterceptorsTxRequired.add(new InterceptorB());
        return defaultCommandInterceptorsTxRequired;
    }
}
```

3）创建测试类

```
public class MyConfig {
    /**
     * @param args
     */
    public static void main(String[] args) {
        ProcessEngines.getDefaultProcessEngine();
        // 创建Activiti配置对象
        ProcessEngineConfiguration config = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("interceptor-config.xml");
        // 初始化流程引擎
        ProcessEngine engine = config.buildProcessEngine();
        // 部署一个最简单的流程
        engine.getRepositoryService().createDeployment().addClasspathResource("bpmn/interceptor/config.bpmn20.xml").deploy();
        // 构建流程参数
        Map<String, Object> vars = new HashMap<String, Object>();
        vars.put("day", 10);
        // 开始流程
        engine.getRuntimeService().startProcessInstanceByKey("vacationProcess",vars);
    }
}
```

注：这里的流程图的设计就不在进行描述了，可以查看疯狂workflow的源码（第四章）。

### 总结

1）在流程引擎启动时

执行SchemaOperationsProcessEngineBuild，该类也是一个Command的实现。

2）进行部署时

执行DeployCmd

3）获取ID块时

执行GetNextIdBlockCmd

4）开始流程时

执行StartProcessInstanceCmd

另外，需要注意的是，执行普通的CRUD时，只会执行getDefaultCommandInterceptorsTxRequired方法中定义的拦截器，而执行产生ID块的操作命令时，会执行getDefaultCommandInterceptorsTxRequiresNew方法定义的拦截器。