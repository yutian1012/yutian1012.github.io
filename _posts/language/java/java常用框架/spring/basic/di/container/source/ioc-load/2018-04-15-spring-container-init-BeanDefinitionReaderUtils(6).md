---
title: spring容器的BeanDefinitionReaderUtils注册BeanDefinition到容器
tags: [spring]
---

DefaultBeanDefinitionDocumentReader类的processBeanDefinition方法，会调用BeanDefinitionReaderUtils工具类，将解析到的beanDefinition对象（BeanDefinitionParserDelegate对象解析的bean标签获取到的对象）注册到BeanFactory容器中。

1）BeanDefinitionReaderUtils工具类的registerBeanDefinition方法

该方法完成beanDefinition对象注入到容器中

```
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    //注入对象到容器，查看DefaultListableBeanFactory类对应方法
    //保存信息到Map<String, BeanDefinition> beanDefinitionMap（ConcurrentHashMap类型）中
    registry.registerBeanDefinition(
        beanName,definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    //将相应的别名注册到容器中，查看SimpleAliasRegistry类，DefaultListableBeanFactory类间接继承到该类
    //保存信息到Map<String, String> aliasMap（ConcurrentHashMap类型）中
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String aliase : aliases) {
            registry.registerAlias(beanName, aliase);
        }
    }
}
```

2）BeanDefinitionRegistry的registerBeanDefinition方法

该方法实现将BeanDefinition对象注入到BeanFactory的beanDefinitionMap集合（ConcurrentHashMap类型）中，而且注入过程中使用了线程安全控制并发。

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

问题：为什么要使用synchronized关键字锁住对象？

beanDefinitionMap本身是ConcurrentHashMap类型的，因此多线程操作内部已经提供了安全的操作，这里为什么在外部要再使用synchronized关键字加锁同步呢？

问题：哪个地方会发生并发操作？

3）registerAlias方法

该方法位于SimpleAliasRegistry类中，DefaultListableBeanFactory间接继承到该类。实现将别名信息注入到SimpleAliasRegistry类的MaliasMap集合中（该集合为ConcurrentHashMap类型对象）。

```
@Override
public void registerAlias(String name, String alias) {

    if (alias.equals(name)) {
        this.aliasMap.remove(alias);
    }
    else {
        if (!allowAliasOverriding()) {
            String registeredName = this.aliasMap.get(alias);
            if (registeredName != null && !registeredName.equals(name)) {
                throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" +
                        name + "': It is already registered for name '" + registeredName + "'.");
            }
        }
        checkForAliasCircle(name, alias);
        this.aliasMap.put(alias, name);
    }
}
```

