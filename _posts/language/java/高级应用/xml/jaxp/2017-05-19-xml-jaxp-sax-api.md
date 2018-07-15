---
title: 使用JAXP以SAX解析xml相关API
tags: [xml]
---

参考：http://blog.csdn.net/redarmy_chen/article/details/12951649

### 涉及类

XMLReaderFactory，XMLReader，SAXParserFactory，SAXParser

1）XMLReaderFactory类

位于org.xml.sax.helpers包中，是XML reader创建工厂类。

2）XMLReader类

读取并解析xml，触发相应事件，并由相应的事件处理器解决

3）事件处理器接口

DefaultHandler实现了ContentHandler，DTDHandler， EntityResolver 和 ErrorHandler四个接口。

相关接口方法包括：startDocument()，startElement()，characters()，endElement()，endDocument（）

### 处理器接口和方法

自定义处理器需要继承DefaultHandler接口，完成事件的处理。

解析节点元素：（节点的解析是按顺序进行的操作）

1）文档开始事件处理方法

重写public void startDocument()方法（不常用）

2）节点开始事件处理方法

重写public void startElement(String uri, String localName, String qName,Attributes attributes)方法，

参数：
uri - 名称空间 URI（如标签的前缀所指的命名空间），如果元素没有名称空间 URI，或者未执行名称空间处理，则为空字符串。；
localName - 本地名称（不带前缀），如果未执行名称空间处理，则为空字符串；
qName - 限定名（带有前缀），如果限定名不可用，则为空字符串；
atts - 连接到元素上的属性。如果没有属性，则它将是空 Attributes 对象。

3）读取节点数据：

重写public void characters(char[] ch, int start, int length)方法（继startElement方法后的执行方法），

4）节点结束事件处理方法

重写public void endElement(String uri, String localName, String qName)方法

5）文档结束事件处理方法，

重写public void endDocument()方法（不常用）