---
title: 通道之间的数据传输
tags: [java]
---

参考：http://ifeve.com/java-nio-channel-to-channel/

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel传输到另外一个channel。这里主要是因为FileChannel提供了通道对接方法，而其他通道没有提供相应的方法。

实现将其他通道数据写入文件通道，或从文件通道数据写入其他通道。

1）transferFrom方法

该方法实现数据从源通道传输到FileChannel中，源通道可以是FileChannel，也可以是SocketChannel，即从网络的Socket中获取网络传输的数据。

```
package org.sjq.test.nio;

import java.io.RandomAccessFile;
import java.nio.channels.FileChannel;

public class ChannelTest {
    public static void main(String[] args)throws Exception {
        RandomAccessFile fromFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/fromFile.txt", "rw");
        FileChannel fromChannel = fromFile.getChannel();

        RandomAccessFile toFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/toFile.txt", "rw");
        FileChannel toChannel = toFile.getChannel();

        long position = 0;
        long count = fromChannel.size();
        
        System.out.print(count);//文件中的字节长度

        toChannel.transferFrom(fromChannel,position, count);
        
        fromChannel.close();
        toChannel.close();
        fromFile.close();
        toFile.close();
    }
}
```

注：这种方式可不借助缓冲区实现文件的拷贝工作。

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于count个字节，则所传输的字节数要小于请求的字节数。

此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据（count个字节）全部传输到FileChannel中。

2）transferTo方法

transferTo方法将数据从FileChannel传输到其他的channel中。当然，对接的channel也可以是SocketChannel，这样就实现将文件数据传递到网络Scoket客户端。

```
package org.sjq.test.nio;

import java.io.RandomAccessFile;
import java.nio.channels.FileChannel;

public class ChannelTest {
    public static void main(String[] args)throws Exception {
        RandomAccessFile fromFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/fromFile.txt", "rw");
        FileChannel fromChannel = fromFile.getChannel();

        RandomAccessFile toFile = new RandomAccessFile("src/test/java/org/sjq/test/nio/toFile.txt", "rw");
        FileChannel toChannel = toFile.getChannel();

        long position = 0;
        long count = fromChannel.size();
        
        System.out.print(count);//文件中的字节长度
        
        //position处开始向目标文件写入数据，count表示最多传输的字节数。
        fromChannel.transferTo(position, count,toChannel);
        
        fromChannel.close();
        toChannel.close();
        fromFile.close();
        toFile.close();
    }
}
```