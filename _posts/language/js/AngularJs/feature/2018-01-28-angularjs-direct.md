---
title: angularjs模块的指令
tags: [angularjs]
---

1）指令

可以通过指令，对大部分的功能进行封装，在页面中简单的引用即可。

2）html代码

```
<!doctype html>
<!-- ng-app指令定义一个AngularJS应用程序，即该页面解析时遇到angular标签时交由angular指令逻辑来处理，并渲染页面。只有ng-app所在标签包裹的内容才会使用angular。
    ng-app=MyModule指定了定义的模块，交由该模块定义的内容来处理，找不到相应的处理标签类，再使用全局的
-->
<html ng-app="MyModule">
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <!-- 自定义类hello标签 -->
        <hello></hello>
    </body>
    <script src="js/angular-1.2.9.js"></script>
    <script src="js/HelloAngular_Directive.js"></script>
</html>
```

3）js代码

```
var myModule=angular.module("MyModule",[]);

//将directive挂接到模块中，传递的名称为页面使用的标签名
//返回的模型数据中的template会代替标签，显示到页面中
myModule.directive("hello",function(){
    return {
        restrict: 'E',
        template: '<div>Hi everyone!<div>',
        replace: true
    }
});
```