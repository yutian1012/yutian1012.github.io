---
title: js对象创建--原型
tags: [js]
---

原型prototype相当于java中的抽象类，对于通用的方法和属性，在原型中定义了，那么在对象实例化时就不需要重新创建了，可以解决构造函数实例化时，方法重新创建，效率低下的问题。

1）原型方法

```
function clock(){
}

//原型中设置属性
clock.prototype.hour=12;
clock.prototype.minute=12;
clock.prototype.second=12;
//原型定义方法
clock.prototype.showTime=function(){
    alert(this.hour+":"+this.minute+":"+this.second);
}

//创建对象
var newClock = new clock();
newClock.showTime();
```

2）原型与构造结合

```
//构造函数
function clock(hour,minute,second){
    this.hour=hour;
    this.minute=minute;
    this.second=second;
}
//原型定义方法
clock.prototype.showTime=function(){
    alert(this.hour+":"+this.minute+":"+this.second);
}

//创建对象
var newClock = new clock(1,2,3);
newClock.showTime();
```

注：调用newClock.showTime方法时，先看实例中有没有，有调之，无追踪到原型，有调之，无出错，调用失败。