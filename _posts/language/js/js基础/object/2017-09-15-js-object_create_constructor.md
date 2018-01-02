---
title: js对象创建--构造函数
tags: [js]
---

构造函数是普通函数，生成对象时必须使用new时，即将其作为构造函数来处理。缺点是必须强制使用new，并且函数中的方法在创建对象时都会重新创建（效率低）。

1）构造函数创建对象（避免直接调用，放置为window全局添加不必要的属性）

```
function clock(hour,minute,second){
  this.hour = hour;
  this.minute = minute;
  this.second = second;
  this.showTime = function(){
   alert(this.hour+":"+this.minute+":"+this.second);
  }
 }

//使用new关键字创建对象
var newClock =new  clock(12,12,12); 
alert(newClock.hour);
newClock instanceof clock;//true
```

注：如果不加new关键字，clock就不会当成构造函数调用，而只是一个普通的函数。同时，还会意外地给他的外部作用域即window添加属性，因为此时构造函数内部的this已经映射到了外部作用域（window）上了。

2）构造函数创建对象（改进）

改进方法，避免用户直接把构造函数当普通函数来调用。

```
function clock(hour,minute,second){
  if(this instanceof clock){
   this.hour = hour;
   this.minute = minute;
   this.second = second;
   this.showTime = function(){
    alert(this.hour+":"+this.minute+":"+this.second);
   }
  }
  else{
   throw new Error("please add 'new' to make a instance");
  }
}

clock(1,2,3);//直接当作普通方法来调用，会抛出错误
```

3）问题

构造函数创建对象必须使用new关键字，否则该函数就不会当成是构造函数，而只是普通函数。

由于this指针在对象实例化的时候发生改变，this将指向新的实例。这时新实例的方法也要重新创建，如果n个实例就要n次重建相同的方法。而方法一般是通用的，不需要每个对象都创建（可使用prototype方式来保证方法无需重新创建）。