---
title: js定时器实现
tags: [js]
---

1）定时器setInterval

```
setInterval(function(){
    //...
},100);
```

注：定时功能，实现每隔多长时间执行方法体代码，单位是毫秒。

2）定时器bug问题

a.定时器执行放缓

浏览器对setInterval的解析问题，如果打开一个页签，该页签中存在setInterval编写的定时器。即使切换离开该页签，浏览器依然会执行未激活页签内的setInterval方法，但定时器的执行会放缓慢。

b.代码执行时间与设置间隔不匹配会出现bug

如果定时器内部的代码执行时间要远大于定时器设置的时间间隔，会出现异常，如时间上出现偏差等问题。

3）使用setTimeout来模拟定时器

```
function(callback){
    ...
    window.setTimeout(callback,1000/60);
}
```

setTimeout方法会当内部代码执行完成后才会执行callback方法。

4）使用setTimeout实现计数器

```
<html>
<span id="content"></span>
<input type="button" value="开始计时" onclick="start()"/>
<input type="button" value="停止计时" onclick="stop()"/>
<script>
var timer;
var num=0;
var view=document.getElementById("content")
function countDown(){
    view.innerHTML=num++;
    timer=setTimeout(function(){
        console.log("----countDown----");
        countDown();
    },1000)
}

//进入页面即开始执行
countDown();

function start(){
    countDown();
}
function stop(){
    clearTimeout(timer);
}
</script>
</html>
```