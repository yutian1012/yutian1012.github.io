---
title: 在java中字节读取文件时的编码
tags: [coding]
---

*读文件要使用和文件编码一致的编码才不会出现乱码。*

参考：http://blog.csdn.net/fgstudent/article/details/43191247

Java读取文件的方式总体可以分为两类：按字节读取和按字符读取。

首先查看java运行环境中使用的字符编码集。

```
//输出UTF-8
System.out.println(Charset.defaultCharset());
```

注：在IDE环境中可以设置Text file encoding（项目右击--properties）

*java默认字符集的查找顺序*

在new String创建字符串或调用String类的getBytes方法时，默认传递的字符编码查找顺序为，IDE环境中设置的编码，操作系统默认使用的编码。

### 字节输入流

首先按字节读取就是采用InputStream.read方法来读取字节，然后保存到一个byte数组中，最后经常用new String把字节数组转换成String。

注：在最后一步隐藏了一个编码的细节，在调用new String方法时会使用操作系统默认的字符集来解码字节数组，中文操作系统就是GBK（如果IDE中设置了文档编码字符集，那么会优先使用该字符集）。而从输入流里读取的字节很可能本身并不是GBK编码的。因为从输入流里读取的字节编码取决于被读取的文件自身的编码。

java代码示例

举个例子：我们在D:盘新建一个名为demo.txt的文件，写入"我们"，并保存。此时demo.txt编码是ANSI，中文操作系统下就是GBK。

a.创建demo.txt文档

编辑文档后直接保存，不要设置编码，默认为ANSI。

b.编写代码读取并显示文本内容

```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testDecode() throws FileNotFoundException {
        String filePath="E:/demo.txt";
        
        FileInputStream fis=new FileInputStream(filePath);
        
        byte[] buf=new byte[1024];
        try {
            while(fis.read(buf)!=-1) {
                //如果new string不设置GBK编码，则出现乱码，因为IDE设置为UTF-8
                System.out.println(new String(buf,"GBK"));
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

从输入字节流读取该文件所得到的字节就是使用GBK方式编码的字节。因此在还原时需要指定编码为GBK。

### 字节输出流

采用字节输出流把字节输出到某个文件，我们是无法指定生成文件的编码的（假设文件以前不存在），那么生成的文件是什么编码的呢？经过测试发现，其实这取决于写入的字节编码格式。

```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testDecode() throws FileNotFoundException {
        
        String filePath="E:/demo.txt";
        
        FileInputStream fis=new FileInputStream(filePath);
        FileOutputStream fos=new FileOutputStream("E:/demoOut.txt");
        
        byte[] buf=new byte[1024];
        try {
            int len=0;
            while((len=fis.read(buf))!=-1) {
                //输出的txt文档是GBK编码
                fos.write(buf,0,len);
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            try {
                fis.close();
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

以UTF-8编码输出（IDE环境设置的编码为UTF-8）

```
byte[] buf=new byte[1024];
try {
    int len=0;
    while((len=fis.read(buf))!=-1) {
        String s=new String(buf, 0, len,"GBK");
        //getBytes方法会将字符串以UTF-8进行编码，获取编码后的字节数组，然后输出
        fos.write(s.getBytes());
    }
}
                
```

总结：InputStream中的字节编码取决文件本身的编码，而OutputStream生成文件的编码取决于字节的编码。

### 把GBK编码的文档转存为UTF-8编码的文档

```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testDecode() throws FileNotFoundException {
        
        String filePath="E:/demo.txt";
        
        FileInputStream fis=new FileInputStream(filePath);
        FileOutputStream fos=new FileOutputStream("E:/demoOut.txt");
        
        byte[] buf=new byte[1024];
        try {
            int len=0;
            while((len=fis.read(buf))!=-1) {
                String s=new String(buf, 0, len,"GBK");
                //IDE环境中设置的字符集为UTF-8
                //getBytes方法会将字符串以UTF-8进行编码，
                //获取编码后的字节数组，然后输出该字节数组
                fos.write(s.getBytes("UTF-8"));//这里也可以不传递UTF-8
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            try {
                fis.close();
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```