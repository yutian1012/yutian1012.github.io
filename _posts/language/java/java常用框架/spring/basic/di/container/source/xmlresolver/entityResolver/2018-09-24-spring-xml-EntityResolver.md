---
title: spring容器的EntityResolver解析器
tags: [spring]
---

org.xml.sax.EntityResolver接口

对于解析一个xml，sax首先会读取该xml文档上的声明，根据声明去寻找相应的dtd定义，以便对文档的进行验证。

注：默认的寻找规则（即：通过网络，实现上就是声明DTD的地址URI地址来下载DTD声明），并进行认证，下载的过程是一个漫长的过程，而且当网络不可用时，这里会报错，就是应为相应的dtd没找到。

1）描述

EntityResolver的作用就是为应用本身提供一个如何寻找DTD或Schema声明的方法。spring中是如何处理的，针对DTD使用BeansDtdResolver处理，针对schema使用PluggableSchemaResolver来处理。

```
public abstract InputSource resolveEntity (String publicId,String systemId)
        throws SAXException, IOException;
```

注：接口方法resolveEntity接收2个参数（publicId，systemId），并返回一个InputStream对象。

2）DelegatingEntityResolver实现EntityResolver接口

```
/** Suffix for DTD files */
public static final String DTD_SUFFIX = ".dtd";
/** Suffix for schema definition files */
public static final String XSD_SUFFIX = ".xsd";

private final EntityResolver dtdResolver;

private final EntityResolver schemaResolver;
```

注：该代理中包含了dtdResolver和schemaResolver两种方式的解析，根据xml的不同方式使用不同的解析器解析。

相关的实现类为BeansDtdResolver和PluggableSchemaResolver类

```
public DelegatingEntityResolver(ClassLoader classLoader) {
    this.dtdResolver = new BeansDtdResolver();
    this.schemaResolver = new PluggableSchemaResolver(classLoader);
}
```

3）ResourceEntityResolver继承DelegatingEntityResolver类

提供了通过ResourceLoader来加载相关的校验问题，先通过DelegatingEntityResolver去加载，如果加载不到，则使用ResourceLoader通过指定路径加载，适用于分模块配置和加载多个xml定义文档。