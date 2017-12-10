---
title: Netty client实例
tags: [architecture]
---

创建一个netty客户端

```
public class Client {

    public static void main(String[] args) {
        //服务类
        ClientBootstrap bootstrap = new  ClientBootstrap();
        
        //线程池
        ExecutorService boss = Executors.newCachedThreadPool();
        ExecutorService worker = Executors.newCachedThreadPool();
        
        //socket工厂
        bootstrap.setFactory(new NioClientSocketChannelFactory(boss, worker));
        
        //管道工厂
        bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
            
            @Override
            public ChannelPipeline getPipeline() throws Exception {
                ChannelPipeline pipeline = Channels.pipeline();
                //解码器
                pipeline.addLast("decoder", new StringDecoder());
                //编码器
                pipeline.addLast("encoder", new StringEncoder());
                //消息接收发送处理类
                pipeline.addLast("hiHandler", new HiHandler());
                return pipeline;
            }
        });
        
        //连接服务端
        ChannelFuture connect = bootstrap.connect(
            new InetSocketAddress("127.0.0.1", 10101));
        //channel理解为一个会话
        Channel channel = connect.getChannel();
        
        System.out.println("client start");
        
        Scanner scanner = new Scanner(System.in);
        while(true){
            System.out.println("请输入");
            channel.write(scanner.next());
        }
    }
}
```

消息接收处理类

```
/**
 * 消息接受处理类
 */
public class HiHandler extends SimpleChannelHandler {

    /**
     * 接收消息
     */
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {      
        String s = (String) e.getMessage();
        System.out.println(s);
        
        super.messageReceived(ctx, e);
    }

    /**
     * 捕获异常
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) throws Exception {
        System.out.println("exceptionCaught");
        super.exceptionCaught(ctx, e);
    }

    /**
     * 新连接
     */
    @Override
    public void channelConnected(ChannelHandlerContext ctx, ChannelStateEvent e) throws Exception {
        System.out.println("channelConnected");
        super.channelConnected(ctx, e);
    }

    /**
     * 必须是链接已经建立，关闭通道的时候才会触发
     */
    @Override
    public void channelDisconnected(ChannelHandlerContext ctx, ChannelStateEvent e) throws Exception {
        System.out.println("channelDisconnected");
        super.channelDisconnected(ctx, e);
    }

    /**
     * channel关闭的时候触发
     */
    @Override
    public void channelClosed(ChannelHandlerContext ctx, ChannelStateEvent e) throws Exception {
        System.out.println("channelClosed");
        super.channelClosed(ctx, e);
    }
}
```