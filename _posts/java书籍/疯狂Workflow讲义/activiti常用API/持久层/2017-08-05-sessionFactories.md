---
title: sessionFactory对象
tags: [activiti]
---

### Session对象获取（操作数据库的接口）

CommandConext类中包含了SessionFactory集合信息，用于获取相应的session，进而执行数据库的操作。

1）CommandContext类获取session

```
protected Map<Class< ? >, SessionFactory> sessionFactories;
protected Map<Class< ? >, Session> sessions = new HashMap<Class< ? >, Session>();
...
//Class对象为DbSqlSession.class，可以查看AbstractManager类调用方法时传递的参数
public <T> T getSession(Class<T> sessionClass) {
    Session session = sessions.get(sessionClass);
    if (session == null) {

      SessionFactory sessionFactory = sessionFactories.get(sessionClass);
      if (sessionFactory==null) {
        throw new ActivitiException("no session factory configured for "+sessionClass.getName());
      }
      session = sessionFactory.openSession();
      sessions.put(sessionClass, session);
    }

    return (T) session;
}
```

2）sessionFactories集合的初始化操作

```
 public CommandContext(Command<?> command, ProcessEngineConfigurationImpl processEngineConfiguration) {
    this.command = command;
    this.processEngineConfiguration = processEngineConfiguration;
    this.failedJobCommandFactory = processEngineConfiguration.getFailedJobCommandFactory();
    //从ProcessEgineConfig配置中获取session工厂信息
    sessionFactories = processEngineConfiguration.getSessionFactories();
    this.transactionContext = processEngineConfiguration
    .getTransactionContextFactory().openTransactionContext(this);
}
```

注：SessionFactory的配置信息位于processEngineConfiguration对象中。

3）ProcessEngineConfigurationImpl类的sessionFactory配置信息

```
protected Map<Class<?>, SessionFactory> sessionFactories;
...
//初始化SessionFactory集合
protected void initSessionFactories() {
    if (sessionFactories==null) {
      sessionFactories = new HashMap<Class<?>, SessionFactory>();

      //添加系统默认的SessionFactory
      //1 DbSqlSessionFactory对象
      dbSqlSessionFactory = new DbSqlSessionFactory();
      dbSqlSessionFactory.setDatabaseType(databaseType);
      dbSqlSessionFactory.setIdGenerator(idGenerator);
      dbSqlSessionFactory.setSqlSessionFactory(sqlSessionFactory);
      dbSqlSessionFactory.setDbIdentityUsed(isDbIdentityUsed);
      dbSqlSessionFactory.setDbHistoryUsed(isDbHistoryUsed);
      dbSqlSessionFactory.setDatabaseTablePrefix(databaseTablePrefix);
      dbSqlSessionFactory.setDatabaseSchema(databaseSchema);
      addSessionFactory(dbSqlSessionFactory);
      
      //2 GenericManagerFactory对象，与系统中的实体对象相关
      addSessionFactory(new GenericManagerFactory(AttachmentManager.class));
      addSessionFactory(new GenericManagerFactory(CommentManager.class));
      addSessionFactory(new GenericManagerFactory(DeploymentManager.class));
      addSessionFactory(new GenericManagerFactory(ExecutionManager.class));
      addSessionFactory(new GenericManagerFactory(HistoricActivityInstanceManager.class));
      addSessionFactory(new GenericManagerFactory(HistoricDetailManager.class));
      addSessionFactory(new GenericManagerFactory(HistoricProcessInstanceManager.class));
      addSessionFactory(new GenericManagerFactory(HistoricTaskInstanceManager.class));
      addSessionFactory(new GenericManagerFactory(IdentityInfoManager.class));
      addSessionFactory(new GenericManagerFactory(IdentityLinkManager.class));
      addSessionFactory(new GenericManagerFactory(JobManager.class));
      addSessionFactory(new GenericManagerFactory(GroupManager.class));
      addSessionFactory(new GenericManagerFactory(MembershipManager.class));
      addSessionFactory(new GenericManagerFactory(ProcessDefinitionManager.class));
      addSessionFactory(new GenericManagerFactory(PropertyManager.class));
      addSessionFactory(new GenericManagerFactory(ResourceManager.class));
      addSessionFactory(new GenericManagerFactory(TableDataManager.class));
      addSessionFactory(new GenericManagerFactory(TaskManager.class));
      addSessionFactory(new GenericManagerFactory(UserManager.class));
      addSessionFactory(new GenericManagerFactory(VariableInstanceManager.class));
      addSessionFactory(new GenericManagerFactory(EventSubscriptionManager.class));
    }
    //自定义的sessionFactory对象
    if (customSessionFactories!=null) {
      for (SessionFactory sessionFactory: customSessionFactories) {
        addSessionFactory(sessionFactory);
      }
    }
}
```

4）sessionFactory集合的key

ProcessEngineConfigurationImpl类的addSessionFactory方法

```
protected void addSessionFactory(SessionFactory sessionFactory) {
    sessionFactories.put(sessionFactory.getSessionType(), sessionFactory);
}
```

注：该方法通过获取传递SessionFactory对象，调用其getSessionType方法作为key。

GenricmanagerFactory类的getSessionType

```
public GenericManagerFactory(Class< ? extends Session> managerImplementation) {
    this.managerImplementation = managerImplementation;
}

public Class< ? > getSessionType() {
    return managerImplementation;
}
```

以GroupManager类为例

```
addSessionFactory(new GenericManagerFactory(GroupManager.class))
```

放置到sessionFactory集合（map的key值）为GroupManager.class对象。

以DbSqlSessionFactory为例

```
addSessionFactory(dbSqlSessionFactory);
```

放置到sessionFactory集合（map的key值）为DbSqlSession.class对象

```
public Class< ? > getSessionType() {
    return DbSqlSession.class;
}
```

5）DbSqlSessionFactory的openSession方法

```
public Session openSession() {
    return new DbSqlSession(this);
}
```

最终返回Session对象，用于数据的操作。