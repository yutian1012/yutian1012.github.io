---
title: spring容器的XmlBeanDefinitionReader初始化过程
tags: [spring]
---

XmlBeanDefinitionReader是一个载入BeanDefinition的读取器（这是一个xml文件读取定义信息的读取器），将bean定义信息设置到BeanFactory（因此，需要设置一个BeanFactory对象，用来接收数据）。

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

注：DefaultListableBeanFactory实现了BeanDefinitionRegister接口，因此可以传入DefaultListableBeanFactory对象实例。

父类AbstractBeanDefinitionReader构造器，这个构造器主要干了三件事，一个是将factory对象绑定到属性registry上，另外两个就是初始化ResourceLoader和environment。

```
protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
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

2）加载bean定义

XmlBeanDefinitionReader类的loadBeanDefinitions方法

```
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}
```

注：这里应用装饰器模式，包装Resource。

注2：java中如BufferedInputSteram就是对InputStream的装饰类。装饰类可以不断的嵌套，提供新的特性。

```
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    ...

    //resourcesCurrentlyBeingLoaded是ThreadLocal类型的，里面保存的是Set类型资源集合。ThreadLocal是为了保证线程安全。
    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();

    if (currentResources == null) {
        currentResources = new HashSet<EncodedResource>(4);
        this.resourcesCurrentlyBeingLoaded.set(currentResources);
    }
    //将encodeResource，资源放入本地线程变量（存放资源的Set中）
    if (!currentResources.add(encodedResource)) {
        throw new BeanDefinitionStoreException(
                "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }

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

3）doLoadBeanDefinitions方法加载Bean定义文档对象

这个方法其实会调用doLoadDocument()方法获得Document，即W3C中说的document。

```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
        throws BeanDefinitionStoreException {
    try {
        Document doc = doLoadDocument(inputSource, resource);
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

调用DefaultBeanDefinitionDocumentReader的registerBeanDefinitions方法完成注册

```
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    logger.debug("Loading bean definitions");
    Element root = doc.getDocumentElement();
    doRegisterBeanDefinitions(root);
}

//从<beans>作为root节点进行解析
protected void doRegisterBeanDefinitions(Element root) {
    //获取profile属性，这里判断profile是否被激活
    String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
    if (StringUtils.hasText(profileSpec)) {
        Assert.state(this.environment != null, "Environment must be set for evaluating profiles");
        String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
                profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
        if (!this.environment.acceptsProfiles(specifiedProfiles)) {
            return;
        }
    }

    //解析XML BeanDefinition的代理
    //由于每个bean可能存在层次关系，而引发递归调用，为了维护网络层次关系，采用parent属性保留其parent Bean
    BeanDefinitionParserDelegate parent = this.delegate;
    this.delegate = createDelegate(this.readerContext, root, parent);

    //空方法，用于子类扩展
    preProcessXml(root);
    parseBeanDefinitions(root, this.delegate);
    //空方法，用于子类扩展
    postProcessXml(root);

    this.delegate = parent;
}
```

补充：嵌套bean

在Spring中，如果某个Bean所依赖的Bean不想被Spring容器直接访问，可以使用嵌套Bean。和普通的Bean一样，使用bean元素来定义嵌套的Bean，嵌套Bean只对它的外部的Bean有效，Spring容器无法直接访问嵌套的Bean，因此定义嵌套Bean也无需指定id属性。

```
<bean id="Student" class="com.abc.Student">
    <!-- 下面是一个普通的属性 -->
    <property name="name" value="张三" />
    <!-- 下面的属性是一个嵌套的Bean，对于和Student平级的Bean来说，这个Bean是不可见的，Spring容器也无法访问 -->
    <bean class="com.abc.School" />
</bean>
```

注：嵌套Bean提高了Bean的内聚性，但是降低了程序的灵活性。

5）DefaultBeanDefinitionDocumentReader类的parseBeanDefintions方法解析xml

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    //判断对象的命名空间是否是beans命名空间，如果是则使用默认命名空间解析方法
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        //变量节点
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;

                if (delegate.isDefaultNamespace(ele)) {
                    parseDefaultElement(ele, delegate);
                }
                else {
                    delegate.parseCustomElement(ele);
                }
            }
        }
    }
    else {
        //使用自定义元素解析方法
        delegate.parseCustomElement(root);
    }
}
```

默认元素解析方法parseDefaultElement

```
//根据元素类型不同，调用的注册方法不同
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    //import标签的处理，import导入新的xml配置文件
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    //alias别名元素
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }
    //bean元素，解析xml的bean定义信息，封装成BeanDefinition对象
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    //嵌套的beans标签处理
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        doRegisterBeanDefinitions(ele);
    }
}
```