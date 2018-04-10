---
title: spring作用域的代理模式
tags: [spring]
---

作用域代理能够延迟注入请求和会话作用域的bean，一般用于web环境下。

1）问题引入

就购物车bean来说，会话作用域是最为合适的，因为它与给定的用户关联性最大。

```
@Component
@Scope(value=WebApplicationContext.SCOPE_SESSION,
    proxyMode=ScopedProxyMode.INTERFACES)
public ShoppingCart cart(){...}
```

将value设置成了WebApplicationContext中的SCOPE_SESSION常量（它的值是session）。这会告诉Spring为Web应用中的每个会话创建一个ShoppingCart。这会创建多个ShoppingCartbean的实例，但是对于给定的会话只会创建一个实例，在当前会话相关的操作中，这个bean实际上相当于单例的。

要注意的是，@Scope同时还有一个proxyMode属性，它被设置成了ScopedProxyMode.INTERFACES。这个属性解决了将会话或请求作用域的bean注入到单例bean中所遇到的问题。

2）spring初始化bean时如何解决依赖会话时期的bean对象问题

```
@Component
public class StoreService{
    @Autowired
    public void setShoppingCart(ShoppingCart shoppingCart){
        this.shoppingCart=shoppingCart;
    }
    ...
}
```

问题：spring初始化StoreService单例对象时，需要将ShoppingCart实例注入到StoreService的属性上，而此时并没有任何会话参数（因为没有用户登录，就没有会话产生）。另外，系统中将会有多个ShoppingCart实例，每个用户一个。我们并不想让Spring注入某个固定的ShoppingCart实例到StoreService中。

3）ScopedProxyMode.INTERFACES

ScopedProxyMode.INTERFACES的作用是，Spring并不会将实际的ShoppingCartbean注入到StoreService中，Spring会注入一个到ShoppingCartbean的代理，这个代理会暴露与ShoppingCart相同的方法，所以StoreService会认为它就是一个购物车。但是，当StoreService调用ShoppingCart的方法时，代理会对其进行懒解析并将调用委托给会话作用域内真正的ShoppingCartbean。

问题：proxyMode属性被设置成了ScopedProxyMode.INTERFACES，这表明这个代理要实现ShoppingCart接口，并将调用委托给实现bean。如果ShoppingCart是接口而不是类的话，这是可以的（也是最为理想的代理模式）。但如果ShoppingCart是一个具体的类的话，Spring就没有办法创建基于接口的代理了。

4）ScopedProxyMode.TARGET_CLASS

如果代理的不是接口而是类时，必须使用CGLib来生成基于类的代理。所以，如果bean类型是具体类的话，我们必须要将proxyMode属性设置为ScopedProxyMode.TARGET_CLASS，以此来表明要以生成目标类扩展的方式创建代理。

注：请求作用域的bean也会面临相同的装配问题。因此，请求作用域的bean也应该以作用域代理的方式进行注入。

5）xml方式配置代理模式

```
<bean id="cart" class="com.myapp.ShoppingCart" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>
```

aop:scoped-proxy标签会告诉Spring为bean创建一个作用域代理。默认情况下，它会使用CGLib创建目标类的代理。但是我们也可以将proxy-target-class属性设置为false，进而要求它生成基于接口的代理。

注：proxy-target-class默认是true