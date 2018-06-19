---
title: 文件上传控件的展示页面
tags: [work]
---

文件上传后的页面展示控件，点击文件名时出现兼容性问题。主要是由于

![](/images/work/bpmx/js/fileAttachemntjs.png)

1）控件的设置

```
<div name="div_attachment_container" right="r">
     <div class="attachement"></div>
     <textarea style="display:none" controltype="attachment" name="m:t_idea:attachmentFree" lablename="上传附件" validate="{}" validatable="true">${tIdea.attachmentFree}</textarea>
</div>
```

2）定位

js文件为AttachMent.js，查看该文件的AttachMent.init方法。附件内容显示调用AttachMent.insertHtml方法。

如果要去掉文件链接可修改AttachMent.getHtml方法。

3）清空上传文档

```

```
