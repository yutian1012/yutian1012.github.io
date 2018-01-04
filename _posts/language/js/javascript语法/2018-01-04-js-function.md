---
title: js函数
tags: [js]
---

函数是完成特定任务的可重复调用的代码段，是JavaScript组织代码的单位。

### 函数的定义

JavaScript的函数属于Function对象，因此可以使用Function对象的构造函数来创建一个函数（变量方式），同时也可以使用function关键字以普通的形式来定义一个函数（普通方式）。

1）普通方式定义

```
function sum(arg1,arg2){
    return arg1+arg2;
}
```

2）函数的变量定义方式

```
var circularityArea=new Function("r","return r*r*Math.PI");
var rCircle=2;
var area=circularityArea(rCircle);
alert("半径为"+rCircle+"的圆面积为："+area);
```

注：Function构造函数最后一个字符串相当于函数体内的程序语句序列，前面的参数为方法参数信息。

注2：直接在Function构造函数中输入程序语句创建函数意义不大，函数对象的特定是动态创建函数。程序内部的指令由外部输入并使之得到执行。

注3：变量式定义函数的一个实际案例，可以实现javascript代码在线编辑和运行。

实例：实现js代码在编辑和运行

```

```

### 函数的指针调用方式（回调函数）

指针调用方式也可称为函数回调，回调函数按照调用者的约定事项函数的功能，即由调用者传入回调函数。通常由自己编写回调函数体，而传递到第三方函数中，由第三方函数内部进行调用执行。

实例：编写一个排序函数，实现数字排序，排序函数可接收一个回调函数，内部执行该回调函数实现排序，具体的排序规则由回调函数来决定。

```
function sortNumber(obj,func){
    //参数验证
    if(!(obj instanceof Array)||!(func instanceof Function)){
        throw "参数无效";
    }
    //外层循环，每一次循环都会把前n个元素按照有回调函数规定的顺序排列好
    for(n in obj){
        for(m in obj){
            //使用回调函数排序，规则由用户设定
            if(func(obj[n],obj[m])){
                alert("第"+n+"层循环，交换："+obj[n]+"---"+obj[m]);
                var tmp=obj[n];
                obj[n]=obj[m];
                obj[m]=tmp;
            }
        }
    }
    return obj;
}
//定义回调函数
function greatThan(arg1,arg2){
    return arg1 > arg2;
}
//执行方法
try{
    var numAry=new Array(5,8,6,32,1,45,7,25);
    alert("排序前："+numAry.join(","));
    sortNumber(numAry,greatThan);
    alert("排序后："+numAry.join(","));
}catch(e){
    alert(e);
}
```

注：通用排序函数本身不定义排序规则，规则由回调函数实现。

### 函数参数

JavaScript的函数参数信息由arguments对象管理。

1）arguments对象

arguments对象的length属性说明函数调用函数时实际传递的参数个数。arguments对象不能显示创建，在函数被调用时，由JavaScript运行时环境创建并设定该对象的属性值。

### 函数的作用域

每一个函数在执行时都处于特定的运行上下文中，该上下文决定了函数可以直接访问到的变量。

1）公有函数

公有函数是指定义在全局作用域中，每个代码都可以调用的函数。公有函数是处于顶级作用域中的函数。

2）私有函数

私有函数是指处于局部作用域中的函数，即嵌套在函数定义中的函数。子级函数就是父级函数的私有函数，外界不能调用私有函数，私有函数只能被拥有该函数的函数代码调用。但可以返回该私有函数的指针，供外部程序调用。

```
//定义一个构造函数
function Card(name,phone,address){
    this.name=name;
    this.phone=phone;
    this.address=address;

    //定义私有函数
    this.getName=function getName(){//将私有函数映射成对象属性
        return name;
    }
}
var card=new Card("zhang","12562312556","xxxxx");
alert(card.getName());
```

### 函数中的this关键字

this关键字引用运行上下文中的当前对象，JavaScript的函数调用通常发生于某一个对象的上下文中，如果尚未指定当前对象，则调用函数那是的默认当前对象是Global。

使用call方法可以改变当前对象为指定对象。

```
var name="global";

function localName(){
    this.name="local";
}
function showName(){
    alert("调用函数的上下文对象的名称"+this.name);
}
//不指定调用对象默认当前对象为Global，输出Global对象的name属性
showName();
//定义一个对象作为运行上下文中的当前对象
var ln=new localName();
showName.call(ln);
```

注：使用Function对象的call方法，改变上下文中的当前对象，即改变this的指向。

注2：this关键字极其重要，使用时必须确定当前上下文对象是什么。

### 函数的返回类型

函数的返回类型主要有两种：即值类型（js的基本类型）和引用类型（符合数据类型或函数指针）。

值类型使用的是值传递方式，即传递数据的副本。而引用类型则是引用传递方式，即传递数据的地址。