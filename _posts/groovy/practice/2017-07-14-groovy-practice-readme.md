---
title: java调用脚本的封装介绍
tags: [groovy]
---

该实例是参考bpmx对Groovy操作的封装设计来进行测试的，通过测试可以更好的理解脚本引擎的执行。

1）组成

该实例主要包括GroovyBindingWrapper.java，GroovyScriptEngineWrapper.java，GroovyScriptEngineWrapperTest.java，ScriptImpl.java类。

![](/images/groovy/practice/components.png)

2）使用

用户可以通过手动编写设置脚本从而完成后端执行脚本，动态获取值。主要应用在流程节点计算执行人等功能上。

GroovyBindingWrapper类中设置的变量一般是spring初始化结束后的bean，spring将service等bean设置到该类中（一般是比较固定的），供脚本引擎直接使用这些bean。注意在向该类中设置对象需要防止内存泄漏（一般设置单例，同一个类对象的实例对应map的key必须是一样的）。