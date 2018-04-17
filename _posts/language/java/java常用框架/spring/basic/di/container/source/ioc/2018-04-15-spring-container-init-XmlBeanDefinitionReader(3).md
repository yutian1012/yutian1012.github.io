---
title: spring容器的XmlBeanDefinitionReader初始化过程
tags: [spring]
---

XmlBeanDefinitionReader是一个载入BeanDefinition的读取器（这是一个xml文件读取定义信息的读取器），将bean定义信息设置到BeanFactory。

在初始化XmlBeanDefinitionReader对象时，接收一个BeanFactory对象。该对象用来注册解析xml得到的BeanDefinition对象。而DefaultListableBeanFactory实现了BeanDefinitionRegister接口，因此，可以直接将容器对象beanFactory传入。

1）XmlBeanDefinitionReader类的初始化

```
DefaultListableBeanFactory factory=new DefaultListableBeanFactory();

XmlBeanDefinitionReader reader=new XmlBeanDefinitionReader(factory);
```

XmlBeanDefinitionReader构造方法

```
public XmlBeanDefinitionReader(BeanDefinitionRegistry registry) {
    super(registry);
}
```

注：DefaultListableBeanFactory实现了BeanDefinitionRegister接口，因此这里传入了DefaultListableBeanFactory对象实例。

2）AbstractBeanDefinitionReader构造函数

父类AbstractBeanDefinitionReader构造器，这个构造器主要干了三件事，一个是将factory对象绑定到属性registry上，另外两个就是初始化ResourceLoader和environment。

```
protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
   
    this.registry = registry;

    // Determine ResourceLoader to use.
    //为什么registry（也就是上面的factory）可能是ResourceLoader呢？其实这里是为ApplicationContext埋下了伏笔，而ApplicationContext实现了ResourceLoader接口
    if (this.registry instanceof ResourceLoader) {
        this.resourceLoader = (ResourceLoader) this.registry;
    }
    else {
        this.resourceLoader = new PathMatchingResourcePatternResolver();
    }

    // Inherit Environment if possible
    //ApplicationContext实现了EnvironmentCapable，因此可以获取相关的Environmen对象
    if (this.registry instanceof EnvironmentCapable) {
        this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
    }
    else {
        this.environment = new StandardEnvironment();
    }
}
```

注：可以看到，这里兼容了ApplicationContext容器的相关设置信息。

3）loadBeanDefinitions方法加载bean定义

XmlBeanDefinitionReader类的loadBeanDefinitions方法

```
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}
```

注：这里应用装饰器模式，包装Resource资源对象。

注2：java中如BufferedInputSteram就是对InputStream的装饰类。装饰类可以不断的嵌套，提供新的特性（增加灵活扩展性）。

```
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    ...

    //变量resourcesCurrentlyBeingLoaded是ThreadLocal类型的，里面保存的是Set类型资源集合。ThreadLocal是为了保证线程安全。
    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();

    ...

    //将encodeResource，资源放入本地线程变量（存放资源的Set中）
    if (!currentResources.add(encodedResource)) {
        //抛出资源的循环依赖异常
        throw new BeanDefinitionStoreException(
                "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }

    //获取资源输入流，从而解析文档
    try {
        //获取资源的输入流
        InputStream inputStream = encodedResource.getResource().getInputStream();
        try {
            //包装成InputSource类型的资源
            InputSource inputSource = new InputSource(inputStream);
            if (encodedResource.getEncoding() != null) {
                inputSource.setEncoding(encodedResource.getEncoding());
            }
            //最终调用doLoadBeanDefinitions方法，解析Bean的定义信息
            return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        }
        finally {
            inputStream.close();
        }
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
                "IOException parsing XML document from " + encodedResource.getResource(), ex);
    }
    finally {
        currentResources.remove(encodedResource);
        if (currentResources.isEmpty()) {
            this.resourcesCurrentlyBeingLoaded.remove();
        }
    }
}
```

doLoadBeanDefinitions方法加载Bean定义文档对象

这个方法其实会调用doLoadDocument()方法获得Document，即W3C中说的document。

```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
        throws BeanDefinitionStoreException {
    try {
        //从流中获取Docuemnt对象
        Document doc = doLoadDocument(inputSource, resource);
        //解析W3C类型的Document对象
        return registerBeanDefinitions(doc, resource);
    }
    //各种异常捕获
    ...
}
```

4）registerBeanDefinitions方法解析并注册

通过registerBeanDefinitions,启动对BeanDefinition的详细解析过程，这个解析过程涉及到了Spring的配置规则。通过这个方法Spring会按照Bean语义要求进行解析，并转化为容器内部的数据结构（BeanDefinition）。

```
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();

    documentReader.setEnvironment(this.getEnvironment());

    int countBefore = getRegistry().getBeanDefinitionCount();

    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));

    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

createBeanDefinitionDocumentReader方法返回BeanDefinitionDocumentReader对象

```
private Class<?> documentReaderClass = DefaultBeanDefinitionDocumentReader.class;
...
protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
    return BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));
}
```

注：这里返回的BeanDefinitionDocumentReader对象是DefaultBeanDefinitionDocumentReader的一个实例。

XmlReaderContext对象，该上下文对象中用于传递XmlBeanDefinitionReader中的相关信息到BeanDefinitionDocumentReader类中。

```
public XmlReaderContext createReaderContext(Resource resource) {
    return new XmlReaderContext(resource,
        this.problemReporter, 
        this.eventListener,
        this.sourceExtractor, 
        this, 
        etNamespaceHandlerResolver());
}
```