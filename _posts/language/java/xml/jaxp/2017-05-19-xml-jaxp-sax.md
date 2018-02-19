---
title: xml文档的解析--使用JAXP以SAX解析xml
tags: [xml]
---

参考：http://blog.csdn.net/redarmy_chen/article/details/12951649

### 原理

SAX采用事件处理的方式解析XML文件，利用 SAX 解析 XML 文档，涉及两个部分：解析器和事件处理器：解析器可以使用JAXP的API创建，创建出SAX解析器后，就可以指定解析器去解析某个XML文档。

1）解析器：

采用SAX方式在解析某个XML文档时，它只要解析到XML文档的一个组成部分（节点），都会去调用事件处理器的一个方法，解析器在调用事件处理器的方法时，会把当前解析到的xml文件内容作为方法的参数传递给事件处理器。

2）事件处理器：

由程序员编写，程序员通过事件处理器中方法的参数，就可以很轻松地得到sax解析器解析到的数据，从而可以决定如何对数据进行处理。

注：SAX API中主要有四种处理事件的接口，它们分别是ContentHandler，DTDHandler， EntityResolver 和 ErrorHandler

### SAX方式解析的特点

1）解析根节点方法

解析xml的根节点用的是startElement方法，而不是startDocument方法。

2）事件处理器接口

DefaultHandler实现了ContentHandler，DTDHandler， EntityResolver 和 ErrorHandler四个接口

3）可注册多个事件处理器

ContentHandler对象可以注册多个，并行接收事件。

4）不能随机访问

SAX解析器对文档的解析是顺序执行的

5）不能修改文档

使用SAX对文档进行解析，只能访问文档内容，无法做到向文档中添加节点，更不能删除和修改文档中的内容