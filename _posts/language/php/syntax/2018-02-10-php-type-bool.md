---
title: php数据类型--布尔型
tags: [php]
---

1）布尔类型的取值

布尔类型只有两个值true或FALSE

```
<?php
    $a=true;
    # 输出bool(true)
    var_dump($a);
?>
```

2）false值：

在判断条件中，false与整形0，浮动型0.0，空字符串以及字符串"0"，不包含任何元素的数组，不包含任何成员变量的对象，特殊类型NULL（包括尚未设定的变量）是可以替代的。

```
<?php
    $a=0;
    if(!$a){
        echo '$a等价于false';
    }
    if(!0){
        echo "0等价于false";
    }
    if(!null){
        echo "null等价于false";
    }
    # 输出1
    echo is_null($b);

    # 非0表示true
    if(1){
        echo "1等价于true";
    }
?>
```