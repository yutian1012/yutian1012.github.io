---
title: spring容器的DocumentLoader接口
tags: [spring]
---

接口：org.springframework.beans.factory.xml.DocumentLoader

1）描述

该接口是加载xml文档的策略接口

2）DefaultDocumentLoader实现了DocumentLoader接口

spring用于加载xml文档的默认实现类，使用JAXP（Java API for XMLProcessing）作为xml文档解析器。

解析过程集中在loadDocument方法中

```
@Override
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
        ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

    //获取工厂实例，并向工厂对象中设置是否支持namspaceAware以及是否校验xml文档
    DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
    ...
    //构建DocumentBuilder对象，builder对象中设置entityResolver和errorHandler
    DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);

    //解析数据输入源，构造出Document对象
    return builder.parse(inputSource);
}
```