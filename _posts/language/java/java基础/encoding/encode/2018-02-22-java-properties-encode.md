---
title: 在java中properties文件的编码
tags: [coding]
---

Properties和ResourceBundle使用的解码编码。Properties类继承了Hashtable类

```
Properties extends Hashtable<Object,Object>
```

Properties和ResourceBundle类，他们在读取文件过程中并不允许我们指定解码编码，那么它们采取什么解码方式呢？

1）查看源码

```
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

import org.junit.Test;

/**
 * Unit test for simple App.
 * 测试
 */
public class AppTest{
    @Test
    public void testDecode() throws IOException {
        Properties properties=new Properties();
        //查看load方法能够查询解析文档时使用的是ISO8859-1
        properties.load(new FileInputStream("E:/test.properties"));
        //获取属性值
        properties.getProperty("test");
    }
}
```

查看源码后发现都是采用iso-8859-1编码来解码的。这样的话我们也不难理解我们写的properties文件为什么都是iso-8859-1的了。因为采取任何一个别的编码都将产生乱码。

2）native2ascii工具

因为iso-8859-1编码是没有中文的，所以我们输入的中文要转换为unicode，通常我们使用插件来完成，也可以使用jdk自带的native2ascii工具。