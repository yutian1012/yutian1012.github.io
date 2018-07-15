---
title: Netty helloworld实例（3.x版本）
tags: [architecture]
---

1）什么是Netty

Netty是由JBOSS提供的一个java开源框架。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。

2）netty的主要用途

分布式进程通信：例如，hadoop、dubbo、akka等具有分布式功能的框架，底层RPC通信都是基于netty实现的，这些框架使用的版本通常都还在用netty3.x。

游戏服务器开发：使用java开发游戏服务器（网游，手游），实现客户端与服务器的通信都是有使用netty来开发的。

3）入门Helloworld

创建服务类

```
import java.net.InetSocketAddress;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.jboss.netty.bootstrap.ServerBootstrap;
import org.jboss.netty.channel.ChannelPipeline;
import org.jboss.netty.channel.ChannelPipelineFactory;
import org.jboss.netty.channel.Channels;
import org.jboss.netty.channel.socket.nio.NioServerSocketChannelFactory;
import org.jboss.netty.handler.codec.string.StringDecoder;
import org.jboss.netty.handler.codec.string.StringEncoder;
/**
 * netty服务端入门
 */
public class Server {
    public static void main(String[] args) {

        //服务类
        ServerBootstrap bootstrap = new ServerBootstrap();
        
        //boss线程监听端口，即客户端连接请求
        //worker线程负责数据读写，即客户端数据交互
        ExecutorService boss = Executors.newCachedThreadPool();
        ExecutorService worker = Executors.newCachedThreadPool();
        
        //设置niosocket工厂
        bootstrap.setFactory(new NioServerSocketChannelFactory(boss, worker));
        
        //设置管道的工厂，管道中设置过滤器
        bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
            @Override
            public ChannelPipeline getPipeline() throws Exception {
                ChannelPipeline pipeline = Channels.pipeline();
                //接收数据将数据解码成string，否则接收到的是ChannelBuffer，
                //即使用时需要在接收方法messageReceived方法中转换
                pipeline.addLast("decoder", new StringDecoder());
                //写回数据时编码成string，否则回写数据是需要使用ChannelBuffer
                pipeline.addLast("encoder", new StringEncoder());
                pipeline.addLast("helloHandler", new HelloHandler());
                return pipeline;
            }
        });
        
        bootstrap.bind(new InetSocketAddress(10101));
        
        System.out.println("start!!!");
    }
}
```

注：StringDecoder实现了ChannelUpstreamHandler接口，是一个上行的handler，即接收数据经过的handler。StringEncoder实现了ChannelDownstreamHandler接口，是一个下行的handler，即回写发送数据经过的handler。

创建一个channelHandler

```
import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.channel.ChannelStateEvent;
import org.jboss.netty.channel.ExceptionEvent;
import org.jboss.netty.channel.MessageEvent;
import org.jboss.netty.channel.SimpleChannelHandler;
/**
 * 消息接受处理类
 */
public class HelloHandler extends SimpleChannelHandler {

    /**
     * 接收消息，telnet发送消息时触发
     */
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {
        String s = (String) e.getMessage();
        System.out.println(s);
        
        //回写数据
        ctx.getChannel().write("hi");
        super.messageReceived(ctx, e);
    }

    /**
     * 捕获异常，messageReceived方法抛出异常时，会触发该方法
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) throws Exception {
        System.out.println("exceptionCaught");
        super.exceptionCaught(ctx, e);
    }

    /**
     * 新连接，使用telnet连接时触发
     * 该方法可以用来进行黑名单过滤，如果一个IP恶意访问服务，可以在创建连接过程中查看该客户端IP是否在黑名单中，如果在黑名单中，则直接不运行连接的创建。
     */
    @Override
    public void channelConnected(ChannelHandlerContext ctx, ChannelStateEvent e) throws Exception {
        System.out.println("channelConnected");
        super.channelConnected(ctx, e);
    }

    /**
     * 关闭telnet，先触发该方法，在触发channelClosed方法
     * 如果使用客户端来操作，如果不建立连接，关闭客户端不会触发该方法
     * 必须是链接已经建立，关闭通道的时候才会触发

     * 如玩家上线需要将玩家信息读入到redis缓存服务中，玩家下线需要抛出相应的事件将玩家信息从redis缓存中清除。
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

注：SimpleChannelHandler可以处理接收消息，回写数据到客户端。

注2：channelClosed主要用于处理资源的回收操作。