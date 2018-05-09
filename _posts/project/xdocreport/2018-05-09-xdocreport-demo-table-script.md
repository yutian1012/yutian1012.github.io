---
title: xdocreport 使用脚本生成表格行数据
tags: [xdocreport]
---

参考：https://github.com/opensagres/xdocreport/wiki/ODTReportingJavaMainListFieldAdvancedTable

xdocreport可以使用FieldsMetadata快速实现多行数据的表格展示。但是这样没有限制了对特殊模板的处理。即没行展示的样式都相同。

### 使用list脚本实现循环展示行

1）创建模板，每个命令都要占一个单独MergeField域

![](/images/project/xdocreport/word/word-table-list-script.png)

a.第一个展示数据的单元格（序号字段）

```
# 第一个mergeField域
@before-row[#list userList as user]
# 第二个mergeField域
${user.xh}
# 第三个mergeField域
@after-row[/#list]
```

b.第二个展示数据的单元格（姓名字段）

```
# 一个mergeField域
${user.username}
```

注：其余字段显示于这里的第二个单元格设置内容相同。

2）程序代码

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

public class TestFreemark2DocListScript {
    
    private static String filePath="D:/xdoc/";
    
    public static void main(String[] args)throws Exception {
        InputStream in = TestFreemark2Doc.class.getResourceAsStream("/template/Test-list-script.docx");
        
        //第一步：创建report
        IXDocReport report = XDocReportRegistry.getRegistry().loadReport(in, TemplateEngineKind.Freemarker);
        
        //第二步：从report中获取上下文环境，并注册内容对象
        IContext context = report.createContext();
        context.put("name", "张三");
        context.put("age",33);
        
        setTableList(report, context);
        
        //第三步：输出文档镀锡
        OutputStream out = new FileOutputStream(new File(filePath+"Test_out.docx"));
        report.process(context, out);
    }
    
    public static void setTableList(IXDocReport report,IContext context){
        
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

### 奇数行和偶数行不同样式

修改样式使word模板的奇数行和偶数行，使不同行显示不用样式。

![](/images/project/xdocreport/word/word-table-list-script2.png)

参考样例中的文档：DocxTableWithoutFieldsMetadataWithFreemarker.docx（需要下载样例源码）

1）第一行设置

a.第一个展示数据的单元格（序号字段）

```
# 第一个mergeField域
@before-row[#list userList as user] [#if user_index % 2 == 0]
# 第二个mergeField域
${user.xh}
# 第三个mergeField域
@after-row[#else]
```

b.第二个展示数据的单元格（姓名字段）

```
# 一个mergeField域
${user.username}
```

注：其余字段显示于这里的第二个单元格设置内容相同。

第二行设置

```
# 第一个mergeField域
${user.xh}
# 第二个mergeField域
@after-row[/#if][/#list]
```

注：其余字段显示于上一行设置内容相同。

2）实例代码

使用上面的代码即可，不需要做任何修改

### 循环展示多个表格

```
# 设置一个mergeField域，标识for循环开始
[#list developers as d]

# 模板设置内容，如果集合中的字段也包含集合，就可以设置表格来展示
Identity
Name : ${d.name}
Last Name : ${d.lastName}
Email : ${d.mail}

# 设置一个mergeField域，标识for循环结束
[/#list]
```