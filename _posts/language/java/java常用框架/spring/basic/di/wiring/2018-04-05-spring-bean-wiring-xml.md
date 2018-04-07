---
title: spring的装配xml配置方式
tags: [spring]
---

1）xml配置文件

但在使用XML时，需要在配置文件的顶部声明多个XML模式（XSD）文件，这些文件定义了配置Spring的XML元素。

```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context">

    <!--configurationdetailsgohere-->
</beans>
```

2）定义bean对象

```
<bean class="soundsystem.SgtPeppers"/>
```

这里声明了一个很简单的bean，创建这个bean的类通过class属性来指定的，并且要使用全限定的类名。因为没有明确给定ID，所以这个bean将会根据全限定类名来进行命名。在本例中，bean的ID将会是"soundsystem.SgtPeppers#0"。

注：可以同id属性指定ID值。

3）构造函数中注入对象

a.借助构造器注入bean对象

具体到构造器注入，有两种基本的配置方案可供选择：

使用constructor-arg元素方式

```
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <constructor-arg ref="compactDisc"/>
</bean>
```

使用Spring3.0所引入的c-命名空间

![](/images/spring/core/c-namespace.png)

```
<?xml version="1.0"encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 使用参数名指定引用对象 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" 
        c:cd-ref="compactDisc" />
    <!-- 使用参数索引位置引用对象 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" 
        c:_0-ref="compactDisc" />
</beans>
```

注：参数名方式需要指定构造函数的参数名，如代码中的c:cd-ref中的cd就是参数名。

注2：索引位置方式，在XML中不允许数字作为属性的第一个字符，因此必须要添加一个下画线作为前缀。

4）构造函数中注入字面量

构造函数中接收字符串常量作为初始化值

```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="Sgt.Pepper'sLonelyHeartsClubBand"/>
    <constructor-arg value="TheBeatles"/>
</bean>

<bean id="compactDisc" class="soundsystem.BlankDisc"
    c:_title="Sgt.Pepper'sLonelyHeartsClubBand"
    c:_artist="TheBeatles"/>

<bean id="compactDisc" class="soundsystem.BlankDisc"
    c:_0="Sgt.Pepper'sLonelyHeartsClubBand"
    c:_1="TheBeatles"/>
```

5）构造函数中集合注入

```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg><null/></constructor-arg>
</bean>

<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg>
        <list>
            <!-- 也可以使用ref引用对象 -->
            <value>Sgt. Pepper's Lonely Hearts Club Band</value>
        </list>
    </constructor-arg>
</bean>
```

注：当构造函数的参数对象是list使用list标签，如果是set需要使用set标签

6）对象属性注入

java中对于强依赖一般采用构造函数注入，而对于可选性依赖一般使用属性的方式进行注入。

```
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <property name="compactDisc" ref="compactDisc"/>
</bean>
```

使用p命名空间设置属性

```
<bean id="cdPlayer" class="soundsystem.CDPlayer"
    p:compactDisc-ref="compactDisc"/>
```

![](/images/spring/core/p-namespace.png)

7）属性方式注入字面量和集合

```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <!-- 注入字面量 -->
    <property name="title" value="Sgt.Pepper'sLonelyHeartsClubBand"/>
    <property name="artist" value="TheBeatles"/>
    <!-- 注入集合属性 -->
    <property name="tracks">
        <list>
            <value>Sgt.Pepper'sLonelyHeartsClubBand</value>
            <value>WithaLittleHelpfromMyFriends</value>
        </list>
    </property>
</bean>
```

8）使用util命名空间简化bean定义

```
<util:listid="trackList">
    <value>Sgt.Pepper'sLonelyHeartsClubBand</value>
    <value>WithaLittleHelpfromMyFriends</value>
</util:list>

<bean id="compactDisc" class="soundsystem.BlankDisc" 
    p:title="Sgt.Pepper'sLonelyHeartsClubBand"
    p:artist="TheBeatles"
    p:tracks-ref="trackList"/>
```