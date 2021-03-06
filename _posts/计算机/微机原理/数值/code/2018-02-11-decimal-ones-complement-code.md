---
title: 计算机中数值的反码表示
tags: [computer]
---

### 反码

反码表示法规定：正数的反码与其原码相同；负数的反码是对其原码逐位取反，但符号位除外。

1）反码的引入

a.原码的加减操作存在问题

计算机中使用原码表示数值，但是在使用原码对数进行算术运算，发现用带符号位的原码进行加减操作时计算不正确。即原码的加减操作是有错误的。

注：两个正整数的原码加法是没有问题的（不考虑溢出），问题出现在带符号位的负数身上。

```
# 1+(-1)=0

# 实际使用原码实现，结果不为0，而变成-2
00000001+10000001=10000010
```

b.反码实现运算

针对负数，对除符号位外的其余各位逐位取反就产生了反码，而在使用反码进行常规计算时是没有问题的。并且反码的取值空间和原码相同且一一对应。


```
# 1+(-1)=0

# 实际使用反码实现
# -1的反码为：11111110
# 11111111反码对应的原码为：10000000，即为-0
(00000001)反 + (11111110)反=(11111111)反
```

2）反码的计算规则

a.反码运算时，其符号位与数值一起参加运算。

b.反码的符号位相加后，如果有进位出现，则要把它送回到最低位去相加（循环进位）。

c.用反码运算，其运算结果亦为反码。在转换为真值时，若符号位为0，数位不变；若符号位为1，应将结果求反才是其真值。

```
# 2+(-1)=1

# 实际使用反码实现，
# 在计算过程中反码发生了一次循环进位，最低位本来为0，进位后变为1
(00000010)反 + (11111110)反 = (00000001)反
```

3）反码存在的问题

问题出现在(+0)和(-0)上，在人们的计算概念中零是没有正负之分的。

注：即思考如何在计算机中消除正零和负零。