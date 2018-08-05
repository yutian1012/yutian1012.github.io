---
title: spring BeanPostProcessor类
tags: [spring]
---

参考：https://www.jianshu.com/p/1417eefd2ab1

BeanPostProcessor是Spring IOC容器给我们提供的一个扩展接口。

Spring容器通过BeanPostProcessor给了我们一个机会对Spring管理的每个bean进行再加工。如可以修改bean的属性，可以给bean生成一个动态代理实例等等。一些SpringAOP的底层处理也是通过实现BeanPostProcessor来执行代理包装逻辑的。

1）接口定义

BeanPostProcessor接口包括2个方法postProcessAfterInitialization和postProcessBeforeInitialization，这两个方法的第一个参数都是要处理的Bean对象，第二个参数都是Bean的name。返回值也都是要处理的Bean对象。

```
public interface BeanPostProcessor { 
    //bean初始化方法调用前被调用 
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException; 
    //bean初始化方法调用后被调用 
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException; 
}
```

注意：接口中两个方法不能返回null，如果返回null那么在后续初始化方法将报空指针异常或者通过getBean()方法获取不到bena实例对象。

2）接口方法调用顺序

当一个BeanPostProcessor的实现类注册到Spring IOC容器后，对于该Spring IOC容器所创建的每个bean实例在初始化方法（如afterPropertiesSet和任意已声明的init方法）调用前，将会调用BeanPostProcessor中的postProcessBeforeInitialization方法，而在bean实例初始化方法调用完成后，则会调用BeanPostProcessor中的postProcessAfterInitialization方法。

```
--> Spring IOC容器实例化Bean

--> 调用BeanPostProcessor的postProcessBeforeInitialization方法

--> 调用bean实例的初始化方法

--> 调用BeanPostProcessor的postProcessAfterInitialization方法
```