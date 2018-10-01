---
title: spring类型转换
tags: [spring]
---

参考：https://blog.csdn.net/dwade_mia/article/details/79340302

1）ConfigurableBeanFactory类中的类型转换

ConfigurableBeanFactory是BeanFactory的子类，提供了包括支持类型转换功能的一系列配置接口

```
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

    void setConversionService(ConversionService conversionService);

    ConversionService getConversionService();

    void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar);

    void registerCustomEditor(Class<?> requiredType, Class<? extends PropertyEditor> propertyEditorClass);

    void setTypeConverter(TypeConverter typeConverter);

    TypeConverter getTypeConverter();

    // other code......
}
```

2）TypeConverter和Converter接口

其中TypeConverter在2.5+版本提供，默认使用SimpleTypeConverter实现，底层基于jdk提供的PropertyEditor，只能实现String与Object的相互转换，由于spring创建bean的时候，配置信息大多是String类型，因此满足大部分的类型转换场景。

但是这种弊端很明显，只能满足String与Object的相互转换，因此spring3.x引入了Converter接口。
