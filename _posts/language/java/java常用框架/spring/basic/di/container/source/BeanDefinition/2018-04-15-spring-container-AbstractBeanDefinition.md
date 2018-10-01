---
title: spring中AbstractBeanDefinition对象的理解
tags: [spring]
---

2）AbstratBeanDefinition抽象类定义

```
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
        implements BeanDefinition, Cloneable {
}
```

BeanMetadataAttributeAccessor接口

```
public class BeanMetadataAttributeAccessor extends AttributeAccessorSupport implements BeanMetadataElement {
```

BeanMetadataAttributeAccessor接口既实现了BeanMetadataElement接口提供的getResource()方法也提供了AttributeAccessorSupport针对属性的增删改查，如上AttributeAccessor的方法。

3）三种构造函数

方式一：创建一个新的默认设置的AbstractBeanDefinition

```
protected AbstractBeanDefinition() {
    this(null, null);
}
```

方式二：根据构造参数值和属性参数值来创建

```
protected AbstractBeanDefinition(ConstructorArgumentValues cargs, MutablePropertyValues pvs) {
    setConstructorArgumentValues(cargs);
    setPropertyValues(pvs);
}
```

方式三：深复制一个原有的beandefinition

```
protected AbstractBeanDefinition(BeanDefinition original) {
    setParentName(original.getParentName());
    setBeanClassName(original.getBeanClassName());
    ...
    copyAttributesFrom(original);

    if (original instanceof AbstractBeanDefinition) {
        AbstractBeanDefinition originalAbd = (AbstractBeanDefinition) original;
        if (originalAbd.hasBeanClass()) {
            setBeanClass(originalAbd.getBeanClass());
        }
        setAutowireMode(originalAbd.getAutowireMode());
        ...
        setResource(originalAbd.getResource());
    }
    else {
        setResourceDescription(original.getResourceDescription());
    }
}
```