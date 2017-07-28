---
title: bpmx动态数据源--使用
tags: [work]
---

在实际使用数据源时，需要手动调用DbContextHolder类的setDataSource方法。另外，该配置只针对当前线程有效，因此无需手动设置回去。下次web访问时，会启动新的线程，这样自然就切换到了默认的数据源上了。

### dao层类调用

在所有dao层类中，不管是mybatis的sqlSessionFactory还是jdbcTemplate的datasouce使用时都会从数据源获取connection，即调用DynamicDataSource类的getConnection方法。而获取的数据源连接却是被动态封装了起来。

### 如何动态的设置数据源信息呢？

1）新数据源的添加

DynamicDataSource类重写了父类的方法

```
@Override  
public void setTargetDataSources(Map<Object, Object> targetDataSources) {  
     super.setTargetDataSources(targetDataSources);  
     //重点  
     super.afterPropertiesSet();  
}
```

注：自定义实现，调用父类的方法完成重新设置动态数据源功能，并将其设置到resolvedDataSources集合中。

2）数据源的切换

在DbContextHolder类或DataSourceHelper类中都提供了相应的切换方法。

```
public static void setDataSource(String alias) {
    SysDataSourceService sysDataSourceService = AppUtil.getBean(SysDataSourceService.class);
    SysDataSource sysDataSource = sysDataSourceService.getByAlias(alias);
    if (sysDataSource == null)
        return;
    DbContextHolder.setDataSource(alias, sysDataSource.getDbType());
}
```

注：获取在bpmx系统中通过页面配置的数据源信息，具体的数据源信息存放到SYS_DATA_SOURCE表中。