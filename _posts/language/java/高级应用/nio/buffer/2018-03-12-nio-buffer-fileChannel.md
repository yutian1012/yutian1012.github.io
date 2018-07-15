---
title: nio读取文档
tags: [java]
---

### NIO利用Buffer读取文件

1）使用Buffer读写数据一般遵循以下四个步骤：

a.写入数据到Buffer，从文件channel中read数据到缓存区

b.调用flip()方法，将Buffer从写模式切换到读模式（读就绪状态）。

注：flip有快速反转的意思，表示重新定位到缓存区的头，表示读就绪状态。

c.从Buffer中读取数据，

d.调用clear()方法或者compact()方法，实现清空缓冲区。

注：clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

2）实例

a.读取文档，文档中没有中文

```
package org.sjq.test.nio;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class BufferTest {
    public static void main(String[] args)throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        //create buffer with capacity of 48 bytes
        ByteBuffer buf = ByteBuffer.allocate(48);

        int bytesRead = inChannel.read(buf); //read into buffer.
        //inChannel.read
        while (bytesRead != -1) {

          buf.flip();  //make buffer ready for read

          while(buf.hasRemaining()){
              System.out.print((char) buf.get()); // read 1 byte at a time
          }

          buf.clear(); //make buffer ready for writing
          bytesRead = inChannel.read(buf);
        }
        aFile.close();
    }
}
```

注：用这种方式读取文档时，如果文档中有中文会出现中文乱码

b.读取中文文档

转换过程中可以看到字符的转换需要使用到nio提供的字符集charset，获取相应字符集的解码对象，对相应的字节数组进行解码或对应的字符，存放到字符缓冲区中。

```
package org.sjq.test.nio;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;

public class BufferTest {
    public static void main(String[] args)throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        //create buffer with capacity of 48 bytes
        ByteBuffer buf = ByteBuffer.allocate(32);
        CharBuffer charBuffer=CharBuffer.allocate(32);
        
        Charset charset=Charset.forName("UtF-8");//设置读取文档的编码，即文档的编辑时使用的编码
        CharsetDecoder decoder=charset.newDecoder();//获取编码器

        int bytesRead = inChannel.read(buf); //read into buffer.
        //inChannel.read
        while (bytesRead != -1) {

          buf.flip();  //make buffer ready for read
          decoder.decode(buf,charBuffer,false);
          charBuffer.flip();
        
          while(charBuffer.hasRemaining()){
              System.out.print(charBuffer.get()); // read 1 byte at a time
          }
        
          charBuffer.clear();
          buf.clear(); //make buffer ready for writing
          //再次从通道中读取
          bytesRead = inChannel.read(buf);
        }
        
        inChannel.close();
        aFile.close();
    }
}
```