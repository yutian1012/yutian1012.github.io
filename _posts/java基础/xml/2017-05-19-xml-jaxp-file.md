---
title: xml文档的解析--JAXP实现dom对象，文件和字符串转换
tags: [xml]
---

### 概念问题

dom和sax（解析模式）：

解析xml时（如浏览器解析html标签），主要存在两种方式：SAX模式和DOM模式。SAX (Simple API for XML) 和 DOM (Document Object Model) 是当前两个主要的XML API，几乎所有商用的xml 解析器都同时实现了这两个接口。

注： SAX 和 DOM 不是相互排斥的，记住这点很重要。您可以使用 DOM 来创建 SAX 事件流，也可以使用SAX来创建DOM树。事实上，用于创建DOM树的大多数解析器实际上都使用 SAX 来完成这个任务（创建xml文档过程）！

#### 使用JAXP实现dom对象与string间的转换

JAXP（Java API for XMLProcessing，意为XML处理的Java API）

1）字符串转document对象

```
StringReader stringReader=new StringReader(xmlStr);//获取xml的string字符串
DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
DocumentBuilder builder=factory.newDocumentBuilder();
org.w3c.dom.Document document= builder.parse(new InputSource(stringReader));//或传递xml的File对象
```

2）document对象输出字符串/file

```
TransformerFactory factory=TransformerFactory.newInstance();
Transformer transformer= factory.newTransformer();
ByteArrayOutputStream byteArrayOutputStream=new ByteArrayOutputStream();
transformer.setOutputProperty("encoding", "utf-8");//中文问题
transformer.transform(new DOMSource(document), new StreamResult(byteArrayOutputStream));
String xmlStr=byteArrayOutputStream.toString();
```

注：如果接文件输出流，则写入到文件。


