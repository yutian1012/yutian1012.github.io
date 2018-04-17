---
title: spring容器的BeanDefinitionDocumentReader解析XML的Docuemnt
tags: [spring]
---

BeanDefinitionDocumentReader接口的一个实现类是DefaultBeanDefinitionDocumentReader。

1）registerBeanDefinitions方法

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
    //获取profile属性，这里判断profile是否被激活
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
    BeanDefinitionParserDelegate parent = this.delegate;
    //创建解析代理对象
    this.delegate = createDelegate(this.readerContext, root, parent);

    //空方法，用于子类扩展
    preProcessXml(root);
    parseBeanDefinitions(root, this.delegate);
    //空方法，用于子类扩展
    postProcessXml(root);

    this.delegate = parent;
}
```

2）DefaultBeanDefinitionDocumentReader类的parseBeanDefintions方法解析xml

```
/* Parse the elements at the root level in the document:"import", "alias", "bean". */
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

注：可以看到最终解析具体元素交由代理对象BeanDefinitionParserDelegate。在该对象中也会设计到xml文档的标签和属性信息的详细解析。

注2：解析后并没有返回对象，那么是如何将解析的bean注入到容器BeanFactory中的呢？原来在创建BeanDefinitionParserDelegate对象时将XmlReaderContext对象传入到了BeanDefinitionParserDelegate对象中，因此可以通过该上下文对象获取容器等引用。

3）补充：嵌套bean

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