---
title: dom4j输出xml文档
tags: [xml]
---

Dom4j包中提供了OutputFormat用来格式化输出数据，设置编码等操作，提供了XMLWriter用于向相关的流中输出数据。

```
import org.dom4j.Document;
import org.dom4j.DocumentHelper;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.XMLWriter;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testStr2Dom() throws Exception {
        String xmlStr = "<employee><empname>@残缺的孤独</empname><age>25</age><title>软件开发工程师</title></employee>";  
        Document document = DocumentHelper.parseText(xmlStr);
        OutputFormat format =OutputFormat.createPrettyPrint();// 指定XML编码     
        format.setEncoding( "utf-8"); 
        XMLWriter writer = new XMLWriter(new FileWriter( "E:/output.xml"),format);
        writer.write(document);
        writer.close();
    }
}
```