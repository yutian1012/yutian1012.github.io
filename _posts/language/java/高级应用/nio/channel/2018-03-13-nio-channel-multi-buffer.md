---
title: 通道对接缓存区
tags: [java]
---

参考：http://ifeve.com/java-nio-scattergather/

Java NIO开始支持通道的scatter/gather。

1）分散scatter（）

分散从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据分散（scatter）到多个Buffer中。

![](/images/java_basic/nio/channel/scatter.png)

```
# 将通道中的数据分别存储到消息头和消息体的缓冲区中
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

//接收缓存区数组，数据的读入顺序与缓冲区在数组的顺序有关
channel.read(bufferArray);
```

注：read方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。

在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

2）聚集gather（Gathering Writes）

聚集写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据聚集（gather）后发送到Channel。

![](/images/java_basic/nio/channel/gather.png)

```
# 将消息头和消息体缓冲区数据写入到通道
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

//接收缓冲区数组，缓冲区数据会按照位于数组的顺序，向通道中写入数据
channel.write(bufferArray);
```

注：write方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入（缓冲区的有效数据）。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

3）使用场景

scatter/gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。