---
title: spring BeanFactoryPostProcessor类
tags: [spring]
---

BeanFactoryPostProcessor可以修改bean的配置信息，能够在容器启动时，对象实例化（调用构造函数前）获取容器中的BeanDefinition对象（bean的配置信息），并能对其进行修改。

注：是spring容器为我们提供的第一个扩展点。

实例：实现对Student类进行属性赋值

1）Student类

student类中包含一个name属性，通过BeanFactoryPostProcessor实现值的注入操作。

```
package com.sjq.bean.beanFactoryPostProcessor;

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

2）BeanFactoryPostProcessorDemo类

实现BeanFactoryPostProcessor，重写postProcessBeanFactory方法，向Student定义信息中注入属性值。

```
package com.sjq.bean.beanFactoryPostProcessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

@Component
public class BeanFactoryPostProcessorDemo implements BeanFactoryPostProcessor{

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        
        System.out.println("【BeanFactoryPostProcessor】调用postProcessBeanFactory方法");
        
        BeanDefinition bd=beanFactory.getBeanDefinition("student");//获取student对象的配置信息
        MutablePropertyValues pv =  bd.getPropertyValues();
        //相当于在xml中通过property设置属性
        /*<bean id="xxx" class="xxxx" >
            <property name="name" value="zhangsan"></property>
        </bean>*/
        pv.addPropertyValue("name", "zhangsan");
    }

}
```

3）新建配置类BeanFactoryPostProcessorConfig

```
package com.sjq.bean.beanFactoryPostProcessor;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackageClasses=BeanFactoryPostProcessorDemo.class)
public class BeanFactoryPostProcessorConfig {
    
}
```

4）启动容器查看输出

```
package com.sjq.bean.beanFactoryPostProcessor;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AnnotationApplicationRun {
    public static void main(String[] args) {
        ApplicationContext context=new AnnotationConfigApplicationContext(BeanFactoryPostProcessorConfig.class);
        Student student=context.getBean(Student.class);
        System.out.println(student.getName());
    }
}
```