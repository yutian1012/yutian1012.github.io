---
title: spring容器中创建bean实例
tags: [spring]
---

AbstractAutowireCapableBeanFactory类中的doCreateBean方法，会调用createBeanInstance是bean的创建。

1）createBeanInstance方法查看

```
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
    ...
    //方式一：通过factory-method构造对象
    //定义bean时（bean标签中）是否提供给了factory-method属性
    if (mbd.getFactoryMethodName() != null)  {
        //使用相应的工厂方法创建对象
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

    //方式二：通过构造函数创建对象
    // Shortcut when re-creating the same bean...
    //查缓存的构造信息，如果已经缓存，说明之前处理过
    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            //spring根据构造函数参数不同，需要进行解析，并将解析对象缓存在RootBeanDefinition中的属性ResolvedConstructorOrFactoryMethod，方便下次直接使用
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    // 解析构造函数并进行构造函数的实例化
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null ||
            mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
            mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // No special handling: simply use no-arg constructor.
    //方式三：使用无参构造函数构造对象
    return instantiateBean(beanName, mbd);
}
```

注：实例化对象时还要查看具体的getInstantiationStrategy策略，其中SimpleInstantiationStrategy实例化策略是利用构造函数的newInstance方式实例化。

2）查看instantiateBean方法

该方法是用来进行实例化对象的具体方法

```
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
    try {
        Object beanInstance;
        final BeanFactory parent = this;
        //安全管理问题的处理
        if (System.getSecurityManager() != null) {
            beanInstance = AccessController.doPrivileged(new PrivilegedAction<Object>() {
                @Override
                public Object run() {
                    return getInstantiationStrategy().instantiate(mbd, beanName, parent);
                }
            }, getAccessControlContext());
        }
        else {//使用初始化策略实现对象实例化操作
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
        }
        //将实例化对象包装成BeanWrapper对象
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);
        return bw;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
    }
}
```

3）getInstantiationStrategy().instantiate实现对象的实例化

AbstractAutowireCapableBeanFactory类instantiationStrategy实例化默认策略为CglibSubclassingInstantiationStrategy。

查看CglibSubclassingInstantiationStrategy类定义的instantiate方法。定位到其父类SimpleInstantiationStrategy中的instantiate方法。

```
@Override
public Object instantiate(RootBeanDefinition beanDefinition, String beanName, BeanFactory owner) {
    //判断是否重写了方法
    if (beanDefinition.getMethodOverrides().isEmpty()) {
        Constructor<?> constructorToUse;
        synchronized (beanDefinition.constructorArgumentLock) {
            //从缓存中获取使用的构造函数
            constructorToUse = (Constructor<?>) beanDefinition.resolvedConstructorOrFactoryMethod;
            //自行处理获取可以使用的构造函数
            if (constructorToUse == null) {
                final Class<?> clazz = beanDefinition.getBeanClass();
                //cglib方式不能代理接口
                if (clazz.isInterface()) {
                    throw new BeanInstantiationException(clazz, "Specified class is an interface");
                }
                try {
                    //安全问题处理
                    if (System.getSecurityManager() != null) {
                        constructorToUse = AccessController.doPrivileged(new PrivilegedExceptionAction<Constructor<?>>() {
                            @Override
                            public Constructor<?> run() throws Exception {
                                return clazz.getDeclaredConstructor((Class[]) null);
                            }
                        });
                    }
                    else {
                        //获取默认构造方法
                        constructorToUse =  clazz.getDeclaredConstructor((Class[]) null);
                    }
                    //将对象缓存到bd对象中，方便下次直接使用
                    beanDefinition.resolvedConstructorOrFactoryMethod = constructorToUse;
                }
                catch (Exception ex) {
                    throw new BeanInstantiationException(clazz, "No default constructor found", ex);
                }
            }
        }
        //利用BeanUtils工具类实例化对象
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        // Must generate CGLIB subclass.
        return instantiateWithMethodInjection(beanDefinition, beanName, owner);
    }
}
```

4）查看BeanUtils类的instantiateClass方法实现

```
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
    Assert.notNull(ctor, "Constructor must not be null");
    try {
        ReflectionUtils.makeAccessible(ctor);
        //使用构造函数的newInstance构造对象
        return ctor.newInstance(args);
    }
    ...
}
```