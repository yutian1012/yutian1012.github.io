---
title: spring容器的BeanDefinitionParserDelegate解析XML的Docuemnt
tags: [spring]
---

DefaultBeanDefinitionDocumentReader对象解析xml元素是交由BeanDefinitionParserDelegate对象进行处理。

1）BeanDefinitionParserDelegate对象的成员变量定义信息

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

//常用方法
//提供了多个创建BeanDefinition方法
createBeanDefinition(...)
//各种解析标签的方法
parseBeanDefinitionElement(...)
parseBeanDefinitionAttributes(...)
parseMetaElements(...)
parseConstructorArgElements(...)
...
```

注：从结构上可以看到，该类定义了解析文档的通用方法。

2）BeanDefinitionParserDelegate解析代理对象的创建

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

3）初始化DocumentDefaultsDefinition对象，

对于DocumentDefaultsDefinition对象，主要是设置Bean定义的初始默认值，如lazyInit，autowire，default-init-method等属性值。

```
public void initDefaults(Element root, BeanDefinitionParserDelegate parent) {
    populateDefaults(this.defaults, 
        (parent != null ? parent.defaults : null), root);

    this.readerContext.fireDefaultsRegistered(this.defaults);
}
```

4）DefaultBeanDefinitionDocumentReader类parseDefaultElement方法解析元素

DefaultBeanDefinitionDocumentReader会调用BeanDefinitionParserDelegate代理对象的解析方法parseDefaultElement解析元素。

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

查看解析生成bean对象方法（这里我们只查看bean元素的解析）

DefaultBeanDefinitionDocumentReader类的processBeanDefinition方法会调用BeanDefinitionParserDelegate对象的parseBeanDefinitionElement方法

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

5）parseBeanDefinitionElement方法生成BeanDefinitionHolder

bean的解析过程调用了三个重载方法，最终返回BeanDefinitionHolder对象

```
//重载方法一
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
    return parseBeanDefinitionElement(ele, null);
}
//重载方法二
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
    //获取bean标签设置的属性id和name
    String id = ele.getAttribute(ID_ATTRIBUTE);
    String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

    List<String> aliases = new ArrayList<String>();
    if (StringUtils.hasLength(nameAttr)) {
        String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
        aliases.addAll(Arrays.asList(nameArr));
    }

    String beanName = id;

    ...

    //解析Bean标签的构造BeanDefinition对象
    AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);

    if (beanDefinition != null) {
        ...
        
        //封装成BeanDefinitionHolder对象，绑定了name，id和BeanDefinition对象
        String[] aliasesArray = StringUtils.toStringArray(aliases);
        return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
    }

    return null;
}
//重载方法三
public AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, BeanDefinition containingBean) {
    //将对象先放到栈中，finally中再从栈中弹出
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
        //主要是向bd中设置parentName和BeanClass两个属性，生成的对象是GenericBeanDefinition，该对象继承了AbstractBeanDefinition，该抽象类中定义了bean相关的属性和相关操作的属性信息，后面的代码会进一步设置这些属性值。
        AbstractBeanDefinition bd = createBeanDefinition(className, parent);

        //解析属性设置到BeanDefinition对象上，bd上定义了相关成员变量，如scope，lazy-init等属性
        parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);

        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

        //meta标签的处理，问题：spring中meta标签有什么用？
        //可以通过meta定义一些特殊属性存放到bd中
        //AbstractBeanDefinition类继承了BeanMetadataAttributeAccessor类，将Meta信息保存到AttributeAccessorSupport类中定义的Map<String, Object> attributes中
        parseMetaElements(ele, bd);

        //lookup-method标签处理，LookupOverride继承了MethodOverride类
        //将信息封装到bd的MethodOverrides集合中（Set类型的集合）
        parseLookupOverrideSubElements(ele, bd.getMethodOverrides());

        //replaced-method标签处理，ReplaceOverride继承了MethodOverride类
        //将信息封装到bd的MethodOverrides集合中（Set类型的集合）
        parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
        
        //处理bean的constructor-arg标签
        //将解析信息设置到bd的ConstructorArgumentValues变量中
        //成员1（LinkedHashMap<Integer, ValueHolder>）：Map<Integer, ValueHolder> indexedArgumentValues，带索引位置
        //成员2（LinkedList<ValueHolder>）：List<ValueHolder> genericArgumentValues
        parseConstructorArgElements(ele, bd);

        //处理bean的property标签
        //将解析信息设置到bd的MutablePropertyValues变量中
        //成员（ArrayList<PropertyValue>）List<PropertyValue>存储属性信息
        parsePropertyElements(ele, bd);

        //处理bean的qualifier标签
        //将信息设置到bd的Map<String, AutowireCandidateQualifier> qualifiers中
        //使用的是LinkedHashMap类型的map集合
        parseQualifierElements(ele, bd);
        ...

        return bd;
    }
    finally {
        this.parseState.pop();
    }
    ...

    return null;
}
```

注：通过上面代码的分析，可以看出，bean中定义的所有属性及子标签都被解析成相应的对象后设置到BeanDefinition对象中。

parseBeanDefinitionAttributes方法，处理bean元素上的属性信息

```
/**
 * Apply the attributes of the given bean element to the given bean * definition.
 * @param ele bean declaration element
 * @param beanName bean name
 * @param containingBean containing bean definition
 * @return a bean definition initialized according to the bean element attributes
 */
public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName, BeanDefinition containingBean, AbstractBeanDefinition bd) {

    if (ele.hasAttribute(SCOPE_ATTRIBUTE)) {//scope标签
        bd.setScope(ele.getAttribute(SCOPE_ATTRIBUTE));
    }
    else if (containingBean != null) {
        // Take default from containing bean in case of an inner bean definition.
        bd.setScope(containingBean.getScope());
    }

    if (ele.hasAttribute(ABSTRACT_ATTRIBUTE)) {//abstract属性
        bd.setAbstract(TRUE_VALUE.equals(ele.getAttribute(ABSTRACT_ATTRIBUTE)));
    }

    String lazyInit = ele.getAttribute(LAZY_INIT_ATTRIBUTE);
    if (DEFAULT_VALUE.equals(lazyInit)) {//lazy-init属性
        lazyInit = this.defaults.getLazyInit();
    }
    bd.setLazyInit(TRUE_VALUE.equals(lazyInit));

   ...

    return bd;
}
```

问题：this.parseState的作用？