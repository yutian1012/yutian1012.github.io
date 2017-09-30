---
title: js对象创建--工厂方式创建
tags: [js]
---

利用js函数内部创建对象并返回。但这样的方式创建的对象没有独立的类型，只能是Object类型。

代码：

```
function createClock(hour,minute,second){
  var clock = new Object();
   clock.hour=hour;
   clock.minute=minute;
   clock.second=second;
   clock.showHour=function(){
   alert(this.hour+":"+this.minute+":"+this.second);
  };
  return clock;
 };

//调用工厂方法，返回对象
var newClock = createClock(12,12,12);
newClock.showHour();//调用对象方法
newClock instanceof Object;//true
```

注：不能识别对象的类型