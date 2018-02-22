---
title: windows系统中常用的编码
tags: [other]
---

在保持txt文本时（另存为）出现的选择编码：ANSI，Unicode，Unicode big endian 和 UTF-8。

![](/images/other/encode/windows-character.png)

### 1. ANSI是默认的编码方式。
1）全称American National Standards Institute(各种外文字符延伸编码方式)

对于英文文件是ASCII编码，对于简体中文文件是GB2312编码（只针对Windows简体中文版，如果是繁体中文版会采用Big5码，与操作系统有关）。

### 2. Unicode编码

指的是UCS-2编码方式，即直接用两个字节存入字符的Unicode码。

UCS-2 (2-byte Universal Character Set)是一种定长的编码方式，UCS-2仅仅简单的使用一个16位码元来表示码位，也就是说在0到0xFFFF的码位范围内，它和UTF-16基本一致。

### 3. Unicode big endian编码
1）Unicode码可以采用UCS-2格式直接存储

以汉字"严"为例，Unicode码是4E25，需要用两个字节存储，一个字节是4E，另一个字节是25。存储的时候，4E在前，25在后，就是Big endian方式；25在前，4E在后，就是Little endian方式。

注：第一个字节在前，就是"大头方式"（Big endian），第二个字节在前就是"小头方式"（Little endian）。

2）Little endian：
地址低位存储值的低位，地址高位存储值的高位。从人的第一观感来说低位值小，就应该放在内存地址小的地方，也即内存地址低位反之，高位值就应该放在内存地址大的地方，也即内存地址高位。"严"为例，Unicode码是4E25，25是低位字节，4E是高位字节，因此25在前（内存地址低），4E在后。

3）BE（big-endian）：
地址低位存储值的高位，地址高位存储值的低位。最直观的字节序，不要考虑对应关系，只需要把内存地址从左到右按照由低到高的顺序写出，就可以把值按照通常的高位到低位的顺序写出。"严"为例，Unicode码是4E25，25是低位字节，4E是高位字节，4E在前（高位存储在了低内存地址上），25在后。如果把内存想成从上到下（内存从低到高）排列，从上到下看就能看到4E25，比较直观。

3）文件规范：
Unicode规范中定义，每一个文件的最前面分别加入一个表示编码顺序的字符，这个字符的名字叫做"零宽度非换行空格"（ZERO WIDTH NO-BREAK SPACE），用FEFF表示。
注：这正好是两个字节，而且FF比FE大1。

Unicode big endian 文件的 BOM 为：FE FF。即如果一个文本文件的头两个字节是FE FF，就表示该文件采用大头方式。
理解为，FE小于FF，递增，称大头，进而高位在前，低位在后。

Unicode little endian 文件的 BOM 为：FF FE。即如果一个文本文件的头两个字节是FF FE，就表示该文件采用小头方式。
理解为，FF大于FE，递减，称小头，进而低位在前，高位在后。

### 4. 实例查看TXT文档的编码方式

打开"记事本"程序Notepad.exe，新建一个文本文件，内容就是一个"严"字，依次采用ANSI，Unicode，Unicode big endian 和 UTF-8编码方式保存。然后，用文本编辑软件UltraEdit中的"十六进制功能"，观察该文件的内部编码方式。

1）ANSI文件的编码：
文件的编码就是两个字节"D1CF"，这正是"严"的GB2312编码，这也暗示GB2312是采用大头方式存储的。

2）Unicode：编码是四个字节"FF FE 25 4E"，其中"FF FE"表明是小头方式存储，真正的编码是4E25。

3）Unicode big endian：编码是四个字节"FE FF 4E 25"，其中"FE FF"表明是大头方式存储。

4）UTF-8：编码是六个字节"EF BB BF E4 B8 A5"，前三个字节"EF BB BF"表示这是UTF-8编码，后三个"E4B8A5"就是"严"的具体编码，它的存储顺序与编码顺序是一致的。

5）Java程序实现
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
                System.out.println(len);
                System.out.println(DatatypeConverter.printHexBinary(buf));
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