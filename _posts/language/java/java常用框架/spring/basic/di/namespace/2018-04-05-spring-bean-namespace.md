---
title: spring中xml的命名空间
tags: [spring]
---

但在使用XML时，需要在配置文件的顶部声明多个XML模式（XSD）文件，这些文件定义了配置Spring的XML元素。可借助Spring Tool Suite创建XML配置文件，简化开发xml文件。

1）引入context命名空间

如当使用component-scan元素标签时，需要引入context命名空间。

```
<?xmlversion="1.0"encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"

    xsi:schemaLocation=
    "http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

2）引入bean定义的命名空间

```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--configurationdetailsgohere-->
</beans>
```

3）c命名空间

c命名空间能够简化构造函数在xml中配置信息，spring为constructor-arg提供了c命名空间。

```
<?xml version="1.0"encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation=
        "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 使用参数名指定引用对象 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" 
        c:cd-ref="compactDisc" />
    <!-- 使用参数索引位置引用对象 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" 
        c:_0-ref="compactDisc" />
</beans>
```

4）p命名空间

Spring提供了更加简洁的p-命名空间，作为property元素的替代方案。为了启用p-命名空间，必须要在XML文件中与其他的命名空间一起对其进行声明。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation=
        "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 使用参数名指定引用对象 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer"
        p:compactDisc-ref="compactDisc"/>
</beans>
```

5）util命名空间

该命名空间提供的常用工具

```
util:constant引用某个类型的public static域，并将其暴露为bean

util:list创建一个java.util.List类型的bean，其中包含值或引用

util:map创建一个java.util.Map类型的bean，其中包含值或引用

util:properties创建一个java.util.Properties类型的bean

util:property-path引用一个bean的属性（或内嵌属性），并将其暴露为bean

util:set创建一个java.util.Set类型的bean，其中包含值或引用
```

util命名空间所提供的功能之一就是util:list元素，它会创建一个列表的bean。

```
<util:listid="trackList">
    <value>Sgt.Pepper'sLonelyHeartsClubBand</value>
    <value>WithaLittleHelpfromMyFriends</value>
</util:list>
```

6）aop命名空间

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
    ...
</beans>
```