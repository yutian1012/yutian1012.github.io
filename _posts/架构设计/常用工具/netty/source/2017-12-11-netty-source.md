---
title: Netty 源码
tags: [architecture]
---

1）查看源码：

创建一个maven项目，然后在maven中引入依赖

```
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty</artifactId>
    <version>3.10.5.Final</version>
</dependency>
```

2）创建测试类并测试

使用netty入门案例作为netty源码查看的入口。

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

注：上面的方式能够正常的查看到源码的实现，但并不能在源码中加入system输出语句。

3）在eclipse中引入netty的源码

netty项目的源码：

```
https://github.com/netty/netty
```

运行gitclone项目到本地

```
git clone https://github.com/netty/netty.git
```

切换版本到3.10.5版本

```
git tag -l 查看远程的tag信息
git checkout netty-3.5.10.Final 获取tag标签处的代码
git status 查看当前的代码情况。
```

4）将下载的netty导入到eclipse中

右击--import--exists maven project导入项目

然后配置源码关联为本地项目：在测试项目上右键--java build path--souce attachment

![](/images/architecture/netty/netty-source.png)

注：在maven项目中关联源码会默认使用maven本地仓库中下载的源码包，不能正确关联到新建的项目代码。

5）在源码中直接添加测试代码

在源码项目下创建测试类，并运行。

注：最好的方式就是在源码项目中运行

6）为断点设置条件

AbstractNioWorker类的实现类包括NioWorker和NioDatagramWorker。在AbstractNioWorker上（即实现类的父类）的run方法添加一个断点，当NioWorker类的线程进入该run方法时，才进入断点，而NioDatagramWorker线程则忽略。

可以在断点上设置条件：断点上右击--breakpoint properties--condition，设置条件：

```
this.thread.currentThread().getClass().getName().contains("NioWorker");
```

![](/images/architecture/netty/netty-source-breakpoint.png)