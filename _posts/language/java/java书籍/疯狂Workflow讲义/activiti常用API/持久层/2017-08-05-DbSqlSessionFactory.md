---
title: DbSqlSessionFactory对象
tags: [activiti]
---

1）不同数据库sql语法

该类定义了多个不同数据库的语法，分别放置到不同map集合中。

```
//用来放置具体的statement语句
protected static final Map<String, Map<String, String>> databaseSpecificStatements = new HashMap<String, Map<String,String>>();

//放置不同数据库分页的实现语句
public static final Map<String, String> databaseSpecificLimitBeforeStatements = new HashMap<String, String>();
public static final Map<String, String> databaseSpecificLimitAfterStatements = new HashMap<String, String>();
```

以mysql数据库为例，展示三个集合的初始化设置

```
//mysql specific
//设置分页信息
databaseSpecificLimitBeforeStatements.put("mysql", "");
databaseSpecificLimitAfterStatements.put("mysql", "LIMIT #{maxResults} OFFSET #{firstResult}");

//设置具体的statement执行sql，将其放置到databaseSpecificStatements集合中
addDatabaseSpecificStatement("mysql", "selectNextJobsToExecute", "selectNextJobsToExecute_mysql");
addDatabaseSpecificStatement("mysql", "selectExclusiveJobsToExecute", "selectExclusiveJobsToExecute_mysql");
addDatabaseSpecificStatement("mysql", "selectProcessDefinitionsByQueryCriteria", "selectProcessDefinitionsByQueryCriteria_mysql");
addDatabaseSpecificStatement("mysql", "selectProcessDefinitionCountByQueryCriteria", "selectProcessDefinitionCountByQueryCriteria_mysql");
addDatabaseSpecificStatement("mysql", "selectDeploymentsByQueryCriteria", "selectDeploymentsByQueryCriteria_mysql");
addDatabaseSpecificStatement("mysql", "selectDeploymentCountByQueryCriteria", "selectDeploymentCountByQueryCriteria_mysql");
```

2）增、删、改、查缓存集合

```
protected Map<Class<?>,String>  insertStatements = Collections.synchronizedMap(new HashMap<Class<?>, String>());
protected Map<Class<?>,String>  updateStatements = Collections.synchronizedMap(new HashMap<Class<?>, String>());
protected Map<Class<?>,String>  deleteStatements = Collections.synchronizedMap(new HashMap<Class<?>, String>());
protected Map<Class<?>,String>  selectStatements = Collections.synchronizedMap(new HashMap<Class<?>, String>());
```

获取新增记录

```
public String getInsertStatement(PersistentObject object) {
    //获取执行sql的statement字符串，传递insertStatements缓存集合
    return getStatement(object.getClass(), insertStatements, "insert");
}
...
private String getStatement(Class<?> persistentObjectClass, Map<Class<?>,String> cachedStatements, String prefix) {
    //先从insertStatements集合中获取以缓存的statement
    String statement = cachedStatements.get(persistentObjectClass);
    if (statement!=null) {
      return statement;
    }
    //这里的statement是mybatis的映射文件的id（获取相应的sql定义信息）
    statement = prefix+ClassNameUtil.getClassNameWithoutPackage(persistentObjectClass);
    statement = statement.substring(0, statement.length()-6);
    cachedStatements.put(persistentObjectClass, statement);
    return statement;
}
```