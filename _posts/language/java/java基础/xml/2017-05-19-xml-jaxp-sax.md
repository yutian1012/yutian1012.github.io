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

### 涉及类

XMLReaderFactory，XMLReader，SAXParserFactory，SAXParser

1）XMLReaderFactory类

位于org.xml.sax.helpers包中，是XML reader创建工厂类。

2）XMLReader类

读取并解析xml，触发相应事件，并由相应的事件处理器解决

### 创建解析器（方式一）

通过XMLReaderFactory、XMLReader完成

```
//通过XMLReaderFactory创建XMLReader对象
XMLReader reader = XMLReaderFactory.createXMLReader();
//设置事件处理器对象：
//这里的MyDefaultHandler是自定义的处理器
reader.setContentHandler(new MyDefaultHandler());

//读取要解析的xml文件： 
FileReader fileReader = new FileReader(
    new File( "src\\sax\\startelement\\web.xml"));
//指定解析的xml文件：
reader.parse(new InputSource(fileReader));

```

### 创建解析器（方式二）

通过SAXParserFactory、SAXParser、XMLReader完成

```
//使用SAXParserFactory创建SAX解析工厂：
SAXParserFactory spFactory = SAXParserFactory.newInstance();
//通过SAX解析工厂得到解析器对象：
SAXParser sParser = spFactory.newSAXParser();
//通过解析器对象得到一个XML的读取器：
XMLReader xmlReader = sp.getXMLReader();
//设置读取器的事件处理器：
xmlReader.setContentHandler(new MyDefaultHandler());
//解析xml文件:
reader.parse(new InputSource(fileReader));
```

### 创建解析器（方式三)

通过SAXParserFactory，SAXParser完成

```
//获取sax解析器的工厂对象：
SAXParserFactory factory = SAXParserFactory.newInstance();
//通过工厂对象 SAXParser创建解析器对象：
SAXParser saxParser = factory.newSAXParser();
//通过解析saxParser的parse方法设定解析的文件和自己定义的事件处理器对象：
saxParser.parse(new File("src//sax//sida.xml"), new MyDefaultHandler());
```

### 创建处理器

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