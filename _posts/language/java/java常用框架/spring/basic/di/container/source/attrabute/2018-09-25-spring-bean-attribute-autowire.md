---
title: spring的bean标签属性autowire自动装配
tags: [spring]
---

参考：https://www.cnblogs.com/ViviChan/p/4981539.html

bean对象的属性依赖注入传统方式可通过ref进行依赖设置，这样比较麻烦，而且一旦bean改了名字，所有引用处都需要修改。autowire属性实现了bean的自动依赖设置。配置有autowire属性的bean，Spring在实例化这个bean的时候会在容器中查找匹配的bean对autowire bean进行属性注入。

注：虽然autowrie给我们带来配置的便利性，但是也有缺点，比如会导致bean的关系没那么显而易见，所以用autowire还是ref还是需要根据项目来决定。

1）属性的注入方式

property+ref方式

```
<property name="xxx" ref="xxx"></property>

<property name="xxx"><ref bean="xxx"/></property>
```

2）使用autowire方式注入

```
<bean id="auto" class="example.autoBean" autowire="byType"/>
```

3）autowire的选项

autowire属性有5种模式

```
no：(默认)不采用autowire机制。
注：这种情况，当我们需要使用依赖注入，只能用<ref/>标签。

default：采用父级标签（即beans的default-autowire属性）的配置。

byName：通过属性的名称自动装配（注入）。Spring会在容器中查找名称与bean属性名称一致的bean，并自动注入到bean属性中。当然bean的属性需要有setter方法。
例如：bean A有个属性master，master的setter方法就是setMaster，A设置了autowire="byName"，那么Spring就会在容器中查找名为master的bean通过setMaster方法注入到A中。

byType：通过类型自动装配（注入）。Spring会在容器中查找类（Class）与bean属性类一致的bean，并自动注入到bean属性中，如果容器中包含多个这个类型的bean，Spring将抛出异常。如果没有找到这个类型的bean，那么注入动作将不会执行。

constructor：类似于byType，但是是通过构造函数的参数类型来匹配。
假设bean A有构造函数A(B b, C c)，那么Spring会在容器中查找类型为B和C的bean通过构造函数A(B b,C c)注入到A中。与byType一样，如果存在多个bean类型为B或者C，则会抛出异常。但是与byType不同的是，如果在容器中找不到匹配的类的bean，将抛出异常，因为Spring无法调用构造函数实例化这个bean。
```

4）全局配置

在beans标签中指定default-autowire属性，那么对于子标签bean如果没有单独的设置autowire属性，那么将采用父标签beans的default-autowire属性的模式，如果单独设置了autowire属性，则采用自己的模式。

5）autowire-candidate属性

该属性可以设置是否作为候选bean被spring容器设置依赖时所使用。

配置有autowire属性的bean，Spring在实例化这个bean的时候会在容器中查找匹配的bean对autowire bean进行属性注入，这些被查找的bean我们称为候选bean。作为候选bean可设置自己不成为候选bean（我凭什么就要被你用，老子不给你用）。所以候选bean可以给自己增加了autowire-candidate="false"属性（默认是true），那么容器就不会把这个bean当做候选bean了，即这个bean不会被当做自动装配对象。

同样，beans标签可以定义default-autowire-candidate="false"属性让它包含的所有bean都不做为候选bean。