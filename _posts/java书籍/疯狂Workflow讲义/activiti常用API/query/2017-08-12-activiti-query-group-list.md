---
title: Activiti数据查询--GroupQuery查询list集合
tags: [activiti]
---

### GroupQuery中list方法的实现

GroupQueryImpl没有定义List方法，因此该方法的实现定义在了AbstractQuery抽象类中了。

1）AbstractQuery对象的list方法

```
public List<U> list() {
    this.resultType = ResultType.LIST;
    if (commandExecutor!=null) {
      return (List<U>) commandExecutor.execute(this);
    }
    return executeList(Context.getCommandContext(), null);
}

public abstract List<U> executeList(CommandContext commandContext, Page page);
```

注：可以看到这是一个模板方法，定义了抽象的executeList模板方法，供具体的子类来实现。

2）GroupQueryImpl类的executeList方法

```
public List<Group> executeList(CommandContext commandContext, Page page) {
    checkQueryOk();
    return commandContext
      .getGroupManager()
      .findGroupByQueryCriteria(this, page);
}
```

注：可以看到底层仍然是使用GroupManagerImpl类来实现查询的。

3）GroupManager类的findGroupByQueryCriteria方法

```
public List<Group> findGroupByQueryCriteria(GroupQueryImpl query, Page page) {
    return getDbSqlSession().selectList("selectGroupByQueryCriteria", query, page);
}
```

4）DbSqlSession类的selectList方法

AbstractQuery类继承了ListQueryParameterObject，因此GroupQueryImpl也继承了该类，因此可以做完实参传入，从而屏蔽了不同查询对象的差异，而只用来获取查询条件。

```
@SuppressWarnings("unchecked")
public List selectList(String statement, ListQueryParameterObject parameter, Page page) {   
    return selectList(statement, parameter);
}

@SuppressWarnings("unchecked")
public List selectList(String statement, ListQueryParameterObject parameter) {
    statement = dbSqlSessionFactory.mapStatement(statement);    
    if(parameter.firstResult == -1 ||  parameter.maxResults==-1) {
      return Collections.EMPTY_LIST;
    }
    List loadedObjects = null;
    String databaseType = dbSqlSessionFactory.databaseType;
    if(databaseType.equals("mssql") || databaseType.equals("db2")) {
      // use mybatis paging (native database paging not yet implemented)
      loadedObjects = sqlSession.selectList(statement, parameter, new RowBounds(parameter.getFirstResult(), parameter.getMaxResults()));
    } else {
      // use native database paging
      loadedObjects = sqlSession.selectList(statement, parameter);
    }
    return filterLoadedObjects(loadedObjects);
}
```

5）查询参数

ListQueryParameterObject只是一个普通的java类，用该类作为查询参数的封装，该类中的属性都提供了get方法。如GroupQueryImpl中定义了name

```
protected String name;
protected String nameLike;

public String getName() {
    return name;
}
public String getNameLike() {
    return nameLike;
}
```

因为GroupQueryImpl间接的继承了ListQueryParameterObject对象，并且作为实参传递给了查询。mybatis执行查询时，就能获取nameLike等参数值，从而封装到查询sql中。

6）查看map映射文件

activiti-engine-5.10-sources.jar\org\activiti\db\mapping\entity\Group.xml文件

```
<select id="selectGroupByQueryCriteria" 
    parameterType="org.activiti.engine.impl.GroupQueryImpl" resultMap="groupResultMap">

    ${limitBefore}
    select G.*
    <include refid="selectGroupByQueryCriteriaSql" />
    <if test="orderBy != null">
      order by ${orderBy}
    </if>
    ${limitAfter}

</select>

<sql id="selectGroupByQueryCriteriaSql">
    from ${prefix}ACT_ID_GROUP G
    <if test="userId != null">
      inner join ${prefix}ACT_ID_MEMBERSHIP M on G.ID_ = M.GROUP_ID_
      inner join ${prefix}ACT_ID_USER U on M.USER_ID_ = U.ID_
    </if>

    <where>
      <if test="id != null">
        G.ID_ = #{id}
      </if>
      <if test="name != null">
        and G.NAME_ = #{name}
      </if>
      <if test="nameLike != null">
        and G.NAME_ like #{nameLike}
      </if>
      <if test="type != null">
        and G.TYPE_ = #{type}
      </if>
      <if test="userId != null">
        and U.ID_ = #{userId}
      </if>
      <if test="procDefId != null">
        and exists (select ID_ from ${prefix} ACT_RU_IDENTITYLINK where PROC_DEF_ID_ = #{procDefId} and GROUP_ID_=G.ID_ )
      </if>
      
    </where>
  </sql> 

```

### listPage方法

需要提供两个int参数，第一个参数为符合查询结果的第一条数据的索引值，从0开始。第二个参数为查询的结果数量。

1）方法调用

```
List<Group> list=identityService.createGroupQuery().groupNameLike("经理").listPage();
```

2）AbstractQuery类的listPage方法

```
public List<U> listPage(int firstResult, int maxResults) {
    this.firstResult = firstResult;
    this.maxResults = maxResults;
    this.resultType = ResultType.LIST_PAGE;
    if (commandExecutor!=null) {
      return (List<U>) commandExecutor.execute(this);
    }
    return executeList(Context.getCommandContext(), new Page(firstResult, maxResults));
}
```

注：将两个int参数值设置到AbstractQuery对象的成员变量中，因此就能在ListQueryParameterObject中获取分页值了。

3）mybatis获取分页信息

mybatis的sql映射中最后插入了limitAfter

```
${limitAfter}
```

4）limitAfter值从何而来

limitAfter是在DbSqlSessionFactory中初始化的。

```
public static final Map<String, String> databaseSpecificLimitAfterStatements = new HashMap<String, String>();

databaseSpecificLimitAfterStatements.put("mysql", "LIMIT #{maxResults} OFFSET #{firstResult}");
```

5）在ProcessEngineConfigurationImpl类中初始化SqlSessionFactory\时设置的属性

```
protected void initSqlSessionFactory() {
    ...
     if(databaseType != null) {
          //设置分页
          properties.put("limitBefore" , DbSqlSessionFactory.databaseSpecificLimitBeforeStatements.get(databaseType));

          properties.put("limitAfter" , DbSqlSessionFactory.databaseSpecificLimitAfterStatements.get(databaseType));
    }
}
```