---
title: spring容器中bean的实例化（通过类型查找bean）
tags: [spring]
---

参考：https://www.cnblogs.com/ToBeAProgrammer/p/5230065.html

bean的实例化过程会涉及到依赖注入的过程（即IOC控制反转到底是怎么实现的），以及单例bean对象的保存和获取。

注：要清楚的分清spring实例化bean和初始化bean是分离的2个阶段。实例化可理解为调用构造函数的newInstance方法，初始化理解为设置对象的属性信息。

1）根据class类型获取bean

DefaultListableBeanFactory类的getBean方法。

整体执行过程：根据type类型转换成对应的beanName数组，如果有多个，先进一步筛选出可作为候选值的beanName数组（进一步筛选缩小候选范围）；如果筛选后数组只有一个，则直接调用getBean方法（重载的方法）；如果筛选后数组仍有多个值，则从数组中获取primaryBean，如果primaryBean有多个则直接抛出异常，如果获取了唯一的primaryBean，则调用getBean方法获（重载的方法）取对象；如果筛选后数组没有任何值，则调用父容器的getBean方法。

```
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
    //根据class类型获取bean的name数组
    /*该方法会先从相应的缓存集合中获取Map<Class<?>, String[]> allBeanNamesByType和Map<Class<?>, String[]> singletonBeanNamesByType中尝试获取，这两个对象都是ConcurrentHashMap类型变量*/
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
    //当只有一个BeanNames值时，直接获取
    if (beanNames.length == 1) {
        return getBean(beanNames[0], requiredType);
    }
    //如果存在多个值，需要从容器中获取primarybean，并且primary不能多个
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

2）getBean方法（AbstractBeanFactory类，完成实例化bean）

```
@Override
public <T> T getBean(String name, Class<T> requiredType) throws BeansException{
    return doGetBean(name, requiredType, null, false);
}
```

doGetBean方法完成bean的实例化，该方法位于AbstractBeanFactory类中。

方法执行过程：

先从缓存的单例对象集合中获取；获取对象后，如果对象是FactoryBean，则调用相应的getObject方法获取实例；如果当前容器不存在，则沿着父容器不断向上查找（查找父容器中缓存的单例对象）；如果缓存中不存在，则获取RootBeanDefinition（合成的bean）对象，通过RootBeanDefinition定义信息实例化对象，将实例化的对象缓存并返回。

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
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else {
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
        }
        
        //判断是否只是类型检查，如果只是类型的检查则不创建对象
        if (!typeCheckOnly) {
            //标记beanName正在创建
            markBeanAsCreated(beanName);
        }

        try {
            //获取RootBeanDefinition，通过BeanDefinition构造实例对象
            final RootBeanDefinition mbd=getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            //获取依赖数组，bean实例化前依赖必须先要实例化
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
                //获取单例的实例对象，并缓存起来
                sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                    @Override
                    public Object getObject() throws BeansException {
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        catch (BeansException ex) {
                          ...
                        }
                    }
                });
                //针对FactoryBean对象的处理
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            
            //创建prototype类型的bean
            else if (mbd.isPrototype()) {
                // It's a prototype -> create a new instance.
                Object prototypeInstance = null;
                try {
                    //这里的befor和after是在线程中设置该bean是否创建的标志
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
                ...
            }
        }
        ...
    }

    // 如果类型不匹配，需要利用TypeConverter进行转换
    if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
        try {
            return getTypeConverter().convertIfNecessary(bean, requiredType);
        }
        catch (TypeMismatchException ex) {
            ...
        }
    }
    return (T) bean;
}
```