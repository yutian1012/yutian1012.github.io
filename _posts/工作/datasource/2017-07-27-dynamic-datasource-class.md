---
title: bpmx动态数据源--DynamicDataSource实现类
tags: [work]
---

### DynamicDataSource的实现

hotent实现DynamicDataSource继承extends AbstractRoutingDataSource。使用spring提供的AbstractRoutingDataSource实现动态数据源。

1）继承体系

AbstractRoutingDataSource继承了AbstractDataSource，而AbstractDataSource又是javax.sql.DataSource的子类。因此DynamicDataSource底层实现了DataSource类。

注：AbstractRoutingDataSource类是由spring提供的。

2）AbstractRoutingDataSource类实现的重要方法（源码查看）

```
public Connection getConnection() throws SQLException {  
    return determineTargetDataSource().getConnection();  
}
```

注：这里determineTargetDataSource()是重点方法，决定了在众多数据源存在的情况下，该使用什么样的策略，使用哪个数据源做为获取数据库连接的数据源。

```
protected DataSource determineTargetDataSource() {
    Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
    Object lookupKey = determineCurrentLookupKey();
    DataSource dataSource = (DataSource) this.resolvedDataSources.get(lookupKey);
    if (dataSource == null) {
        dataSource = this.resolvedDefaultDataSource;
    }
    if (dataSource == null) {
        throw new IllegalStateException(
"Cannot determine target DataSource for lookup key [" + lookupKey + "]");
    }
    return dataSource;
}
```

注：determineCurrentLookupKey()需要进行实现的抽象方法，该方法返回需要使用的DataSource的key值，然后根据这个key从resolvedDataSources这个map里取出对应的DataSource，如果找不到，则用默认的resolvedDefaultDataSource。

注2：determineCurrentLookupKey是一个抽象的方法，用于被子类实现。

重点：要在我们自定义的dynamicDatsource类中重写或实现determineCurrentLookupKey方法。

3）AbstractRoutingDataSource类获取配置属性

配置文件

```
<bean id="dataSource" class="com.hotent.core.db.datasource.DynamicDataSource">
    <property name="targetDataSources"  >
        <map>
            <entry key="dataSource_Default" value-ref="dataSource_Default" />
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="dataSource_Default" />
</bean>
```

AbstractRoutingDataSource实现了InitializingBean接口，因此，在spring容器初始化后会调用afterPropertiesSet方法

```
public void afterPropertiesSet() {
    if (this.targetDataSources == null) {
        throw new IllegalArgumentException("Property 'targetDataSources' is required");
    }
    this.resolvedDataSources = new HashMap<Object, DataSource>(this.targetDataSources.size());
    for (Map.Entry entry : this.targetDataSources.entrySet()) {
        Object lookupKey = resolveSpecifiedLookupKey(entry.getKey());
        DataSource dataSource = resolveSpecifiedDataSource(entry.getValue());
        this.resolvedDataSources.put(lookupKey, dataSource);
    }
    if (this.defaultTargetDataSource != null) {
        this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
    }
}
```

注：可以看到解析配置文件中定义的属性，所有的数据源都会被放置到resolveDataSources集合中。另外又单独指定了默认的DataSource（该DataSource也在集合中）。

4）自定义数据源类DynamicDataSource，实现determineCurrentLookupKey方法

```
protected Object determineCurrentLookupKey() {
    return DbContextHolder.getDataSource();  
}
```

注：自定义的实现类的determineCurrentLookupKey方法通过获取当前线程设置的数据源，从而实现动态查询数据源。即在执行相应的数据源操作时，需要向当前线程中设置数据源信息。