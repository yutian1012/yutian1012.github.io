---
title: spring中BeanDefinition对象的理解
tags: [spring]
---

参考：https://www.cnblogs.com/davidwang456/p/4192318.html

BeanDefinition相当于一个数据结构，这个数据结构的生成过程是根据解析的resource资源对象中的bean定义而来的，这些bean在Spirng IoC容器内部表示成了的BeanDefintion这样的数据结构，IoC容器对bean的管理和依赖注入的实现都是通过操作BeanDefinition来进行的。

一般情况下，BeanDefinition只在容器启动时加载并解析，除非容器刷新或重启，这些信息不会发生变化，当然如果用户有特殊的需求，也可以通过编程的方式在运行期调整BeanDefinition的定义。

1）查看BeanDefinition接口定义

一个BeanDefinition描述了一个bean的实例，包括属性值，构造方法参数值和继承自它的类的更多信息。BeanDefinition仅仅是一个最简单的接口，主要功能是允许BeanFactoryPostProcessor在构建对象时修改并配置相关属性值。例如PropertyPlaceHolderConfigure能够检索并修改属性值和别的bean的元数据。

```
//bean标签的parent属性信息
String getParentName();
//bean标签的class属性信息
String getBeanClassName();
//bean标签的factory-bean属性信息
String getFactoryBeanName();
//bean标签的factory-method属性信息
String getFactoryMethodName();
//bean标签的scope属性信息
String getScope();
//bean标签的lazy-init属性信息
boolean isLazyInit();
//bean标签的depends-on属性信息
String[] getDependsOn();
//bean标签的autowire-candidate属性信息
boolean isAutowireCandidate();
//bean标签的primary属性信息    
boolean isPrimary();
//构造函数参数对象constructor-arg标签
ConstructorArgumentValues getConstructorArgumentValues();
//property标签
MutablePropertyValues getPropertyValues();
//是否是单例
boolean isSingleton();
//通过拷贝的方式实现BeanDefinition时，会保留被拷贝对象的引用
BeanDefinition getOriginatingBeanDefinition();
...
```

ConstructorArgumentValues类构造参数，保存了构造方法所有的参数值，通常作为beanDefinition的一部分来使用。它不仅支持类型匹配的普通参数，也支持根据参数列表中的索引位置来提供参数。提供了两个变量来保存参数：带索引的和不带索引的

```
public class ConstructorArgumentValues {

    private final Map<Integer, ValueHolder> indexedArgumentValues = new LinkedHashMap<Integer, ValueHolder>(0);

    private final List<ValueHolder> genericArgumentValues = new LinkedList<ValueHolder>();
```

MutablePropertyValues类是PropertyValues接口的默认实现，支持对属性的简单操作，为构造方法提供深度复制和使用map获取构造的支持。

2）查看BeanDefinition实现接口

```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement
```

AttributeAccessor接口定义

AttributeAccessor接口定义了最基本的对任意对象的元数据的修改或者获取，即设置的meta标签信息会封装到该接口定义的属性中。

```
String[] attributeNames() 
Object getAttribute(String name) 
boolean hasAttribute(String name) 
Object removeAttribute(String name) 
void setAttribute(String name, Object value) 
```

BeanMetadataElement接口提供了一个getResource()方法,用来传输一个可配置的源对象。

3）常见的实现类

beanDefinition接口的直接实现类ChildBeanDefinition，GenericBeanDefinition，RootBeanDefinition。