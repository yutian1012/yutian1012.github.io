---
title: Netty 源码
tags: [architecture]
---

问题：多个selector如何协作？？

线程池是如何利用的？？

pipeline如何执行的？？

如何理解基于nio实现？？

netty数据包协议？？

netty线程模型？？

netty架构剖析？？

缓冲区，选择器，通道？？

protocol buff??

粘包分包socket工具？？

直播聊天室实战？？

1）分析源码实例

```
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

2）NioServerSocketChannelFactory类

该类构建时传递boss和worker线程池，通过构造函数可以看出创建的worker线程数量为默认的cpu内核数量*2（即2倍的cpu内核数量），默认的boss线程数量是1。

核心类AbstractNioWorker

ChannelPipeline管道的处理（fireMessageReceived方法）