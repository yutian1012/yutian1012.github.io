---
title: freemarker常用指令
tags: [freemarker]
---

FreeMarker的常用指令

FreeMarker的FTL指令也是模板的重要组成部分,这些指令可实现对数据模型所包含数据的循环迭代,分支控制。除此之外,还有一些重要的功能,也是通过FTL指令来实现的。

### 1. if指令

```
<#assign age=23>
<#if (age>60)>老年人
<#elseif (age>40)>中年人
<#elseif (age>20)>青年人
<#else> 少年人
</#if>
```

注：逻辑表达式用括号括起来主要是因为里面有>符号,由于FreeMarker会将>符号当成标签的结束字符,可能导致程序出错,为了避免这种情况,我们应该在凡是出现这些符号的地方都使用括号。

### 2. switch , case , default , break指令

这些指令显然是分支指令,作用类似于Java的switch语句

switch指令的语法结构如下:

```
<#switch value>
<#case refValue>...<#break>
<#case refValue>...<#break>
<#default>...
</#switch>
```

### 3. list, break指令

1）语法

list指令是一个迭代输出指令,用于迭代输出数据模型中的集合,list指令的语法格式如下:

```
<#list sequence as item>
...
</#list>
```

两个特殊的循环变量:

item_index:当前变量的索引值

item_has_next:是否存在下一个对象

可以使用<#break>指令跳出迭代，这里的item是可以变化的

2）实例:

```
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as x>
${x_index + 1}.${x}<#if x_has_next>,</if>
<#if x="星期四"><#break></#if>
</#list>
```

### 4. include指令

include指令的作用类似于JSP的包含指令,用于包含指定页

include指令的语法格式如下:

```
<#include filename [options]>
```

注：参数意义

filename:该参数指定被包含的模板文件

options:该参数可以省略,指定包含时的选项,包含encoding和parse两个选项,其中encoding指定包含页面时所用的解码集,而parse指定被包含文件是否作为FTL文件来解析,如果省略了parse选项值,则该选项默认是true.

### 5. import指令（导入变量）

该指令用于导入FreeMarker模板中的所有变量,并将该变量放置在指定的Map对象中。

import指令的语法格式如下:

```
<#import "/lib/common.ftl" as com>
```

上面的代码将导入/lib/common.ftl模板文件中的所有变量，放置在一个名为com的Map对象中。

### 6. noparse指令

noparse指令指定FreeMarker不处理该指定里包含的内容

该指令的语法格式如下:

```
<#noparse>...</#noparse>
```

实例:

```
<#noparse>
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
</#noparse>
输出如下:
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
```

注：原样输出文本内容。

### 7. escape , noescape指令

1）escape指令导致body区的插值（ftl表达式）都会被自动加上escape表达式（使用escape指定的表达式进行解析）,但不会影响字符串内的插值,只会影响到body内出现的插值。

使用escape指令的语法格式如下:

```
<#escape identifier as expression>...
<#noescape>...</#noescape>
</#escape>
```

实例:

```
<#escape x as x?html>  //body中所有的插值都使用?html来格式化
First name:${firstName}
Last name:${lastName}
Maiden name:${maidenName}
</#escape>
上面的代码等同于:
First name:${firstName?html}
Last name:${lastName?html}
Maiden name:${maidenName?html}
```

注：escape指令在解析模板时起作用而不是在运行时起作用。

场景：为了防止跨站脚本攻击等问题, 必须在对model中的值进行转义。

2）escape指令也嵌套使用,子escape继承父escape的规则

实例:

```
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
```

对于放在escape指令中所有的插值而言,这此插值将被自动加上escape表达式,如果需要指定escape指令中某些插值无需添加escape表达式,则应该使用noescape指令,放在noescape指令中的插值将不会添加escape表达式.

### 8. assign指令

assign指令在前面已经使用了多次,它用于为该模板页面创建或替换一个顶层变量。assign指令的用法有多种,包含创建或替换一个顶层变量,或者创建或替换多个变量等。

1）创建变量指定命名空间

```
<#assign name=value [in namespacehash]>
```

注：这个用法用于指定一个名为name的变量,该变量的值为value。

FreeMarker允许在使用assign指令里增加in子句,in子句用于将创建的name变量放入namespacehash命名空间中。

2）创建多个变量

```
<#assign name1=value1 name2=value2 ... nameN=valueN [in namespacehash]>
```

注：可以同时创建或替换多个顶层变量，空格分开每个变量的定义。

3）创建复杂表达式

```
<#assign name [in namespacehash]>capture this</#assign>
```

注：将assign指令的内容赋值给name变量

实例：

```
<#assign x>
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as n>
${n}
</#list>
</#assign>
${x}
输出:星期一 星期二 星期三 星期四 星期五 星期六 星期天
```

### 9. setting指令

该指令用于设置FreeMarker的运行环境

```
<#setting name=value>
```

name的取值范围包含如下几个:

locale:该选项指定该模板所用的国家/语言选项

number_format:指定格式化输出数字的格式

boolean_format:指定两个布尔值的语法格式,默认值是true,false

date_format,time_format,datetime_format:指定格式化输出日期的格式

time_zone:设置格式化输出日期时所使用的时区

### 10. macro, nested, return指令

1）macro可以用于实现自定义指令,通过使用自定义指令,可以将一段模板片段定义成一个用户指令。

macro指令的语法格式如下:

```
<#macro name param1 param2 ... paramN>
...
<#nested loopvar1, loopvar2, ..., loopvarN>
...
<#return>
</#macro>
```

在上面的格式片段中,包含了如下几个部分:

name属性指定的是该自定义指令的名字

param属性指使用自定义指令时传递的参数

2）nested指令:输出使用自定义指令时的中间部分

nested指令中的循环变量:由macro定义部分指定,传给使用标签的模板

3）return指令:用于随时结束该自定义指令

4）实例:

```
<#macro book>   //定义一个自定义指令
j2ee
</#macro>
<@book />    //使用刚才定义的指令
```

上面的代码输出结果为:j2ee

5）为自定义指令指定参数

```
<#macro book booklist>     //定义一个自定义指令booklist是参数
<#list booklist as book>
   ${book}
</#list>
</#macro>
<@book booklist=["spring","j2ee"] />   //使用刚刚定义的指令
```

输出结果为:spring j2ee

6）使用nested指令来输出自定义指令的中间部分

```
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
```

注：将HTML页面模板定义成一个page指令

在其他页面中使用此page指令:

```
<#import "/common.ftl" as com>  //假设上面的模板页面名为common.ftl,导入页面
<@com.page title="book list">
<u1>
<li>spring</li>
<li>j2ee</li>
</ul>
</@com.page>
```

注：使用macro和nested指令可以非常容易地实现页面装饰效果。

这里的nested的值为命令的标签体

```
<@com.page title="book list">
<u1>
<li>spring</li>
<li>j2ee</li>
</ul>
```

7）使用nested指令时,指定一个或多个循环变量

```
<#macro book>
<#nested 1>   //使用book指令时指定了一个循环变量值
<#nested 2>
</#macro>
nested后的值为循环变量提供值
```

使用：

```
<@book ;x> ${x} .图书</@book>
命令接收nested传递的循环变量值时使用";+变量名"的形式来实现
```

注：当使用nested指令传入变量值时,在使用该自定义指令时,就需要使用一个占位符（如book指令后的;x）

输出：1 .图书    2 .图书

注2：<#nested 1>中的数值1会传递给占位符x

8）在nested指令中使用循环变量时,可以使用多个循环变量

```
<#macro repeat count>
<#list 1..count as x>     //使用nested指令时指定了三个循环变量
   <#nested x, x/2, x==count>
</#list>
</#macro>
```

使用：

```
<@repeat count=4 ; c halfc last>
${c}. ${halfc}<#if last> Last! </#if>//将该指令嵌入到repeat指令中。
</@repeat>
```

输出结果为：1. 0.5   2. 1   3. 1.5   4. 2 Last;

9）return指令用于结束macro指令

注：一旦在macro指令中执行了return指令，则FreeMarker不会继续处理macro指令里的内容。

```
<#macro book>
spring
<#return>
j2ee
</#macro>
```

使用：

```
<@book />
```

输出:spring,而j2ee位于return指令之后,不会输出.

### 11. local命令

使用local可以声明局部变量，在宏中替换全局变量。

```
<#local username="王五"/>
```