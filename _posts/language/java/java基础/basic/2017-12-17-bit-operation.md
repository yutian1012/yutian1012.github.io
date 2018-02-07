---
title: java位运算
tags: [java]
---

参考：http://blog.csdn.net/xiaochunyong/article/details/7748713

Java提供的位运算符有：

左移( << )、右移( >> ) 、无符号右移( >>> ) 、位与( & ) 、位或( | )、位非( ~ )、位异或( ^ )，除了位非( ~ )是一元操作符外，其它的都是二元操作符。

Java中int类型占32位，可以表示一个正数，也可以表示一个负数。正数换算成二进制后的最高位为0，负数的二进制最高为为1。

### 移位操作

1）左移操作

```
public static void main(String[] args) {  
    System.out.println(5<<2);//运行结果是20，相当于乘2的2次幂（即4）
}
```

首先会将5转为2进制表示形式(java中，整数默认就是int类型,也就是32位):

0000 0000 0000 0000 0000 0000 0000 0101  5换算成二进制，左移2位后，低位补0：

0000 0000 0000 0000 0000 0000 0001 0100  换算成10进制为20

2）右移操作

```
System.out.println(5>>2);//运行结果是1，相当于除2的2次幂（即4）
```

首先会将5转为2进制表示形式(java中，整数默认就是int类型,也就是32位):

0000 0000 0000 0000 0000 0000 0000 0101 5换算成二进制，右移2位，高位补0：

0000 0000 0000 0000 0000 0000 0000 0001 换算成10进制为1

3）无符号右移

```
System.out.println(-5>>2);//运算结果是
```

1111 1111 1111 1111 1111 1111 1111 1011 -5换算成二进制，右移2位，高位补1

1111 1111 1111 1111 1111 1111 1111 1110 换算成十进制为-2

4）综合比较

正数右移，高位用0补，负数右移，高位用1补，当负数使用无符号右移时，用0进行部位（自然而然的，就由负数变成了正数了）。

正数或者负数左移，低位都是用0补。

注：右移操作高位补符号位，左移操作低位补0（总结）


```
System.out.println(5>>3);//结果是0  
System.out.println(-5>>3);//结果是-1  
System.out.println(-5>>>3);//结果是536870911
```

5右移过程：

0000 0000 0000 0000 0000 0000 0000 0101 5换算成二进制，右移3位，用0进行补位

0000 0000 0000 0000 0000 0000 0000 0000  换算成10进制为0

-5右移过程：

1111 1111 1111 1111 1111 1111 1111 1011  -5换算成二进制，右移3位，用1进行补位

1111 1111 1111 1111 1111 1111 1111 1111   换算成10进制为-1

注：原码为：除符号位外各位取反，再加1，所以是-1

-5无符号右移过程：

1111 1111 1111 1111 1111 1111 1111 1011  -5换算成二进制，右移3位，用0进行补位

0001 1111 1111 1111 1111 1111 1111 1111  换算成10进制为536870911

### 与/或/非/异或操作

不区分符号位和数值位，统一运算。

1）与操作

```
System.out.println(5 & 3);//结果为1
```

与操作的运算过程：

0000 0000 0000 0000 0000 0000 0000 0101  5转换为二进制

0000 0000 0000 0000 0000 0000 0000 0011  3转换为二进制

---------------------------------------

0000 0000 0000 0000 0000 0000 0000 0001  结果换算成10进制为1

2）或操作

```
System.out.println(5 | 3);//结果为7
```

0000 0000 0000 0000 0000 0000 0000 0101  5转换为二进制

0000 0000 0000 0000 0000 0000 0000 0011  3转换为二进制

---------------------------------------

0000 0000 0000 0000 0000 0000 0000 0111  结果换算成10进制为7

3）非操作

```
System.out.println(~5);//结果为-6  
```

0000 0000 0000 0000 0000 0000 0000 0101  5转换为二进制

---------------------------------------

1111 1111 1111 1111 1111 1111 1111 1010  结果换算成10进制为-6

4）异或操作

```
System.out.println(5 ^ 3);//结果为6
```

0000 0000 0000 0000 0000 0000 0000 0101  5转换为二进制

0000 0000 0000 0000 0000 0000 0000 0011  3转换为二进制

---------------------------------------

0000 0000 0000 0000 0000 0000 0000 0110  结果换算成10进制为6

### 二进制的输出操作

1）java整型类型的表示

包括：int,long,short,byte,char

注：可以使用十进制，十六进制（前缀0x或0X），八进制（前缀0）以及二进制（前缀0b或0B）表示。

int类型的数据（占4个字节）

```
public static void intData() {
    int a = 0x0A;//十六进制方式表示（等价于0x002f），使用0x做前缀表示十进制
    int b = 10;//十进制方式表示
    int c = 012;//八进制方式表示，使用0做前缀表示八进制
    int d = 0b00001010;//二进制方式表示，需要jre7以上的编译环境支持，使用前缀0b
    //输出十进制数值
    System.out.println("a="+a+";"+"b="+b+";"+"c="+c+";"+"d="+d+";");
    
    System.out.println("==========================================");
    
    //输出二进制的表示形式
    System.out.println("a二进制为："+Integer.toBinaryString(a));
    System.out.println("b二进制为："+Integer.toBinaryString(b));
    System.out.println("c二进制为："+Integer.toBinaryString(c));
    System.out.println("d二进制为："+Integer.toBinaryString(d));
}
```

long类型的数据（占8个字节）

```
public static void longData() {
    long a = 0x0A;//十六进制方式表示（等价于0x002f），使用0x做前缀表示十进制
    long b = 10;//十进制方式表示
    long c = 012;//八进制方式表示，使用0做前缀表示八进制
    //二进制方式表示，需要jre7以上的编译环境支持，使用前缀0b
    long d = 0b00001010;
    //输出十进制数值
    System.out.println("a="+a+";"+"b="+b+";"+"c="+c+";"+"d="+d+";");
    
    System.out.println("==========================================");
    
    //输出二进制的表示形式
    System.out.println("a二进制为："+Long.toBinaryString(a));
    System.out.println("b二进制为："+Long.toBinaryString(b));
    System.out.println("c二进制为："+Long.toBinaryString(c));
    System.out.println("d二进制为："+Long.toBinaryString(d));
}
```

short类型的数据（占2个字节）

```
public static void shortData() {
    short a = 0x0A;//十六进制方式表示（等价于0x002f），使用0x做前缀表示十进制
    short b = 10;//十进制方式表示
    short c = 012;//八进制方式表示，使用0做前缀表示八进制
    //二进制方式表示，需要jre7以上的编译环境支持，使用前缀0b
    short d = 0b00001010;
    //输出十进制数值
    System.out.println("a="+a+";"+"b="+b+";"+"c="+c+";"+"d="+d+";");
    
    System.out.println("==========================================");
    
    //输出二进制的表示形式
    System.out.println("a二进制为："+Integer.toBinaryString(a));
    System.out.println("b二进制为："+Integer.toBinaryString(b));
    System.out.println("c二进制为："+Integer.toBinaryString(c));
    System.out.println("d二进制为："+Integer.toBinaryString(d));
}
```

byte类型的数据（占1个字节）

```
public static void byteData() {
    byte a = 0x0A;//十六进制方式表示（等价于0x002f），使用0x做前缀表示十进制
    byte b = 10;//十进制方式表示
    byte c = 012;//八进制方式表示，使用0做前缀表示八进制
    //二进制方式表示，需要jre7以上的编译环境支持，使用前缀0B
    byte d = 0b00001010;
    //输出十进制数值
    System.out.println("a="+a+";"+"b="+b+";"+"c="+c+";"+"d="+d+";");
    
    System.out.println("==========================================");
    
    //输出二进制的表示形式
    System.out.println("a二进制为："+Integer.toBinaryString(a));
    System.out.println("b二进制为："+Integer.toBinaryString(b));
    System.out.println("c二进制为："+Integer.toBinaryString(c));
    System.out.println("d二进制为："+Integer.toBinaryString(d));
}
```

char类型的数据（占1个字节）

```
public static void charData() {
    char a = 0x0A;//十六进制方式表示（等价于0x002f），使用0x做前缀表示十进制
    char b = 10;//十进制方式表示
    char c = 012;//八进制方式表示，使用0做前缀表示八进制
    //二进制方式表示，需要jre7以上的编译环境支持，使用前缀0B
    char d = 0B00001010;
    //输出十进制数值
    System.out.println("a="+a+";"+"b="+b+";"+"c="+c+";"+"d="+d+";");
    
    System.out.println("==========================================");
    
    //输出二进制的表示形式
    System.out.println("a二进制为："+Integer.toBinaryString(a));
    System.out.println("b二进制为："+Integer.toBinaryString(b));
    System.out.println("c二进制为："+Integer.toBinaryString(c));
    System.out.println("d二进制为："+Integer.toBinaryString(d));
}
```

2）java浮点型

包括：float,double，对于浮点型数据一般不会使用上述的赋值方式，浮点型数据包括底数，指数和尾数的概念。

注：编写java代码时，一般不会使用浮点型做位运算。
