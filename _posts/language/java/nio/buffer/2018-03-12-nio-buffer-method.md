---
title: nio缓冲区的常用方法
tags: [java]
---

参考：http://ifeve.com/buffers/

在理解了buffer对象的核心属性后，再来查看buffer相关方法的执行会更清楚。方法的执行会改变相关的属性值。

1）Buffer的分配

要想获得一个Buffer对象首先要进行分配。每一个Buffer类都有一个allocate方法。

```
# 分配48字节capacity的ByteBuffer
ByteBuffer buf = ByteBuffer.allocate(48);

# 分配一个可存储1024个字符的CharBuffer
CharBuffer buf = CharBuffer.allocate(1024);
```

2）向Buffer中写数据

写数据到Buffer有两种方式：

a.从Channel写到Buffer

b.通过Buffer的put()方法写到Buffer里

```
# 从Channel写到Buffer，将通道中的内容写入buffer
RandomAccessFile aFile = new RandomAccessFile("nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
int bytesRead = inChannel.read(buf); //read into buffer.

# 通过put方法写到Buffer，从内存写到buffer中
buf.put(127);
```

注：put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如，写到一个指定的位置，或者把一个字节数组写入到Buffer。

3）flip()方法

flip方法将Buffer从写模式切换到读模式。

调用flip()方法会将position设回0，并将limit设置成之前position的值。换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等。即表示现在能读取多少个byte、char等。

4）从Buffer中读取数据

从Buffer中读取数据有两种方式：

a.从Buffer读取数据到Channel

b.使用get()方法从Buffer中读取数据

```
# 从Buffer读取数据到Channel，将buf数据读入到通道
int bytesWritten = inChannel.write(buf);

# 使用get()方法从Buffer中读取数据，将buf的数据读入到内存
byte aByte = buf.get();
```

注：get方法有很多版本，允许你以不同的方式从Buffer中读取数据。例如，从指定position读取，或者从Buffer中读取数据到字节数组。

5）rewind()方法--实现重新读取数据

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

6）clear()与compact()方法

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成capacity的值。换句话说，Buffer被清空了。其实Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据，调用clear方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

7）mark()与reset()方法

通过调用Buffer.mark方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset方法恢复到这个position。因此这两个方法在使用时一般是成对出现的。

```
# 标记当前position的位置
buffer.mark();
# 将position设置回mark处
buffer.reset();  //set position back to mark.
```
