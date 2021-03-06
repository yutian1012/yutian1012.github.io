---
title: 计算机中数值的移码表示
tags: [computer]
---

移码（又叫增码）是符号位取反的补码。移码表示法是在数X上增加一个偏移量来定义的，常用来表示浮点数中的阶码，所以是整数。

1）移码的引入

目的是便于浮点数比大小。

如果阶码（指数）也用补码来表示，就会使得一个浮点数中出现两个符号位，即浮点数尾数部分的表示和指数部分的表示。这样的结果是，在比较两个浮点数大小时，无法像比较整数时一样使用简单的无逻辑的二进制比较。

因此，为了处理负指数的情况，实际的指数值按要求需要加上一个偏差（Bias）值作为保存在指数域中的值。

2）移码的计算

如果机器字长为n，规定偏移量为2^(n-1)。若X是整数，则X移=2^(n-1)+X。

注：移码和补码只有符号位相反，其余都一样。

```
# 45的移码，45的补码为00101101，偏移量即最高位为1
[+45]移=00101101+10000000=10101101
  00101101
+ 10000000
------------
  10101101

# -45的移码，-45的补码为11010011，偏移量为10000000
[-45]=11010011+10000000=01010011
  11010011
+ 10000000
------------
  01010011
```

注：在移码表示中，0也编码是相同的，[+0]移=1000000，[-0]移=1000000

### IEEE754标准

用移码表示阶码和用IEEE754标准表示阶码是两回事，IEEE754标准中偏移值是127，而移码表示阶码时偏移值是128。

注：单精度数的偏差值为127（0111 1111）(8位)，而双精度数的偏差值为1023(01 1111 1111)（10位）。

总结，IEEE754标准的偏移量的定义与移码定义有些出入的地方。

1）移码计算中的问题--阶码上溢和下溢

在浮点数的阶码中，00000000与11111111被保留用作特殊情况，所以阶码可用范围只有1~254，总共有254个值。

a.阶码下溢

移码表示中，0有唯一的编码（1000...000，补码的0为000...000，符号位变动后即为1000...000），当出现00...00时（表示-2^n），属于浮点数下溢。

b.阶码上溢

当一个浮点数阶码大于机器的最大阶码，称为上溢。