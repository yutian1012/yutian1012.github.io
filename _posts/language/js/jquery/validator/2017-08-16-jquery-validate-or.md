---
title: 两个空间保证至少有一个必填
tags: [jquery]
---

页面中存在两个控件，在提交表单时至少保证有一个控件必填。

1）页面

假设页面有两个控件（name和idcard），现在要求这两个控件至少有一个是必填的。

```
<form id="signupForm" method="post" action="">
    <input id="name" name="name" value="">
    <input id="idcard" name="idcard" value="">
    ...
</form>
```

2）设置校验

添加校验方法

```
jQuery.validator.addMethod("check", function() {
    return $("#idcard").val()!="" || $("#name").val()!="";
}, "姓名和身份证号必须一个有值！");
```

使用定义的校验方法

```
$( "#signupForm" ).validate( {
    rules: {
        idcard: {
            check:true
        },
        name: {
            check:true
        }
    },
    messages: {
        /*name: {
            check: "姓名必须填写!"
        }*/
    }
});
```

注：这里的messages信息可以不会填写，因为addMethod方法中提供了一个默认的值，如果在messages中提供了name控件的check校验输出信息，将会覆盖掉方法提供的默认值。