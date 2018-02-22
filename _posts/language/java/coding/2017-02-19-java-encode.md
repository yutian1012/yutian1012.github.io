---
title: java字符编码
tags: [other]
---

Java中的String类是按照unicode进行编码的。char在java中是2个字节。java采用unicode，2个字节(16位)来表示一个字符。

### 1. 正确理解这句话

1）java中所有的字符串的编码都是使用unicode

2）String(byte[] bytes, String encoding)方法

当使用String(byte[] bytes, String encoding)构造字符串时，encoding所指的是bytes中的数据是按照那种方式编码的，而不是最后产生的String是什么编码方式。java会根据指定的encode进行解码，然后再使用unicode进行编码成String类型变量。

换句话说，是让系统把bytes中的数据由encoding编码方式转换成unicode编码。如果不指明，bytes的编码方式将由jdk根据操作系统决定。

3）实例

汉字"严"的GB2312码为："D1 CF"，Unicode编码为："25 4E"，UTF-8编码为："E4 B8 A5"

```
public class JavaUnicode {
    public static void main(String[] args) throws Exception {
        //show();
        byte[] buf=new byte[]{(byte) 0XD1,(byte) 0XCF};
        System.out.println(DatatypeConverter.printHexBinary(buf));//D1 CF
        System.out.println(new String(buf));//乱码
        System.out.println(new String(buf,"GB2312"));//指定编码，告诉java这个字节数组使用的是GB2312编码后的字节数组
        //输出新转换成的string类型的字节数组，从而可以确认new String得到的String类型对象使用的是UTF-8编码
        System.out.println(DatatypeConverter.printHexBinary(new String(buf,"GB2312").getBytes()));//E4 B8 A5
    }
    /**
     * 直接查看
     * @throws UnsupportedEncodingException
     */
    public static void show() throws UnsupportedEncodingException{
        String character="严";
        System.out.println(DatatypeConverter.printHexBinary(character.getBytes()));//E4 B8 A5
        System.out.println(DatatypeConverter.printHexBinary(character.getBytes("GB2312")));//D1 CF
    }
}
```

注：String类型的数据只有存储时才会编码成相应的字节数组（根据指定的编码规则），程序中直接看到的是经过解码后的字符。而该字符使用不同的编码规则，编码得到的字节数组是不同的。这是有不同的编码规则决定的，编码规则可以理解为字符对应的编码表。

### 2. java中getBytes()方法

1）String.getBytes(String decode)方法会根据指定的decode编码返回某字符串在该编码下的byte数组表示

```
public static void showGetBytes() throws UnsupportedEncodingException{
         byte[] b_gbk = "深".getBytes("GBK");//C9EE
         System.out.println(DatatypeConverter.printHexBinary(b_gbk));
         byte[] b_utf8 = "深".getBytes("UTF-8");//E6B7B1
         System.out.println(DatatypeConverter.printHexBinary(b_utf8));
         byte[] b_iso88591 = "深".getBytes("ISO8859-1");//3F
         System.out.println(DatatypeConverter.printHexBinary(b_iso88591));
         byte[] b_unicode = "深".getBytes("unicode");//FEFF6DF1
         System.out.println(DatatypeConverter.printHexBinary(b_unicode));
    }
```

注：分别返回"深"这个汉字在GBK、UTF-8、ISO8859-1和unicode编码下的byte数组表示，此时b_gbk的长度为2，b_utf8的长度为3，b_iso88591的长度为1，unicode为4。

2）与getBytes相对的，可以通过new String(byte[], decode)的方式来还原这个"深"字时，这个new String(byte[], decode)实际是使用decode指定的编码来将byte[]解析成字符串。

```
public static void recoveryBytes() throws UnsupportedEncodingException{
         byte[] b_gbk = "深".getBytes("GBK");
         byte[] b_utf8 = "深".getBytes("UTF-8");
         byte[] b_iso88591 = "深".getBytes("ISO8859-1");
         byte[] b_unicode = "深".getBytes("unicode");
         System.out.println(new String(b_gbk,"GBK"));//正确还原
         System.out.println(new String(b_utf8,"UTF-8"));//正确还原
         System.out.println(new String(b_iso88591,"ISO8859-1"));//乱码
         System.out.println(new String(b_unicode, "unicode"));//正确还原
    }
```

注：通过打印s_gbk、s_utf8、s_iso88591和unicode，会发现，s_gbk、s_utf8和unicode都是"深"，而只有s_iso88591是一个不认识的字符。

3）为什么使用ISO8859-1编码再组合之后，无法还原"深"字呢

因为ISO8859-1编码的编码表中，根本就没有包含汉字字符，当然也就无法通过"深".getBytes("ISO8859-1");，来得到正确的"深"字在ISO8859-1中的编码值了（获取"深"的编码字节数组），所以再通过new String()来还原就无从谈起了。

注：通过String.getBytes(String decode)方法来得到byte[]时，一定要确定decode的编码表中确实存在String表示的码值，这样得到的byte[]数组才能正确被还原。

### 3. web协议支持中文的解决方案

1）web前端将中文使用iso转成字符串

有时候，为了让中文字符适应某些特殊要求（如http header头要求其内容必须为iso8859-1编码），可能会通过将中文字符按照字节方式来编码的情况。如 

```
String s_iso88591 = new String("深".getBytes("UTF-8")"ISO8859-1")， 
```

2）服务器端解析

这样得到的s_iso8859-1字符串实际是三个在 ISO8859-1中的字符（保证服务器端不会将中文字符解析成iso，类似上面的解析，中文的多个字节，解析为了一个iso字节），在将这些字符传递到目的地后，目的地程序再通过相反的方式String s_utf8 = new String(s_iso88591.getBytes("ISO8859-1"),"UTF-8")来得到正确的中文汉字"深"。这样就既保证了遵守协议规定、也支持中文。 

### 4. 数据库长度与字符编码

1）检查字符长度，以免数据库字段的长度不够而报错

考虑到中英文的差异，肯定不能用String.length()方法判断,而需采用String.getBytes().length;而本方法将返回该操作系统默认的编码格式的字节数组。而且应该传入编码类型。

因此，在java开发时，服务器端一般都会选择使用UTF-8编码，而数据库的存储引擎的编码类型也统一使用UTF-8.

2）实例

如字符串"Hello!你好！"，在一个中文WindowsXP系统下，结果为12，而在英文的UNIX环境下，结果将为9。（将其保存到文件中，通过java程序读取）

因为该方法和平台（编码）相关的。在中文操作系统中，getBytes方法返回的是一个GBK或者GB2312的中文编码的字节数组，其中中文字符，各占两个字节，而在英文平台中，一般的默认编码是"ISO-8859-1",每个字符都只取一个字节（而不管是否非拉丁字符）。所以在这种情况下，应该给其传入字符编码字符串，即String.getBytes("GBK").length。

参考：http://blog.csdn.net/cxy_d/article/details/52722919

参考：http://blog.csdn.net/tianjf0514/article/details/7854624
