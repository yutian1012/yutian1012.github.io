---
title: dom4j接口
tags: [xml]
---

Dom4j是一个简单、灵活的开放源代码的库。Dom4j是由早期开发JDOM的人分离出来而后独立开发的。与JDOM不同的是，dom4j使用接口和抽象基类，虽然Dom4j的API相对要复杂一些，但它提供了比JDOM更好的灵活性。

注：使用Dom4j开发，需下载dom4j相应的jar文件。

1）获取Docuemnt对象

DOM4j中，获得Document对象的方式有三种；

a.读取XML文件,获得document对象

```
SAXReader reader = newSAXReader();
Document document = reader.read(new File("input.xml"));
```
 
b.解析XML形式的文本,得到document对象.

```
String text = "<members></members>";
Document document = DocumentHelper.parseText(text);
```
 
c.主动创建空document对象.

```
Document document = DocumentHelper.createDocument();  
```

2）Element对象操作

a.创建Element对象

```
# 创建Element对象
Element root =document.addElement("members");

# 添加一个指定名称的子元素
Element childEle = parentEle.addElement("书名");
```

b.获取Element对象

```
# 获取根Element对象
Element root = document.getRootElement();

# 获取某个元素的指定名称的第一个子节点
Elementelement = element.element("书名");

# 获取某个元素的指定名称的所有子元素的集合
List list = element.elements("书名");

# 
```

c.删除Element对象

```
# 删除某个元素指定的子元素
parentEle.remove(childEle);
```

3）属性的操作

a.获取属性及属性值

```
# 获取某个元素的指定名称的属性对象
Attribute attr = element.attribute("id");

# 获取元素属性值
String id = element.attributeValue("id");
```

b.给元素添加属性或更新其值

```
Attribute attr =element.addAttribute("id","123");
```

c.删除某个元素的指定属性

```
element.remove(attribute);
```

4）文本Text操作

a.获取某个元素的文本内容

```
String text = element.getText();
```

b.给某个元素添加或更新文本内容

```
element.setText("Tom");
```