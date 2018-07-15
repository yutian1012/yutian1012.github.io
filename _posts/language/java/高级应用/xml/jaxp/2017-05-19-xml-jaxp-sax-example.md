---
title: 使用JAXP以SAX解析xml实例
tags: [xml]
---

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