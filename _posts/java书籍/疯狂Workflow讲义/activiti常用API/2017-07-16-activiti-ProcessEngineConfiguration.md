---
title: ProcessEngineConfiguration流程引擎配置
tags: [activiti]
---

ProcessEngineConfiguration对象代表一个Activiti流程引擎的全部配置，该类提供了一系列创建ProcessEngineConfiguration实例的镜头方法。这些方法用于读取和解析相应的配置文件，并返回ProcessEngineConfiguration的实例。

1）提供了很多实例化对象的静态方法

主要的静态方法可参考想要的api文档，使用非常简单。常用的5个静态方法创建ProcessEngineConfiguration对象。

```
createProcessEngineConfigurationFromResourceDefault
createProcessEngineConfigurationFromResource
createProcessEngineConfigurationFromInputStream
createStandaloneInMemProcessEngineConfiguration
createStandaloneProcessEngineConfiguration
```

2）数据源配置

该类包括流程的数据源信息配置，Acitiviti支持多种数据库。数据源的配置包括jdbc，dbcp（数据库连接池，datasource），c3p0。

设置databaseSchemaUpdate属性可以设置流程引擎启动和关闭时数据库执行的策略（数据库表创建和删除）。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 流程引擎配置的bean -->
    <bean id="processEngineConfiguration"
        class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/act" />
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="root" />
        <property name="databaseSchemaUpdate" value="drop-create" />
    </bean>
</beans>
```

注：可以看到activiti定义bean的方式与spring容器定义bean相同，通过源代码可以看到activiti就是利用了spring中DefaultListableBeanFactory来初始化bean的定义的。

3）history配置

在流程执行的过程中，会产生一些与流程相关的数据。如流程实例、流程任务和流程参数等数据。随着流程的进行与结束，这些数据将会被从流程数据表中删除，为了能保存这些数据，activiti提供了历史数据表，可以让这些数据保存到历史数据表中。

设置级别（保存粒度）：none（0），activity（1），audit（2），full（3），后面的数值会保存到ACT_GE_PROPERTY表的historyLevel属性中。

4）邮件服务器配置

activiti支持邮件服务，当流程执行到某一个节点时，activiti会根据流程文件配置（EmailTask），发送邮件到相应的邮箱。

5）ProcessEngineConfiguration及子类

![](/images/book/workflow/activiti/api/processEngineConfig_subimpl.png)

可以实现自定义的ProcessEngineConfiguration类，添加自定义属性，并提供相应的set方法，这样就可以在xml中配置property节点中添加该属性，并设置属性值。

6）命令拦截器配置

主要是通过基础ProcessEngineConfigImpl类（抽象类），并实现方法来实现添加拦截器。
