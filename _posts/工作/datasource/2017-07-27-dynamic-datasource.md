---
title: bpmx动态数据源--页面动态配置数据源
tags: [work]
---

### 数据源对象的生成

页面上设置数据源，保存或会将信息设置到动态数据源集合中

1）设置数据源信息

![](/images/work/bpmx/datasource/datasourceset.png)

后台生成DataSource对象放置到动态数据源的resolvedDataSources集合中。

2）SysDataSourceController类的save方法

```
// 加入系统数据源
if (sysDataSource.getEnabled()) {
    try {
        DataSourceUtil.addDataSource(sysDataSource.getAlias(), sysDataSourceService.getDsFromSysSource(sysDataSource));
    } catch (Exception e) {
    }
}
```

3）sysDataSourceService.getDsFromSysSource方法根据配置信息生成DataSource对象。

```
public DataSource getDsFromSysSource(SysDataSource sysDataSource) {
    try {
        // 获取对象
        Class<?> _class = null;
        //类路径org.apache.commons.dbcp.BasicDataSource
        //可以查看sys_data_source表的class_path字段
        _class = Class.forName(sysDataSource.getClassPath());

        javax.sql.DataSource sqldataSource = null;
        // 初始化对象
        sqldataSource = (javax.sql.DataSource) _class.newInstance();

        // 开始set它的属性，解析配置的json数据，json为数据库配置信息
        String settingJson = sysDataSource.getSettingJson();
        JSONArray ja = JSONArray.fromObject(settingJson);

        for (int i = 0; i < ja.size(); i++) {
            JSONObject jo = ja.getJSONObject(i);

            Object value = BeanUtils.convertByActType(jo.getString("type"), jo.getString("value"));
            //设置属性到datasource对象中
            BeanUtils.setProperty(sqldataSource, jo.getString("name"), value);
        }

        // 如果有初始化方法，需要调用，必须是没参数的
        String initMethodStr = sysDataSource.getInitMethod();
        if (!StringUtil.isEmpty(initMethodStr)) {
            Method method = _class.getMethod(initMethodStr);
            method.invoke(sqldataSource);
        }

        return sqldataSource;

    } catch (Exception e) {
        LOGGER.debug(e.getMessage());
    }
    return null;
}
```

4）DataSourceUtil工具类添加数据源

```
public static void addDataSource(String key, DataSource dataSource) throws IllegalAccessException, NoSuchFieldException {
    DynamicDataSource dynamicDataSource = (DynamicDataSource) AppUtil.getBean(GLOBAL_DATASOURCE);
    dynamicDataSource.addDataSource(key, dataSource);
}
```

注：设置动态数据源，即调用DynamicDataSource类的addDataSource方法。