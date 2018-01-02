---
title: easy-ui中日期字段的回显
tags: [easy-ui]
---

### 问题1：修改数据时，日期显示当前日期

![](/images/js/easy-ui/datetimebox.png)

修改方法，设置字符串转换日期然后设置值。

```
$.extend($.fn.datebox.defaults,{
    parser:function(s){
        var t = Date.parse(s);
        if (!isNaN(t)){
            return new Date(t);
        } else {
            return new Date();
        }
    }
});
```