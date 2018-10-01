---
title: spring中RootBeanDefinition对象的理解
tags: [spring]
---

获取RootBeanDefinition的过程就是子bean合并父bean配置信息的过程。

注：spring beanFactory运行期间，可以返回一个特定的bean（是运行期间生成的Bean对象）。RootBeanDefinition可以作为一个重要的通用的beanDefinition视图。可以理解当我们通过BeanFactory的getBean方法获取对象时，会从容器中加载整合RootBeanDefinition，然后根据RootBeanDefinition的指导来实例化对象。

注2：生成对象是都是使用RootBeanDefinition最为生成对象的指导对象。

注3：理解时结合ChildBeanDefinition分析

从spring2.5后，编写注册bean definition有了更好的的方法：GenericBeanDefinition。GenericBeanDefinition支持动态定义父类依赖，而非硬编码作为root bean definition。

1）理解merge Bean的概念

在xml中配置的BeanDefinition一般都是GenriceBeanDefinition类型的bean。而指定了相应的parent属性的bean可以使用ChildBeanDefinition类型来表示（GenriceBeanDefinition可代替该bean）。这里ChildBeanDefinition只包含了xml中的配置信息，而父Bean的信息却并不包含。

在使用BeanDefinition创建对象实例时，需要完整的BeanDefinition信息。因此引入了RootBeanDefinition对象，该对象整合了ChildBeanDefintion和对应的parent定义的Bean信息。是一个完整的BeanDefinition。

2）RootBeanDefinition的生成

BeanFacotry的getBean方法利用AbstractBeanFactory中定义的getMergedLocalBeanDefinition方法获取RootBeanDefinition

```
protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException {
    // Quick check on the concurrent map first, with minimal locking.
    //首先从缓存中获取mergedBeanDefinitions（Map<String, RootBeanDefinition>）
    RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);
    if (mbd != null) {
        return mbd;
    }
    return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));
}
```

查看RootBeanDefinition对象的构造过程

```
protected RootBeanDefinition getMergedBeanDefinition(
        String beanName, BeanDefinition bd, BeanDefinition containingBd)
        throws BeanDefinitionStoreException {

    synchronized (this.mergedBeanDefinitions) {
        RootBeanDefinition mbd = null;

        // 尝试从缓存中获取，保证线程安全
        if (containingBd == null) {
            mbd = this.mergedBeanDefinitions.get(beanName);
        }

        if (mbd == null) {
            if (bd.getParentName() == null) {//没有定义parent属性
                // Use copy of given root bean definition.
                if (bd instanceof RootBeanDefinition) {
                    mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
                }
                else {
                    mbd = new RootBeanDefinition(bd);
                }
            }
            else {//定义了parent属性
                // Child bean definition: needs to be merged with parent.
                BeanDefinition pbd;//获取父Bean
                try {
                    String parentBeanName = transformedBeanName(bd.getParentName());
                    if (!beanName.equals(parentBeanName)) {
                        //递归调用获取父Bean
                        pbd = getMergedBeanDefinition(parentBeanName);
                    }
                    else {
                        if (getParentBeanFactory() instanceof ConfigurableBeanFactory) {
                            //从父容器中获取
                            pbd = ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(parentBeanName);
                        }
                        ...
                    }
                }
                ...
                // Deep copy with overridden values.
                //使用父Bean作为RootBeanDefinition的构造参数，配置对象
                mbd = new RootBeanDefinition(pbd);
                //将子bean定义的信息配置到RootBeanDefinition对象上
                mbd.overrideFrom(bd);
            }

            ...

            // 缓存对象到mergedBeanDefinitions集合中
            if (containingBd == null && isCacheBeanMetadata() && isBeanEligibleForMetadataCaching(beanName)) {
                this.mergedBeanDefinitions.put(beanName, mbd);
            }
        }

        return mbd;
    }
}
```