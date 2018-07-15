---
title: NIO通信实例
tags: [java]
---

实例：使用NIO进行客户端与服务器端的通信。

有一个服务器通道ServerSocketChannel serverChannel，一个客户端通道 SocketChannel clientChannel；服务器缓冲区：serverBuffer，客户端缓冲区：clientBuffer。

    当服务器想向客户端发送数据时，需要调用：clientChannel.write(serverBuffer)。当客户端要读时，调用 clientChannel.read(clientBuffer)

    当客户端想向服务器发送数据时，需要调用：serverChannel.write(clientBuffer)。当服务器要读时，调用 serverChannel.read(serverBuffer)

    这样，通道和缓冲区的关系似乎更好理解了。在实践中，未必会出现这种双向连接的蠢事（然而这确实存在的，后面的内容还会涉及），但是可以理解为在NIO中：如果想将Data发到目标端，则需要将存储该Data的Buffer，写入到目标端的Channel中，然后再从Channel中读取数据到目标端的Buffer中。
