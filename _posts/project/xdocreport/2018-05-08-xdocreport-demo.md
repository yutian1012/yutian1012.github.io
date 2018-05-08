---
title: xdocreport 实例
tags: [xdocreport]
---

参考：https://blog.csdn.net/tototuzuoquan/article/details/71273986

1）创建一个maven项目

添加依赖

```
<dependencies>
    <dependency>
        <groupId>fr.opensagres.xdocreport</groupId>
        <artifactId>fr.opensagres.xdocreport.template.freemarker</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>fr.opensagres.xdocreport</groupId>
        <artifactId>fr.opensagres.xdocreport.document.docx</artifactId>
        <version>2.0.1</version>
    </dependency>
</dependencies>
```

2）创建word模板

a.新建word；

b.替换简单动态变量：

Ctrl+F9插入一个空域，右键--编辑域--选择MergeField--编辑域代码。

![](/images/project/xdocreport/word/word-field.png)

![](/images/project/xdocreport/word/word-field-edit.png)

相同的方式设置name和age字段，并保存模板文件为Test.docx

3）编写代码输出文本

```
package org.ipph.demo;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;

import fr.opensagres.xdocreport.document.IXDocReport;
import fr.opensagres.xdocreport.document.registry.XDocReportRegistry;
import fr.opensagres.xdocreport.template.IContext;
import fr.opensagres.xdocreport.template.TemplateEngineKind;

public class TestFreemark2Doc {
    
    private static String filePath="D:/xdoc/";
    
    public static void main(String[] args)throws Exception {
        InputStream in = TestFreemark2Doc.class.getResourceAsStream("/template/Test.docx");
        
        //第一步：创建report
        IXDocReport report = XDocReportRegistry.getRegistry()
                .loadReport(in, TemplateEngineKind.Freemarker);
        
        //第二步：从report中获取上下文环境，并注册内容对象
        IContext context = report.createContext();
        context.put("name", "张三");
        context.put("age",33);
        
        //第三步：输出文档镀锡
        OutputStream out = new FileOutputStream(new File(filePath+"Test_out.docx"));
        report.process(context, out);
    }
}
```

注：将文档输出到D:/xdoc目录下。