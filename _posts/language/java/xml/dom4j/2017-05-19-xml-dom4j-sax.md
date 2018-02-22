---
title: dom4j以sax方式解析xml文档
tags: [xml]
---

参考：http://blog.csdn.net/java2k/article/details/6095777

DOM4J利用SAXReader类来读取文件，然后为SAXReader对象设置元素处理解析器ElementHandler，有doStart和doEnd两个方法，分别在遇到元素的开始符号和结束符号时候触发。只需在相应的方法内实现自己的业务逻辑即可。

1）新建testinfo.xml文档

```
<?xml version="1.0" encoding="UTF-8"?>  
<TESTINFO version="1.0">  
    <UPDATEINFO>  
        <PARAM_TIMESTAMP>2010-12-22 09:54:09</PARAM_TIMESTAMP>  
        <CITYINFO_LIST>  
            <CITYINFO City="591" Name="XYZ" No="334" />  
            <CITYINFO City="591" Name="BJD" No="335" />  
            <CITYINFO City="591" Name="UUS" No="390" />  
        </CITYINFO_LIST>  
    </UPDATEINFO>  
</TESTINFO>  
```

2）创建处理类，继承ElementHandler

```
import org.dom4j.Element;
import org.dom4j.ElementHandler;
import org.dom4j.ElementPath;

public class MySaxHandler implements ElementHandler {
    /**
     * 解析到元素的结尾标签时触发
     */
    public void onEnd(ElementPath ep) {  
        Element e = ep.getCurrent(); //获得当前节点  
        if(e.getName().equals("PARAM_TIMESTAMP"))  
            System.out.println("解析到时间:"+e.getText());  
        else if(e.getName().equals("CITYINFO")){  
            System.out.printf("解析到CITYINFO,属性值为：%s,%s,%s/n", e  
                    .attributeValue("City"), e.attributeValue("Name"), e  
                    .attributeValue("No"));  
        }  
        //从内存中移去
        e.detach();   
    }  
    /**
     * 解析到元素的开始标签时触发
     */
    public void onStart(ElementPath ep) {  
    }  
}
```

3）测试解析

```
import java.io.InputStream;

import org.dom4j.DocumentException;
import org.dom4j.io.SAXReader;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testDom4jSax() throws DocumentException {
        InputStream is = MySaxHandler.class.getResourceAsStream("/testinfo.xml");   
        SAXReader reader = new SAXReader();
        //设置解析xml的handler
        reader.setDefaultHandler(new MySaxHandler());  
        reader.read(is);  
    }
}
```