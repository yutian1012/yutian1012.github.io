---
title: nio缓冲区的类型
tags: [java]
---

参考：https://www.cnblogs.com/ironPhoenix/p/4199728.html

1）Java NIO对象类型缓冲区

Buffer类型包括：ByteBuffer，MappedByteBuffer，CharBuffer，DoubleBuffer，FloatBuffer，IntBuffer，LongBuffer，ShortBuffer。

这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float或double类型来操作缓冲区中的字节。

2）Java NIO与channel交互

Buffer在nio中的主要作用就是与channel交互。但这几种类型中能与channel交互的只有ByteBuffer。所以在用其他类型Buffer的时候，一般都是先将ByteBuffer转化为想用的类型。

使用ByteBuffer类提供的byteBuffer.asCharBuffer()、byteBuffer.asIntBuffer()等方法进行转换相应的Buffer类型。意思就是底层是ByteBuffer，但看起来是其它种类的Buffer，可以用相应的方法，但是视图发生了读写，底层的ByteBuffer也会发生变化。

```
package org.sjq.test.nio;

import java.nio.ByteBuffer;
import java.nio.CharBuffer;

public class BufferTest {
    public static void main(String[] args)throws Exception {
        ByteBuffer bb = ByteBuffer.allocate(1024);
        //将ByteBuffer转化为CharBuffer视图后，再调用put，ByteBuffer中的position指针不会移动
        CharBuffer charBuffer=bb.asCharBuffer().put("Hello World中国");//以字符串的形式写入
        bb.limit("Hello World中国".length()*Character.BYTES);//字符数组长度*每个字符占的字节数
        
        /*while(bb.hasRemaining()) {
            //前面是以字符串的形式写入的，因此以字符串的形式读出是没有问题的
            System.out.print(bb.getChar());
        }*/
        
        charBuffer.flip();
        while(charBuffer.hasRemaining()) {
            System.out.print(charBuffer.get());
        }
        
    }
}
```