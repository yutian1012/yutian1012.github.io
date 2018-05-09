---
title: xdocreport 输入table行实例
tags: [xdocreport]
---

1）在word模板中创建表格

表格中数据展示的是一行一行的记录。

![](/images/project/xdocreport/word/word-table-list.png)

注：模板中的userList是Context中存放的list对象集合

2）java代码实现

在展示列表数据是需要使用FieldsMetadata类，对字段进行封装。并且设定的参数名要与模板中设置的占位符相匹配。

```
package org.ipph.demo;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import fr.opensagres.xdocreport.document.IXDocReport;
import fr.opensagres.xdocreport.document.registry.XDocReportRegistry;
import fr.opensagres.xdocreport.template.IContext;
import fr.opensagres.xdocreport.template.TemplateEngineKind;
import fr.opensagres.xdocreport.template.formatter.FieldsMetadata;

public class TestFreemark2Doc {
    
    private static String filePath="D:/xdoc/";
    
    public static void main(String[] args)throws Exception {
        InputStream in = TestFreemark2Doc.class.getResourceAsStream("/template/Test.docx");
        
        //第一步：创建report
        IXDocReport report = XDocReportRegistry.getRegistry()
                .loadReport(in, TemplateEngineKind.Freemarker);
        
        //第二步：从report中获取上下文环境，并注册内容对象
        IContext context = report.createContext();
        
        setTableList(report, context);
        
        //第三步：输出文档镀锡
        OutputStream out = new FileOutputStream(new File(filePath+"Test_out.docx"));
        report.process(context, out);
    }
    
    public static void setTableList(IXDocReport report,IContext context){
        
        FieldsMetadata metadata = new FieldsMetadata();
        metadata.addFieldAsList("userList.xh");
        metadata.addFieldAsList("userList.username");
        metadata.addFieldAsList("userList.graduate");
        metadata.addFieldAsList("userList.email");
        report.setFieldsMetadata(metadata);
        
        List<Map<String,Object>> result=new ArrayList<Map<String,Object>>();
        
        for(int i=0;i<5;i++) {
            Map<String,Object> data=new HashMap<String, Object>();
            data.put("xh",i+1);
            data.put("username", "张三"+i);
            data.put("graduate", "硕士"+i);
            data.put("email","test"+i+"@qq.com");
            
            result.add(data);
        }
        
        context.put("userList", result);
    }
}
```

3）对模板进行修改

![](/images/project/xdocreport/word/word-table-list2.png)

左侧固定不动，右侧是多行展示数据集。使用上面的方式也能实现，并且左侧会自动扩展合并多行。

![](/images/project/xdocreport/word/word-table-list2-result.png)

4）针对特殊列表的处理

![](/images/project/xdocreport/word/word-table-special-list.png)

上图展示了部分样式，如何针对这样的列表信息进行数据展示？

编辑模板

![](/images/project/xdocreport/word/word-table-special-list-template.png)

问题，由于生成的代码行位于顶部，没有标题行，造成左侧单元格也被重复生成了，没有进行有效的合并。

![](/images/project/xdocreport/word/word-table-special-list-result.png)

解决方法：

只要for循环不是第一行就不会出现这类问题，即左侧能够很好的合并单元格。因此，可以从集合中抽取出一条来，单独设置值，余下的使用for循环遍历即可。

可能存在的问题是如果集合中只有一条，将该条抽取出来显示到第一行，第二行的for循环由于集合空，没有值则留空，那么会多出一个空记录（可通过创建2个模板来解决单独一条集合记录的问题）。

![](/images/project/xdocreport/word/word-table-special-list-result2.png)