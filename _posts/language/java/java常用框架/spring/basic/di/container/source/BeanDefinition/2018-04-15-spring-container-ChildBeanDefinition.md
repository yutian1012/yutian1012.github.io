---
title: spring中ChildBeanDefinition对象的理解
tags: [spring]
---

ChildBeanDefinition是一种beanDefinition，它可以继承它父类的设置，即ChildBeanDefinition对RootBeanDefinition有一定的依赖关系。

1）ChildBeanDefinition定义

```
public class ChildBeanDefinition extends AbstractBeanDefinition {
    private String parentName;
    ...
}
```

注：定义了parentName属性，用来设置parent的name信息。

2）ChildBeanDefinition与parent

ChildBeanDefinition从父类继承构造参数值，属性值并可以重写父类的方法，同时也可以增加新的属性或者方法。(类同于java类的继承关系)。若指定初始化方法，销毁方法或者静态工厂方法，　　ChildBeanDefinition将重写相应父类的设置。

而depends-on，autowiremode，dependency-check，sigleton，lazy-init的设置信息一般由子类自行设定。

3）使用GenericBeanDefinition代替ChildBeanDefinition

从spring2.5开始，提供了一个更好的注册beanDefinition的类，即GenericBeanDefinition，它支持动态定义父依赖，方法是GenericBeanDefinition.setParentName方法，GenericBeanDefinition可以有效的替代ChildBeanDefinition的绝大分部使用场合。

4）实例

```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

子BeanDefinition继承父BeanDefinition的scorp、constructor 、property 、method overrides，并添加新配置。子BeanDefinition会覆盖父子BeanDefinition的配置有：initialization method、destroy method、static factory method settings。

子BeanDefinition有些配置始终不会继承父BeanDefinition：depends on、autowire mode、dependency check、singleton、lazy init。

5）父bean定义为抽象

```
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>
```

通过abstract属性将一个bean定义为抽象的。如果父bean没有指定class，则需要将其定义为抽象的。如果一个bean被指定为抽象，那么也就可以认为该bean就是一个模板，子bean通用配置的抽象。