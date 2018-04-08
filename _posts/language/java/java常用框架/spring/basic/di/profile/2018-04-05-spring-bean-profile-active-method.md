---
title: spring的环境激活方式
tags: [spring]
---

1）作为DispatcherServlet的初始化参数；

```
<servlet>
    ...
    <init-param>
      <param-name>spring.profiles.active</param-name>
      <param-value>production</param-value>
    </init-param>
</servlet>
```

注：web.xml中配置

2）作为Web应用的上下文参数；

```
<context-param>
  <param-name>spring.profiles.active</param-name>
  <param-value>production</param-value>
</context-param>
```

注：web.xml中配置

3）作为JNDI条目；

4）作为环境变量；

```
ConfigurableEnvironment.setActiveProfiles("test")
```

5）作为JVM的系统属性

tomcat中catalina.bat（.sh中不用“set”）添加JAVA_OPS，通过设置active选择不同配置文件

```
set JAVA_OPTS="-Dspring.profiles.active=test"
```

eclipse中启动tomcat。项目右键run as –> run configuration–>Arguments–> VM arguments中添加

```
-Dspring.profiles.active="local"
```

6）在集成测试类上，使用@ActiveProfiles注解设置。

测试框架中提供的注解@ActiveProfiles

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={PersistenceTestConfig.class})\
@ActiveProfiles("dev")
public class PersistenceTest{}
```