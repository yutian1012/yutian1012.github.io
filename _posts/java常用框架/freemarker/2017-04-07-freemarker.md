---
title: freemarker语法
tags: [freemarker]
---

参考：http://www.cnblogs.com/liugang/archive/2012/10/19/2730340.html

1、FreeMarker模板文件组成:
1）文本:直接输出的部分
2）注释:<#-- ... -->格式部分,不会输出
3）插值:即${...}或#{...}格式的部分,将使用数据模型中的部分替代输出
插值分两种类型：1,通用插值${expr};2,数字格式化插值:#{expr}或#{expr;format}
4）FTL指令:FreeMarker指令和HTML标记类似,名字前加#予以区分,不会输出。
注：使用标签时前面的符号#也可能变成@,如果该指令是一个用户指令而不是系统内建指令时,应将#符号改成@符号。即系统指令使用#做前缀，用户指令使用@做前缀。

2、通用插值
1字符串
插值结果为字符串值:直接输出表达式结果。如：${“helloworld”}
2数值
插值结果为数字值:根据默认格式(由#setting指令设置)将表达式结果转换成文本输出，也可以使用内建的字符串函数格式化单个插值。
如代码：${answer?string.number}//?表示方法调用。
3日期值
插值结果为日期值:根据默认格式(由#setting指令设置)将表达式结果转换成文本输出。可以使用内建的字符串函数格式化单个插值
如代码：${lastUpdated?string("yyyy-MM-dd HH:mm:ss zzzz")}
4布尔值
插值结果为布尔值:根据默认格式(由#setting指令设置)将表达式结果转换成文本输出。可以使用内建的字符串函数格式化单个插值。
如代码：<#assign foo=true/>  ${foo?string("yes", "no")}

5数字格式化插值
数字格式化插值可采用#{expr;format}形式来格式化数字,其中expr表示数值表达式，format可以是:
mX:小数部分最小X位
MX:小数部分最大X位
如代码：
```
<#assign x=2.582/>  <#assign y=4/>  
#{x; M2} <#-- 输出2.58 --> 
#{y; m1M2} <#-- 输出4.0 -->
```

3、freemarker表达式
表达式是FreeMarker模板的核心功能,表达式放置在插值语法${}之中时,表明需要输出表达式的值;表达式语法也可与FreeMarker标签结合,用于控制输出。实际上FreeMarker的表达式功能非常强大,它不仅支持直接指定值,输出变量值,也支持字符串格式化输出和集合访问等功能。
1直接指定值
直接指定值可以是字符串,数值,布尔值,集合和MAP对象。
1）字符串
直接指定字符串值使用单引号或双引号限定,如果字符串值中包含特殊字符需要转义。
如代码：
${"helloworld"} 输出：helloworld
注：转义字符：\";双引号(u0022)，\';单引号(u0027)，\\;反斜杠(u005C)，\t;Tab(u0009)等
如代码：
${"\"helloworld\""}输出："helloworld"
注：如果某段文本中包含大量的特殊符号。FreeMarker提供了另一种特殊格式:可以在指定字符串内容的引号前增加r标记,在r标记后的文件将会直接输出。
${r"${foo}"} 输出：${foo}

2）数值
表达式中的数值直接输出,不需要引号。小数点使用"."分隔,不能使用分组","符号。FreeMarker目前还不支持科学计数法,所以"1E3"是错误的。
在FreeMarker表达式中使用数值需要注意以下几点:
第一：数值不能省略小数点前面的0,所以".5"是错误的写法
第二：数值8 , +8 , 8.00都是相同的
如：${8}

3）布尔值
直接使用true和false,不使用引号。如：${true}

4）集合
集合以方括号包括,各集合元素之间以英文逗号","分隔
如代码：
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as x>
${x}
</#list>

5）Map对象
Map对象使用花括号包括,Map中的key-value对之间以英文冒号":"分隔,多组key-value对之间以英文逗号","分隔。
如代码：
{"语文":78, "数学":80}
Map对象的key和value都是表达式,但是key必须是字符串

2输出变量值
FreeMarker的表达式输出变量时,这些变量可以是顶层变量,也可以是Map对象中的变量,还可以是集合中的变量,并可以使用点(.)语法来访问Java对象的属性。
1）顶层变量的输出
所谓顶层变量就是直接放在数据模型中的值
代码：
Map root = new HashMap();   //创建数据模型
root.put("name","annlee");   //name是一个顶层变量
${name} 输出 annlee字符串
注：对于顶层变量,直接使用${variableName}来输出变量值,变量名只能是字母,数字,下划线,$,@和#的组合,且不能以数字开头号。

2）输出集合元素
如果需要输出集合元素,则可以根据集合元素的索引来输出集合元素,集合元素的索引以方括号指定（下标从0开始）。

3）输出Map元素
Map对象可以是直接HashMap实例,甚至包括JavaBean实例,对于JavaBean实例而言,我们一样可以把其当成属性为key,属性值为value的Map实例。为了输出Map元素的值,可以使用点语法或方括号语法
代码：
Map root = new HashMap();//hashMap实例
Book book = new Book();//javaBean对象实例
Author author = new Author();
author.setName("annlee");
author.setAddress("gz");
book.setName("struts2");
book.setAuthor(author);//对象中嵌入对象

//放入到map中
root.put("info","struts");
root.put("book", book);

访问变量：
book.author.name    //全部使用点语法
book["author"].name
book.author["name"]    //混合使用点语法和方括号语法
book["author"]["name"]   //全部使用方括号语法

3字符串操作
FreeMarker的表达式对字符串操作非常灵活,可以将字符串常量和变量连接起来,也可以返回字符串的子串等。
1）字符串连接有两种语法:
第一：使用${..}或#{..}在字符串常量部分插入表达式的值,从而完成字符串连接.
第二：直接使用连接运算符+来连接字符串
代码：
Map root = new HashMap(); 
root.put("user","annlee");
下面将user变量和常量连接起来:
${"hello, ${user}!"}   //使用第一种语法来连接
${"hello, " + user + "!"} //使用+号来连接
上面的输出字符串都是hello,annlee!,可以看出这两种语法的效果完全一样
注：${user}是引用的顶层变量值
值得注意的是,${..}只能用于文本部分,不能用于表达式,下面的代码是错误的:
<#if ${isBig}>Wow!</#if>
<#if "${isBig}">Wow!</#if>
应该写成:<#if isBig>Wow!</#if>

2）截取子串
可以根据字符串的索引来进行,截取子串时如果只指定了一个索引值,则用于取得字符串中指定索引所对应的字符;如果指定两个索引值,则返回两个索引中间的字符串子串
代码：
Map root = new HashMap();
root.put("book","struts2,freemarker");
可以通过如下语法来截取子串:
${book[0]}${book[4]}   //结果是su
${book[1..4]}     //结果是tru

4集合链接运算
将两个集合连接成一个新的集合,连接集合的运算符是+
代码：
<#list ["星期一","星期二","星期三"] + ["星期四","星期五","星期六","星期天"] as x>
${x}
</#list>

5 Map连接运算符
Map对象的连接运算符也是将两个Map对象连接成一个新的Map对象,Map对象的连接运算符是+。如果两个Map对象具有相同的key,则右边的值替代左边的值。
代码：
<#assign scores = {"语文":86,"数学":78} + {"数学":87,"Java":93}>

6算术运算
FreeMarker表达式中完全支持算术运算,FreeMarker支持的算术运算符包括:+, - , * , / , %
代码：
${ 12 %10 }

在表达式中使用算术运算符时要注意以下几点:
1,运算符两边的运算数字必须是数字
2,使用+运算符时,如果一边是数字,一边是字符串,就会自动将数字转换为字符串再连接,如:${3 + "5"},结果是:35

使用内建的int函数可对数值取整,如:
<#assign x=5>
${ (x/2)?int } 输出2
${ 1.1?int } 输出1
${ 1.999?int } 输出1
${ -1.1?int } 输出-1
${ -1.999?int } 输出-1

7比较运算符
表达式中支持的比较运算符有如下几个:
1,=或者==:判断两个值是否相等（=不是赋值运算）
2,!=:判断两个值是否不等.
3,>或者gt:判断左边值是否大于右边值
4,>=或者gte:判断左边值是否大于等于右边值
5,<或者lt:判断左边值是否小于右边值
6,<=或者lte:判断左边值是否小于等于右边值
注意：=和!=可以用于字符串,数值和日期来比较是否相等,但=和!=两边必须是相同类型的值,否则会产生错误。而且FreeMarker是精确比较,"x","x ","X"是不等的。
其它的运算符可以作用于数字和日期,但不能作用于字符串,大部分的时候,使用gt等字母运算符代替>会有更好的效果,因为FreeMarker会把>解释成FTL标签的结束字符。当然,也可以使用括号来避免这种情况,如:<#if (x>y)>。

8逻辑运算符
逻辑运算符有如下几个:
逻辑与:&&
逻辑或:||
逻辑非:!
逻辑运算符只能作用于布尔值,否则将产生错误

9内建函数（用于转换输出）
FreeMarker还提供了一些内建函数来转换输出,可以在任何变量后紧跟?,?后紧跟内建函数,就可以通过内建函数来轮换输出变量。
1）常用的内建的字符串函数:
html:对字符串进行HTML编码
cap_first:使字符串第一个字母大写
lower_case:将字符串转换成小写
upper_case:将字符串转换成大写
trim:去掉字符串前后的空白字符

2）集合的常用内建函数
size:获取序列中元素的个数

3）是数字值的常用内建函数
int:取得数字的整数部分,结果带符号

10空值处理运算符
FreeMarker对空值的处理非常严格。FreeMarker的变量必须有值,没有被赋值的变量就会抛出异常,因为FreeMarker未赋值的变量强制出错可以杜绝很多潜在的错误，如缺失潜在的变量命名,或者其他变量错误。
这里所说的空值,实际上也包括那些并不存在的变量,对于一个Java的null值而言,我们认为这个变量是存在的,只是它的值为null,但对于FreeMarker模板而言,它无法理解null值,null值和不存在的变量完全相同。
1）为了处理缺失变量,FreeMarker提供了两个运算符:
!:指定缺失变量的默认值
??:判断某个变量是否存在

2）!运算符的用法有如下两种:
variable!或variable!defaultValue,第一种用法不给缺失的变量指定默认值,表明默认值是空字符串,长度为0的集合,或者长度为0的Map对象。
注：使用!指定默认值时,并不要求默认值的类型和变量类型相同

3）??运算符非常简单,它总是返回一个布尔值
用法为:variable??,如果该变量存在,返回true,否则返回false

11运算符的优先级
FreeMarker中的运算符优先级如下(由高到低排列):
1,一元运算符:!
2,内建函数:?
3,乘除法:*, / , %
4,加减法:- , +
5,比较:> , < , >= , <= (lt , lte , gt , gte)
6,相等:== , = , !=
7,逻辑与:&&
8,逻辑或:||
9,数字范围: ..（集合操作时）
注：实际上,在开发过程中应该使用括号来严格区分,这样的可读性好,出错少。

4、FreeMarker的常用指令
FreeMarker的FTL指令也是模板的重要组成部分,这些指令可实现对数据模型所包含数据的循环迭代,分支控制。除此之外,还有一些重要的功能,也是通过FTL指令来实现的。
1 if指令
<#assign age=23>
<#if (age>60)>老年人
<#elseif (age>40)>中年人
<#elseif (age>20)>青年人
<#else> 少年人
</#if>
注：逻辑表达式用括号括起来主要是因为里面有>符号,由于FreeMarker会将>符号当成标签的结束字符,可能导致程序出错,为了避免这种情况,我们应该在凡是出现这些符号的地方都使用括号。

2 switch , case , default , break指令
这些指令显然是分支指令,作用类似于Java的switch语句
switch指令的语法结构如下:
<#switch value>
<#case refValue>...<#break>
<#case refValue>...<#break>
<#default>...
</#switch>

3 list, break指令
list指令是一个迭代输出指令,用于迭代输出数据模型中的集合,list指令的语法格式如下:
<#list sequence as item>
...
</#list>
两个特殊的循环变量:
item_index:当前变量的索引值
item_has_next:是否存在下一个对象
可以使用<#break>指令跳出迭代，这里的item是可以变化的

例子如下:
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as x>
${x_index + 1}.${x}<#if x_has_next>,</if>
<#if x="星期四"><#break></#if>
</#list>
4 include指令
include指令的作用类似于JSP的包含指令,用于包含指定页
include指令的语法格式如下:<#include filename [options]>
注：参数意义
filename:该参数指定被包含的模板文件
options:该参数可以省略,指定包含时的选项,包含encoding和parse两个选项,其中encoding指定包含页面时所用的解码集,而parse指定被包含文件是否作为FTL文件来解析,如果省略了parse选项值,则该选项默认是true.

5 import指令（导入变量）
该指令用于导入FreeMarker模板中的所有变量,并将该变量放置在指定的Map对象中。
import指令的语法格式如下:
<#import "/lib/common.ftl" as com>
上面的代码将导入/lib/common.ftl模板文件中的所有变量，放置在一个名为com的Map对象中。

6 noparse指令
noparse指令指定FreeMarker不处理该指定里包含的内容
该指令的语法格式如下:
<#noparse>...</#noparse>

看如下的例子:
<#noparse>
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
</#noparse>
输出如下:
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
注：原样输出文本内容。
7 escape , noescape指令
1）escape指令导致body区的插值都会被自动加上escape表达式,但不会影响字符串内的插值,只会影响到body内出现的插值,使用escape指令的语法格式如下:
<#escape identifier as expression>...
<#noescape>...</#noescape>
</#escape>

看如下的代码:
<#escape x as x?html>  //body中所有的插值都使用?html来格式化
First name:${firstName}
Last name:${lastName}
Maiden name:${maidenName}
</#escape>
上面的代码等同于:
First name:${firstName?html}
Last name:${lastName?html}
Maiden name:${maidenName?html}
注：escape指令在解析模板时起作用而不是在运行时起作用。

2）escape指令也嵌套使用,子escape继承父escape的规则
如下例子:
<#escape x as x?html>
Customer Name:${customerName}
Items to ship;
<#escape x as itemCodeToNameMap[x]>
   ${itemCode1}
   ${itemCode2}
   ${itemCode3}
   ${itemCode4}
</#escape>
</#escape>
上面的代码类似于:
Customer Name:${customerName?html}
Items to ship;
${itemCodeToNameMap[itemCode1]?html}
${itemCodeToNameMap[itemCode2]?html}
${itemCodeToNameMap[itemCode3]?html}
${itemCodeToNameMap[itemCode4]?html}
对于放在escape指令中所有的插值而言,这此插值将被自动加上escape表达式,如果需要指定escape指令中某些插值无需添加escape表达式,则应该使用noescape指令,放在noescape指令中的插值将不会添加escape表达式.

8 assign指令
assign指令在前面已经使用了多次,它用于为该模板页面创建或替换一个顶层变量。assign指令的用法有多种,包含创建或替换一个顶层变量,或者创建或替换多个变量等。
1）创建变量指定命名空间
语法如下:<#assign name=value [in namespacehash]>
注：这个用法用于指定一个名为name的变量,该变量的值为value。
FreeMarker允许在使用assign指令里增加in子句,in子句用于将创建的name变量放入namespacehash命名空间中。

2）创建多个变量
用法:<#assign name1=value1 name2=value2 ... nameN=valueN [in namespacehash]>
注：可以同时创建或替换多个顶层变量。

3）创建复杂表达式
语法格式:<#assign name [in namespacehash]>capture this</#assign>
注：将assign指令的内容赋值给name变量.如下例子:
<#assign x>
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as n>
${n}
</#list>
</#assign>
${x}
输出:星期一 星期二 星期三 星期四 星期五 星期六 星期天

9 setting指令
该指令用于设置FreeMarker的运行环境
语法格式:<#setting name=value>
name的取值范围包含如下几个:
locale:该选项指定该模板所用的国家/语言选项
number_format:指定格式化输出数字的格式
boolean_format:指定两个布尔值的语法格式,默认值是true,false
date_format,time_format,datetime_format:指定格式化输出日期的格式
time_zone:设置格式化输出日期时所使用的时区

10 macro , nested , return指令
1）macro可以用于实现自定义指令,通过使用自定义指令,可以将一段模板片段定义成一个用户指令。
macro指令的语法格式如下:
<#macro name param1 param2 ... paramN>
...
<#nested loopvar1, loopvar2, ..., loopvarN>
...
<#return>
</#macro>
在上面的格式片段中,包含了如下几个部分:
name属性指定的是该自定义指令的名字
param属性指使用自定义指令时传递的参数

2）nested指令:输出使用自定义指令时的中间部分
nested指令中的循环变量:由macro定义部分指定,传给使用标签的模板

3）return指令:用于随时结束该自定义指令

4）一个简单的例子:
<#macro book>   //定义一个自定义指令
j2ee
</#macro>
<@book />    //使用刚才定义的指令
上面的代码输出结果为:j2ee

5）为自定义指令指定参数
<#macro book booklist>     //定义一个自定义指令booklist是参数
<#list booklist as book>
   ${book}
</#list>
</#macro>
<@book booklist=["spring","j2ee"] />   //使用刚刚定义的指令
输出结果为:spring j2ee

6）使用nested指令来输出自定义指令的中间部分
<#macro page title>
<html>
<head>
   <title>FreeMarker示例页面 - ${title?html}</title>
</head>
<body>
   <h1>${title?html}</h1>
   <#nested>      //用于引入用户自定义指令的标签体
</body>
</html>
</#macro>
注：将HTML页面模板定义成一个page指令
在其他页面中如此page指令:
<#import "/common.ftl" as com>  //假设上面的模板页面名为common.ftl,导入页面
<@com.page title="book list">
<u1>
<li>spring</li>
<li>j2ee</li>
</ul>
</@com.page>
注：使用macro和nested指令可以非常容易地实现页面装饰效果。

7）使用nested指令时,指定一个或多个循环变量
<#macro book>
<#nested 1>   //使用book指令时指定了一个循环变量值
<#nested 2>
</#macro>
使用：<@book ;x> ${x} .图书</@book>
注：当使用nested指令传入变量值时,在使用该自定义指令时,就需要使用一个占位符(如book指令后的;x)
输出：1 .图书    2 .图书
注：<#nested 1>中的数值1会传递给站位符x

8）在nested指令中使用循环变量时,可以使用多个循环变量
<#macro repeat count>
<#list 1..count as x>     //使用nested指令时指定了三个循环变量
   <#nested x, x/2, x==count>
</#list>
</#macro>
使用：
<@repeat count=4 ; c halfc last>
${c}. ${halfc}<#if last> Last! </#if>//将该指令嵌入到repeat指令中。
</@repeat>
输出结果为：1. 0.5   2. 1   3. 1.5   4. 2 Last;

9）return指令用于结束macro指令
注：一旦在macro指令中执行了return指令，则FreeMarker不会继续处理macro指令里的内容。
<#macro book>
spring
<#return>
j2ee
</#macro>
使用：
<@book />
输出:spring,而j2ee位于return指令之后,不会输出.

11 local命令
使用local可以声明局部变量，在宏中替换全局变量。
<#local username="王五"/>
