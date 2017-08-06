---
title: AbstractManager封装访问持久层的抽象类
tags: [activiti]
---

以Group对象插入数据为例，查看具体的实现过程

1）创建并保存Group对象

```
Group group = identityService.newGroup("1");
group.setName("经理组");
group.setType("manager");
group.setId(null);
//保存Group到数据库
identityService.saveGroup(group);
```

IdentityService.saveGroup方法通过查看源码，可以发现该方法会调用GroupManager类的insertGroup方法（命令对象CreateGroupCmd的execute）。

2）GroupManager类继承了AbstractManager类

```
public void insertGroup(Group group) {
    //调用父类AbstractManager类获取数据库操作session
    getDbSqlSession().insert((PersistentObject) group);
}
```

3）AbstractManager类

getDbSqlSession()方法定义在AbstracManager类中

```
protected DbSqlSession getDbSqlSession() {
    return getSession(DbSqlSession.class);
}

protected <T> T getSession(Class<T> sessionClass) {
    return Context.getCommandContext().getSession(sessionClass);
}
```

注：通过CmmandContext上下文环境获取数据库操作的Session对象。Context类是一个工具类，该工具类能够从当前线程堆栈中获取CommandContext对象。

4）Context类

```
protected static ThreadLocal<Stack<CommandContext>> commandContextThreadLocal = new ThreadLocal<Stack<CommandContext>>();
...
public static CommandContext getCommandContext() {
    //从当前线程中获取栈对象，然后从栈顶获取CommandContext对象
    Stack<CommandContext> stack = getStack(commandContextThreadLocal);
    if (stack.isEmpty()) {
      return null;
    }
    return stack.peek();
}
```

问题：CommandContext如何设置到当前线程中的

5）CommandContext的注入到线程变量中

CommandContextInterceptor类的execute方法中会创建ComandConext，并将其设置到当前线程堆栈中。

```
public <T> T execute(Command<T> command) {
    CommandContext context = commandContextFactory.createCommandContext(command);

    try {
      //设置到当前线程堆栈中
      Context.setCommandContext(context);
      Context.setProcessEngineConfiguration(processEngineConfiguration);
      //调用下一个拦截器链
      return next.execute(command);
      
    } catch (Exception e) {
      context.exception(e);
      
    } finally {
      try {
        context.close();
      } finally {
        //从线程中移除CommandContext对象
        Context.removeCommandContext();
        Context.removeProcessEngineConfiguration();
      }
    }
    
    return null;
}
```

在ProcessEngineConfig对象中，默认都会将其配置到拦截器链中，因此在执行任何命令时，都会先经过该拦截器将CommandContext对象设置到线程中。

6）CommandContext类的getSession方法

```
public <T> T getSession(Class<T> sessionClass) {
    Session session = sessions.get(sessionClass);
    if (session == null) {
      //从SessionFactories集合中获取SessionClass类对应的Factory
      //这里传递的值为DbSqlSession.class，因此获取的是DbSqlSessionFactory对象
      SessionFactory sessionFactory = sessionFactories.get(sessionClass);
      if (sessionFactory==null) {
        throw new ActivitiException("no session factory configured for "+sessionClass.getName());
      }
      //DbSqlSessionFactory对象openSession获取的是DbSqlSession对象
      session = sessionFactory.openSession();
      sessions.put(sessionClass, session);
    }

    return (T) session;
}
```

注：最终获取的是DbSqlSession对象，使用对象能够实现对数据库的操作。