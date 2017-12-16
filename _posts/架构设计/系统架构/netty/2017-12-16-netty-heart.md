---
title: Netty 心跳（3.x版本）
tags: [architecture]
---

心跳是为了访问客户端长期连接，而不使用的情况，为了避免资源浪费，可通过心跳来中断长时间不操作的客户端。

1）改造server端代码

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
import org.jboss.netty.handler.timeout.IdleStateHandler;
import org.jboss.netty.util.HashedWheelTimer;
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
        
        //定时器--用于定时检测客户端与服务器的连接情况
        final HashedWheelTimer hashedWheelTimer=new HashedWheelTimer();
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
                
                //IdleStateHandler类的第一个参数是定时器，第二个参数是读超时秒
                //数，第三个参数是写超时秒数，第四个是读写超时的秒数
                //产生Idle事件
                pipeline.addLast("idle",
                        new IdleStateHandler(hashedWheelTimer,5,5,10));
                
                pipeline.addLast("idleHandler", new IdleHandler());
                return pipeline;
            }
        });
        
        bootstrap.bind(new InetSocketAddress(10101));
        
        System.out.println("start!!!");
    }
}
```

2）helloHandler类

继承IdleStateAwareChannelHandler，接收会话状态的事件，并根据会话状态信息决定是否关闭客户端连接。

```
import java.text.SimpleDateFormat;
import java.util.Date;

import org.jboss.netty.channel.ChannelFuture;
import org.jboss.netty.channel.ChannelFutureListener;
import org.jboss.netty.channel.ChannelHandler;
import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.handler.timeout.IdleState;
import org.jboss.netty.handler.timeout.IdleStateAwareChannelHandler;
import org.jboss.netty.handler.timeout.IdleStateEvent;
/**
 * 消息接受处理类
 */
public class IdleHandler extends IdleStateAwareChannelHandler implements ChannelHandler {

    @Override
    public void channelIdle(final ChannelHandlerContext ctx,IdleStateEvent e)throws Exception{
        SimpleDateFormat dateFormat=new SimpleDateFormat("ss");
        //READER_IDLE,WIRTER_IDLE,ALL_IDLE
        System.out.println(e.getState()+dateFormat.format(new Date()));
        //长时间不通信关闭会话
        if(e.getState()==IdleState.ALL_IDLE){
            ChannelFuture write=ctx.getChannel().write("timeout you will close");
            //设置回调，当write方法结束后执行
            write.addListener(new ChannelFutureListener(){
                //监听操作完成后执行的动作
                @Override
                public void operationComplete(ChannelFuture future)throws Exception{
                    ctx.getChannel().close();
                }
            });
        }
    }
}
```