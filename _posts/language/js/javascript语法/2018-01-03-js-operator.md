---
title: js运算符
tags: [js]
---

运算符是指程序设计语言中有运算意义的符号。JavaScript的运算符包含算术运算符、逻辑运算符和一些特殊的运算符。

### 算术运算符

通常在数学表达式中使用，实现数值类型操作数间的数学计算。JavaScript中算术运算符包括加法、减法、乘法、除法、取模/取余（%）、征服号、递增、递减等。

注：加法运算符与字符串型操作数结合的情况，操作数类型为字符串时其意义为连接运算。
注2：JavaScript中数学运算符运用于非数字型操作数时，将发生隐式类型转换。

注3：当算术运算返回一个未定义的或无法表示的值时，NaN就产生了。

```
typeof NaN;//输出number
```

### 关系运算符

关系运算符是比较两个操作数大于、小于或相等的运算符。返回一个布尔值，表示给定关系是否成立。

1）相等运算符（==）和等同运算符（===）

```
1=="1";//返回true
1==true;//返回true
```

注：相等运算符“==”不是严格相等性判断，只要类型转换后的数据仍然相等的话就返回true。

注2：等同运算符“===”是严格意义上的相等，两个值和它们的类型完全一致时才返回true。

2）不等运算符（!=）和不等同运算符（!==）

注：与相等运算符和等同运算符进行类比。

### in运算符

in运算符检查对象中是否有某特定的属性，in运算符可以用来变量数组集合。

```
var fruit=new Array("梨",3.5,"葡萄",7,"香蕉",2,"苹果",3,"荔枝",6);
for(index in fruit){
    if(index%2==0){
        //alert(typeof index);//string类型
        alert(fruit[index]+"价格："+fruit[index-0+1]);
    }
}
```

注：in运算符返回的是集合的下标，并且类型为string。

### instanceof运算符

instanceof运算符表明某对象是否是某个类的实例，用于确定对象的身份。

注：使用instanceof可以判断某一对象是否是某一个类的实例，当要确定某个对象的类型时可以使用typeof运算符。

### 字符串运算符

字符串运算符主要包括+、>、<、>=、<=这几种。

其中+表示字符串连接符

比较运算符是字符串的大小比较，字符串比较会依次比较每个字符的ascii码值。

### 逻辑运算符

逻辑运算符包括：逻辑与（&&）、逻辑或（||）、逻辑非（!）。

注：在使用逻辑运算符时，操作数被当成布尔类型。

注2：非零值为true，零值为false。

### 位运算符

位运算符是对变量的二进制位进行逻辑运算，包括按位与（&）、按位或（|）、按位异或（^）、按位非（~）和移位运算（包括有符号和无法符号的左移和右移操作）等。

1）按位异或操作

通常按位异或运算应用在信息加/解密场合

```
# A看作明文，B看作密钥，C看作密文
A^B=C;//A与B异或得到C
B^C=A;//B与C异或就能还原出A
```

实例：在网络上传送敏感信息时通常需要加密处理，以防被他人窃取。现在要求编写一个信息加密解密的应用程序，可以将数据信息加密成不可阅读的数据，同时又能将加密后的数据解密还原。

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

2）按位非（按位取反）

JavaScript中对数据的按位非运算有其独特的规律，并不完全等同于其他编程语言所进行的位操作。

```
~"abc";//字符串取反-1
~true;//布尔true取反-2
~false;//布尔false取反-1
~3;//正数取反-4
~-3;//负数取反2
```

注：对于字符串数据，按位取反后值为-1

注2：对布尔值true和false按位取反分别得到-2和-1

注3：对数值数据+N的-(N+1)，-N得N-1，针对数值型取反操作需要将值转换成补码，然后在补码的位上进行取反操作获取出来的值仍然是补码，在将补码转换成原码，即输出得到的结果。

### 移位运算

移位时一定要考虑因为符号位的移动而带来的影响。

1）左移运算符（<<）

移位规则：整体向左移动低位补0，向左移动n位，相当于原数乘以2^n

2）右移运算符（>>）

当移动的是带有符号的数值时，左边空出的位用书的符号位填充，向右移动超出的位将被丢弃。

注：负数高位为1，正数高位为0

3）高位补0的右移运算（>>>）

左侧空出的位用0填充，右边超出的位被丢弃。

### 特殊运算符

JavaScript中存在一些特殊的运算符应用在特殊的场合中。这些特殊运算符包括：条件运算符（三目运算符），new运算符，void运算符，typeof运算符，点运算符，数组存取运算符，delete运算符，逗号运算符，this运算符等。

1）new运算符

使用new运算符能够创建对象，但要删除对象需要将变量所指向的对象赋值为null，等待系统回收。

注：delete只能删除对象的属性，不能删除对象。

注2：使用new创建的对象，若要删除须对引用对象的变量赋null值。

2）void运算符

使用void运算符可以避免表达式返回值，即忽略表达式返回的值。void可以让表达式被执行而结果被忽略。

在浏览器的地址栏中执行语句：

```
javascript:void(window.open("http://www.baidu.com");)
```

注：window对象的open方法可以打开一个新窗口，并加载指定地址的文件。open方法打开一个值引用新打开的窗口，如果不是void运算符屏蔽返回值，当前窗口的内容将被清除。

3）类型检查运算符typeof

typeof可获得数据的类型名，返回6种可能的值：Number、String、Boolean、Object、Function和undefined。

4）对象属性存取运算符（点运算符）

对象的属性类型也包括对象的方法。主要包括两种形式的调用：第一种是调用实例对象的属性或方法，第二种是调用类的静态方法，静态方法是所有对象公有的，类无线实例化即可使用的方法。

```
var nameYZN="杨宗楠";
var xing=nameYZN.charAt(0);
var unicodeOfYang=String.fromCharCode(26472);
```

注：fromCharCode可将unicode码转为汉字。

5）数组存取运算符（[]）

方括号中使用数组的索引下标，下标从0开始编码。

注：中括号运算符也可以用来获取对象的属性值

```
var student=new Object();
student.name="zhang";
alert(student["name"]);
```

6）delete运算符

要删除使用new运算符创建的对象需要将对象的引用赋值null，当引用为0时系统自动回收对象所占的资源。

delete运算符则可以删除对象的一个属性或数组的一个元素，JavaScript对象的属性可以动态添加，对于动态添加的属性可以使用deleteyunsf将其删除。

delete对象属性后，再次访问该属性时为undefined，表示属性以及不存在了。

7）函数调用运算符符（call）

函数调用运算符call，作用于Function对象，主要功能是调用对象的一个方法，并以另一个对象替换为当前对象，以改变this指针的指向。

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