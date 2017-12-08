---
title: 表单打印
tags: [bpmx]
---

参考：http://blog.csdn.net/hdchangchang/article/details/46504417

1）web页面的打印

打印使用的是浏览器提供的打印功能。实际上，是浏览器打印功能菜单的一种程序调用。

```
window.print()
```

注：缺点是不能设置打印参数，比如纸型，页边距，选择打印机等等

2）设置打印前和打印后的事件onbeforeprint、onafterprint

```
<script type="text/javascript">
    window.onbeforeprint = function () {
        //将一些不需要打印的隐藏
        alert('Hello');
    }
    window.onafterprint = function () {
        //放开隐藏的元素
        alert('Bye');
    }
</script>
```

注：只有火狐和ie支持，chrome浏览器是不支持这两个方法的。

执行过程：

使用ctrl+p打开打印对话框时，会执行window.onbeforeprint方法，当关闭掉打印对话框后（包括手动关闭），会调用window.onafterprint方法。

3）可以利用css来控制页面中的东西是否显示

实例：

```
<html>
<script type="text/javascript">
    window.onbeforeprint = function () {
        alert('Hello');
    }
    window.onafterprint = function () {
        alert('Bye');
    }
</script>
<style>  
@media print{
.noprint{display:none}
}
</style>
<input type="button" onclick="window.print()" value="打印" class="noprint">

<table width="757" height="174" border="0" align="center" cellpadding="0" cellspacing="0">  
 <!-- tr内容不会被打印 -->
 <tr class="noprint">  
  <td height="133" align="center" valign="top">  
   <img src="http://www.baidu.com/img/baidu_jgylogo3.gif" width="757" height="133">
  </td>    
 </tr>  
 <tr>
  <td height="133" align="center" valign="top" id="content">  
    调用window.print()时，可以利用css来控制页面中的东西是否显示
  </td>
 </tr>
</table>

</html>
```

注意：media的浏览器兼容问题

4）开启新窗口实现打印

针对要打印的页面排版和原web页面相差很大，可以点打印按钮弹出新窗口，把需要打印的内容显示到新窗口中，在新窗口中调用window.print()方法，然后自动关闭新窗口。

```
<!DOCTYPE html>  
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<title>打印测完</title>
</head>
<body>
    <div>....</div>
    <input type="button" value="打印" id="js_print" />
</body>
<script>
    //打印功能单击事件
    var oPrintBtn = document.getElementById("js_print");
    oPrintBtn.onclick=function(){
        //oPop为新窗体对象，
        var oPop = window.open('','oPop');
        //设置输出内容到新窗体中
        var str = '<!DOCTYPE html>'
            str +='<html>'
            ...
        //设置输出内容
        oPop.document.write(str);
        //打印
        oPop.print();
        //打印完毕后关闭
        oPop.close();
    }
</script>
</html>
```