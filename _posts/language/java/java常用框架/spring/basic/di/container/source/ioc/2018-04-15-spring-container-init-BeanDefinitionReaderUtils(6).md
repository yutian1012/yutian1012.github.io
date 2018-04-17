---
title: spring容器的BeanDefinitionReaderUtils注册BeanDefinition到容器
tags: [spring]
---

BeanDefinitionParserDelegate对象解析xml文档时，会调用BeanDefinitionReaderUtils工具类，将解析到的beanDefinition对象注册到BeanFactory容器中。

1）BeanDefinitionReaderUtils工具类的registerBeanDefinition方法

该方法完成beanDefinition对象注入到容器中

```
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    //注入对象到容器
    registry.registerBeanDefinition(
        beanName,definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String aliase : aliases) {
            registry.registerAlias(beanName, aliase);
        }
    }
}
```

2）BeanDefinitionRegistry的registerBeanDefinition方法

该方法实现将BeanDefinition对象注入到BeanFactory的beanDefinitionMap集合中，而且注入过程中使用了线程安全控制并发。

这里传入的BeanDefinitionRegistry对象实际上是DefaultListableBeanFactory对象。查看该类的registerBeanDefinition方法。

```
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    
    ...

    BeanDefinition oldBeanDefinition;

    synchronized (this.beanDefinitionMap) {
        oldBeanDefinition = this.beanDefinitionMap.get(beanName);
        ...
        //将beanDefinition信息设置到beanDefinitionMap成员变量中
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }

    if (oldBeanDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
}
```

3）问题：为什么要使用synchronized关键字锁住对象？

4）问题：哪个地方会发生并发操作？