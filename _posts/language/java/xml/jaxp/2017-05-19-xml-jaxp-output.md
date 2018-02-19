---
title: java格式化输出xml文档
tags: [xml]
---

Transformer类用于把代表XML文件的Document对象转换为某种格式后进行输出

1）格式化化输出需要设置相应的属性

```
transformer.setOutputProperty(OutputKeys.INDENT,"yes");
```

2）输出实例

```
import java.io.StringReader;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;

import org.junit.Test;
import org.w3c.dom.Document;
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
        
        //输出
        TransformerFactory transformerFactory = TransformerFactory.newInstance();
        Transformer transformer = transformerFactory.newTransformer();
        transformer.setOutputProperty(OutputKeys.INDENT, "yes");  
        transformer.setOutputProperty(OutputKeys.CDATA_SECTION_ELEMENTS, "yes");
        transformer.transform(new DOMSource(document), new StreamResult(System.out));
    }
}
```