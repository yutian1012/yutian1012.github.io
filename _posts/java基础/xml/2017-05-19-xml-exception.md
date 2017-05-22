---
title: xml解析常见异常信息
tags: [xml]
---

### 忽略DTD校验

参考：http://blog.csdn.net/lan861698789/article/details/20492661

xml文件的校验信息

```
<!DOCTYPE dmh SYSTEM "200602.dtd">
```

解析时的错误信息：系统找不到DTD文件

dom4j的解决方案

```
SAXReader saxReader=new SAXReader();
saxReader.setValidation(false);//不校验DTD或schema
Document document=saxReader.read(new File(xmlPath));
```

注：虽然设置了不校验，但是还是会去加载dtd文件，因此，可以定义一个类，该类中返回空的校验信息

```
import java.io.ByteArrayInputStream;
import java.io.IOException;

import org.xml.sax.EntityResolver;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
/**
 * 忽略dtd校验，即返回空的文件
 */
public class IgnoreDTDEntityResolver implements EntityResolver {

    @Override
    public InputSource resolveEntity(String publicId, String systemId)
      throws SAXException, IOException {
        return new InputSource(new ByteArrayInputStream("<?xml version='1.0' encoding='UTF-8'?>".getBytes()));
    }
}
```

配置该视图类的解析

```
SAXReader saxReader=new SAXReader();
saxReader.setValidation(false);//不校验DTD或schema
saxReader.setEntityResolver(new IgnoreDTDEntityResolver());
Document document=saxReader.read(new File(xmlPath));
```