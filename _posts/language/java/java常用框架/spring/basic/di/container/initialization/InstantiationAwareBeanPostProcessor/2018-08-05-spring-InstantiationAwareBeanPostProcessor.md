---
title: spring InstantiationAwareBeanPostProcessor类
tags: [spring]
---

InstantiationAwareBeanPostProcessor接口本质是BeanPostProcessor的子接口。

1）接口方法的执行时机

InstantiationAwareBeanPostProcessor接口相应方法的执行时机是在类实例化前后，即调用对象的构造函数的前后执行的。before方法是在对象构造函数执行之前，propertyValues是在构造函数之后设置属性之前执行，而after方法则是在BeanPostProcessor接口的after方法后才执行的（跨度比较大）。

注：执行时机是与BeanPostProcessor接口完全不同的，BeanPostProcessor接口的方法执行时机是在对象的初始化过程中，before方法在InitializingBean接口和init-method之前，after是在这之后。

2）查看方法的执行时机

一般我们继承Spring为其提供的适配器类InstantiationAwareBeanPostProcessor Adapter来使用它。

3）创建Student类（待实例化的对象）

```
package com.sjq.bean.instantiationAwareBeanPostProcessor;

import org.springframework.stereotype.Component;

@Component
public class Student {
    private String name;
    
    public Student() {
        System.out.println("【student类】构造函数");
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

4）InstantiationAwareBeanPostProcessorAdapter实现类

创建InstantiationAwareBeanPostProcessorDemo类继承InstantiationAwareBeanPostProcessorAdapter适配器。

```
package com.sjq.bean.instantiationAwareBeanPostProcessor;

import java.beans.PropertyDescriptor;

import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessorAdapter;
import org.springframework.stereotype.Component;

@Component
public class InstantiationAwareBeanPostProcessorDemo extends InstantiationAwareBeanPostProcessorAdapter  {
    
    public InstantiationAwareBeanPostProcessorDemo() {
        super();
        System.out.println("【InstantiationAwareBeanPostProcessorDemo类】构造函数");
    }
    
    /**
     * 实例化对象之前调用，即调用对象的构造函数之前执行
     */
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
         System.out.println("【InstantiationAwareBeanPostProcessorDemo】调用postProcessBeforeInstantiation方法");
         //System.out.println("【InstantiationAwareBeanPostProcessorDemo】"+beanName);
         return null;
    }
    
    /**
     * 实例化对象后，且设置对象属性前执行
     */
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean,
            String beanName) throws BeansException {
        System.out.println("【InstantiationAwareBeanPostProcessorDemo】调用postProcessPropertyValues方法");
        //System.out.println("【InstantiationAwareBeanPostProcessorDemo】"+beanName);
        return pvs;
    }
    /**
     * BeanPostProcessor的after方法执行之后才执行
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("【InstantiationAwareBeanPostProcessorDemo】调用postProcessAfterInitialization方法");
        //System.out.println("【InstantiationAwareBeanPostProcessorDemo】"+beanName);
        return bean;
    }
}
```

5）创建配置文件

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
            http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
            
    <bean id="student" class="com.sjq.bean.instantiationAwareBeanPostProcessor.Student"></bean>
    
    <bean id="demo" class="com.sjq.bean.instantiationAwareBeanPostProcessor.InstantiationAwareBeanPostProcessorDemo"></bean>
</beans>
```

6）创建容器启动类

```
package com.sjq.bean.instantiationAwareBeanPostProcessor;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ClasspathXmlApplicationRun {
    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("applicationContext_InstantiationAware.xml");
    }
}
```

执行测试结果

```
【InstantiationAwareBeanPostProcessorDemo类】构造函数
【InstantiationAwareBeanPostProcessorDemo】调用postProcessBeforeInstantiation方法
【student类】构造函数
【InstantiationAwareBeanPostProcessorDemo】调用postProcessPropertyValues方法
【InstantiationAwareBeanPostProcessorDemo】调用postProcessAfterInitialization方法
```