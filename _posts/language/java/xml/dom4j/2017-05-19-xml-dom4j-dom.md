---
title: dom4j以dom方式解析xml文档
tags: [xml]
---

参考：http://blog.csdn.net/gui66497/article/details/53105846

dom方式解析xml一般是用来编辑xml文档，如增加，修改，删除节点等操作。

获取Document对象

```
//创建SAXReader对象  
SAXReader reader = new SAXReader();  
//读取文件 转换成Document  
Document document = reader.read(AppTest.class.getResource("/test.xml")); 
```

1）xml文档

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

2）添加节点

调用addAttribute方法添加属性，调用addElement方法添加元素。

```
import java.io.IOException;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    /**
     * 添加节点
     * <staff smoker="true">
     *    <age>40</age>
     *    <birthDay>1966-10-15</birthDay>
     *    <name>周华健</name>
     * </staff>
     * @throws DocumentException
     * @throws IOException 
     */
    @Test
    public void testDom4jDomAdd() throws Exception {
        //创建SAXReader对象  
        SAXReader reader = new SAXReader();  
        //读取文件 转换成Document  
        Document document = reader.read(AppTest.class.getResource("/test.xml")); 
        //获取根节点
        Element ele=document.getRootElement();
        //新增节点
        Element staffElement=ele.addElement("staff");
        //设置节点的属性
        staffElement.addAttribute("smoker", "true");
        //添加age节点
        Element ageElement=staffElement.addElement("age");
        //设置文本值
        ageElement.setText("40");
        
        Element birthDayElement=staffElement.addElement("birthDay");
        birthDayElement.setText("1966-10-15");
        
        Element nameElement=staffElement.addElement("name");
        nameElement.setText("周华健");
        
        //输出xml内容
        OutputFormat format = OutputFormat.createPrettyPrint();  
        format.setEncoding("utf-8");  
        XMLWriter writer = new XMLWriter(System.out,format);  
        writer.write(document);  
        writer.close();  
        System.out.println("执行结束！");
    }
}
```

2）修改节点

修改周星驰节点的age文本值，以及smoker为false。调用setValue方法设置属性值，调用setText方法设置节点文本值。

```
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    /**
     * 修改节点
     * 修改周星驰节点的age文本值，以及smoker为false
     */
    @Test
    public void testDom4jDomUpdate() throws Exception {
        //创建SAXReader对象  
        SAXReader reader = new SAXReader();  
        //读取文件 转换成Document  
        Document document = reader.read(AppTest.class.getResource("/test.xml")); 
        
        List<Element> staffElements = document.getRootElement().elements("staff");
        if(null!=staffElements&&staffElements.size()>0) {
            for(Element ele:staffElements) {
                if("周星驰".equals(ele.element("name").getText())) {
                    ele.attribute("smoker").setValue("false");;
                    ele.element("age").setText("36");
                    break;
                }
            }
        }
        
        //输出xml内容
        OutputFormat format = OutputFormat.createPrettyPrint();  
        format.setEncoding("utf-8");  
        XMLWriter writer = new XMLWriter(System.out,format);  
        writer.write(document);  
        writer.close();  
        System.out.println("执行结束！");
    }
}
```

3）删除节点

删除周杰伦的smoker属性，并删除周星驰的birthday节点。属性和元素的删除使用detach方法完成。

```
import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    /**
     * 删除节点
     * 删除周杰伦的smoker属性，并删除周星驰的birthDay节点
     */
    @Test
    public void testDom4jDomUpdate() throws Exception {
        //创建SAXReader对象  
        SAXReader reader = new SAXReader();  
        //读取文件 转换成Document  
        Document document = reader.read(AppTest.class.getResource("/test.xml")); 
        
        List<Element> staffElements = document.getRootElement().elements("staff");
        if(null!=staffElements&&staffElements.size()>0) {
            for(Element ele:staffElements) {
                if("周杰伦".equals(ele.element("name").getText())) {
                    ele.attribute("smoker").detach();
                }else if("周星驰".equals(ele.element("name").getText())) {
                    ele.element("birthDay").detach();
                }
            }
        }
        
        //输出xml内容
        OutputFormat format = OutputFormat.createPrettyPrint();  
        format.setEncoding("utf-8");  
        XMLWriter writer = new XMLWriter(System.out,format);  
        writer.write(document);  
        writer.close();  
        System.out.println("执行结束！");
    }
}
```