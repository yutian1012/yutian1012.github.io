---
title: 流程表单导出word
tags: [bpmx]
---

1）导出word按钮事件

```
//导出word文档
function downloadToWord(runId){
    var cl=$(".panel-body").clone();//获取导出的页面内容
    var form=cl.children();
    handFile(form);//处理附件控件，只显示文件名

    //生成表单数据提交到后台处理导出word
    var frm=new com.hotent.form.Form();
    //查看processRunController类的downloadToWord方法
    frm.creatForm("bpmPreview",
        __ctx+"/platform/bpm/processRun/downloadToWord.ht");
    frm.addFormEl("form",cl.html());
    frm.addFormEl("runId",runId);
    frm.submit();
}
```

2）查看word文档的生成

在processRunController中的downloadToWord方法中调用了genFile方法

```
/**
 * 生成文件。
 * 
 * @param content
 * @param fileName
 */
@SuppressWarnings("unused")
private void genFile(String content, String fileName) {
    ByteArrayInputStream bais = null;
    FileOutputStream fos = null;
    POIFSFileSystem poifs = null;//POI接口
    byte b[];
    try {
        //替换掉所有的script脚本
        String resultString = content.replaceAll("(?si)<script[\\s]*[^>]*>.*?</script>", "");
        b = resultString.getBytes("gbk");
        bais = new ByteArrayInputStream(b);
        //生成word文档
        poifs = new POIFSFileSystem();
        DirectoryEntry directory = poifs.getRoot();
        DocumentEntry documentEntry = directory.createDocument("WordDocument", bais);

        fos = new FileOutputStream(fileName);
        poifs.writeFilesystem(fos);

    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            bais.close();
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

3）下载文档

```
private void donwload(String filePath, HttpServletResponse response, String fileName, boolean isIE) {
    String filedisplay = fileName + ".doc";

    //设置response的ContentType类型
    response.reset();
    response.setContentType("application/x-msdownload;charset=utf-8");
    response.setCharacterEncoding("UTF-8");

    try {
        if (!isIE) {//判断浏览器是否是IE
            //使用Base64编码方式
            String enableFileName = "=?UTF-8?B?" + (new String(Base64.getBase64(filedisplay))) + "?=";

            response.setHeader("Content-Disposition", "attachment;filename=" + enableFileName);
        } else {
            filedisplay = URLEncoder.encode(filedisplay, "utf-8");
            
            response.addHeader("Content-Disposition", "attachment;filename=" + filedisplay);
        }

        OutputStream sops = response.getOutputStream();
        InputStream ips = new FileInputStream(filePath);
        byte[] content = new byte[ips.available()];
        int length = -1;
        while ((length = ips.read(content)) != -1) {
            sops.write(content, 0, length); // 读入流,保存在Byte数组中
        }
        ips.close();
        sops.flush();
        sops.close();
    } catch (UnsupportedEncodingException e) {
        log.debug(e.getMessage(), e);

    } catch (IOException e) {
        log.debug(e.getMessage(), e);
    }
}
```

4）后台判断浏览器是否是IE

```
String agent = (String) request.getHeader("USER-AGENT");
boolean isIe = true;
if (agent != null && agent.indexOf("MSIE") == -1 && agent.indexOf("Trident") == -1) {
    isIe = false;
}
```

5）文件下载中文乱码问题

参考：http://blog.csdn.net/baidu_18607183/article/details/44002987

在IE下，filename必须保留扩展名部分（xxx.doc）, 文件名中的中文才能正确解码, 否则可能不识别%20(空格), 甚至在ie6下全部都是未解码的格式(%xx)。

另外原始的空格使用urlEncode编码后转换为+号（基于历史原因）,而ie解析时会直接作为+号处理, 因此需要手工替换一下这个特殊字符。

```
URLEncoder.encode("中文+   en", "UTF-8").replaceAll("\\+", "%20");
```

safari相对比较变态，filename部分只能使用utf-8的原始字节，而http的header必须使用单字节编码的字符串，因此需要将原始内容重新构造为iso-8859-1单字节编码的字符串。

```
new String(filename.getBytes("UTF-8"),"ISO8859-1") 
```

参考：http://java-xp.iteye.com/blog/903048

浏览器能正确识别的编码格式，只要按照这样的编码来设置对应的Content-Disposition，那么应该就不会出现中文文件名的乱码问题了。

首先，Content-Disposition值可以有以下几种编码格式

1. 直接urlencode：

```
Content-Disposition:attachment;filename="struts2.0%E4%B8%AD%E6%96%87%E6%95%99%E7%A8%8B.chm"
```

2. Base64编码：

```
Content-Disposition:attachment;filename="=?UTF8?B?c3RydXRzMi4w5Lit5paH5pWZ56iLLmNobQ==?="
```

3. RFC2231规定的标准：

```
Content-Disposition: attachment; filename*=UTF-8''%E5%9B%9E%E6%89%A7.msg
```

4. 直接ISO编码的文件名：

```
Content-Disposition: attachment;filename="测试.txt"
```

然后，各浏览器支持的对应编码格式为：

```
IE浏览器，采用URLEncoder编码
Opera浏览器，采用filename*方式
Safari浏览器，采用ISO编码的中文输出
Chrome浏览器，采用Base64编码或ISO编码的中文输出
FireFox浏览器，采用Base64或filename*或ISO编码的中文输出 
```

实现代码：

```
//默认使用IE的方式进行编码
new_filename = URLEncoder.encode(filename, "UTF8"); 

rtn = "filename=\"" + new_filename + "\""; 
if (userAgent != null){
    userAgent = userAgent.toLowerCase(); 
    
    // IE浏览器，只能采用URLEncoder编码 
    if (userAgent.indexOf("msie") != -1){ 
        rtn = "filename=\"" + new_filename + "\""; 
    }
    // Opera浏览器只能采用filename* 
    else if (userAgent.indexOf("opera") != -1){ 
        rtn = "filename*=UTF-8''" + new_filename; 
    }
    // Safari浏览器，只能采用ISO编码的中文输出 
    else if (userAgent.indexOf("safari") != -1 ){ 
        rtn = "filename=\"" + 
            new String(filename.getBytes("UTF-8"),"ISO8859-1") + "\""; 
    }
    // Chrome浏览器，只能采用MimeUtility编码或ISO编码的中文输出 
    else if (userAgent.indexOf("applewebkit") != -1 ){ 
        new_filename = MimeUtility.encodeText(filename, "UTF8", "B"); 
        rtn = "filename=\"" + new_filename + "\""; 
    } 
    // FireFox浏览器，可以使用MimeUtility或filename*或ISO编码的中文输出 
    else if (userAgent.indexOf("mozilla") != -1){ 
        rtn = "filename*=UTF-8''" + new_filename; 
    }
}  
```

注：在javaEE中提供了MimeUtility类，位于javax.mail.internet包中。