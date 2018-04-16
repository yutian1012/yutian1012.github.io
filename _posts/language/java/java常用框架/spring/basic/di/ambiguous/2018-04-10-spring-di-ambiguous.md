---
title: spring自动装配的歧义问题
tags: [spring]
---

1）歧义问题

自动装配让Spring完全负责将bean引用注入到构造参数和属性中。不过，仅有一个bean匹配所需的结果时，自动装配才是有效的。如果不仅有一个bean能够匹配结果的话，这种歧义性会阻碍Spring自动装配属性、构造器参数或方法参数。

2）出现歧义的一个实例

```
@Autowired
public void setDessert(Dessert dessert){
    this.dessert=dessert;
}
```

注：使用@Autowired注解实现依赖的注入

3）spring容器中有多个Dessert接口实现类

Dessert是一个接口，并且有三个类实现了这个接口，分别为Cake、Cookies和IceCream：

```
@Component
public class Cake implements Dessert{...}

@Component
public class Cookies implements Dessert{...}

@Component
public class IceCream implements Dessert{...}
```

注：三个实现均使用了@Component注解，在组件扫描的时候，能够发现它们并将其创建为Spring应用上下文里面的bean。

4）自动装配时异常信息

当Spring试图自动装配setDessert()中的Dessert参数时，它并没有唯一、无歧义的可选值。在从多种甜点中做出选择时，Spring无法做出选择。Spring此时别无他法，只好宣告失败并抛出异常。

Spring会抛出NoUniqueBeanDefinitionException：

```
nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException:
No qualifying bean of type[com.desserteater.Dessert]is defined:expected single matching bean but found 3:cake,cookies,iceCream
```