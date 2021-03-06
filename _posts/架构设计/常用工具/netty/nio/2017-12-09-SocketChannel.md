---
title: NIO方式ServerSocketChannel通信
tags: [architecture]
---

NIO通过异步的注册和监听相应的socket通信事件，实现使用少量的线程处理多个客户端的连接读写等请求。

1）NIO的处理过程

处理过程是通过事件注册监听的方式来实现的，如向ServerSocketChannel中注册socket连接事件SelectionKey.OP_ACCEPT，当有客户端请求连接，就会触发连接事件。向SocketChannel中注册接受客户端消息事件SelectionKey.OP_READ，当客户端发送消息时，才能读取客户端发送的消息。

2）NIOServer类

ServerSocketChannel相当于传统的ServerSocket，SocketChannel相当于Socket。

该类通过selector通道管理器来管理ServerSocketChannel通道和SocketChannel通道。通过调用通道的register方法将相应的事件（如SelectionKey.OP_ACCEPT）绑定到selector上。利用selector的select方法来监听通道的注册的事件，从而调用相应的处理方法完成通信。

```
/**
 * NIO服务端
 */
public class NIOServer {
    // 通道管理器
    private Selector selector;

    /**
     * 获得一个ServerSocket通道，并对该通道做一些初始化的工作
     * @param port 绑定的端口号
     * @throws IOException
     */
    public void initServer(int port) throws IOException {
        // 获得一个ServerSocket通道
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        // 设置通道为非阻塞
        serverChannel.configureBlocking(false);
        // 将该通道对应的ServerSocket绑定到port端口
        serverChannel.socket().bind(new InetSocketAddress(port));
        // 获得一个通道管理器
        this.selector = Selector.open();
        // 将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_ACCEPT事件,
        //注册该事件后，当该事件到达时，selector.select()会返回，
        //如果该事件没到达selector.select()会一直阻塞。
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
    }

    /**
     * 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理
     * @throws IOException
     */
    public void listen() throws IOException {
        System.out.println("服务端启动成功！");
        // 轮询访问selector
        while (true) {
            // 当注册的事件到达时，方法返回；否则,该方法会一直阻塞
            selector.select();
            // 获得selector中选中的项的迭代器，选中的项为注册的事件
            Iterator<?> ite = this.selector.selectedKeys().iterator();
            while (ite.hasNext()) {
                SelectionKey key = (SelectionKey) ite.next();
                // 删除已选的key,以防重复处理
                ite.remove();

                handler(key);
            }
        }
    }

    /**
     * 处理请求，从SelectionKey中能够判断出该事件是什么类型的事件
     * 请求来南京事件，可读事件等
     * @param key
     * @throws IOException
     */
    public void handler(SelectionKey key) throws IOException {
        
        // 客户端请求连接事件
        if (key.isAcceptable()) {
            handlerAccept(key);
            // 获得了可读的事件
        } else if (key.isReadable()) {
            handelerRead(key);
        }
    }

    /**
     * 处理连接请求
     * 
     * @param key
     * @throws IOException
     */
    public void handlerAccept(SelectionKey key) throws IOException {
        ServerSocketChannel server = (ServerSocketChannel) key.channel();
        // 获得和客户端连接的通道
        SocketChannel channel = server.accept();
        // 设置成非阻塞
        channel.configureBlocking(false);

        // 在这里可以给客户端发送信息哦
        System.out.println("新的客户端连接");
        // 在和客户端连接成功之后，为了可以接收到客户端的信息，
        //需要给通道设置读的权限。
        channel.register(this.selector, SelectionKey.OP_READ);
    }

    /**
     * 处理读的事件
     * 
     * @param key
     * @throws IOException
     */
    public void handelerRead(SelectionKey key) throws IOException {
        // 服务器可读取消息:得到事件发生的Socket通道
        SocketChannel channel = (SocketChannel) key.channel();
        // 创建读取的缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        int read = channel.read(buffer);
        if(read > 0){
            byte[] data = buffer.array();
            String msg = new String(data).trim();
            System.out.println("服务端收到信息：" + msg);
            
            //回写数据
            ByteBuffer outBuffer = ByteBuffer.wrap("好的".getBytes());
            channel.write(outBuffer);// 将消息回送给客户端
        }else{
            System.out.println("客户端关闭");
            key.cancel();
        }
    }

    /**
     * 启动服务端测试
     * 
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        NIOServer server = new NIOServer();
        server.initServer(8000);
        server.listen();
    }
}
```

注：selector注册ServerSocketChannel用来监听服务端口，注册ServerSocketChannel用来建立客户端连接

2）客户端连接

客户端连接可以使用telnet等工具进行连接测试。

3）核心API介绍

```
ServerSocketChannel--ServerSocket
SocketChannel--Socket
Selector用于监听ServerSocketChannel和SocketChannel
```

SelectionKey类比Map的key，value为注册在selector上的Channel对象，如ServerSocketChannel，和SocketChannel。

```
serverChannel.register(selector, SelectionKey.OP_ACCEPT);//ServerSocketChannel
channel.register(this.selector, SelectionKey.OP_READ);//SocketChannel
```

注：每一个客户端的连接都对应一个SocketChannel对象。