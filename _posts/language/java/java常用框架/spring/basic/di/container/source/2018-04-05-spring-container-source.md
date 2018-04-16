---
title: spring容器的初始化
tags: [spring]
---

参考：https://www.cnblogs.com/wuzhe1991/p/8336746.html

https://blog.csdn.net/owen_william/article/details/50939976

参考：https://blog.csdn.net/xyh930929/article/details/79196069

1）入口类

以java类的配置方式查看spring的初始化过程

```
public static void main(String[] args) {
    new AnnotationConfigApplicationContext(CDPlayerConfig.class);
}
```

配置类

```
@Configuration
public class CDPlayerConfig {
  @Bean
  public CompactDisc compactDisc() {
    return new SgtPeppers();
  }
  @Bean
  public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);
  }
}
```

2）处理过程

首先注册配置类到，然后调用refresh方法

```
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
    this();
    //注册配置类的定义信息到应用上下文中
    register(annotatedClasses);
    //相当于重新加载容器的配置
    refresh();
}
```

3）调用refresh方法

该方法是一个需要重点分析的方法

```
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        //初始化前的准备工作，例如对系统属性或者环境进行准备及验证。
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        //创建并初始化BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        //对BeanFactory进行各种功能填充，即进一步的初始化
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            //在应用上下文的子类中扩展BeanFactory的后置处理能力
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            //激活各种BeanFactory处理器
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            logger.warn("Exception encountered during context initialization - cancelling refresh attempt", ex);

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }
    }
}
```