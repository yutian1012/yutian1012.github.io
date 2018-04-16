---
title: spring自动装配的歧义问题--设置限定符
tags: [spring]
---

Spring的限定符能够在所有可选的bean上进行缩小范围的操作，最终能够达到只有一个bean满足所规定的限制条件。如果将所有的限定符都用上后依然存在歧义性，那么你可以继续使用更多的限定符来缩小选择范围（限定符的组合使用）。

1）@Qualifier注解的使用（设置查找的bean id）

@Qualifier注解是使用限定符的主要方式。它可以与@Autowired和@Inject协同使用，在注入的时候指定想要注入进去的是哪个bean。

```
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert){
    this.dessert=dessert;
}
```

注：为@Qualifier注解所设置的参数就是想要注入的bean的ID。

注2：所有使用@Component注解声明的类都会创建为bean，并且bean的ID为首字母变为小写的类名。

2）问题（类名耦合）

这里的问题在于setDessert()方法上所指定的限定符与要注入的bean的名称是紧耦合的。对类名称的任意改动都会导致限定符失效。

基于默认的beanID作为限定符是非常简单的，但这有可能会引入一些问题。如果你重构了IceCream类，将其重命名为Gelato的话，那此时就无法匹配setDessert方法中的限定符，自动装配会失败。

3）定义bean时指定限定符（解决耦合类名）

```
@Component
@Qualifier("cold")
public class IceCream implements Dessert{...}

@Bean
@Qualifier("cold")
public Dessert iceCream(){
    return new IceCream();
}
```

在这种情况下，cold限定符分配给了IceCreambean。因为它没有耦合类名，因此你可以随意重构IceCream的类名，而不必担心会破坏自动装配。在注入的地方，只要引用cold限定符就可以了。

```
@Autowired
@Qualifier("cold")
public void setDessert(Dessert dessert){
    this.dessert=dessert;
}
```

4）自定义限定符注解（解决多个限定问题）

如果多个bean都具备相同特性的话（同时使用了相同的限定符表示），这种做法也会出现注入时歧义问题。

java不允许在一个类上添加多个相同的注解，因此，我们可以自定义注解，继承@Qualifier，从而实现在一个类上添加多个限定符。

```
# 错误的方式添加多限定符
@Component
@Qualifier("cold")
@Qualifier("creamy")
public class IceCream implements Dessert{...}
```

定义限定符注解，在注解上添加了@Qualifier，那么该注解也就具备了限定符的功能了。

```
@Target({ElementType.CONSTRUCTOR,ElementType.FIELD,ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Creamy{}
```

定义bean时添加自定义限定符，可以添加多个限定符

```
//这里使用多个限定符修饰bean的定义
@Component
@Cold
@Creamy
public class IceCream implements Dessert{...}
```

注入bean时添加自定义限定符，可以添加多个限定符，从而缩小选择范围

```
//注入时也可以使用多个限定符缩小选择范围
@Autowired
@Cold
@Creamy
public void setDessert(Dessert dessert){
    this.dessert=dessert;
}
```