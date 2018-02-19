---
title: xml文档的解析--JAXP以dom方式解析实例
tags: [xml]
---

Document，Element和Node都位于org.w3c.dom包下。

1）dom方式设置xml元素

标签内部的数据都是节点的形式组织的，如文本值也是一个节点。

```
import java.io.StringReader;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;

import org.junit.Test;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.xml.sax.InputSource;

/**
 * Unit test for simple App.
 */
public class AppTest{
    
    @Test
    public void testJaxp()throws Exception {
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

2）dom解析xml字符串

遍历时文本节点可以通过getTextContent直接输出，不用在访问文本节点

```
import java.io.StringReader;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

import org.junit.Test;
import org.w3c.dom.Document;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;

/**
 * Unit test for simple App.
 */
public class AppTest{
    
    @Test
    public void testJaxp()throws Exception {
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
                    System.out.println(node.getNodeValue());//null
                    System.out.println(node.getTextContent());//文本值
                    NodeList children=node.getChildNodes();
                    Node childNode=children.item(0);
                    if(Node.TEXT_NODE==childNode.getNodeType()){
                        //数值"陈红军"是文本节点
                        System.out.println(childNode.getTextContent());//文本值
                        System.out.println(childNode.getNodeValue());//文本值
                    }
                }
            }
        }
    }
}
```