---
title: 页面弹框的参数信息
tags: [bpmx]
---

场景：在流程页面中点击沟通按钮，弹出一个对话框，该对话框是一个jsp页面。希望在该jsp页面中获取弹框时设置的参数信息。

1）页面调用

taskToStart.jsp页面（流程待办页面），点击沟通按钮。弹出对话框（taskToStartCommunicate.jsp）

沟通按钮事件

```
//显示沟通意见。
function showTaskCommunication(conf){
    var obj = {data:CustomForm.getData()};
    isTaskEnd(function(){
        if(!conf) conf={};
        //输入子页面
        var url=__ctx + "/platform/bpm/task/toStartCommunicate.ht?taskId=${task.id}";
        
        /*KILLDIALOG*/
        DialogUtil.open({
            height:350,
            width: 500,
            title : '沟通意见',
            url: url, 
            isResize: true,
            //自定义参数
            obj: obj
        });
    })
}
```

注：参数的设置分为两部分，一部分拼接在url后面，一部分在DialogUtil的open方法中传递了js对象。

taskToStartCommunicate.jsp页面接收参数

```
//使用el表达式接收url地址拼接的参数
var taskId=${param.taskId};

//使用dialog对象获取DialogUtil的open方法传递的参数对象
var dialog = frameElement.dialog;
var formData = dialog.get("obj").data;
```

注：window.frameElement即为包含本页面的iframe或frame对象

2）分析DialogUtil参数对象的获取

DialogUtil对象的定义，该对象定义在DialogUtil.js文件中

```
/**
 * 为了ligerUi的顶层dialog显示
 */
var DialogUtil = {
    open : function(opts){
        var outerWindow =window.top;
        return outerWindow.$.ligerDialog.open(opts);
    }
}
```

注：该对象内部调用了$.ligerDialog.open方法。

3）ligerui框架

$.ligerDialog对象是系统引入的ligerui框架中定义的。

查看ligerui.all.js文件

```
//核心对象
window.liger = $.ligerui = {
    ....
    run:function (plugin, args, ext)
        {
            ...
            var parms = Array.apply(null, args);
            parms.shift();
            //调用方法
            return manager[method].apply(manager, parms);  //manager method
        }
}

var l = $.ligerui;

$.ligerDialog = function ()
{
    return l.run.call(null, "ligerDialog", arguments, { isStatic: true });
};

//open方法，这里的p参数会传递到上面方法调用的arguments中
$.ligerDialog.open = function (p)
{
    return $.ligerDialog(p);
};
```