---
title: dom4j校验xml
tags: [xml]
---

dom4j默认不会使用schema校验xml，通过方法setValidation传递参数true，可实现xml文档的校验。

```
import org.dom4j.Document;
import org.dom4j.io.SAXReader;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testDom4jValidate() throws Exception {
        //创建SAXReader对象  
        SAXReader reader = new SAXReader();  
        //设置是否校验xml文档
        reader.setValidation(true);
        //读取文件 转换成Document  
        Document document = reader.read(AppTest.class.getResource("/shiporder.xml")); 
    }
}
```