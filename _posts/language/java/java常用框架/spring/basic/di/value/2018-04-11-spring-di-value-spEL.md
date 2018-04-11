---
title: spring表达式装配数据
tags: [spring]
---

Spring3引入了Spring表达式语言（SpringExpressionLanguage，SpEL），它能够以一种强大和简洁的方式将值装配到bean属性和构造器参数中，在这个过程中所使用的表达式会在运行时计算得到值。使用SpEL，你可以实现超乎想象的装配效果，这是使用其他的装配技术难以做到的（甚至是不可能的）。

1）SpEL表达式装配数据

如果通过组件扫描创建bean的话，在注入属性和构造器参数时，可以使用@Value注解，结合SpEL表达式来传递参数。

```
//如从系统属性中获取相关的属性
public BlankDisc(@Value("#{systemProperties['disc.title']}")String title,@Value("#{systemProperties['disc.artist']}")String artist){
    this.title=title;
    this.artist=artist;
}
```

注：可以看到SpEL表达式与占位符很相似，但SpEL表达式功能更加强大。

2）xml方式装配数据

在XML配置中，你可以将SpEL表达式传入property或constructor-arg的value属性中，或者将其作为p-命名空间或c-命名空间条目的值。

```
<bean id="sgtPeppers" class="soundsystem.BlankDisc" 
    c:_title="#{systemProperties['disc.title']}"
    c:_artist="#{systemProperties['disc.artist']}"/>
```