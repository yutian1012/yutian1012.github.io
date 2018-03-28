---
title: 二维码编码
tags: [weixin]
---

参考：https://blog.csdn.net/hk_5788/article/details/50839790

1）QR码支持的编码：

数字编码（Numeric mode），字符编码（Alphanumeric mode），字节编码（Byte mode），日文编码（Kanji mode），特殊的字符集编码（Extended Channel Interpretation (ECI) mode），混合编码（Structured Append mode），一些特殊的工业或行业用编码（FNC1 mode）等。

2）使用的编码标识

在Format Information中定义了编码格式的“编号”。

![](/images/weixin/develop/QRCode/mode-indicators.png)

3）版本与相应的编码位数

不同版本（尺寸）的二维码，对于，数字，字符，字节和Kanji模式下，对于单个编码的2进制的位数。

![](/images/weixin/develop/QRCode/number-bits-count.png)

### 编码过程

1）数字编码

Numeric mode数字编码，从0到9。如果需要编码的数字的个数不是3的倍数，那么，最后剩下的1或2位数会被转成4或7bits，则其它的每3位数字会被编成10，12，14bits，编成多长还要看二维码的尺寸。

注：这里的10，12，14的长度是由version决定的

实例：在Version 1的尺寸下，纠错级别为H的情况下，编码：01234567

a.分组

把上述数字分成三组: 012 345 67

b.对分组进行编码

把他们转成二进制，因为使用的是version1，所有编码数字使用10位

```
012 转成 0000001100  
345 转成 0101011001  
67 转成 1000011
```

c.把这三个二进制串起来: 

```
0000001100 0101011001 1000011
```

d.对数字个数进行编码

把数字的个数转成二进制 (version 1-H是10bits): 

```
# 8个数字的二进制是 
0000001000
```

e.添加编码格式标志

把数字编码的标志0001拼接到二进制串上

```
0001 0000001000 0000001100 0101011001 1000011
```