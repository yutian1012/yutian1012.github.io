---
title: spring容器中bean的实例化（通过类型查找bean）
tags: [spring]
---

参考：https://www.cnblogs.com/ToBeAProgrammer/p/5230065.html

bean的实例化过程会涉及到依赖注入的过程（即IOC控制反转到底是怎么实现的），以及单例bean对象的保存和获取。

1）根据class类型获取bean

DefaultListableBeanFactory类的getBean方法

```
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
    //根据class类型获取bean的name数组
    String[] beanNames = getBeanNamesForType(requiredType);
    if (beanNames.length > 1) {
        ArrayList<String> autowireCandidates = new ArrayList<String>();
        for (String beanName : beanNames) {
            //判断BeanDefinition对象是否存在，并且BeanDefinition是否默认候选
            if (!containsBeanDefinition(beanName) || getBeanDefinition(beanName).isAutowireCandidate()) {
                autowireCandidates.add(beanName);
            }
        }
        if (autowireCandidates.size() > 0) {
            beanNames = autowireCandidates.toArray(new String[autowireCandidates.size()]);
        }
    }
    if (beanNames.length == 1) {
        return getBean(beanNames[0], requiredType);
    }
    else if (beanNames.length > 1) {
        T primaryBean = null;
        //for循环不是找到就返回，而是要遍历所有的数据，判断是否包含多个
        for (String beanName : beanNames) {
            //获取实例
            T beanInstance = getBean(beanName, requiredType);
            if (isPrimary(beanName, beanInstance)) {
                if (primaryBean != null) {
                    throw new NoUniqueBeanDefinitionException(requiredType, beanNames.length,
                            "more than one 'primary' bean found of required type: " + Arrays.asList(beanNames));
                }
                primaryBean = beanInstance;
            }
        }
        if (primaryBean != null) {
            return primaryBean;
        }
        throw new NoUniqueBeanDefinitionException(requiredType, beanNames);
    }
    //从父类中查找
    else if (getParentBeanFactory() != null) {
        return getParentBeanFactory().getBean(requiredType);
    }
    else {//找不到时抛出异常
        throw new NoSuchBeanDefinitionException(requiredType);
    }
}
```

2）getBean方法（实例化bean）

```
@Override
public <T> T getBean(String name, Class<T> requiredType) throws BeansException{
    return doGetBean(name, requiredType, null, false);
}
```

doGetBean方法完成bean的实例化，该方法位于AbstractBeanFactory类中

```
protected <T> T doGetBean(final String name, final Class<T> requiredType,
    final Object[] args, boolean typeCheckOnly) throws BeansException {

    //获取格式的bean名称，如&开头的name表示引用，该方法对其进行格式化
    //该方法同时实现将传递的别名转成正确的beanname
    final String beanName = transformedBeanName(name);
    Object bean;

    //先从缓存中获得bean，处理那些已经被创建过的单例bean，单例bean不重复创建。
    //DefaultSingletonBeanRegistry类中singletonObjects是一个ConcurrentHashMap，用来保存单例Bean实例
    Object sharedInstance = getSingleton(beanName);

    if (sharedInstance != null && args == null) {
        //处理FactoryBean的情况，如果sharedInstance实例不是FactoryBean类型的对象，则直接返回，否则使用FactoryBean创建bean对象
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }

    else {
        // Fail if we're already creating this bean instance:
        // We're assumably within a circular reference.
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        // Check if bean definition exists in this factory.
        //如果当前Factory中没有找到，则查找parentBeanFactory，并且沿着双亲链一直往上找
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // Not found -> check parent.
            String nameToLookup = originalBeanName(name);
            if (args != null) {
                // Delegation to parent with explicit args.
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else {
                // No args -> delegate to standard getBean method.
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
        }
        
        //判断是否只是类型检查，如果只是类型的检查则不创建对象
        if (!typeCheckOnly) {
            //标记beanName正在创建
            markBeanAsCreated(beanName);
        }

        try {
            //获取RootBeanDefinition
            final RootBeanDefinition mbd=getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            //获取依赖数组
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                //处理依赖bean的实例化
                for (String dependsOnBean : dependsOn) {
                    //判断beanName是否依赖，结合下面的registerDependentBean方法理解依赖关系
                    if (isDependent(beanName, dependsOnBean)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                "Circular depends-on relationship between '" + beanName + "' and '" + dependsOnBean + "'");
                    }
                    //注册所依赖的bean
                    registerDependentBean(dependsOnBean, beanName);
                    //实例化依赖的bean，这里其实是一个递归操作
                    getBean(dependsOnBean);
                }
            }

            // Create bean instance.
            //创建单例bean
            if (mbd.isSingleton()) {
                //匿名内部类，实现了ObjectFactory接口，接口的getObject()方法是一个回调的方法
                sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                    @Override
                    public Object getObject() throws BeansException {
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        catch (BeansException ex) {
                            // Explicitly remove instance from singleton cache: It might have been put there
                            // eagerly by the creation process, to allow for circular reference resolution.
                            // Also remove any beans that received a temporary reference to the bean.
                            destroySingleton(beanName);
                            throw ex;
                        }
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            
            //创建prototype类型的bean
            else if (mbd.isPrototype()) {
                // It's a prototype -> create a new instance.
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            else {//创建指定scope的bean
                String scopeName = mbd.getScope();
                final Scope scope = this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope '" + scopeName + "'");
                }
                try {
                    Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                        @Override
                        public Object getObject() throws BeansException {
                            beforePrototypeCreation(beanName);
                            try {
                                return createBean(beanName, mbd, args);
                            }
                            finally {
                                afterPrototypeCreation(beanName);
                            }
                        }
                    });
                    bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                }
                catch (IllegalStateException ex) {
                    throw new BeanCreationException(beanName,
                            "Scope '" + scopeName + "' is not active for the current thread; " +
                            "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                            ex);
                }
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    // Check if required type matches the type of the actual bean instance.
    if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
        try {
            return getTypeConverter().convertIfNecessary(bean, requiredType);
        }
        catch (TypeMismatchException ex) {
            if (logger.isDebugEnabled()) {
                logger.debug("Failed to convert bean '" + name + "' to required type [" +
                        ClassUtils.getQualifiedName(requiredType) + "]", ex);
            }
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
        }
    }
    return (T) bean;
}
```

补充：子BeanDefinition和父BeanDefinition

参考：https://blog.csdn.net/disiwei1012/article/details/77142167

参考：https://www.cnblogs.com/davidwang456/p/4192318.html

理解：dependentBeanMap和dependenciesForBeanMap的区别？

DefaultSingletonBeanRegistry类的registerDependentBean方法中的这两个变量。