---
title: spring中Bean的生命周期
tags: [spring]
---

参考：https://blog.csdn.net/cqh_jslr/article/details/53134719

1）理解Bean的生命周期

正确理解spring中Bean的生命周期，能够更好的利用spring提供的扩展点来自定义Bean的创建过程。

3）容器中Bean的初始化过程

![](/images/spring/core/spring-initialization.png)

第一步：spring对bean进行实例化

```
// 如果一个类提供了默认构造函数（无参构造），则实例化时会调用该构造函数
public class Student{
    public Student(){
        System.out.println("【student类】构造函数");
    }
}
```

注：容器启动时，创建容器中的bean实例时，会首先调用bean的无参构造，创建出实例对象

第二步：设置属性

```
// 设置对象属性值，如使用xml配置时，向属性传递初始化值
<bean id="beanInitialDemo" class="com.sjq.bean.BeanInitialDemo" p:name="张三"/>

//或者装配引用对象
private Student student;

// 使用autowired注解完成属性的依赖注入操作
@Autowired
public void setStudent(Student student) {
    System.out.println("第二步：【装配属性】spring装配引用对象Student");
    this.student = student;
}
```

spring将值和bean的引用注入到bean对应的属性中。

第三步：BeanNameAware接口

如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递一个setBeanName方法。

```
public interface BeanNameAware extends Aware {
    void setBeanName(String name);
}

// Bean实现上述接口，并提供一个属性beanName，用来获取bean的Id值
private String beanName;

// spring容器会调用setBeanName方法，并将bean的ID作为参数传入方法
@Override
public void setBeanName(String name) {
    System.out.println("【BeanNameAware接口】调用BeanNameAware.setBeanName()");
    this.beanName = name;
}
```

注：方便用户通过获取BeanName从而获取spring容器的bean的Id定义值

第四步：BeanFactoryAware接口

如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory方法，将BeanFactory容器实例传入。

```
public interface BeanFactoryAware extends Aware {
    void setBeanFactory(BeanFactory beanFactory) throws BeansException;
}

// Bean实现上述接口，并提供一个属性beanFactory，用来获取bean的工厂对象
private BeanFactory beanFactory;

// spring容器会调用setBeanFactory方法，并将BeanFactory容器实例作为参数传入方法
@Override
public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
    System.out.println(
        "【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
    this.beanFactory = beanFactory;
}
```

注：通过工厂对象可进一步获取容器相关信息

第五步：ApplicationContextAware接口

ApplicationContextAware接口与BeanFactoryAware接口是类似的，都可以获取容器相关的信息，进而可以从容器中获取想要的对象和组件。

如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext方法，将bean所在的应用上下文的引用传入进来。

```
public interface ApplicationContextAware extends Aware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}

// 接收spring注入到bean实例的ApplicationContext实例（容器应用上下文）
private ApplicationContext applicationContext;

// ApplicationContextAware接口
@Override
public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    System.out.println("第五步：【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()");
    this.applicationContext=applicationContext;
}
```

注：通过该接口可以获取容器应用的上下文，对于某些工具类来讲非常方便就能获取容器信息。

第六步：BeanPostProcessor接口

如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessBeforeInitialization方法

```
//实现了BeanPostProcessor接口的类，在该类实例化是，方法postProcessBeforeInitialization不会得到调用
public class BeanPostProcessorDemo implements BeanPostProcessor{
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        ....
    }
    ....
}

// 我们再定义一个普通的bean对象，但是，该bean对象实例化时，却会调用到BeanPostProcessor接口的postProcessBeforeInitialization方法
public class Student{
    ...
}
```

注：该接口中方法的调用是为了扩展系统中所有bean实例化时的对象扩展，而本身实现了BeanPostProcessor接口的类却不会调用相关的接口方法。因此，该接口实现的是一个全局实例化的统一配置。

第七步：InitializingBean接口

如果bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法。类似地，如果bean使用init-method声明了初始化方法，该方法也会被调用。

```
// 接口定义
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}

@Override
public void afterPropertiesSet() throws Exception {
    System.out.println("第七步：【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
}
```

第八步：init-method初始化方法

紧接着，如果配置了init-method，则会调用init-method指定的方法

```
// 
// 方式一，xml配置
<bean id="beanInitialDemo" class="com.sjq.bean.BeanInitialDemo" 
    init-method="initMethod"/>
// 方式二，注解配置
@Bean(initMethod="initMethod")

// 对象自定义的init方法
public void initMethod() {
    System.out.println("第八步：【init-method】调用<bean>的init-method属性指定的初始化方法");
}
```

第九步：BeanPostProcesser接口

如果bean实现了BeanPostProcesser接口，Spring将调用它们的postProcessAfterInitialization方法。

注：参照第六步

第十步：bean创建完成

此时bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到应用上下文被销毁。

第十一步：DisposableBean接口

销毁bean对象，如果bean实现了DisposableBean接口，Spring将调用它的destroy方法

```
// 接口定义
public interface DisposableBean {
    void destroy() throws Exception;
}

// DisposableBean接口
@Override
public void destroy() throws Exception {
    System.out.println("第十一步：【DiposibleBean接口】调用DiposibleBean.destory()");
}
```

第十二步：destroy-method方法

同样如果bean使用destroy-method声明了销毁方法，该方法也会被调用。

```
// 对象自定义的destroy方法
public void destroyMethod() {
    System.out.println("第十二步：【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
}
```

注：配置方式参考init-method