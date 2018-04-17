---
title: spring容器的BeanDefinitionParserDelegate解析XML的Docuemnt
tags: [spring]
---

DefaultBeanDefinitionDocumentReader对象解析xml元素是交由BeanDefinitionParserDelegate对象进行处理。

1）BeanDefinitionParserDelegate解析代理对象创建

DefaultBeanDefinitionDocumentReader会创建BeanDefinitionParserDelegate代理对象，然后调用该对象的解析方法。

```
protected BeanDefinitionParserDelegate createDelegate(
    XmlReaderContext readerContext, Element root, BeanDefinitionParserDelegate parentDelegate) {

    BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext, this.environment);
    
    //初始化BeanDefinitionParserDelegate对象的DocumentDefaultsDefinition
    delegate.initDefaults(root, parentDelegate);
    
    return delegate;
}
```

2）初始化DocumentDefaultsDefinition对象，

对于DocumentDefaultsDefinition对象，主要是设置Bean定义的初始默认值，如lazyInit，autowire，default-init-method等属性值。

```
public void initDefaults(Element root, BeanDefinitionParserDelegate parent) {
    populateDefaults(this.defaults, 
        (parent != null ? parent.defaults : null), root);

    this.readerContext.fireDefaultsRegistered(this.defaults);
}
```

3）parseCustomElement方法解析元素

DefaultBeanDefinitionDocumentReader会调用BeanDefinitionParserDelegate代理对象的解析方法parseCustomElement解析元素。

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

4）BeanDefinitionParserDelegate对象的成员变量定义信息

通过查看该类的组成，可以发现该类提供了final static类型的字符串来标识xml中元素标签名，或属性信息。并且能够通过XMLReaderContext对象获取容器相关的引用。

```
//beans的命名空间
public static final String BEANS_NAMESPACE_URI = 
    "http://www.springframework.org/schema/beans";

//xml中的常用属性名称
public static final String NAME_ATTRIBUTE = "name";

public static final String PARENT_ATTRIBUTE = "parent";

public static final String CLASS_ATTRIBUTE = "class";

public static final String ID_ATTRIBUTE = "id";

....

//常用标签定义
public static final String BEAN_ELEMENT = "bean";

public static final String META_ELEMENT = "meta";

public static final String PROPERTY_ELEMENT = "property";

public static final String LIST_ELEMENT = "list";

public static final String SET_ELEMENT = "set";

...

//该上下文对象能够获取容器等对象的引用
private final XmlReaderContext readerContext;

//bean定义的默认属性信息存放到该对象中
private final DocumentDefaultsDefinition defaults = new DocumentDefaultsDefinition();

private Environment environment;

private final Set<String> usedNames = new HashSet<String>();
```

5）processBeanDefinition方法解析生成bean对象

这里我们只查看bean元素的解析，查看具体的处理过程。

```
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    //处理在xml文件中定义的<bean>,以及他的对应的属性，并封装成holder
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            //最后将对象注册到FactoryBean中的beanDefinitionMap属性中
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                    bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

注：BeanDefinition可以看成是对bean定义的抽象，是容器中的基本数据类型，封装了Bean所需要的所有的数据。

注2：BeanDefinitionHolder是BeanDefinition对象的封装类，封装了BeanDefinition的BeanDefinition，名字和别名。

```
public class BeanDefinitionHolder implements BeanMetadataElement {

    private final BeanDefinition beanDefinition;

    private final String beanName;

    private final String[] aliases;
}
```

注：通过BeanDefinitionHolder对象的属性字段，也可以看出BeanDefinitionHolder对BeanDefinition的封装。

6）parseBeanDefinitionElement方法生成BeanDefinitionHolder

bean的解析过程调用了三个重载方法，最终返回BeanDefinitionHolder对象

```
//重载方法一
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
    return parseBeanDefinitionElement(ele, null);
}
//重载方法二
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
    String id = ele.getAttribute(ID_ATTRIBUTE);
    String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

    List<String> aliases = new ArrayList<String>();
    if (StringUtils.hasLength(nameAttr)) {
        String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
        aliases.addAll(Arrays.asList(nameArr));
    }

    String beanName = id;

    ...

    AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);

    if (beanDefinition != null) {
        ...

        String[] aliasesArray = StringUtils.toStringArray(aliases);
        return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
    }

    return null;
}
//重载方法三
public AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, BeanDefinition containingBean) {

    this.parseState.push(new BeanEntry(beanName));

    String className = null;//class属性的处理
    if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
        className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
    }

    try {
        String parent = null;//parent属性
        if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
            parent = ele.getAttribute(PARENT_ATTRIBUTE);
        }
        //具体查看BeanDefinition对象的数据结果和创建过程
        AbstractBeanDefinition bd = createBeanDefinition(className, parent);

        //解析属性设置到BeanDefinition对象上
        parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);

        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

        //meta标签的处理
        parseMetaElements(ele, bd);
        //lookup-method标签处理
        parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        //replaced-method标签处理
        parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
        
        ...

        return bd;
    }
    ...

    return null;
}
```