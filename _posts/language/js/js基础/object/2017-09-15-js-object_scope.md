---
title: js对象作用域
tags: [js]
---

1）私有作用域（规定）

ECMAScript只有公用作用域，即ECMAScript中的所有对象的所有属性和方法都是公用的。

约定：规定在属性前后加下划线来表示该对象属性为私有的（即不建议在外边直接访问，而应在对象定义内部使用）。

```
obj._color_ = "blue";
```

2）静态作用域

这里的静态作用域是为了模拟不创建对象，而直接调用类的方法（类似于java中的工具类的静态变量和方法）。ECMAScript并没有静态作用域。不过，它可以给构造函数提供属性和方法。

```
function sayHello() {
  alert("hello");
}

//为函数提供属性
sayHello.alternate = function() {
  alert("hi");
}

sayHello();     //输出 "hello"
sayHello.alternate();   //输出 "hi"，相当于使用类名直接访问方法。
```

注：构造函数是函数，函数是对象，对象可以有属性和方法。