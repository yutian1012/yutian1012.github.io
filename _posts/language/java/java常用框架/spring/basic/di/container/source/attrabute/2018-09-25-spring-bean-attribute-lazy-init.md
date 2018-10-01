---
title: spring的bean标签属性lazy-init
tags: [spring]
---

参考：https://www.cnblogs.com/ViviChan/p/4981563.html

1）懒加载的理解

默认情况下，容器初始化的时候便会把bean实例化，如果希望某个对象只有在使用时才创建，就可以使用lazy-init属性配置成懒加载。这样不但可以节省系统资源，而且能够提升容器的启动速度。

2）懒加载的配置

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
```

3）懒加载被引用

容器初始化时非懒加载的bean都会被实例化，而设置了懒加载的bean不会被实例化。但如果一个配置了lazy-init="true"属性的bean被另外一个bean依赖，那Spring还是会在容器初始化的时候实例化这个bean。

4）全局设置懒加载

如果希望某个beans的配置文件中的所有bean都是懒加载的，那可以给beans标签添加default-lazy-init="true"属性。