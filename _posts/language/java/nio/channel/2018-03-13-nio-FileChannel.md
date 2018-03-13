---
title: FileChannel文件通道
tags: [java]
---

参考：http://ifeve.com/file-channel/

Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。

FileChannel类可以实现常用的read，write以及scatter/gather操作，同时它也提供了很多专用于文件的新方法，如transferFrom方法和ransferTo方法，这两个方法是专门针对FileChannel提供的方法。

1）打开FileChannel

在使用FileChannel之前，必须先打开它。但是，我们无法直接打开一个FileChannel，需要通过使用一个FileInputStream、FileOutputStream或RandomAccessFile来获取一个FileChannel实例。

```
# 使用FileInputStream打开一个FileChannel
FileInputStream in=new FileInputStream("data/nio-data.txt");
FileChannel inChannel=in.getChannel();

# 使用RandomAccessFile打开一个FileChannel
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```

注意：使用FileInputStream获取的FileChannel不能向channel中写入数据。