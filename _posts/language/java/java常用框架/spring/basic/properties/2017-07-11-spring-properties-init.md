---
title: spring属性文件
tags: [spring]
---

参考：http://www.cnblogs.com/hafiz/p/5876243.html

### 通过PropertiesFactoryBean创建属性类

```
<bean id="prop"
    class="org.springframework.beans.factory.config.PropertiesFactoryBean">
   <!-- 这里是PropertiesFactoryBean类，它也有个locations属性，也是接收一个数组，跟上面一样 -->
   <property name="locations">
       <array>
          <value>classpath:jdbc.properties</value>
       </array>
   </property>
</bean>
```

注：返回对象为Properties。

### 使用util:properties标签

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-3.2.xsd
        http://www.springframework.org/schema/util 
　　　　 http://www.springframework.org/schema/util/spring-util.xsd"> 

    <util:properties id="propertiesReader" location="classpath:jdbc.properties"/>

</beans>
```

注：需要也纳入util的命名空间