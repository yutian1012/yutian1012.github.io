---
title: spring中Environment对象的属性值获取
tags: [spring]
---

通过spring的Environment对象可以检索应用的属性信息，一个应用的属性有很多来源: 属性文件(properties files)，JVM系统属性，系统环境变量，JNDI，servlet上下文参数，临时属性对象等。Environment对于property所扮演的角色是提供给使用者一个方便的服务接口用于配置属性源，或从属性源中获取属性。

1）获取属性方法

getProperty()方法有四个重载的变种形式：

```
String getProperty(String key)
String getProperty(String key,String defaultValue)
T getProperty(String key,Class<T> type)
T getProperty(String key,Class<T> type,T defaultValue)
```

前两种形式的getProperty方法都会返回String类型的值。后两种可以指定返回的属性类型。从而有针对性的处理业务数据，如返回Int类型的数据。

2）属性必须要存在

使用getRequiredProperty方法获取属性，如果属性没有定义的话，将会抛出IllegalStateException异常。

```
@Bean
public BlankDisc disc(){
    return new BlankDisc(env.getRequiredProperty("disc.title"),
        env.getRequiredProperty("disc.artist"));
}
```

3）判断属性是否存在

如果想检查一下某个属性是否存在的话，那么可以调用Environment的containsProperty方法

```
boolean titleExists=env.containsProperty("disc.title");
```

4）占位符获取属性

直接从Environment中检索属性是非常方便的，尤其是在Java配置中装配bean的时候。但是，Spring也提供了通过占位符装配属性的方法，这些占位符的值会来源于一个属性源。