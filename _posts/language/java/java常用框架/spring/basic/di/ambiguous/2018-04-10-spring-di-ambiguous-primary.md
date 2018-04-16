---
title: spring自动装配的歧义问题--设置首选
tags: [spring]
---

在声明bean的时候，通过将其中一个可选的bean设置为首选（primary）bean能够避免自动装配时的歧义性。当遇到歧义性的时候，Spring将会使用首选的bean，而不是其他可选的bean。

在Spring中，可以通过@Primary来设置首选bean。

@Primary能够与@Component组合用在组件扫描的bean上，也可以与@Bean组合用在Java配置的bean声明中。设置可以在xml上设置primary属性实现。

1）与@Component结合使用

```
@Component
@Primary
public class IceCream implements Dessert{...}
```

2）与@Bean组合使用

```
@Bean
@Primary
public Dessert iceCream(){
    return new IceCream();
}
```

3）使用xml方式配置

```
<bean id="iceCream" class="com.desserteater.IceCream" primary="true"/>
```

4）存在的问题

设置首选bean的局限性在于@Primary无法将可选方案的范围限定到唯一一个无歧义性的选项中。它只能标示一个优先的可选方案。当首选bean的数量超过一个时，我们并没有其他的方法进一步缩小可选范围。