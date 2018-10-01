---
title: spring的bean标签属性depends-on自动装配
tags: [spring]
---

参考：https://www.cnblogs.com/ViviChan/p/4981518.html

depends-on定义了bean的依赖关系，即标识了bean创建和销毁的先后顺序。

1）depends-on标签

depends-on是bean标签的属性之一，表示一个bean对其他bean的依赖关系。

2）与ref的区别

ref标签是一个bean对其他bean的引用，而depends-on属性只是表明依赖关系（不一定会引用）。

3）存在的意义

depends-on依赖关系决定了被依赖的bean必定会在依赖bean之前被实例化，反过来，容器关闭时，依赖bean会在被依赖的bean之前被销毁。

4）实例

manager和accoutDao会先于beanOne被实例化，会慢于beanOne被销毁，而beanOne不引用accountDao（或者说beanOne不会将accountDao注入到自己的属性中）

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```