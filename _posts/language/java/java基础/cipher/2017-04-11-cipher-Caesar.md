---
title: 凯撒密码
tags: [security]
---

### 1. 工作原理

对数据进行移位操作，如abc（明文）将信息向右位移3加密为def（密文）。仅作用于字母的加密操作，不适合其他字符。

### 2. 代码实现

```
package org.ipph.web.cipher;

public class Caesar {
    public static void main(String[] args) {
        String txt="helloworld";
        //获取所有字符
        char[] charArray=txt.toCharArray();
        for(char c:charArray){
            //字符转换成ASCII码值
            System.out.print((char)(c+3));
        }
    }
}
```

### 3. 问题

如果字母超出了26个英文字母表示的范围，必须重新跳到a。但如果单纯从加密的角度来讲是没有问题的。

代码修改：

### 4. 密码破解

1）主要是根据字母的频率确定出移位值

使用字符出现频率就可以破解。如e在26个英文字母中出现的频率是最高的，与加密中出现频率最高的字母存在一定关系，从而确定出位移。

2）字母频率

```
E是使用频率最高的，下面是英文字母使用频率表:(%)
A 8.19 B 1.47 C 3.83 D 3.91 E 12.25 F 2.26 G 1.71
H 4.57 I 7.10 J 0.14 K 0.41 L 3.77 M 3.34 N 7.06
O 7.26 P 2.89 Q 0.09 R 6.85 S 6.36 T 9.41
U 2.58 V 1.09 W 1.59 X 0.21 Y 1.58 Z 0.08
```

### 5. 统计字符

统计一篇文章中的字母出现频率，并从小到大排列。

第一步：将文件读入到String字符串中，将字符串转换成字符数组。

第二步：定义一个26个长度的int数组，用来记录字符出现的个数

第三步：循环字符数组，累加int数组与字母相应位置的值。

第四步：使用排序算法对结果进行排序。

参考：http://blog.csdn.net/qq_37567881/article/details/55668587