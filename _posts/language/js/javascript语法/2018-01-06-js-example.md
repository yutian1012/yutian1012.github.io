---
title: js实例
tags: [js]
---

### 异或操作实现加密

在网络上传送敏感信息时通常需要加密处理，以防被他人窃取。现在要求编写一个信息加密解密的应用程序，可以将数据信息加密成不可阅读的数据，同时又能将加密后的数据解密还原。

```
var msgCoded;//原文
var msgEncoded;//密文
//定义加密和解密的方法
function CodeAndEncode( pkey, date )
{
    var codedStr = "";
    for( i = 0; i<date.length; i++ )
    {
        var dateCoded=date.charCodeAt(i);
        for( j = 0; j<pkey.length; j++ )
        {
            var keyCoded = pkey.charCodeAt( j );
            dateCoded =  dateCoded^ keyCoded;
        }
        codedStr += String.fromCharCode( dateCoded );
    }
    return codedStr;
}
//点击按钮实现加密
function BtnCode_onclick()
{
    var date = TextArea1.value;//获取控件的value值，即明文
    var key = Password1.value;//获取密钥
    msgCoded = CodeAndEncode( key, date );//对数据进行加密
    TextArea1.value = msgCoded;//设置为加密后的值
}
//点击按钮实现解密
function BtnEncode_onclick() 
{
     var date = TextArea1.value;//获取密文
     var key = Password1.value;//获取密钥
     msgEncoded = CodeAndEncode( key, date );//对数据进行解密
     TextArea1.value = msgEncoded;
}
```

注：明文的每一位都与密钥的每一位进行异或得到密码，解密时密文的每一位都与密钥的每一位进行异或，异或操作满足交换律，即A^B^C=C^B^A，从而能够正确的解密。

### js代码在线编辑运行

实例：实现js代码在线编辑和运行

```
//点击按钮运行文本框中输入的js代码
function runJs_onclick()
{
    try
    {
        // 获取文本框的引用，从而获取在线编辑的代码
        var cmdWin = document.getElementById("TextArea1");  
        // 构造函数体
        var str = "try{" + cmdWin.value + 
            "}catch(e){alert('你的代码有错：'+e.description);}";
        //使用Function定义函数
        var cmd = new Function(str);
        // 调用函数，实现函数的运行
        cmd(); 
    }
    catch(e)                            
    {
        alert("错误："+e.description);  // 输出错误信息
    }
}
```

注：使用Function构造函数动态创建方法对象，然后调用该方法。

### call运算符的应用

call运算符作用域Function对象，可理解为Function类的成员方法，传递一个对象替换js运行代码的当前对象，即改变this指针的指向。

```
function showStudentInfo(){
    alert(this.name+" "+this.age);
}
//定义student的构造函数
function Student(_name,_age){
    this.name=_name;
    this.age=_age;
}
//创建两个对象
var stu1=new Student("tom",20);
var stu2=new Student("lily",21);
//使用function对象的call方法实现方法的调用
showStudentInfo.call(stu1);
showStudentInfo.call(stu2);
```

### 回调函数应用

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

注：通用排序函数中的外层循环，每一次循环都会把前n个元素按照有回调函数规定的顺序排列好

注2：通用排序函数本身不定义排序规则，规则由回调函数实现。

### 将对象转换为本地字符串

toLocalString方法作用是将一个对象转换为本地字符串，主要针对日期对象。

```
var date=new Date();
alert(date.toLocaleString());
```