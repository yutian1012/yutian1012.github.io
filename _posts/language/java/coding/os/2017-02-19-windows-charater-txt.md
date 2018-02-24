---
title: windows系统中常用的编码
tags: [coding]
---

在保持txt文本时（另存为）出现的选择编码：ANSI，Unicode，Unicode big endian和UTF-8。

![](/images/other/encode/windows-character.png)

### ANSI是默认的编码方式

全称American National Standards Institute（各种外文字符延伸编码方式）。

对于英文文件是ASCII编码，对于简体中文文件是GB2312编码（只针对Windows简体中文版，如果是繁体中文版会采用Big5码，与操作系统有关）。

### Unicode编码（windows默认小端方式存储）

指的是UCS-2编码方式，即直接用两个字节存入字符的Unicode码。

UCS-2 (2-byte Universal Character Set)是一种定长的编码方式，UCS-2仅仅简单的使用一个16位码元来表示码位，也就是说在0到0xFFFF的码位范围内，它和UTF-16基本一致。

### Unicode big endian编码

指的是UCS-2编码方式，但存储数据时最低位地址存放高位字节，即内存从最低地址开始按顺序存放（高数位数字先写）。

### 实例查看TXT文档的编码方式

1）新建一个文本文件

内容就是一个"严"字，依次采用ANSI，Unicode，Unicode big endian 和 UTF-8编码方式保存。然后，用文本编辑软件UltraEdit中的"十六进制功能"，观察该文件的内部编码方式。

2）ANSI文件的编码：

文件的编码就是两个字节"D1CF"，这正是"严"的GB2312编码，这也暗示GB2312是采用大头方式存储的。

3）Unicode编码

文件的编码是四个字节"FF FE 25 4E"

注：其中"FF FE"表明是小端方式存储，真正的编码是4E25。

4）Unicode big endian编码

文件编码是四个字节"FE FF 4E 25"

注：其中"FE FF"表明是大端方式存储。

5）UTF-8编码

文件编码是六个字节"EF BB BF E4 B8 A5"，

注：前三个字节"EF BB BF"表示这是UTF-8编码（BOM头），后三个"E4B8A5"就是"严"的具体UTF-8编码值，它的存储顺序与编码顺序是一致的。

6）Java程序实现

```
public class WindowEncoding {
    
    public static void main(String[] args) throws Exception {
        readFile("E:/test_ANSI.txt","GB2312");
        readFile("E:/test_Unicode_little.txt","UTF-16");
        readFile("E:/test_Unicode_big.txt","UTF-16");
        readFile("E:/test_utf-8.txt","UTF-8");
    }
    /**
     * 读取文档内容
     * @param filePath
     * @param charsetName
     */
    public static void readFile(String filePath,String charsetName){
        FileInputStream fis=null;
        BufferedInputStream bis=null;
        try{
            fis=new FileInputStream(filePath);
            bis=new BufferedInputStream(fis);
            byte[] buf=new byte[8];
            int len=0;
            while((len=bis.read(buf))!=-1){
                //输出字节长度
                System.out.println(len);
                //输出十六进制数值
                System.out.println(DatatypeConverter.printHexBinary(buf));
                //输出字符，new String会自动过滤掉BOM头，也能很好的理解大端小端
                System.out.println(new String(buf,0,len,charsetName));
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=bis){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```