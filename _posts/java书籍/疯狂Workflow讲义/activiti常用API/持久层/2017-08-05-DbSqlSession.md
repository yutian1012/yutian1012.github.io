---
title: DbSqlSession对象
tags: [activiti]
---

activiti内部使用的是mybatis来操作数据库，通过引用的jar包也能看出来。

1）SaveGroupCmd保存用户组

SaveGroupCmd命令对象的execute方法

```
commandContext.getGroupManager().insertGroup(group);
```

GroupManager类的insertGroup方法

```
public void insertGroup(Group group) {
    getDbSqlSession().insert((PersistentObject) group);
}
```

2）DbSqlSession类的insert方法

```
public void insert(PersistentObject persistentObject) {
    if (persistentObject.getId()==null) {
      String id = dbSqlSessionFactory.getIdGenerator().getNextId();  
      persistentObject.setId(id);
    }
    //将待保存的对象放入到insertedObjects集合中（List<PersistentObject>）
    insertedObjects.add(persistentObject);
    //将对象同时放入到cachedObjects集合中
    //对象类型（Map<Class<?>,Map<String, CachedObject>>）
    cachePut(persistentObject, false);
}
```

3）对象何时被真正保存到数据库中的？

CommandContextInterceptor类中的execute方法在执行完成拦截器后会调用CommandContext类的close方法，close方法会调用flushSessions()方法，

CommandContext类的flushSessions方法

```
protected void flushSessions() {
    for (Session session : sessions.values()) {
      session.flush();
    }
}
```

会调用DbSqlSession类的flush方法会调用flushInserts()方法

```
protected void flushInserts() {
    for (PersistentObject insertedObject: insertedObjects) {
      //获取DbSqlSessionFactory缓冲集合中定义的mybaits的sql映射文件的id值
      String insertStatement = dbSqlSessionFactory.getInsertStatement(insertedObject);
      //判断insertStatement是否在statementMappings集合中
      //针对需要特殊处理的一些sql
      insertStatement = dbSqlSessionFactory.mapStatement(insertStatement);

      if (insertStatement==null) {
        throw new ActivitiException("no insert statement for "+insertedObject.getClass()+" in the ibatis mapping files");
      }
      
      log.fine("inserting: "+toString(insertedObject));
      //执行插入操作
      //sqlSession类型为org.apache.ibatis.session.SqlSession
      sqlSession.insert(insertStatement, insertedObject);
    }
    //清空待插入集合中的对象
    insertedObjects.clear();
}
```

注：通过运行测试类，发现mybatis使用的SqlSession对象为DefaultSqlSession。

下面会介绍具体的底层执行过程：Mybatis的sql映射信息。

### Mybatis的sql映射文件

1）mybatis类的SqlSessionFactory初始化

ProcessEngineConfigImpl类中的initSqlSessionFactory方法用于初始化Mybatis持久层框架的SqlSessionFactory对象。

```
protected void initSqlSessionFactory() {
    if (sqlSessionFactory==null) {
      InputStream inputStream = null;
      try {
        //读取mybatis配置文件信息，可以解压所activiti的jar包
        //文件路径：org/activiti/db/mapping/mappings.xml
        //该配置文件中同时也指定了mapper文件的位置
        //org/activiti/db/mapping/entity/xxx.xml
        inputStream = getMyBatisXmlConfigurationSteam();

        // update the jdbc parameters to the configured ones...
        Environment environment = new Environment("default", transactionFactory, dataSource);
        //读取mybatis配置文件
        Reader reader = new InputStreamReader(inputStream);
        //设置属性信息
        Properties properties = new Properties();
        properties.put("prefix", databaseTablePrefix);
        if(databaseType != null) {
          properties.put("limitBefore" , DbSqlSessionFactory.databaseSpecificLimitBeforeStatements.get(databaseType));
          properties.put("limitAfter" , DbSqlSessionFactory.databaseSpecificLimitAfterStatements.get(databaseType));
        }
        XMLConfigBuilder parser = new XMLConfigBuilder(reader,"", properties);
        //获取配置类
        Configuration configuration = parser.getConfiguration();
        configuration.setEnvironment(environment);
        configuration.getTypeHandlerRegistry().register(VariableType.class, JdbcType.VARCHAR, new IbatisVariableTypeHandler());
        configuration = parser.parse();
        //生成SqlSessionFactory
        sqlSessionFactory = new DefaultSqlSessionFactory(configuration);
      } catch (Exception e) {
        throw new ActivitiException("Error while building ibatis SqlSessionFactory: " + e.getMessage(), e);
      } finally {
        IoUtil.closeSilently(inputStream);
      }
    }
  }
```

2）查看sql映射文件

以Group用户组为例，查看文件org/activiti/db/mapping/entity/Group.xml

```
<?xml version="1.0" encoding="UTF-8" ?> 

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
  
<mapper namespace="org.activiti.engine.impl.persistence.entity.GroupEntity">

  <!-- GROUP INSERT -->

  <insert id="insertGroup" parameterType="org.activiti.engine.impl.persistence.entity.GroupEntity">
    insert into ${prefix}ACT_ID_GROUP (ID_, REV_, NAME_, TYPE_)
    values (
      #{id ,jdbcType=VARCHAR},
      1, 
      #{name ,jdbcType=VARCHAR},
      #{type ,jdbcType=VARCHAR}
    )
  </insert>
  ....
</mapper>
```

注：可以看到命名空间为GroupEntity类的全限定类名。

3）使用是如何获取的？

```
//这里的prefix前缀为insert，statement的值为insertGroupEntity
statement = prefix+ClassNameUtil.getClassNameWithoutPackage(persistentObjectClass);
//去掉末尾的Entity6位字符，得到insertGroup
statement = statement.substring(0, statement.length()-6);
```

最后调用SqlSession向数据库中保存数据

```
//insertStatement的值为insertGroup
sqlSession.insert(insertStatement, insertedObject);
```

注：org.apache.ibatis.session.SqlSession由Mybatis提供。

4）底层mybatis的sqlmap集合

Mybatis的Configaration类的mappedStatements成员集合变量中包含所有的sql信息。

```
org.activiti.engine.impl.persistence.entity.GroupEntity.insertGroup=org.apache.ibatis.mapping.MappedStatement@52de51b6
...
insertGroup=org.apache.ibatis.mapping.MappedStatement@52de51b6
```

注：不难看出，map中放置的信息包括两部分，一部分是sql映射文件中定义的sql的id值，另一部分为sql映射文件的命名空间+id值。这两部分的值引用的MappedStatement对象是相同的对象。因此在获取查询sql的statement时，可以用这两种方式获取。