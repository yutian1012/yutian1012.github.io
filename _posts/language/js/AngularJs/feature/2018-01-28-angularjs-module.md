---
title: angularjs模块的定义
tags: [angularjs]
---

1）模块化

模块化使代码更清晰，层次划分更明确。模块是个基础，而controller，director甚至service等都需要挂接到模块上，然后再调用模型上的controller...，模块可以理解为一个容器。

![](/imags/js/angularJs/feature/angular-module.png)

2）html代码

```
<!doctype html>
<!-- ng-app指令定义一个AngularJS应用程序，即该页面解析时遇到angular标签时交由angular指令逻辑来处理，并渲染页面，只有ng-app所在标签包裹的内容才会使用angular。
    ng-app=HelloAngular指定了定义的模块，交由该模块定义的内容来处理，找不到相应的处理标签类，再使用全局的
-->
<html ng-app="HelloAngular">
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <!-- ng-controller中指定了MVC的C，指向一个函数，由该函数充当控制器
            该值是注册controller时的值，而不是内部函数名称（与全局区分）
         -->
        <div ng-controller="helloAngular">
            <!-- 这里页面显示的内容为视图view，
             {{}}用来读取模型数据，数据是由控制器方法返回的json对象 -->
            <p>{{greeting.text}},Angular</p>
        </div>
    </body>
    <script src="js/angular-1.2.9.js"></script>
    <script src="js/HelloAngular_module.js"></script>
</html>
```

3）js代码

```
var myModule=angular.module("HelloAngular",[]);

//$scope要求angular注入该变量（绑定），那么就能从变量中获取模型数据
myModule.controller("helloAngular",['$scope',
    function HelloAngular($scope){
        $scope.greeting={
            text:'Hello'
        }
    }
]);
```