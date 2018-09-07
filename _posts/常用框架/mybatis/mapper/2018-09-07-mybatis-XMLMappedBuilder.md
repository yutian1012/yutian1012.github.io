---
title: mybatis XMLMappedBuilder类
tags: [mybatis]
---

参考：http://www.cnblogs.com/fangjian0423/p/mybatis-cache.html

解析每个mapper配置文件的解析类，每一个mapper配置都会实例化一个XMLMapperBuilder类。

1）文件的解析过程

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

2）查看对select节点的解析

XMLMapperBuilder会解析select节点，解析select节点的时候使用XMLStatementBuilder进行解析（包括其他insert，update，delete节点）。

查看XMLStatementBuilder类的parseStatementNode方法实现