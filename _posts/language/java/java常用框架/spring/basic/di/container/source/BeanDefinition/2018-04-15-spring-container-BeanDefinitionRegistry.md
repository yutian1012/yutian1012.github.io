---
title: spring中BeanDefinitionRegistry对象的理解
tags: [spring]
---

参考：https://blog.csdn.net/u011179993/article/details/51598567

BeanDefinitionRegistry接口定义了注册/获取BeanDefinition的方法。

Spring容器的BeanDefinitionRegistry就像是Spring配置信息的内存数据库，主要是以map的形式保存，后续操作直接从BeanDefinitionRegistry中读取配置信息。

1）类定义信息

```
public interface BeanDefinitionRegistry extends AliasRegistry {
    //注册一个BeanDefinition  
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
            throws BeanDefinitionStoreException; 

    //根据name,从自己持有的多个BeanDefinition 中 移除一个
    void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

    // 获取某个BeanDefinition
    BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException; 

    boolean containsBeanDefinition(String beanName);//是否包含 
    String[] getBeanDefinitionNames();//获取所有名称 
    int getBeanDefinitionCount();//获取持有的BeanDefinition数量 
    boolean isBeanNameInUse(String beanName); //判断某个BeanDefinition是否在使用
}
```

注：具体的实现类可查看常用容器类DefaultListableBeanFactory

2）注册BeanDefinition集合

实现类一般使用Map保存多个BeanDefinition

```
Map<String, BeanDefinition> beanDefinitionMap = new HashMap<String, BeanDefinition>();
```

查看DefaultlistableBeanFactory类的registerBeanDefinition方法

```
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {

    ...
    BeanDefinition oldBeanDefinition;

    //处理注册逻辑，将BeanDefinition对象放置到map集合中
    synchronized (this.beanDefinitionMap) {
        oldBeanDefinition = this.beanDefinitionMap.get(beanName);
        if (oldBeanDefinition != null) {
            ...
        }
        else {
            this.beanDefinitionNames.add(beanName);
            this.frozenBeanDefinitionNames = null;
        }
        //设置到集合中
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }

    if (oldBeanDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
}
```

DefaultlistableBeanFactory类对象beanDefinitionMap的数据结构

```
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);
```