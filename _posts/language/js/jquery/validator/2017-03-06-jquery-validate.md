---
title: jquery-validate浏览器端校验
tags: [jquery]
---

### 1. 失去焦点校验

设置失去焦点使校验onfocusout:true

```
$("#signForm").validate({onfocusout:true,rules: {...},messages: {...}});
```

报错TypeError: e[d].call is not a function

解决方法：

```
$("#signForm").validate(
    {
        onfocusout: function(element) { $(element).valid(); },
        rules: {...},
        messages: {...}
    }
);
```

参考：http://www.steveluo.name/jquery-validation-onfocusout-onkeyup-error/

参考：http://www.runoob.com/jquery/jquery-plugin-validate.html

