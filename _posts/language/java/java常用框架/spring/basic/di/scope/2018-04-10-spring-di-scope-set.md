---
title: spring作用域设置
tags: [spring]
---

要使用@Scope注解，它可以与@Component或@Bean一起使用，同时也可以以xml方式配置。

1）与@Component结合使用

```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad{...}
```

使用ConfigurableBeanFactory类的SCOPE_PROTOTYPE常量设置了原型作用域。当然也可以使用@Scope("prototype")，但是使用SCOPE_PROTOTYPE常量更加安全并且不易出错。

2）与@Bean结合使用

```
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Notepad notepad(){
    return new Notepad();
}
```

3）xml中配置，使用bean的scope属性

```
<bean id="notepad" class="com.myapp.Notepad" scope="prototype"/>
```