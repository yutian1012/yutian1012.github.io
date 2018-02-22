---
title: dom4j解析xml
tags: [xml]
---

参考：http://blog.csdn.net/yyywyr/article/details/38361367

### Document与String的转换

1）新建test.xml文档

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<department name="娱乐圈">
    <staff smoker="false">
        <age>30</age>
        <birthDay>1963-12-16</birthDay>
        <name>周杰伦</name>
    </staff>
    <staff smoker="false">
        <age>28</age>
        <birthDay>1962-11-13</birthDay>
        <name>周笔畅</name>
    </staff>
    <staff smoker="true">
        <age>40</age>
        <birthDay>1966-10-15</birthDay>
        <name>周星驰</name>
    </staff>
</department>
```

2）document对象转字符串。

```
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.io.SAXReader;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    
    @Test
    public void testDom2Str() throws DocumentException {
        //创建SAXReader对象  
        SAXReader reader = new SAXReader();  
        //读取文件 转换成Document  
         Document document = reader.read(AppTest.class.getResource("/test.xml")); 
        //document转换为String字符串  
        String documentStr = document.asXML();
        System.out.println(documentStr);
    }
}
```

2）字符串转document对象

使用dom4j提供的DocumentHelper来获取document对象

```
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.DocumentHelper;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testStr2Dom() throws DocumentException {
        String xmlStr = "<employee><empname>@残缺的孤独</empname><age>25</age><title>软件开发工程师</title></employee>";  
        Document document = DocumentHelper.parseText(xmlStr);
        System.out.println(document.asXML());
    }
}
```

dom4j以dom的形式处理xml

参考：http://blog.csdn.net/redarmy_chen/article/details/12969219
参考2：http://blog.csdn.net/mark_lq/article/details/45040731