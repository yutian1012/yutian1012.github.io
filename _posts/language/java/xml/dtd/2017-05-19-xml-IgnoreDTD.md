---
title: xml解析常见异常信息
tags: [xml]
---

### SAX方式忽略DTD校验

参考：http://blog.csdn.net/lan861698789/article/details/20492661

1）xml文件的校验信息

```
<!DOCTYPE dmh SYSTEM "200602.dtd">
```

解析时的错误信息：系统找不到DTD文件

2）dom4j的解决方案

```
import java.io.File;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.io.SAXReader;
import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testXml() throws DocumentException {
        SAXReader saxReader=new SAXReader();
        saxReader.setValidation(false);//不校验DTD或schema
        Document document=saxReader.read(new File("E:\\xmltest\\src\\test\\java\\test.xml"));
    }
}
```

注：虽然设置了不校验，但是还是会去加载dtd文件，因此，可以定义一个类，该类中返回空的校验信息

3）设置忽略dtd文件处理类

```
import java.io.ByteArrayInputStream;
import java.io.IOException;

import org.xml.sax.EntityResolver;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
/**
 * 忽略dtd校验，即返回空的文件
 */
public class IgnoreDTDEntityResolver implements EntityResolver{
    public InputSource resolveEntity(String publicId, String systemId)
      throws SAXException, IOException {
        return new InputSource(new ByteArrayInputStream("<?xml version='1.0' encoding='UTF-8'?>".getBytes()));
    }
}
```

4）配置xml解析

```
@Test
public void testXml() throws DocumentException {
    SAXReader saxReader=new SAXReader();
    saxReader.setValidation(false);//不校验DTD或schema
    saxReader.setEntityResolver(new IgnoreDTDEntityResolver());
    Document document=saxReader.read(new File("E:\\work\\workspace\\eclipse-test\\xmltest\\src\\test\\java\\test.xml"));
}
```