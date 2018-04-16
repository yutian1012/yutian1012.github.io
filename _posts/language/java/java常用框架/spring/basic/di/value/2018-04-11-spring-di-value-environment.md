---
title: spring注入外部值
tags: [spring]
---

在Spring中，处理外部值的最简单方式就是声明属性源并通过Spring的Environment来检索属性。

注：这种方式可以快速获取属性文件中定义的属性信息。

1）实例

```
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class EnvironmentConfig {

  @Autowired
  Environment env;
  
  @Bean
  public BlankDisc blankDisc() {
    return new BlankDisc(
        env.getProperty("disc.title"),
        env.getProperty("disc.artist"));
  }
  
}
```

注：spring中Bean对象使用Environment对象，可以通过@Autowired注解将该对象注入，然后调用相应的方法即可。

2）Environment对象

Spring的环境抽象提供了用于检索一系列的property sources属性配置文件。

可以从spring应用上下文中获取Environment对象。进而获取相关的环境信息。

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment(); 
boolean containsFoo = env.containsProperty("foo"); 
```

3）@PropertySource注解

@PropertySource注解提供了一个方便的方式，用于增加一个PropertySource到Spring的环境中