---
title: dom4j结合xpath解析xml
tags: [xml]
---

### 获取单个子节点

多个节点值的获取可以直接写路径（全路径），但若想获取单个的读取，则必须在父路径后追加获取的节点位置。

1）新建maven项目

引入项目依赖

```
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.0</version>
</dependency>
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
```

2）xml文件内容

新建test.xml文档

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

3）获取第一个节点staff下的name

```
import java.util.List;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testXmlXpath() throws DocumentException {
        SAXReader reader = new SAXReader(); 
        Document document = reader.read(AppTest.class.getResource("/test.xml"));
        List<Node> list=document.selectNodes("/department/staff/name");
        System.out.println(list.size());//3
        //在name路径下筛选时不能有效的排除
        list=document.selectNodes("/department/staff/name[1]");
        System.out.println(list.size());//3
        //父路径后追加获取的节点位置，通过传递[1],
        list=document.selectNodes("/department/staff[1]/name");
        System.out.println(list.size());//1第一个节点
    }
}
```

注意：xpath节点是的访问层级

### 从节点下继续获取节点

节点下的查找不可以在路径前添加斜杠/

```
@Test
public void testXmlXpathLevel() throws DocumentException {
    SAXReader reader = new SAXReader(); 
    Document document = reader.read(AppTest.class.getResource("/test.xml"));

    List<Node> list=document.selectNodes("/department/staff[1]");
    Node node=list.get(0);
    list=node.selectNodes("/name");//找不到节点值，需要去掉前面的斜杆
    System.out.println(list.size());//0
    list=node.selectNodes("name");
    System.out.println(list.size());//1
    System.out.println(list.get(0).getText());
}
```