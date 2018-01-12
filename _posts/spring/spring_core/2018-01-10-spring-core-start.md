---
title: spring容器的启动过程
tags: [spring core]
---

1）容器的启动入口

通过ClassPathXmlApplicationContext对象的创建来查看spring容器的启动过程。

```
new ClassPathXmlApplicationContext("applicationContext.xml");
```

ClassPathXmlApplicationContext类的继承体系

![](/images/spring/core/ClassPathXmlApplicationContext-hierarchy.png)

2）AbstractApplicationContext类的refresh方法

该方法是Spring中比较核心的方法，Spring所有的初始化都在这个方法中完成。

```
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        //Prepare this context for refreshing
        //准备工作包括设置启动时间，是否激活标识位，
        //初始化属性源(property source)配置
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        //创建beanFactory
        //过程是根据xml为每个bean生成BeanDefinition并注册到生成的beanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        //准备创建好的beanFactory（
        //给beanFactory设置ClassLoader，
        //设置SpEL表达式解析器，
        //设置类型转化器【能将xml String类型转成相应对象】，
        //增加内置ApplicationContextAwareProcessor对象，
        //忽略各种Aware对象，
        //注册各种内置的对账对象【BeanFactory，ApplicationContext】等，
        //注册AOP相关的一些东西，
        //注册环境相关的一些bean
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            //注册request/session环境
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            //实例化并调用BeanFactory中扩展了
            //BeanFactoryPostProcessor的Bean的postProcessBeanFactory方法
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            //实例化和注册beanFactory中扩展了BeanPostProcessor的bean
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            //实例化，注册和设置国际化工具类MessageSource
            initMessageSource();

            // Initialize event multicaster for this context.
            //实例化，注册和设置消息广播类
            //如果没有自己定义使用默认的SimpleApplicationEventMulticaster实现
            //此广播使用同步的通知方式
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            //设置样式工具ThemeSource
            onRefresh();

            // Check for listener beans and register them.
            //添加用户定义的消息接收器
            //到上面设置的消息广播ApplicationEventMulticaster
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            //设置自定义的类型转化器ConversionService，
            //设置自定义AOP相关的类LoadTimeWeaverAware，
            //清除临时的ClassLoader，
            //冻结配置
            //实例化所有的类（懒加载的类除外）
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            //注册和设置跟bean生命周期相关的类
            //默认使用DefaultLifecycleProcessor
            //调用扩展了SmartLifecycle接口的start方法，
            //使用上面注册的广播类消息广播类ApplicationEventMulticaster
            //广播ContextRefreshedEvent事件 
            finishRefresh();
        }

        catch (BeansException ex) {
            //针对异常状况的处理，销毁对象，清空容器
            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```