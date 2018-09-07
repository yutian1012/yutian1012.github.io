---
title: mybatis一级缓存
tags: [mybatis]
---

参考：http://www.cnblogs.com/fangjian0423/p/mybatis-cache.html

1. 一级缓存使用实例

```
@Test
public void test() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
        User user = (User)sqlSession.selectOne("org.format.mybatis.cache.UserMapper.getById", 1);
        log.debug(user);
        User user2 = (User)sqlSession.selectOne("org.format.mybatis.cache.UserMapper.getById", 1);
        log.debug(user2);
    } finally {
        sqlSession.close();
    }
}
```

注：这里查询是在同一个SqlSession中，如果存在两个sqlSession查询相同的sql语句，那么会发送两条查询语句。而在一个sqlSession中只会发送一条查询语句（可通过查看输出日志确定）。

2. 源码分析

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

2）Executor的创建过程（装饰器）

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

其中cacheEnabled这个属性是mybatis总配置文件中settings节点中cacheEnabled子节点的值，默认就是true，也就是说我们在mybatis总配置文件中不配cacheEnabled的话，它也是默认为打开的。

3）查看CachingExecutor的query方法执行

SqlSession.selectOne可以最终跟踪代码到Executor类的query方法上

```
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    Cache cache = ms.getCache();//获取二级缓存
    if (cache != null) {
      flushCacheIfRequired(ms);//判断ms是否需要flush缓存
      //根据ms配置判断是否使用缓存
      if (ms.isUseCache() && resultHandler == null) { 
        ensureNoOutParams(ms, parameterObject, boundSql);
        if (!dirty) {//如果数据没有标记为dirty，则使用读写锁来读取缓存内容
          cache.getReadWriteLock().readLock().lock();
          try {
            @SuppressWarnings("unchecked")
            List<E> cachedList = (List<E>) cache.getObject(key);
            if (cachedList != null) return cachedList;
          } finally {
            cache.getReadWriteLock().readLock().unlock();
          }
        }
        //delegate也就是SimpleExecutor
        List<E> list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);

        // issue #578. Query must be not synchronized to prevent deadlocks
        tcm.putObject(cache, key, list);//
        
        return list;
      }
    }
    return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

注：delegate也就是SimpleExecutor,SimpleExecutor没有Override父类的query方法，因此最终执行了SimpleExecutor的父类BaseExecutor的query方法。

注2：一级缓存位于BaseExecutor类的query方法中