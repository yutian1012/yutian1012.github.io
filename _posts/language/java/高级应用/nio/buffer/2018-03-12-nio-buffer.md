---
title: nio缓冲区的核心属性
tags: [java]
---

参考：http://ifeve.com/buffers/

Buffer是一个对象，用于暂存一些要写入或者读出的数据。

在NIO库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的；在写入数据时，也是写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

1）对结构化数据的访问缓冲区

缓冲区实际上是一个数组，并提供了对数据结构化访问以及维护读写位置等信息。具体的缓存区有这些：ByteBuffe、CharBuffer、 ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer。他们实现了相同的接口：Buffer。

2）使用

Java NIO中的Buffer用于和NIO通道进行交互。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO的Buffer对象，并提供了一组方法，用来方便的访问该块内存。

3）Buffer的核心属性

a.mark属性:

定义缓冲区后，存储数据的初始位置，默认为-1。

b.position属性：

存储或者读取到的数据的当前位置，下一个数据将从该position处后面的一个字节开始存储或读取。

c.limit属性：

缓冲区中数据的有效位置，默认和容量一样。

d.capacity属性：

初始容量，在构造对象时可使用allocate方法分配字节数。

4）Buffer的各个属性状态值

a.ByteBuffer初始化状态

mark指向数组的开始位置的前一个（值为-1），position因为没有任何数据也指向开始位置（值为0）。capacity指向数组的最后一个单元下一个位置（即数组长度length），limit为数组的有效长度（即数组的长度length）。

b.插入字符串hello之后的状态

插入字符串后，position位置指向字符串的后一个地址处，其他属性不变。即向缓存中写入数据时，只会改变position的指向。

c.调用flip方法之后的状态（读就绪状态）

flip方法改变position和limit的指向。

调用flip方法后，position重新指向mark位置处（数组的开始位置），即从position指向的位置处开始读取缓冲区中的数据。limit指向原position的位置，即读取的有效数据量为重position到limit之间的子数组。

d.调用clear方法后的状态

position指向开始位置，limit指向capacity位置，即相当于重置缓存区属性状态。

e.调用compact方法后的状态

compact方法会改变position的指向，将未读取的数据复制到前面，而position指向最后一个元素的下一个位置处的下标。最好通过观察代码的执行查看相关的属性变化。

f.调用缓冲区的get方法后的状态

get方法是从缓冲区中读取数据，每调用一次，position都会自增一次。

5）代码查看buffer各个属性的状态

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

在debug模型下运行代码，并在相关的变量框中查看变量的值变化情况。

![](/images/java_basic/nio/buffer/buffer-debug.png)