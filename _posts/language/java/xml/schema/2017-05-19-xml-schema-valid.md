---
title: schema校验
tags: [xml]
---

校验过程是由使用JAXP（Java API for XMLProcessing）提供的类和即可，即jdk内置的处理xml的类。其中SchemaFactory位于javax.xml.validation包下。

参考：http://lavasoft.blog.51cto.com/62575/97597/

参考2：http://www.w3school.com.cn/schema/schema_example.asp

### 校验原理

校验的原理是通过读取解析XML的时候设置校验的XSD和校验错误处理器。

1）创建校验文档xsd

```
<?xml version="1.0" encoding="UTF-8"?>
 <!--
  schema 是根元素

  xmlns:xs="http://www.w3.org/2001/XMLSchema"指明了在schema中使用的元素和数据种类来自http://www.w3.org/2001/XMLSchema名称空间（namespace）。
  同时也指定了来自"http://www.w3.org/2001/XMLSchema"名称空间（namespace）的元素和数据种类必须带前缀"xs:"

  targetNamespace="http://www.zhong.cn"(将全部元素绑定给这个名称空间)，暗示了由这份schema(shiporder, orderperson, shipto, ....)定义的元素来自"http://www.zhong.com"名称空间

  elementFormDefault="qualified" 将所有元素绑定到名称空间 
 -->
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" targetNamespace="http://www.zhong.cn" elementFormDefault="qualified">      
                     
    <!--xs:element  指的是element这个元素来自于xs名称空间-->  
    <!-- 定义一个元素 shiporder -->                   
    <xs:element name="shiporder">
     <!-- 类型是：复合类型（里面包含元素或者属性） -->
     <xs:complexType>
      <!-- 元素要有顺序 -->          
      <xs:sequence>     
       <!-- 定义一个元素 orderperson 类型为：字符串 -->               
       <xs:element name="orderperson" type="xs:string"/>  

       <!-- 定义一个元素 shipto 最少出现1次，最多出现1次  -->         
       <xs:element name="shipto" minOccurs="1" maxOccurs="1"> 
        <!-- shipto元素也是复合类型 -->  
        <xs:complexType> 
         <!-- 元素要有顺序 -->
         <xs:sequence>     
          <!-- 在shipto元素中定义一个元素 name 类型为：字符串 -->
          <xs:element name="name" type="xs:string"/>   
          <xs:element name="address" type="xs:string"/>  
          <xs:element name="city" type="xs:string"/>  
          <xs:element name="country" type="xs:string"/>  
         </xs:sequence>  
        </xs:complexType>  
       </xs:element>  

       <!-- 在shiporder元素中定义一个元素 item 出现次数可以无限次 -->
       <xs:element name="item" maxOccurs="unbounded">    
        <xs:complexType>  
         <xs:sequence>  
          <xs:element name="title" type="xs:string"/>  
          <xs:element name="note" type="xs:string" minOccurs="0"/>  
          <xs:element name="quantity" type="xs:positiveInteger"/>  
          <xs:element name="price" type="xs:decimal"/>  
         </xs:sequence>  
        </xs:complexType>  
       </xs:element>  
      </xs:sequence>  
      <xs:attribute name="orderid" type="xs:string" use="required"/>  
     </xs:complexType>  
    </xs:element>   
</xs:schema>
```

2）创建xml文档

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- xmlns指定默认的命名空间，xsi:schemaLocation指定命名空间和xsd文档位置 -->
<shiporder orderid="889923" 
    xmlns="http://www.zhong.cn"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.zhong.cn shiporder.xsd">
 <orderperson>George Bush</orderperson>
 <shipto>
  <name>John Adams</name>
  <address>Oxford Street</address>
  <city>London</city>
  <country>UK</country>
 </shipto>
 <item>
  <title>Empire Burlesque</title>
  <note>Special Edition</note>
  <quantity>1</quantity>
  <price>10.90</price>
 </item>
 <item>
  <title>Hide your heart</title>
  <quantity>1</quantity>
  <price>9.90</price>
 </item>
</shiporder>
```

3）使用javax提供的校验方式校验

```
import javax.xml.XMLConstants;
import javax.xml.transform.stream.StreamSource;
import javax.xml.validation.Schema;
import javax.xml.validation.SchemaFactory;
import javax.xml.validation.Validator;

import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    
    @Test
    public void testValidSchema()throws Exception {
        SchemaFactory schemaFactory=SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
        Schema schema=schemaFactory.newSchema(new StreamSource(AppTest.class.getResourceAsStream("/shiporder.xsd")));
        Validator validator=schema.newValidator();
        validator.validate(new StreamSource(AppTest.class.getResourceAsStream("/shiporder.xml")));
    }
}
```

4）如果校验不通过会抛出异常：

```
Exception in thread "main" org.xml.sax.SAXParseException
```

5）常见的错误

```
org.xml.sax.SAXParseException; lineNumber: 3; columnNumber: 138; cvc-elt.1: 找不到元素 'shiporder' 的声明。
```

主要原因可能是在xml中没有使用xmlns声明命名空间

3、DOM4j结合java xml api使用XSD来校验一个xml有效性