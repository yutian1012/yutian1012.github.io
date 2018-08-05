---
title: spring生命周期实例
tags: [spring]
---

参考：https://www.cnblogs.com/zrtqsk/p/3735273.html

1）创建BeanInitialDemo类

该类实现Bean级生命周期接口BeanNameAware，BeanFactoryAware，InitializingBean，DisposableBean和ApplicationContextAware

```
package com.sjq.bean.container;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class BeanInitialDemo implements BeanFactoryAware, BeanNameAware,
         InitializingBean, DisposableBean, ApplicationContextAware{
    // 对象属性
    private String name;
    
    // 对象引用属性
    private Student student;
    
    // 接收spring注入到bean实例的beanFactory
    private BeanFactory beanFactory;
    
    // 接收spring实例化该bean的ID
    private String beanName;
    
    // 接收spring注入到bean实例的ApplicationContext实例（容器应用上下文）
    private ApplicationContext applicationContext;
    
    public BeanInitialDemo() {
        System.out.println("第一步：【构造器】调用构造器实例化");
    }
    
    public String getName() {
        return name;
    }
    public void setName(String name) {
        System.out.println("第二步：【注入属性】注入属性name");
        this.name = name;
    }
    
    public Student getStudent() {
        return student;
    }

    @Autowired
    public void setStudent(Student student) {
        System.out.println("第二步：【装配属性】spring装配引用对象Student");
        this.student = student;
    }
    
    // 用于获取容器的BeanFactory实例
    public BeanFactory getBeanFactory() {
        return beanFactory;
    }
    // 用于获取该bean在容器的ID
    public String getBeanName() {
        return beanName;
    }
    // 用于获取spring容器引用上下文对象
    public ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    // BeanNameAware接口
    @Override
    public void setBeanName(String beanName) {
        System.out.println("第三步：【BeanNameAware接口】调用BeanNameAware.setBeanName()");
        this.beanName=beanName;
    }
    
    // BeanFactoryAware接口
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("第四步：【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
        this.beanFactory=beanFactory;
    }
    
    // ApplicationContextAware接口
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("第五步：【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()");
        this.applicationContext=applicationContext;
    }
    
    // InitializingBean接口
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("第七步：【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
    }

    // 对象自定义的init方法
    public void initMethod() {
        System.out.println("第八步：【init-method】调用<bean>的init-method属性指定的初始化方法");
    }
    
    // DisposableBean接口
    @Override
    public void destroy() throws Exception {
        System.out.println("第十一步：【DiposibleBean接口】调用DiposibleBean.destory()");
    }
    
    // 对象自定义的destroy方法
    public void destroyMethod() {
        System.out.println("第十二步：【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
    }
}
```

注：这里同时创建了一个student类，该类用于完成属性注入工作

```
package com.sjq.bean.container;

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

2）创建BeanPostProcessorDemo

该类实现BeanPostProcessor接口，同时观察，在容器创建Student和BeanInitialDemo对象时，都会调用实现了BeanPostProcessor接口的方法。

```
package com.sjq.bean.container;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class BeanPostProcessorDemo implements BeanPostProcessor{
    
    public BeanPostProcessorDemo() {
        super();
        System.out.println("【BeanPostProcessor】实现类构造器！！");
    }
    /**
     * 第一个参数都是要处理的Bean对象，第二个参数都是Bean的name。
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第六步：【BeanPostProcessor接口】方法postProcessBeforeInitialization对属性进行更改！");
        System.out.println("【BeanPostProcessor接口】方法postProcessBeforeInitialization，beanName="+beanName);
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第九步：【BeanPostProcessor接口】方法postProcessAfterInitialization对属性进行更改！");
        System.out.println("【BeanPostProcessor接口】方法postProcessAfterInitialization，beanName="+beanName);
        return bean;
    }
}
```

注：BeanPostProcessor接口两个方法的参数分别为实例化的bean对象和bean的id属性值。

3）创建注解配置类

使用@Configuration注解来表示该类是一个配置类，同时将容器管理的bean都在配置类中初始化

```
package com.sjq.bean.container;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeaninitialConfig {
    
    @Bean(initMethod="initMethod",destroyMethod="destroyMethod")
    public BeanInitialDemo beanInitialDemo() {
        return new BeanInitialDemo();
    }
    
    @Bean
    public BeanPostProcessorDemo beanPostProcessorDemo() {
        return new BeanPostProcessorDemo();
    }
    
    @Bean
    public Student student() {
        return new Student();
    }
}
```

4）初始化容器

运行该类实现容器的初始化，从而观察到容器中bean初始化的过程。由于BeanInitialDemo需要依赖student类，因此会先创建student类实例，然后再创建BeanInitialDemo实例，并将student实例对象注入到BeanInitialDemo实例中

```
package com.sjq.bean.container;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AnnotationApplicationRun {
    public static void main(String[] args) {
        System.out.println("************************************************************************");
        
        System.out.println("开始初始化容器并创建对象.........");
        
        ApplicationContext context=new AnnotationConfigApplicationContext(BeaninitialConfig.class);
        
        System.out.println("************************************************************************");
        
        System.out.println("第十步：对象创建就绪，可以使用容器内的对象了");
        
        BeanInitialDemo demo=context.getBean(BeanInitialDemo.class);
        
        System.out.println(demo.getBeanName());
        
        System.out.println("现在开始关闭容器！");
        
        System.out.println("************************************************************************");
        
        ((AnnotationConfigApplicationContext)context).registerShutdownHook();
    }
}
```

5）运行结果查看

```
************************************************************************
开始初始化容器并创建对象.........
【BeanPostProcessor】实现类构造器！！

# 以上为BeanPostProcessorDemo的实例构造过程

第一步：【构造器】调用构造器实例化

# 构造beanInitialDemo实例，发现属性需要依赖student，因此转而构造student实例

【student类】构造函数
第六步：【BeanPostProcessor接口】方法postProcessBeforeInitialization对属性进行更改！
【BeanPostProcessor接口】方法postProcessBeforeInitialization，beanName=student
第九步：【BeanPostProcessor接口】方法postProcessAfterInitialization对属性进行更改！
【BeanPostProcessor接口】方法postProcessAfterInitialization，beanName=student

# 上面为student类的构造过程，会调用BeanPostProcessor接口的实现方法

第二步：【装配属性】spring装配引用对象Student
第三步：【BeanNameAware接口】调用BeanNameAware.setBeanName()
第四步：【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
第五步：【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()
第六步：【BeanPostProcessor接口】方法postProcessBeforeInitialization对属性进行更改！
【BeanPostProcessor接口】方法postProcessBeforeInitialization，beanName=beanInitialDemo
第七步：【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
第八步：【init-method】调用<bean>的init-method属性指定的初始化方法
第九步：【BeanPostProcessor接口】方法postProcessAfterInitialization对属性进行更改！
【BeanPostProcessor接口】方法postProcessAfterInitialization，beanName=beanInitialDemo

************************************************************************
第十步：对象创建就绪，可以使用容器内的对象了
beanInitialDemo
现在开始关闭容器！

************************************************************************
第十一步：【DiposibleBean接口】调用DiposibleBean.destory()
第十二步：【destroy-method】调用<bean>的destroy-method属性指定的初始化方法
```

6）xml方式配置

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
            http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanInitialDemo" class="com.sjq.bean.container.BeanInitialDemo" init-method="initMethod"
        destroy-method="destroyMethod" scope="singleton" p:name="张三"/>

    <bean id="beanPostProcessorDemo" class="com.sjq.bean.container.BeanPostProcessorDemo"/>
    
    <bean id="student" class="com.sjq.bean.container.Student"/>
</beans>
```

7）初始化容器

```
package com.sjq.bean.container;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ClasspathXmlApplicationRun {
    public static void main(String[] args) {
        System.out.println("************************************************************************");
        
        System.out.println("开始初始化容器并创建对象.........");
        
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext_Container.xml");
        
        System.out.println("************************************************************************");
        
        System.out.println("第十步：对象创建就绪，可以使用容器内的对象了");
        
        BeanInitialDemo demo=context.getBean(BeanInitialDemo.class);
        
        System.out.println(demo.getBeanName());
        
        System.out.println("现在开始关闭容器！");
        
        System.out.println("************************************************************************");
        
        ((ClassPathXmlApplicationContext)context).registerShutdownHook();
    }
}
```