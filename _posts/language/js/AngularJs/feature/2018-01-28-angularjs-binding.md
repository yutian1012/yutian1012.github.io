---
title: angularjs双向数据绑定
tags: [angularjs]
---

1）数据绑定

a.单向数据绑定

目前大多数前端框架都是单向数据绑定，如JqueryUI，BackBone，Flex等。单向数据绑定的特点是，通过定义的模板（template）+模型数据（model），通过一个js引擎进行渲染，生成一个html片段，将html片段插入到页面中显示。

缺点：生成的html片段生成完成后无法再改变，当有新的数据时，只能重新生成，整体替换。

b.双向数据绑定

视图view和数据model是一个双向的操作过程。当视图发生变化时，数据模型中的数据立即发生变化，同理，当模型数据发生变化时，视图也立即发生变化。

注：这里主要是借助事件机制进行实现。

2）html代码

通过在input标签中输入信息，通过页面立即显示输入的内容。

```
<!doctype html>
<html ng-app>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <!-- 使用ng-model绑定控件，并将控件的值作为模型变量数据 -->
        <input ng-model="greeting.text"/>
        <p>{{greeting.text}},AngularJs</p>
    </body>
    <script src="js/angular-1.2.9.js"></script>
</html>
```