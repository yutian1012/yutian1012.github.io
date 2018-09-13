---
title: activiti的命令模式分析
tags: [project]
---

参考：https://www.cnblogs.com/yg_zhang/p/3676804.html

流程引擎所有的操作都采用命令模式，使用命令执行器进行执行，命令执行器是一个采用拦截器链式执行模式。

命令模式的相关结构（可查看相应的设计类图），命令对象Command中包含receiver对象，通过execute方法执行receiver相应的action方法。Invoker是命令发布者，内部通过创建或设置相应的Command对象，调研command对象的execute方法实现命令的执行（具体执行者再Command中维护着）。

1）核心类

```
org.activiti.engine.impl.interceptor.CommandExecutor

CommandInterceptor接口，默认配置了相应的拦截器，并自动注入设置拦截器执行链

CommandContextInterceptor 拦截器
```

2）设置流程变量查看命令模式的执行过程

```
RuntimeService setVariable(String executionId, String variableName, Object value);
```

注：实现类RuntimeServiceImpl。

实现代码中使用了SetExecutionVariablesCmd命令对象

```
Map<String, Object> variables = new HashMap<String, Object>();
variables.put(variableName, value);
commandExecutor.execute(new SetExecutionVariablesCmd(executionId, variables, false));
```

3）持久化操作

在命令中并不执行数据库持久化，持久化在此拦截器中调用context.close();执行。