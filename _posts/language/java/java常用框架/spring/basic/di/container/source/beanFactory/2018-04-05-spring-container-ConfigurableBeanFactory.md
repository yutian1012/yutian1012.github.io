---
title: spring容器的ConfigurableBeanFactory接口
tags: [spring]
---

ConfigurableBeanFactory接口定义BeanFactory的配置，如类加载器，类型转化，属性编辑器，,BeanPostProcessor，作用域，bean定义，处理bean依赖关系，合并其他ConfigurableBeanFactory，bean如何销毁等。

```
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {
    //Bean对象的作用域
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";
    
    //父容器设置
    void setParentBeanFactory(BeanFactory parentBeanFactory) throws IllegalStateException;

    /类加载器设置与获取，默认使用当前线程中的类加载器
    void setBeanClassLoader(ClassLoader beanClassLoader);

    ClassLoader getBeanClassLoader();

    void setTempClassLoader(ClassLoader tempClassLoader);

    ClassLoader getTempClassLoader();

    //是否需要缓存beanmetadata,比如beandifinition和解析好的classes.默认开启缓存
    void setCacheBeanMetadata(boolean cacheBeanMetadata);

    boolean isCacheBeanMetadata();

    //定义用于解析bean definition的表达式解析器
    void setBeanExpressionResolver(BeanExpressionResolver resolver);

    BeanExpressionResolver getBeanExpressionResolver();

    //类型转化器
    void setConversionService(ConversionService conversionService);

    ConversionService getConversionService();

    //属性编辑器
    void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar);

    //注册自定义的属性编辑器
    void registerCustomEditor(Class<?> requiredType, Class<? extends PropertyEditor> propertyEditorClass);

    //复制注册的属性编辑器
    void copyRegisteredEditorsTo(PropertyEditorRegistry registry);

    //设置自动以类型转换器
    void setTypeConverter(TypeConverter typeConverter);

    TypeConverter getTypeConverter();

    //内嵌String类型的处理器
    void addEmbeddedValueResolver(StringValueResolver valueResolver);

    String resolveEmbeddedValue(String value);

    //添加BeanPostProcessor，实例化处理类，处理类使用有顺序的
    //用于增强bean初始化功能
    void addBeanPostProcessor(BeanPostProcessor beanPostProcessor);

    int getBeanPostProcessorCount();

    //设置作用域
    void registerScope(String scopeName, Scope scope);

    String[] getRegisteredScopeNames();

    Scope getRegisteredScope(String scopeName);

    //访问权限控制
    AccessControlContext getAccessControlContext();

    //合并其他ConfigurableBeanFactory的配置，包括上面说到的BeanPostProcessor，作用域等
    void copyConfigurationFrom(ConfigurableBeanFactory otherFactory);

    void registerAlias(String beanName, String alias) throws BeanDefinitionStoreException;

    void resolveAliases(StringValueResolver valueResolver);

    /**
     * Return a merged BeanDefinition for the given bean name,
     * merging a child bean definition with its parent if necessary.
     * Considers bean definitions in ancestor factories as well.
     * @param beanName the name of the bean to retrieve the merged definition for
     * @return a (potentially merged) BeanDefinition for the given bean
     * @throws NoSuchBeanDefinitionException if there is no bean definition with the given name
     * @since 2.5
     */
    BeanDefinition getMergedBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

    //判断是否是FactoryBean
    boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException;
    
    //bean创建状态控制
    void setCurrentlyInCreation(String beanName, boolean inCreation);

    boolean isCurrentlyInCreation(String beanName);

    //设置bean的依赖，销毁对象前会先销毁依赖的bean
    void registerDependentBean(String beanName, String dependentBeanName);

    String[] getDependentBeans(String beanName);

    String[] getDependenciesForBean(String beanName);

    void destroyBean(String beanName, Object beanInstance);

    void destroyScopedBean(String beanName);

    void destroySingletons();
}
```