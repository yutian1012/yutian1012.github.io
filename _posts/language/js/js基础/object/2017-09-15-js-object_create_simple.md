---
title: js对象创建--直接对象
tags: [js]
---

直接创建对象，而不是先创建类，然后由类来生成对象。

1）以json的方式来定义js对象。

```
var clock={
  hour:12,
  minute:10,
  second:10,
  showTime:function(){
   alert(this.hour+":"+this.minute+":"+this.second);
  }
}

clock.showTime();//调用
```

2）创建Object实例

```
var clock = new Object();
clock.hour=12;
clock.minute=10;
clock.showHour=function(){alert(clock.hour);};

clock.showHour();//调用
```

注：属性是可以动态添加，修改的

