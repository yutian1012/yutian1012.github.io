---
title: spring容器的启动时解析xml文档
tags: [spring core]
---

### 解析xml定义bean信息

AbstractXmlApplicationContext-->XmlBeanDefinitionReader-->BeanDefinitionDocumentReader-->BeanDefinitionParserDelegate

注：解析完成后获取BeanDefinition对象，将BeanDefinition对象，及name名称，别名数组封装到BeanDefinitionHolder中，并注册到容器中。

1）AbstractXmlApplicationContext类

AbstractXmlApplicationContext类的loadBeanDefinitions方法：由refresh方法中的obtainFreshBeanFactory方法调用。（refresh-->obtainFreshBeanFactory-->loadBeanDefinitions）

该方法实现根据xml为每个bean生成BeanDefinition并注册到生成的beanFactory。通过XmlBeanDefinitionReader类来加载bean的定义。

2）XmlBeanDefinitionReader类

交由XmlBeanDefinitionReader类的loadBeanDefinitions方法中，该方法接收配置文件的Resource对象（通常为applicationContext.xml）。

```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
        throws BeanDefinitionStoreException {
    try {
        //使用dom对象解析xml文档
        Document doc = doLoadDocument(inputSource, resource);
        return registerBeanDefinitions(doc, resource);
    }
    ...
}
```

3）BeanDefinitionDocumentReader类

交由BeanDefinitionDocumentReader对象来读取文档，实际使用的是DefaultBeanDefinitionDocumentReader类

```
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    //SPI方式定义documentreader（文档的读写者）
    //实现类：DefaultBeanDefinitionDocumentReader
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    //bean定义前的数量
    int countBefore = getRegistry().getBeanDefinitionCount();
    //解析并注册bean定义信息
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    //解析并注册bean的数量
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

4）BeanDefinitionParserDelegate类

交由解析文档的实现类BeanDefinitionParserDelegate，用该类来解析xml文档，通过BeanDefinitionParserDelegate类信息，可以看到在xml中使用的标签以及标签的属性信息都在该类中以常量的形式提供了。

```
//beans标签的命名空间，在该命名空间下定义的标签都交由该类来处理
public static final String BEANS_NAMESPACE_URI = 
    "http://www.springframework.org/schema/beans";
...

//定义的常量信息，可以看出是在xml中经常使用的标签名称和属性
public static final String BEAN_ELEMENT = "bean";
public static final String NAME_ATTRIBUTE = "name";
public static final String ID_ATTRIBUTE = "id";
...
```

5）解析节点过程：

DefaultBeanDefinitionDocumentReader类的parseBeanDefinitions方法，将BeanDefinitionParserDelegate解析类作为参数

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    //判断根节点的命名空间与该解析类是否匹配
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                //判断子节点的命名空间与该解析类是否匹配
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
        delegate.parseCustomElement(root);
    }
}
```

BeanDefinitionParserDelegate解析节点过程：

```
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    //判断并解析import标签
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    //判断并解析alias标签
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }
    //判断并解析bean标签，即类对象定义标签
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    //判断并解析嵌套的beans标签，重新再去回调一遍之前的操作
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        doRegisterBeanDefinitions(ele);
    }
}
```

6）BeanDefinitionParserDelegate解析bean标签

解析完成后，返回BeanDefinitionHolder对象，该对象包含了beanDefinition，bean的id属性值，bean的alia别名数组信息。用于容器的注入时方便查找。

```
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    //parseBeanDefinitionElement方法具体解析bean节点的定义
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        ...
    }
}
```

解析xml中bean节点定义信息

```
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
    //xml中定义的属性，名称，别名集合
    String id = ele.getAttribute(ID_ATTRIBUTE);
    String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
    List<String> aliases = new ArrayList<String>();
    //把名称信息放置到别名集合中
    if (StringUtils.hasLength(nameAttr)) {
        String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
        aliases.addAll(Arrays.asList(nameArr));
    }

    //如果没有定义id，则从别名集合中获取第一个作为id值，并从集合中移除
    String beanName = id;
    if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
        beanName = aliases.remove(0);
        if (logger.isDebugEnabled()) {
            logger.debug("No XML 'id' specified - using '" + beanName +
                    "' as bean name and " + aliases + " as aliases");
        }
    }

    if (containingBean == null) {
        checkNameUniqueness(beanName, aliases, ele);
    }

    //解析标签元素，将属性信息设置到beanDefinition中
    AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
    if (beanDefinition != null) {
        ...
        String[] aliasesArray = StringUtils.toStringArray(aliases);
        return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
    }

    return null;
}
```