---
title: mybatis的核心类
tags: [mybatis]
---

参考：http://www.cnblogs.com/fangjian0423/p/mybatis-cache.html

org.apache.ibatis.session.Configuration类：MyBatis全局配置信息类

org.apache.ibatis.session.SqlSessionFactory接口：操作SqlSession的工厂接口，具体的实现类是DefaultSqlSessionFactory

org.apache.ibatis.session.SqlSession接口：执行sql，管理事务的接口，具体的实现类是DefaultSqlSession

org.apache.ibatis.executor.Executor接口：sql执行器，SqlSession执行sql最终是通过该接口实现的，常用的实现类有SimpleExecutor和CachingExecutor,这些实现类都使用了装饰者设计模式

1）SqlSession接口的实现类

DefaultSqlSession，是SqlSession接口实现类，MyBatis默认使用这个类。很多的查询方法都是通过调用该类的方法来实现的。

查看DefaultSqlSession类的创建

```
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);

      final Executor executor = configuration.newExecutor(tx, execType, autoCommit);
      return new DefaultSqlSession(configuration, executor);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
}
```

可以看出DefaultSqlSession中内部持有Executor，并且Executor持有事务。

2）Executor的创建过程

Executor接口的实现类是由Configuration构造的

```
public Executor newExecutor(Transaction transaction, ExecutorType executorType, boolean autoCommit) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor, autoCommit);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

注：Executor根据ExecutorType的不同而创建，最常用的是SimpleExecutor。最后我们发现如果cacheEnabled这个属性为true的话，那么executor会被包一层装饰器，这个装饰器是CachingExecutor。