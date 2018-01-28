---
title: angularjs前端mvc的实现
tags: [angularjs]
---

1）MVC框架

mvc的好处是职责清晰，代码模块化。

2）html代码

```
<!doctype html>
<!-- ng-app指令定义一个AngularJS应用程序，即该页面解析时遇到angular标签时交由angular指令逻辑来处理，并渲染页面，只有ng-app所在标签包裹的内容才会使用angular。-->
<html ng-app>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <!-- ng-controller中指定了MVC的C，指向一个函数，由该函数充当控制器 -->
        <div ng-controller="HelloAngular">
            <!-- 这里页面显示的内容为视图view，
             {{}}用来读取模型数据，数据是由控制器方法返回的json对象 -->
            <p>{{greeting.text}},Angular</p>
        </div>
    </body>
    <script src="js/angular-1.2.9.js"></script>
    <script src="js/HelloAngular_MVC.js"></script>
</html>
```

3）js代码

```
//定义全局函数作为controller控制器，并向$scope中注入模型数据（greeting），方便页面获取并展示到视图中
function HelloAngular($scope){
    $scope.greeting={
        text:'Hello'
    };
}
```

4）问题：

如果使用angular1.3.0.js及以上，会出现Argument 'xxx' is not a function, got undefined错误信息。

主要原因是，在angular1.3.0.js+，函数定义必须要放置到模块下，不再支持全局函数定义了。目的主要是为了规范程序编写。