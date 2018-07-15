---
title: nio缓冲区的类型
tags: [java]
---

参考：https://www.cnblogs.com/ironPhoenix/p/4199728.html

1）Java NIO对象类型缓冲区

Buffer类型包括：ByteBuffer，MappedByteBuffer，CharBuffer，DoubleBuffer，FloatBuffer，IntBuffer，LongBuffer，ShortBuffer。

这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float或double类型来操作缓冲区中的字节。如在CharBuffer类中的get和put方法返回和传递的数据类型就是char。

2）Java NIO与channel交互

Buffer在nio中的主要作用就是与channel交互。但这几种类型中能与channel交互的只有ByteBuffer。所以在用其他类型Buffer的时候，一般都是先将ByteBuffer转化为想用的类型。

使用ByteBuffer类提供的byteBuffer.asCharBuffer()、byteBuffer.asIntBuffer()等方法进行转换相应的Buffer类型。意思就是底层是ByteBuffer，但看起来是其它种类的Buffer，可以用相应的方法，但是视图发生了读写，底层的ByteBuffer也会发生变化。

3）ByteBuffer的常用操作

```
import java.nio.ByteBuffer;

public class NioTest {
    public static void main(String[] args) {
        ByteBuffer bb = ByteBuffer.allocate(48);
        /*向ByteBuffer中put数据的时候，一下四种形式都可以
         * put(byte b)  
         * put(byte[] src) 
         * put(byte[] src, int offset, int length)
         * put(ByteBuffer src)
         * 四种形式都会移动position指针
         */
        bb.put(new byte[]{1,2,4,2,-13});

        //将Buffer从写模式切换到读模式
        bb.flip();
        //hasRemaining()的作用是看看position到没到limit位置
        while(bb.hasRemaining()) {
            System.out.println(bb.get());
        }
    }
}
```

4）ByteBuffer与其他类型缓冲区的操作

注意，在使用ByteBuffer转换成其他类型的缓冲区时，底层还是使用的是ByteBuffer，而只是在上层包装了一层。

a.使用字符串缓冲区（asCharBuffer方法）

使用asCharBuffer方法获取CharBuffer缓冲区，然后调用put方法设置数据并不会改变ByteBuffer的position指向，因此如果调用flip方法，将不会有任何有效数据。只能通过调用limit方法重新设置limit属性的指向，进而再读取数据。

```
import java.nio.ByteBuffer;

public class NioTest {
    public static void main(String[] args)throws Exception {
        ByteBuffer bb = ByteBuffer.allocate(1024);
        //将ByteBuffer转化为CharBuffer视图后，再调用put，ByteBuffer中的position指针不会移动
        String test="Hello World中国";
        
        bb.asCharBuffer().put(test);
      //字符数组长度*每个字符占的字节数，JDK1.8中提供了一个常量：Character.BYTES（值为2）
        bb.limit(test.length()*2);
        
        while(bb.hasRemaining()) {
            //前面是以字符串的形式写入的，因此以字符串的形式读出是没有问题的
            //getChar每次会获取2个字节的数据
            System.out.print(bb.getChar());
        }
    }
}
```

b.使用字符串缓冲区2（asCharBuffer方法）

调用put方法设置数据虽然不会改变ByteBuffer对象的position指向，但会改变CharBuffer对象的position指向，因此在读取数据前，可以调用CharBuffer对象的flip，然后冲CharBuffer中读取数据。CharBuffer底层仍然使用的是ByteBuffer对象。

```
import java.nio.ByteBuffer;
import java.nio.CharBuffer;

public class NioTest {
    public static void main(String[] args)throws Exception {
        ByteBuffer bb = ByteBuffer.allocate(1024);
        //将ByteBuffer转化为CharBuffer视图后，再调用put，ByteBuffer中的position指针不会移动
        String test="Hello World中国";
        
        CharBuffer charBuffer=bb.asCharBuffer().put(test);
        
        charBuffer.flip();

        while(charBuffer.hasRemaining()) {
            System.out.print(charBuffer.get());
        }
    }
}
```

c.查看CharBuffer对ByteBuffer的包装

asCharBuffer获取的CharBuffer对象的类型是ByteBufferAsCharBufferB，而是要CharBuffer的allocate方法获取的CharBuffer对象的类型是HeapCharBuffer。

ByteBufferAsCharBufferB对象中并不会存放任何数据，实际的数据存放在ByteBuffer对象的缓冲区中。

```
import java.nio.ByteBuffer;
import java.nio.CharBuffer;

public class NioTest {
    public static void main(String[] args)throws Exception {
        
        String test="Hello World中国";
        
        //直接使用CharBuffer缓冲区
        CharBuffer charBuffer1=CharBuffer.allocate(32);
        charBuffer1.put(test);
        
        //使用ByteBuffer缓冲区转换CharBuffer缓冲区
        ByteBuffer bb = ByteBuffer.allocate(32);
        CharBuffer charBuffer2=bb.asCharBuffer().put(test);
        
        System.out.println();
    }
}
```

![](/images/java_basic/nio/buffer/ByteBuffer-CharBuffer.png)

![](/images/java_basic/nio/buffer/ByteBuffer-CharBuffer-data.png)