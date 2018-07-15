---
title: jxls整合到spring mvc中
tags: [jxls]
---

参考：https://blog.csdn.net/zjl103/article/details/49666101

1）依赖引入

```
<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls</artifactId>
    <version>2.4.2</version>
</dependency>

<dependency>
    <groupId>org.jxls</groupId>
    <artifactId>jxls-poi</artifactId>
    <version>1.0.13</version>
</dependency>
```

2）新建类继承AbstractView

renderMergedOutputModel方法用来将模型数据导出到excel文档中。

```
package com.ipph.migratecore.controller;

import java.io.InputStream;
import java.util.Base64;
import java.util.Map;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.jxls.common.Context;
import org.jxls.util.JxlsHelper;
import org.springframework.web.servlet.view.AbstractView;

public class JxlsExcelView extends AbstractView{
    
    private static final String CONTENT_TYPE = "application/vnd.ms-excel";

    private String templatePath;
    private String exportFileName;

    /**
     * @param templatePath   模版相对于当前classpath路径
     * @param exportFileName 导出文件名
     */
    public JxlsExcelView(String templatePath, String exportFileName) {
        this.templatePath = templatePath;
        this.exportFileName = exportFileName;
        //设置文档内容
        setContentType(CONTENT_TYPE);
    }
    /**
     * 输出文档
     * @throws Exception 
     */
    @Override
    protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        Context context = new Context(model);
        response.setCharacterEncoding("utf-8");
        response.setContentType(getContentType());
        response.setHeader("content-disposition","attachment;filename=" + getFileName(request.getHeader("User-Agent").toUpperCase(), exportFileName));
        ServletOutputStream os = response.getOutputStream();
        //try-with resource方式处理流资源
        try( InputStream is = this.getClass().getResourceAsStream(templatePath)){
            //到处excel文档
            JxlsHelper.getInstance().processTemplate(is, os, context);
        }
    }
    /**
     * 获取文件名
     * @param browser
     * @param fileName
     * @return
     * @throws Exception
     */
    public static String getFileName(String browser,String fileName)throws Exception{
        if(browser.indexOf("MSIE")!=-1){//IE浏览器
            fileName = new String((fileName+".xls").getBytes("gb2312"), "ISO8859-1");
        }else if(browser.indexOf("FIREFOX")!=-1){//google,火狐浏览器，会自动添加文件扩展名。
            fileName = "=?UTF-8?B?" + (new String (Base64.getEncoder().encode(fileName.getBytes("UTF-8")))) + "?=";  //火狐文件名空格被截断问题
        }else{
            fileName = new String((fileName+".xls").getBytes("gb2312"), "ISO8859-1");
        }
        return fileName;
    }

}
```

3）controller类方法

将数据封装成一个Map中，然后传递到上面的View对象中。

```
/**
 * 导出数据
 * @param batchLogId
 * @param tableId
 * @param size
 * @param page
 * @param request
 * @return
 */
@RequestMapping("/table/success/export/{batchLogId}/{tableId}")
public ModelAndView exportSuccess(@PathVariable("batchLogId")Long batchLogId,
        @PathVariable("tableId")Long tableId,
        @RequestParam(value="size",defaultValue="20")int size,
        @RequestParam(value="page",defaultValue="0")int page,
        HttpServletRequest request,HttpServletResponse response) {
    
    Pageable pageable=PageRequest.of(page, size);
    
    List<LogModel> tableLogList=logService.getSuccessLogs(batchLogId,tableId,pageable);
    Map<String, Object> model=new HashMap<>();
    
    model.put("tableLogList", tableLogList);
    
    return new ModelAndView(new JxlsExcelView("/export/logexport.xls", "导出执行日志"),model);
}
```

4）创建模板

在单元格中添加批注，批注内容为jxls语法相关的命令。设置完成批注后，只需在excel单元格中填写相应的变量占位符即可。

```
# 定义处理模板的区域
jx:area(lastCell="E2")

# 使用for循环将数据集合设置到excel文档中
jx:each(items="tableLogList" var="log" lastCell="E2")

# 占位符变量，如在单元格A2中设置Log对象的tablename属性
${log.tablename}
```

![](/images/java_structure/jxls/jxls-template.png)