---
title: spring容器的EntityResolver解析器中xml文档中的SystemId等信息
tags: [spring]
---

1）查看spring加载时使用的systemId信息

配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="SgtPeppers" class="com.sjq.bean.container.beanfactory.bean.SgtPeppers"></bean>
</beans>
```

日志输出

```
public id [null] and system id [http://www.springframework.org/schema/beans/spring-beans.xsd]
```

2）如何配置日志输出级别

如果spring中不引入歧途日志框架，则默认使用的是commons-logging。commons-logging是apache最早提供的日志的门面接口。

新建commons-logging.properties文件

```
org.apache.commons.logging.Log=org.apache.commons.logging.impl.SimpleLog
```

新建simplelog.properties文件

```
org.apache.commons.logging.simplelog.defaultlog=trace
```

设置为trace日志级别，才能看到解析xml文档时SystemId等信息。

3）分析xml中的systemId

参考：https://www.cnblogs.com/mjorcen/p/3642855.html

xml文档-schema校验，publicId为null

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
...
</beans>

publicId : null
systemId : http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
```

xml文档-dtd校验

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC  "-//SPRING//DTD BEAN//EN"  "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
...
</beans>

publicId : -//SPRING//DTD BEAN//EN
systemId : http://www.springframework.org/dtd/spring-beans.dtd
```

4）查看PluggableSchemaEntityResolver如何把URL远程转换成本的文件

查看文档spring-xml-PluggableSchemaResolver.md