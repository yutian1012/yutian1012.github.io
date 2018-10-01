---
title: spring容器中应用InstantiationAwareBeanPostProcessor
tags: [spring]
---

参考：https://blog.csdn.net/u010634066/article/details/80321854

InstantiationAwareBeanPostProcessor实现了BeanPostProcessor接口

```
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
    ...
}
```

在获取bean实例时，AbstractAutowireCapableBeanFactory类的createBean方法，会调用resolveBeforeInstantiation方法。容器获取对象前执行InstantiationAwareBeanPostProcessor接口，根据传递的class类型，判断是否返回代理对象。

注：这是spring容器返回代理对象的一个扩展点

1）AbstractAutowireCapableBeanFactory类resolveBeforeInstantiation方法

```
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    //mdb.beforeInstantiationResolved默认为null，因此分支体会得到执行
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {

        // Make sure bean class is actually resolved at this point.
        //判断容器中是否配置了InstantiationAwareBeanPostProcessor接口的实现类
        if (mbd.hasBeanClass() && !mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            bean = applyBeanPostProcessorsBeforeInstantiation(mbd.getBeanClass(), beanName);
            if (bean != null) {
                bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}
```

注：如果applyBeanPostProcessorsBeforeInstantiation返回bean不为null，则外层createBean方法将直接返回该bean对象。如果返回值为null，调用doCreateBean方法返回实例化bean对象。

2）实例

利用InstantiationAwareBeanPostProcessor接口返回代理对象。用cglib动态代理生成一个代理类，并返回对象。

注：一般我们继承Spring为其提供的适配器类InstantiationAwareBeanPostProcessorAdapter来使用它。

```
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    //postProcessBeforeInstantiation生成并返回了代理类
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        System.out.print("beanName:"+beanName+"执行..postProcessBeforeInstantiation\n");
        //利用 其 生成动态代理
        if(beanClass==TestFb.class){
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(beanClass);
            enhancer.setCallback(new MyMethodInterceptor());
            TestFb testFb = (TestFb)enhancer.create();
            System.out.print("返回动态代理\n");
            return testFb;
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        System.out.print("beanName:"+beanName+"执行..postProcessAfterInstantiation\n");

        return false;
    }

    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        System.out.print("beanName:"+beanName+"执行..postProcessPropertyValues\n");
        return pvs;
    }

    //针对TestFb.class类型的对象不会执行下面的方法，因为直接返回了
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.print("beanName:"+beanName+"执行..postProcessBeforeInitialization\n");

        return bean;
    }
    
    //针对TestFb.class类型的对象不会执行下面的方法，因为直接返回了
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.print("beanName:"+beanName+"执行..postProcessAfterInitialization\n");
        return bean;
    }
}
```

注：这里利用自定义InstantiationAwareBeanPostProcessor接口，在postProcessBeforeInstantiation方法针对类型为TestFb.class生成并返回了一个cglib生成的代理对象。因此后续不会再调用doCreateBean方法，利用RootBeanDefinition实例化bean，因此，也不会再执行生命周期中的postProcessBeforeInitialization等方法。

查看文件2018-08-05-spring-InstantiationAwareBeanPostProcessor.md