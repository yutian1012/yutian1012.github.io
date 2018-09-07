---
title: mybatis二级缓存
tags: [mybatis]
---

参考：http://www.cnblogs.com/fangjian0423/p/mybatis-cache.html

二级缓存的作用域是全局，换句话说，二级缓存已经脱离SqlSession的控制了。

1）二级缓存配置

一级缓存不需要配置任何东西，且默认打开。二级缓存需要在mapper文件上加上cache标签配置。

配置方式

```
mybatis全局配置文件中的setting中的cacheEnabled需要为true(默认为true，不设置也行)

mapper配置文件中需要加入<cache>节点

mapper配置文件中的select节点需要加上属性useCache需要为true(默认为true，不设置也行)
```

2）实例代码

配置二级缓存后，缓存作用域大于sqlSession，因此，在两个sqlSession作用域中能够共享二级缓存数据。

```
@Test
public void testCache2() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    try {
        String sql = "org.format.mybatis.cache.UserMapper.getById";
        User user = (User)sqlSession.selectOne(sql, 1);
        log.debug(user);
        sqlSession.commit();//一定要提交

        User user2 = (User)sqlSession2.selectOne(sql, 1);
        log.debug(user2);
    } finally {
        sqlSession.close();
        sqlSession2.close();
    }
}
```

注意，sqlSession第一次查询一定要提交，不提交二级缓存不会生效，sqlSession2还是会查询数据库。

3）二级缓存标签的解析

在mapper文件中配置二级缓存cache后，是如何生效的？

XMLMappedBuilder类用于解析每个mapper配置文件，每一个mapper配置都会实例化一个XMLMapperBuilder类。

XMLMappedBuilder类configurationElement方法

```
private void configurationElement(XNode context) {
    try {
      String namespace = context.getStringAttribute("namespace");
      if (namespace.equals("")) {
          throw new BuilderException("Mapper's namespace cannot be empty");
      }
      builderAssistant.setCurrentNamespace(namespace);
      cacheRefElement(context.evalNode("cache-ref"));
      cacheElement(context.evalNode("cache"));
      parameterMapElement(context.evalNodes("/mapper/parameterMap"));
      resultMapElements(context.evalNodes("/mapper/resultMap"));
      sqlElement(context.evalNodes("/mapper/sql"));
      buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
    }
}
```

解析cache方法cacheElement

```
private void cacheElement(XNode context) throws Exception {
    if (context != null) {
        //获取标签定义的属性信息
      String type = context.getStringAttribute("type", "PERPETUAL");
      Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
      String eviction = context.getStringAttribute("eviction", "LRU");
      Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
      Long flushInterval = context.getLongAttribute("flushInterval");
      Integer size = context.getIntAttribute("size");
      boolean readWrite = !context.getBooleanAttribute("readOnly", false);
      Properties props = context.getChildrenAsProperties();

      //使用MapperBuilderAssistant帮助类创建cache
      builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, props);
    }
}
```

解析完cache标签之后会使用builderAssistant的userNewCache方法，这里的builderAssistant是一个MapperBuilderAssistant类型的帮助类，每个XMLMappedBuilder构造的时候都会实例化这个属性，MapperBuilderAssistant类内部有个Cache类型的currentCache属性，这个属性也就是mapper配置文件中cache节点所代表的值

```
public Cache useNewCache(Class<? extends Cache> typeClass,
  Class<? extends Cache> evictionClass,
  Long flushInterval,
  Integer size,
  boolean readWrite,
  Properties props) {

    typeClass = valueOrDefault(typeClass, PerpetualCache.class);
    evictionClass = valueOrDefault(evictionClass, LruCache.class);
    Cache cache = new CacheBuilder(currentNamespace)
        .implementation(typeClass)
        .addDecorator(evictionClass)
        .clearInterval(flushInterval)
        .size(size)
        .readWrite(readWrite)
        .properties(props)
        .build();

    configuration.addCache(cache);
    currentCache = cache;
    return cache;
}
```

注：最终mapper配置文件中的cache节点被解析到了XMLMapperBuilder实例中的builderAssistant属性中的currentCache值里。