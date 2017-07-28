---
title: bpmx动态数据源--配置
tags: [work]
---

### 配置信息

1）app-resources.xml文件中配置

```
<bean id="dataSource_Default" class="com.alibaba.druid.pool.DruidDataSource" 
init-method="init" destroy-method="close">
```

注：类似于c3p0的一个数据库连接池

2）配置本地数据库的dataSource

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

注：配置dataSource，该dataSource包含上面的配置信息

3）配置容器刷新和启动，对数据源的监听

```
<bean id="dataSourceInitListener"
 class="com.hotent.platform.event.listener.DataSourceInitListener"></bean>
```

4）dao层数据源配置

jdbcTemplate数据源配置

```
<!--对数据库操作都从这里取jdbcTemplate-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

mybatis数据源配置

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:/conf/configuration.xml"/>
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations" >
        <list>
            <value>classpath:/com/hotent/*/maper/*.map.xml</value>
            <!-- 专利接口扫描文件 -->
            <value>classpath:/org/ipph/patent/net/zhihui/map/*.map.xml</value>
        </list>
    </property>
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```