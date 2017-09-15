---
title: js类型（原始数据类型）
tags: [js]
---

参考：http://www.cnblogs.com/hongxuejiao/p/4783150.html

js是一种专门设计用来给网页增加交互性的编程语言。

js的数据类型分为原始数据类型和引用数据类型（对象）。对象又分为原生对象，内置对象，宿主对象。

### js的数据类型分为原始数据类型

1）原始数据类型

分为:Undefined,Null,Boolean,Number,String。

js提供了typeof来判断一个值是否表示一种原始类型，如果是，还可以判断他表示的是哪种原始类型。

```
typeof "abc" =="string";//true
typeof 1=="number";//true
typeof true=="boolean";//true
typeof undefined =="undefined";//true
typeof null=="object";//true
```

注：js不区分整形和浮点型，只有number这一个数值类型。

2）数据类型转换

```
//转成number
parseInt("abc");//NaN
typeof parseInt("1");//"number"

typeof parseFloat("1.0");//"number"

//转成字符串
typeof (1+"");//"string"
```

3）boolean类型判断

空字符串，0，undefined，null，false判断是都为false，其他的值都为true。

```
if("") "true"; else "false";//false
if("12") "true"; else "false";//true
if(1) "true"; else "false";//true
if(0) "true"; else "false";//false

if(undefined) true; else false;//false
if(null) true; else false;//false
```