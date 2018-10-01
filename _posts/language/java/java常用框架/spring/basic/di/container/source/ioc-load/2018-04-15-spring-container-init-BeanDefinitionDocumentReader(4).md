---
title: spring容器的BeanDefinitionDocumentReader解析XML的Docuemnt
tags: [spring]
---

BeanDefinitionDocumentReader接口的一个实现类是DefaultBeanDefinitionDocumentReader。

该类用于读取Document对象，并利用BeanDefinitionParserDelegate对象对Document元素进行解析，将获取的BeanDefinition信息注册到容器中。

注：BeanDefinitionParserDelegate包含Environment（容器设置的环境信息，影响Bean的加载）和XmlReaderContext（读取文档的上下文信息，包含了BeanFactory容器的引用路径）。

1）DefaultBeanDefinitionDocumentReader类的定义信息

```
public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {
    public static final String BEAN_ELEMENT = BeanDefinitionParserDelegate.BEAN_ELEMENT;//bean
    public static final String NESTED_BEANS_ELEMENT = "beans";
    public static final String ALIAS_ELEMENT = "alias";
    public static final String NAME_ATTRIBUTE = "name";
    public static final String ALIAS_ATTRIBUTE = "alias";
    public static final String IMPORT_ELEMENT = "import";
    public static final String RESOURCE_ATTRIBUTE = "resource";
    public static final String PROFILE_ATTRIBUTE = "profile";

    private Environment environment;
    private XmlReaderContext readerContext;
    private BeanDefinitionParserDelegate delegate;
    ...
}
```

注：这里可以发现定义在xml中的标签，这里是使用静态常量来表示的。

注2：BeanDefinitionParserDelegate是真正实现解析的代理类，很多不同标签的具体解析都是使用该代理类来实现的。

2）registerBeanDefinitions方法（向容器中注入BeanDefinition）

查看DefaultBeanDefinitionDocumentReader类的registerBeanDefinitions方法。该方法接收Document（待解析的xml文档对象），XmlReaderContext（该对象封装了XmlBeanDefinitionReader对象中的相关对象引用）

```
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    logger.debug("Loading bean definitions");
    //获取文档节点，即xml的根节点beans
    Element root = doc.getDocumentElement();
    //解析文档节点
    doRegisterBeanDefinitions(root);
}

//从<beans>作为root节点进行解析
protected void doRegisterBeanDefinitions(Element root) {
    //获取profile属性，这里判断profile是否被激活，profile用于多环境配置管理
    String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
    if (StringUtils.hasText(profileSpec)) {
        //获取profile的数组
        String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
            profileSpec,BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);

        //判断指定的profile是否被激活，如果没有被激活则跳出方法
        if (!this.environment.acceptsProfiles(specifiedProfiles)) {
            return;
        }
    }

    //XML解析BeanDefinition的代理对象
    //由于每个bean可能存在层次关系，而引发递归调用，为了维护网络层次关系，采用parent属性保留其parent Bean
    //<beans>标签内还可以嵌套定义beans
    BeanDefinitionParserDelegate parent = this.delegate;
    //创建解析代理对象
    this.delegate = createDelegate(this.readerContext, root, parent);

    //空方法，用于子类扩展
    preProcessXml(root);
    parseBeanDefinitions(root, this.delegate);
    //空方法，用于子类扩展
    postProcessXml(root);
    
    //重新绑定到当前成员变量中
    this.delegate = parent;
}
```

3）DefaultBeanDefinitionDocumentReader类的parseBeanDefintions方法解析xml

```
/* Parse the elements at the root level in the document:"import", "alias", "bean". */
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    //判断对象的命名空间是否是beans命名空间，如果是则使用默认命名空间解析方法
    //命名空间为：http://www.springframework.org/schema/beans
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        //遍历子节点
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;

                if (delegate.isDefaultNamespace(ele)) {
                    //处理import，alias，bean，beans标签
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

注：可以看到最终解析具体元素交由代理对象BeanDefinitionParserDelegate。在该对象中也会涉及到xml文档的标签和属性信息的详细解析。

注2：解析后并没有返回对象，那么是如何将解析的bean注入到容器BeanFactory中的呢？原来在创建BeanDefinitionParserDelegate对象时将XmlReaderContext对象传入到了BeanDefinitionParserDelegate对象中，因此可以通过该上下文对象获取容器等引用。

XmlReaderContext类获取容器引用，BeanDefinitionRegister本身就是BeanFactory。还记得DefaultListableBeanFactory对象实现了BeanDefinitionRegister接口。

```
public final BeanDefinitionRegistry getRegistry() {
    return this.reader.getRegistry();
}
```

4）具体的标签解析

DefaultBeanDefinitionDocumentReader类的parseDefaultElement方法，该方法解析import，alias，bean，beans标签

```
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);//处理import标签
    }
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);//处理alias标签
    }
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);//处理Bean标签
    }
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        doRegisterBeanDefinitions(ele);//处理Beans标签，可以看到这里是递归调用
    }
}
```

查看processBeanDefinition方法处理Bean标签

```
/**
 * Process the given bean element, parsing the bean definition
 * and registering it with the registry.
   处理bean标签，并注册到Registry上
 */
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    //利用BeanDefinitionParserDelegate来解析bean标签，并封装成BeanDefinitionHolder对象
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        //装饰bdHolder对象
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            //实现将BeanDefinitionHolder注册到regisry上
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                    bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        // 触发registred事件
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

具体的解析查看spring-container-init-BeanDefinitionParserDelegate(5).md文件
