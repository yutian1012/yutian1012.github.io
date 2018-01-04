---
title: js控制语句
tags: [js]
---

控制语句主要包括两大类：选择和循环。选择类语句包括if、switch系列，循环语句包括while，for等。

另外程序执行过程可能发生异常，发生异常时，程序常常被迫中止，可以使用try-catch语句捕捉并处理异常。

### 分支语句

1）switch语句

switch实现多路选择功能，在给定的多个选择中选择一个符合条件的分支来执行，注意每个分支要加break，否则会继续执行下面的分支。

### 循环语句

循环语句主要包括for、while、do-while、for-in这4种循环。while和do-while都是条件成立时才执行循环体，而do-while会至少执行一遍循环体。

1）循环的中断

break和continue语句能够中断循环的执行。break语句将无条件跳出并结束当前的循环结构，continue语句的作用是忽略其后的语句并结束本轮循环和开始新的一轮循环。

2）for-in 语句

```
var arr=new Array("apple","banana","orange");
for( n in arr){
    alert(arr[n]+";");
}
```

### 异常处理语句

程序运行过程中难免会出错，出错后的运行结果往往是不正确的，因此运行时出错的程序通常被强制终止。

运行时的错误统称为异常，为了能在错误发生时得到一个处理的机会，JavaScript提供了异常处理语句。

JavaScript的错误为运行时错误和语法错误，语法错误在编译阶段发现，而运行时错误在运行过程中发现。错误处理语句仅能处理运行时错误。

注：try-catch语句能够编写更安全更健壮的代码。

1）try-catch-finally

实例：for循环中的变量m未定义就使用，会抛出异常。

```
try{
    var arr=new Array("apple","banana","orange");
    for( n=0;n<arr.length;m++){
        alert(arr[n]+";");
    }
}catch(e){
    alert((e.number&0XFFFF)+"号错误："+e.description);
}finally{
    arr=null;//清空数组，防止内存泄漏
    alert("数组内容已被情况!");
}
```

注：finally语句块中的语句是无条件执行的。

2）throw语句

多个异常处理语句可以嵌套使用，父级的try-catch语句可以接收到子级抛出的异常，异常的抛出操作使用throw语句。

```
try{
    throw "手动抛出异常";
}catch(e){
    alert(e);
}
```

注：抛出异常时执行在throw关键字后添加抛出的异常信息字符串即可。

3）创建Error对象并抛出

```
try{
    var e=new Error();
    e.serial=100001;
    e.message="手动抛出异常测试"
    throw e;
}catch(e){
    alert(e.serial+"\t错误信息:"+e.message);
}
```