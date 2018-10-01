---
title: spring容器中bean的单例对象的创建
tags: [spring]
---

1）获取单例对象

getBean方法会先尝试从单例缓冲中获取对象，处理那些已经被创建过的单例bean，单例bean不重复创建。DefaultSingletonBeanRegistry类中singletonObjects是一个ConcurrentHashMap，用来保存单例Bean实例。

调用层次DefaultListableBeanFactory类的getBean方法，调用AbstractBeanFactory的getBean方法，进而调用到DefaultSingletonBeanRegistry类的getSingleton（AbstractBeanFactory类通过继承FactoryBeanRegistrySupport间接继承到DefaultSingletonBeanRegistry）.

2）查看DefaultSingletonBeanRegistry类的getSingleton方法

会先遍历缓存集合（singletonObjects，earlySingletonObjects）获取对象，如果获取不到，则从单例工厂集合中获取对象（singletonFactories），然后调用接口的getObject生成实例，并将实例对象设置到earlySingletonObjects缓存集合中。

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //Map<String, Object>（ConcurrentHashMap类型），该集合用于缓存对象
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                //判断是否是工厂Bean，如果是，则调用getObject创建并缓存对象
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

注：单例对象会进行缓存，获取时能够从缓存中快速得到对象。