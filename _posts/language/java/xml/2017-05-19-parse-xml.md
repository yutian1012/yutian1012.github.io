---
title: xml文档的解析方式
tags: [xml]
---

XML的解析从数据结构上来讲，分两大类: Streaming和Tree。

Streaming又分为SAX和StAX。Tree就是DOM。SAX和StAX均是顺序解析XML，并生成读取事件。我们可以通过监听事件来得到我们想要的内容。DOM是一次性的以tree结构形式载入内存。

### Stream解析方式与dom解析方式的区别

1）占用内存

DOM需要更多内存，对于大文档或者多文档，DOM性能差。还有，在android手机上就少用DOM这种占内存的东东吧。

2）Streaming是实时性的，它没有上下文

如果一个XML的element需要上下文才能理解，使用DOM会方便。

3）xml结构不明确使用Streaming方式

如果XML来自网络我们对其结构并不明朗，使用Streaming比较好。DOM适合对XML的结构非常清楚。比如web.xml的结构就是一个人人皆知的结构。

4）修改节点

需要对XML进行增删改查，则使用DOM

### SAX与StAX解析区别

Streaming又包含SAX和StAX，SAX是推(push)解析方法，而StAX是拉(pull)解析方法：

1）Pull可以让我们的代码掌握主动权，在合适的时候去调用解析器继续工作。Push是被动的听从解析器只会。解析器会不停的读，并把事件push到handler中。

2）Pull的代码简单，小，Lib也小。

3）Pull可以一个线程同时解析多个文档。因为主动权在我们

4）StAX可以将一个普通的数据流伪造成一个个XML的读取事件，从而构造成一个XML。


参考：http://blog.csdn.net/dongfengkuayue/article/details/50240157

### java解析xml的API

J2SE的JAXP提供了5个包,用于支持XML. 

1）javax.xml.parsers - 为各种第三方parser提供了接口. 

2）org.w3c.dom - 提供了DOM类

3）org.xml.sax - 提供了SAX类

4）javax.xml.transform - 提供了XSLT的API.

5）javax.xml.stream - 提供了STAX的API. STAX比SAX简单,比DOM快.

6）javax.xml.xpath - 使用xpath对DOM进行字段查询.

### 常用解析xml技术

XPath，Stax，JAXB，JAXP

注：SAX和StAX都基于事件机制，而JDOM和DOM4J借助了XPath

首先，XML解析器存在SAX，StAX和DOM，而XML文件生成方法又有StAX和DOM。XPath是一个查询DOM的工具。XSLT是转换XML格式的工具.

参考：https://my.oschina.net/xpbug/blog/104412

2）
xslt，css

3）
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="test/xsl" href="bookStore.xsl"?>
<!DOCTYPE bookStore PUBLIC "bookStore.dtd" "bookStore.dtd">
```

### 扩展

JAXP的parser以及如何使用第三方parser.

XML的解析方法SAX,DOM以及STAX.

XML的写出方法STAX和XSLT.

使用XPath搜索DOM.

JAXP使用XSLT转换XML.

DOM与JDOM,DOM4J的区别.

JAXP验证XML.

JAXP支持namespace
