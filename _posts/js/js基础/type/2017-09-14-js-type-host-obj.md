---
title: js类型（宿主对象类型）
tags: [js]
---

1）定义：

所有非原生对象都是宿主对象，即由ECMAScript实现的宿主环境提供的对象。

js的宿主环境一般指浏览器，在浏览器中内置了js运行虚拟机，该虚拟机才是真正的宿主。

注：所有的BOM和DOM对象都是宿主对象

2）浏览器对象（BOM）

Window对象：JavaScript层级中的顶层对象。Window对象表示浏览器窗口。每当body或者frameset标签出现，Window对象就会被自动创建。

Navigator对象：包含客户端浏览器的信息。

Screen对象：包含客户端显示屏的信息。

History对象：包含了浏览器窗口访问过的URL。

Location对象：包含了当前URL的信息。

注：这些对象是由宿主环境创建，并设置到全局对象上，便可在js中直接访问。

测试代码（浏览器F12直接在console中运行测试即可）

```
history.back();
screen.width;
```

3）dom对象

当网页被加载时，浏览器会创建页面的文档对象模型。

![](/images/js/base/type/dom.jpg)

宿主环境会创建document对象，将其设置到全局对象上，因此可以在js中访问dom元素。使用window.document或document即可访问对象。

测试代码

```
window.document.body
```

4）补充：HTML DOM（w3c标准）

HTML DOM定义了用于HTML的一系列标准的对象，以及访问和处理HTML文档的标准方法。通过DOM，可以访问所有的 HTML 元素，连同它们所包含的文本和属性。可以对其中的内容进行修改和删除，同时也可以创建新的元素。

注：DOM独立于平台和编程语言。它可被任何编程语言诸如Java、JavaScript和VBScript使用。