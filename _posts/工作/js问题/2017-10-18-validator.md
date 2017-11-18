---
title: 系统js校验问题
tags: [work]
---

1）富文本框的必填效果无法显示

js校验使用的是jquery validator来实现的，富文本框使用kindeditor，对于普通的input，必填校验在未填写时，会有红色边框提示，而在富文本框中却无法显示出来。

首先，jquery validator不校验富文本框的原因是，富文本框将textarea包装起来了，实际存储数据的textarea被设置为hidden，而jquery配置时会默认将hidden控件的校验信息忽略掉。

```
//jquery validator屏蔽了hidden控件的校验
$.extend($.validator.defaults,{ignore:""});
```

注：修改ignore为空后，jquery就能校验相应的textarea信息了