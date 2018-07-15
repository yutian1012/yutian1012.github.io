---
title: java编码与解码
tags: [coding]
---

### 正确理解编码与解码

1）编码

编码是指由字符串编码成二进制流的过程，通过指定不同的编码字符集，实现字符到二进制的转换。

2）解码

解码是指由进制流解码吃呢个字符串的过程，通过指定编码时使用的字符集来进行解码，从而恢复编码前的字符串。

### new String解码

char在java中是2个字节。java采用unicode，2个字节（16位）来表示一个字符。

注：Java中的String类是按照unicode进行编码的。

1）new String方法（解码）

```
new String(byte[] bytes, String encoding)
```

构造字符串时，encoding参数所指的是bytes中的数据是按照那种方式编码的，而不是最后产生的String是什么编码方式。即，指定了字节数组的解码字符集。java会根据指定的encode进行解码成字符串。

注：如果不指明，bytes的编码方式将由jdk根据操作系统决定。

2）实例

汉字"严"的GB2312码为："D1 CF"，Unicode编码为："25 4E"，UTF-8编码为："E4 B8 A5"

```
public class JavaUnicode {
    public static void main(String[] args) throws Exception {
        //show();
        byte[] buf=new byte[]{(byte) 0XD1,(byte) 0XCF};
        System.out.println(DatatypeConverter.printHexBinary(buf));//D1 CF

        System.out.println(new String(buf));//乱码
        //指定编码，告诉java这个字节数组使用的是GB2312编码后的字节数组
        System.out.println(new String(buf,"GB2312"));
    }
    /**
     * 直接查看
     * @throws UnsupportedEncodingException
     */
    public static void show() throws UnsupportedEncodingException{
        String character="严";
        //E4 B8 A5
        System.out.println(DatatypeConverter.printHexBinary(character.getBytes()));
        //D1 CF
        System.out.println(DatatypeConverter.printHexBinary(character.getBytes("GB2312")));
    }
}
```

注：String类型的数据只有存储时才会编码成相应的字节数组（根据指定的编码规则），程序中直接看到的是经过解码后的字符。而该字符使用不同的编码规则，编码得到的字节数组是不同的。这是有不同的编码规则决定的，编码规则可以理解为字符对应的编码表。

### getBytes编码

1）java中getBytes()方法

```
String.getBytes(String decode)
```

方法会根据指定的decode编码返回某字符串在该编码字符集下的byte数组。

2）实例

```
public static void showGetBytes() throws UnsupportedEncodingException{
         byte[] b_gbk = "深".getBytes("GBK");
         //C9EE
         System.out.println(DatatypeConverter.printHexBinary(b_gbk));
         System.out.println(new String(b_gbk,"GBK"));//正确还原
         
         byte[] b_utf8 = "深".getBytes("UTF-8");
         //E6B7B1
         System.out.println(DatatypeConverter.printHexBinary(b_utf8));
         System.out.println(new String(b_utf8,"UTF-8"));//正确还原
         
         //ISO8859-1不能正确的表示中文字符
         byte[] b_iso88591 = "深".getBytes("ISO8859-1");
         //3F
         System.out.println(DatatypeConverter.printHexBinary(b_iso88591));
         System.out.println(new String(b_iso88591,"ISO8859-1"));//乱码
         
         byte[] b_unicode = "深".getBytes("unicode");
         //FEFF6DF1（大端）
         System.out.println(DatatypeConverter.printHexBinary(b_unicode));
         System.out.println(new String(b_unicode, "unicode"));//正确还原
    }
```

注：分别返回"深"这个汉字在GBK、UTF-8、ISO8859-1和unicode编码下的byte数组，此时b_gbk的长度为2，b_utf8的长度为3，b_iso88591的长度为1，unicode为4。

2）为什么使用ISO8859-1编码再组合之后，无法还原"深"字呢

因为ISO8859-1编码的编码表中，根本就没有包含汉字字符，当然也就无法通过"深".getBytes("ISO8859-1")进行编码。所以再通过new String()来还原就无从谈起了。

注：通过String.getBytes(String decode)方法来得到byte数组时，一定要确定decode的编码表中确实存在String表示的码值，这样得到的byte数组才能正确被还原。

3）getBytes方法和平台（编码）相关

如字符串"Hello!你好！"，在一个中文WindowsXP系统下，结果为12，而在英文的UNIX环境下，结果将为9。（将其保存到文件中，通过java程序读取）

a.在中文操作系统中

getBytes方法返回的是一个GBK或者GB2312的中文编码的字节数组，其中中文字符，各占两个字节。

b.在英文平台中

一般的默认编码是"ISO-8859-1"，每个字符都只取一个字节（而不管是否非拉丁字符）。

c.在这种情况下，应该给其传入字符编码字符串

```
String.getBytes("GBK").length
```