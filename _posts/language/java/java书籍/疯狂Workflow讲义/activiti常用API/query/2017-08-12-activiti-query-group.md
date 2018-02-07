---
title: Activiti数据查询--以Group为例
tags: [activiti]
---

### 查询对象

1）各个服务组件的createXXXQuery方法来获取这些查询对象。

如IdentityService类提供了createGroupQuery和createUserQuery方法来实现用户组和用户对象的查询。TaskService提供了createTaskQuery方法来查询Task对象。

2）IdentityServiceImpl实现类

```
public GroupQuery createGroupQuery() {
    return commandExecutor.execute(new CreateGroupQueryCmd());
}
```

最终会调用CreateGroupQueryCmd对象，返回GroupQuery

3）CreateGroupQueryCmd类

```
public GroupQuery execute(CommandContext commandContext) {
    return commandContext
      .getGroupManager()
      .createNewGroupQuery();
}
```

注：commandContext是从ProcessEngineConfigurationImpl类中初始化好的sessionFactories（map类型）集合中获取的GroupManager。

4）GroupManager类

```
public GroupQuery createNewGroupQuery() {
    return new GroupQueryImpl(Context.getProcessEngineConfiguration().getCommandExecutorTxRequired());
}
```

GroupQueryImpl类的继承体系

```
GroupQueryImpl extends AbstractQuery<GroupQuery, Group> implements GroupQuery
```

5）查询单个对象

```
public U singleResult() {
    this.resultType = ResultType.SINGLE_RESULT;
    if (commandExecutor!=null) {
      return (U) commandExecutor.execute(this);
    }
    return executeSingleResult(Context.getCommandContext());
}
```

这里的commandExecutor不空，是在createNewGroupQuery中将命令拦截器注入到commandExecutor上。

```
实际上的commandExecutor对象
commandExecutor=Context.getProcessEngineConfiguration().getCommandExecutorTxRequired()
```

因此，会调用命令对象，AbstractQuery对象本身实现了Command接口，所以，会调用AbstractQuery类的execute方法（命令接口中定义的）。

6）AbstractQuery中命令接口方法

```
public Object execute(CommandContext commandContext) {
    if (resultType==ResultType.LIST) {
      return executeList(commandContext, null);
    } else if (resultType==ResultType.SINGLE_RESULT) {
      return executeSingleResult(commandContext);
    } else if (resultType==ResultType.LIST_PAGE) {
      return executeList(commandContext, null);
    } else {
      return executeCount(commandContext);
    }
}
```

注：resultType是在调用singleResult方法时赋值的，因此根据if判断会执行executeSingleResult方法。

7）AbstractQuery中的executeSingleResult方法

```
public U executeSingleResult(CommandContext commandContext) {
    //AbstractQuery中的抽象方法，由具体的子类来实现
    List<U> results = executeList(commandContext, null);
    if (results.size() == 1) {
      return results.get(0);
    } else if (results.size() > 1) {
     throw new ActivitiException("Query return "+results.size()+" results instead of max 1");
    } 
    return null;
}
```

注：调用了AbstractQuery中定义的executeList抽象方法，因此会交由GroupQueryImpl实现类中重写的executeList方法执行。

8）GroupQueryImpl类的executeList方法

```
public List<Group> executeList(CommandContext commandContext, Page page) {
    checkQueryOk();
    return commandContext
      .getGroupManager()
      .findGroupByQueryCriteria(this, page);
}
```

最终还是交由GroupManager类调用底层数据库操作完成查询。

### 方法链

GroupQuery中的方法基本都会返回GroupQuery对象，因此可以以链的形式不断调用，为其设置参数，实现查询。

```
GroupQuery groupId(String groupId);
GroupQuery groupName(String groupName);
```

具体实现GroupQueryImpl类

```
public GroupQuery groupId(String id) {
    if (id == null) {
      throw new ActivitiException("Provided id is null");
    }
    this.id = id;
    return this;
}
  
public GroupQuery groupName(String name) {
    if (name == null) {
      throw new ActivitiException("Provided name is null");
    }
    this.name = name;
    return this;
}
```

注：设置查询条件，并返回this对象，从而返回对象继续调用该类的方法。