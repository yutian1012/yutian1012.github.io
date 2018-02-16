---
title: php基本语法
tags: [php]
---

### 基本语法

a.语句以分号结束；

b.定义变量，在php中定义一个变量要以$符号开头，

```
# 定义变量以$符号开头
$a=890;
# 会输出变量的信息，包括变量类型，会输出int(890)
Var_dump($a);
```

c.Php变量的数据类型是变化的，是由运行时的上下文决定的，与js相似

注：php是弱数据类型的编程语言

c.php区分大小写

```
# 区分大小写，下面是定义两个变量
$a=89;$A=90;
```

d.变量名规范

php的变量名称应该以字母或下划线开头，不能以数字开头，也不要用特殊字符开头。

### html代码与php混合编写

1）html嵌入php代码方式一

```
# 必须使用<?php  ?>包起来
<?php xxx  ?>

# 实例
<!DOCTYPE html>
<html>
<head>
    <title>测试</title>
</head>
<body>
    <h1>html元素</h1>
    <?php echo "helloworld"; ?>
</body>
</html>
```

2）html中输出php变量

```
<?=变量?>

# 实例
<!DOCTYPE html>
<html>
<head>
    <title>测试</title>
</head>
<body>
    <h1>html元素</h1>

    <?php  $hello="hellowrold!!" ?>
    <?=$hello?>
</body>
</html>
```

### html注释信息

Php注释，支持但行注释，多行注释，以及unix形式的注释

```
多行注释：/*  */
单行注释：//
Unix形式的注释：#

# 实例，在使用注释的时候需要在<?php ?>中。
<?php
    # 注释代码1
    /* 多行方式注释代码*/
    //但行注释代码
    echo "注释代码";
?>
```