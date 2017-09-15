---
title: js类型（原生对象类型）
tags: [js]
---

原生对象类型是js中引用数据类型的一种。

1）原生对象

定义：独立于宿主环境的ECMAScript实现提供的对象。

![](/images/js/base/type/primeobj.jpg)

2）Math和Gloabl两个特殊的原生对象

不必明确实例化这两个对象，它已被实例化了，可以直接使用。

```
Math.abs(-1);//1

isNaN("s");//true，该方法是属于Global对象的
```

注：isNaN，parseInt等方法属于Global对象方法。

3）如何获取Global对象

web浏览器将Global作为window对象的一部分加以实现，因此全局对象中声明的变量和函数都成为了window对象的属性。

注：js中window对象扮演了ECMAScript规定的Global角色。

```
window.isNaN("a");//true
```