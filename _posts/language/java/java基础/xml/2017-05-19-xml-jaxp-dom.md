---
title: xml文档的解析--JAXP以dom方式解析
tags: [xml]
---

参考：http://blog.csdn.net/redarmy_chen/article/details/12912065

### 涉及类

DocumentBuilderFactory，DocumentBuilder，Document

1）DocumentBuilderFactory类

javax.xml.parsers包中的DocumentBuilderFactory用于创建DOM模式的解析器对象，DocumentBuilderFactory是一个抽象工厂类，它不能直接实例化，但该类提供了一个newInstance方法 ，这个方法会根据本地平台默认安装的解析器，自动创建一个工厂的对象并返回。

调用DocumentBuilderFactory.newInstance()方法得到创建DOM解析器的工厂。

2）DocumentBuilder类

调用工厂对象（DocumentBuilderFactory）的newDocumentBuilder方法得到DOM解析器对象

3）Document类

调用DOM解析器对象（DocumentBuilder）的parse方法解析XML文档，得到代表整个文档的Document对象，进行可以利用DOM特性对整个XML文档进行操作了。

4）Element类

使用Document对象获取根节点

```
Element root = document.getDocumentElement(); 
```

### 节点遍历

1）节点（Element）类型：

Node接口中定义了节点类型，如Node.ELEMENT_NODE,Node.TEXT_NODE

```
if (node.getNodeType() == Node.ELEMENT_NODE) {
    Element element = (Element) node;
}
```

注：Element接口继承自Node接口（org.w3c.dom包）

2）获取子节点：

```
NodeList listnode = element.getChildNodes();
for (int j = 0; j < listnode.getLength(); j++) 
{ 
    Node nd = listnode.item(j);
    nd.getNodeName();//节点名
    nd.getNodeType();//节点类型
    nd.getNodeValue();//节点值
}
```


3）获取节点属性:

```
NamedNodeMap namenm = element.getAttributes();
for(int k=0;k<namenm.getLength();k++){  
    Attr attr = (Attr) namenm.item(k);
    attr.getNodeName();//属性名
    attr.getNodeValue();//属性值
    attr.getNodeType();//属性类型
}
```

注：getAttributes方法位于Node接口中。Attr接口继承自Node接口。

### 常用操作

1）根据节点名称获取节点

```
NodeList nodelist = document.getElementsByTagName("teacher");
```

2）删除节点

通过父节点调用removeChild(node)把子节点给删除掉

```
Node node1 = node.getParentNode().removeChild(node); 
```

3）增加节点:

```
Element nd = document.createElement("student");
nd.appendChild(document.createTextNode("陈红军"));//创建文本节点。

node.appendChild(nd);//追加到node节点的下面。
```

### 理解节点的含义

标签内部的数据都是节点的形式组织的，如文本值也是一个节点。

```
public class NodeParser {
    public static void main(String[] args)throws Exception {
        String xml="<class></class>";
        StringReader stringReader=new StringReader(xml);
        DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
        DocumentBuilder builder=factory.newDocumentBuilder();
        Document document=builder.parse(new InputSource(stringReader));
        
        Element nd=document.createElement("Student");
        nd.appendChild(document.createTextNode("陈红军"));
        
        Node node=document.getDocumentElement();//根节点
        node.appendChild(nd);//追加节点
        
        //输出
        TransformerFactory transformerFactory = TransformerFactory.newInstance();
        Transformer transformer = transformerFactory.newTransformer();
        transformer.transform(new DOMSource(document), new StreamResult(System.out));
    }
}
```

输出内容：

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<class>
    <Student>陈红军</Student>
</class>
```

遍历时文本节点可以通过getTextContent直接输出，不用在访问文本节点

```
public class NodeParser {
    public static void main(String[] args)throws Exception {
        String xml="<class><Student>陈红军</Student></class>";
        StringReader stringReader=new StringReader(xml);
        DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
        DocumentBuilder builder=factory.newDocumentBuilder();
        Document document=builder.parse(new InputSource(stringReader));
        
        NodeList nodeList=document.getElementsByTagName("Student");
        if(null!=nodeList){
            for(int i=0;i<nodeList.getLength();i++){
                Node node=nodeList.item(i);
                if(Node.ELEMENT_NODE==node.getNodeType()){
                    System.out.println("element");
                    System.out.println(node.getNodeValue());
                    System.out.println(node.getTextContent());//文本值
                    NodeList children=node.getChildNodes();
                    Node childNode=children.item(0);
                    if(Node.TEXT_NODE==childNode.getNodeType()){
                        System.out.println(childNode.getTextContent());//文本值
                        System.out.println(childNode.getNodeValue());//文本值
                    }
                }
            }
        }
    }
}
```

#### 输出xml文档

Transformer类用于把代表XML文件的Document对象转换为某种格式后进行输出

```
TransformerFactory transformerFactory = TransformerFactory.newInstance();
Transformer transformer = transformerFactory.newTransformer();
transformer.transform(new DOMSource(document), new StreamResult(new File("src//c.xml")));
```

注：上面的输出会在一行中显示，

格式化输出

```
transformer.setOutputProperty(OutputKeys.INDENT,"yes");
```

注：格式化后会多行显示，增加缩进