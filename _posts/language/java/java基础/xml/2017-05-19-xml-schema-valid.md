---
title: schema校验
tags: [xml]
---

参考：http://lavasoft.blog.51cto.com/62575/97597/

### 校验原理

校验的原理是通过读取解析XML的时候设置校验的XSD和校验错误处理器。

### 使用javax提供的校验方式校验

```
public class XmlValidator {
    public static void main(String[] args) throws Exception{
        SchemaFactory schemaFactory=SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
        Schema schema=schemaFactory.newSchema(new StreamSource(XmlValidator.class.getResourceAsStream("/shiporder.xsd")));
        Validator validator=schema.newValidator();
        validator.validate(new StreamSource(XmlValidator.class.getResourceAsStream("/shiporder.xml")));
    }
}
```

注：以上使用的类都是有JAXP（Java API for XMLProcessing）提供，jdk内置的处理xml的类，SchemaFactory位于javax.xml.validation包下。

如果校验不通过会抛出异常：

```
Exception in thread "main" org.xml.sax.SAXParseException
```

3、DOM4j结合java xml api使用XSD来校验一个xml有效性