---
title: Netty 心跳（3.x版本）
tags: [architecture]
---

参考：http://blog.csdn.net/linuu/article/details/51360609

protobuf是由Google开发的一套对数据结构进行序列化的方法，可用做通信协议，数据存储格式，等等。其特点是不限语言、不限平台、扩展性强。

protobuf与xml，json这样的数据格式一样，都有自己的一套语法，且语法很简单，很容易掌握，xml文件的后缀名是xml，json的后缀名是json，以此类推，那么protobuf的后缀名就是proto。

protobuf语法参考：http://www.cnblogs.com/dkblog/archive/2012/03/27/2419010.html

### 场景

客户端发送一个信息，这个信息用Protobuf来做序列化，然后服务器端接收这个信息，解码，读取信息。

1）定义一个proto文件

定义一个"富人"类，他有多辆车，写一个RichMan.proto文件。该类定义了一个富人，该富人有id，名称，邮箱，而且该富人有多个名车（集合），这些名车的类型有奥迪，奔驰，兰博基尼，大众（枚举实现）。

```
package netty;  
  
option java_package = "com.lyncc.netty.codec.protobuf.demo";  
option java_outer_classname = "RichManProto";  
  
message RichMan {  
  
   required int32 id = 1;  
   required string name = 2;  
   optional string email = 3;  
     
   enum CarType {  
     AUDI = 0;  
     BENZ = 1;  
     LAMBORGHINI = 2;  
     DASAUTO = 3;  
   }  
     
   message Car {  
      required string name = 1;  
      optional CarType type = 2 [default = BENZ];  
   }  
     
   repeated Car cars = 4;  
} 
```

语法的解释

java_package值得是该文件生成的java文件的包路径

java_outer_classname值的是生成的class的名称

message和enum是它的基本类型，很类似于java的class和枚举

required表名这个字段是必须的，option表明这个字段可选，default表明这个字段有默认值

repeat表明这个字段可以重复，类似于java中的List，该例子中Car的声明中，就相当于java中的List<Car>。

每个声明成员变量后面的数字，例如1，2，3,4等等，表示成员变量的顺序（同级的声明不能重复）。

2）使用工具转成java文件

Google提供了一个类似脚本的工具，可以将proto文件转化成java文件。

工具下载地址及编译参考：

```
https://www.cnblogs.com/xuhuajie/p/6890252.html
```

编译好直接可以使用的工具

```
https://github.com/google/protobuf/releases
```

使用压缩包下的protoc.exe工具生成java代码（exe文件必须与.proto文件位于同一目录中），在cmd中运行

```
protoc.exe ./RichMan.proto --java_out=./
```

3）将生成的java文件拷贝到eclipse的项目中

引入依赖的protobuf的jar包

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.1.0</version>
</dependency>
```

注：这里引入的jar的版本要与生成java代码使用的protobuf版本统一。

4）protobuf的序列化和反序列化

```
package com.lyncc.netty.codec.protobuf.demo;

import com.google.protobuf.InvalidProtocolBufferException;
import com.lyncc.netty.codec.protobuf.demo.RichManProto.RichMan;
import com.lyncc.netty.codec.protobuf.demo.RichManProto.RichMan.Builder;

public class PB2Byte {
    
    public static void main(String[] args) throws InvalidProtocolBufferException {
        byte[] bs=toBytes();
        toObject(bs);
    }
    
    public static byte[] toBytes() {
        //获取构造器
        Builder builder=RichManProto.RichMan.newBuilder();
        //设置数据内容并返回对象
        //addCar方法对集合添加数据
        //针对require的字段必须要设置值，否则序列化数据是会抛出异常。
        RichMan richMan=builder.setId(1).setEmail("xxxx").setName("ddd").build();
        //序列化对象
        byte[] byteArray=richMan.toByteArray();
        
        return byteArray;
    }
    
    public static void toObject(byte[] byteArray) throws InvalidProtocolBufferException {
         RichMan richMan = RichManProto.RichMan.parseFrom(byteArray);
         
         System.out.println(richMan.getId());
         System.out.println(richMan.getName());
         System.out.println(richMan.getEmail());
    }
}
```

注：使用Builder方式来创建对象。

5）java本身也能进行序列化操作，为什么要使用protobuf呢？

java本身存在ObjectOutputStream对象用于序列化，ObjectInputStream对象用于反序列化操作。但是比较生成的字节码大小能够发现java生成的字节码使用protobuf生成字节码的好几倍。数据量小能够节省很多带宽，传输效率更高。

protobuf序列化和反序列化都需要.proto配置文件，java是不需要的。java将所有的信息都保存到了序列化的字节数组中，而protobuf确并没有将所有的信息保存到字节数组中，而只是将必要信息存入数组中，反序列化时会根据相关的配置信息生成对象。配置信息在客户端和服务器端都是存在的。

protobuf的字段类型是有伸缩性的，如int在java中占4个字节，而protobuf却会根据值的实际大小分配1-5个字节。

### netty 与 protobuf实例

1）客户端代码

在客户端使用protobuf工具生成java代码，导入到eclipse中编写与服务器端的交互

```
package com.lyncc.netty.codec.protobuf.demo;

import java.net.InetSocketAddress;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.jboss.netty.bootstrap.ClientBootstrap;
import org.jboss.netty.channel.Channel;
import org.jboss.netty.channel.ChannelFuture;
import org.jboss.netty.channel.ChannelPipeline;
import org.jboss.netty.channel.ChannelPipelineFactory;
import org.jboss.netty.channel.Channels;
import org.jboss.netty.channel.socket.nio.NioClientSocketChannelFactory;
import org.jboss.netty.handler.codec.protobuf.ProtobufEncoder;
import org.jboss.netty.handler.codec.protobuf.ProtobufVarint32LengthFieldPrepender;

public class PBClient {
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
                //protobuf对java类型的处理
                pipeline.addLast("encoder1",new ProtobufVarint32LengthFieldPrepender());
                //编码器，将对象进行序列化操作
                pipeline.addLast("encoder", new ProtobufEncoder());
                //消息接收发送处理类
                pipeline.addLast("ProtoBufClientHandler", new ProtoBufClientHandler());
                return pipeline;
            }
        });
        
        //连接服务端
        ChannelFuture connect = bootstrap.connect(
            new InetSocketAddress("127.0.0.1", 10101));
        //channel理解为一个会话
        Channel channel = connect.getChannel();
        
    }
}
```

消息接收处理类

```
package com.lyncc.netty.codec.protobuf.demo;

import java.util.ArrayList;  
import java.util.List;

import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.channel.ChannelStateEvent;
import org.jboss.netty.channel.ExceptionEvent;
import org.jboss.netty.channel.SimpleChannelHandler;

import com.lyncc.netty.codec.protobuf.demo.RichManProto.RichMan.Car;  
import com.lyncc.netty.codec.protobuf.demo.RichManProto.RichMan.CarType;  
  
public class ProtoBufClientHandler extends SimpleChannelHandler {

    /**
     * 新连接
     */
    @Override
    public void channelConnected(ChannelHandlerContext ctx, ChannelStateEvent e) throws Exception {
        System.out.println("=======================================");  
        RichManProto.RichMan.Builder builder = RichManProto.RichMan.newBuilder();  
        builder.setName("王思聪");  
        builder.setId(1);  
        builder.setEmail("wsc@163.com");  
          
        List<RichManProto.RichMan.Car> cars = new ArrayList<RichManProto.RichMan.Car>();  
        Car car1 = RichManProto.RichMan.Car.newBuilder().setName("上海大众超跑").setType(CarType.DASAUTO).build();  
        Car car2 = RichManProto.RichMan.Car.newBuilder().setName("Aventador").setType(CarType.LAMBORGHINI).build();  
        Car car3 = RichManProto.RichMan.Car.newBuilder().setName("奔驰SLS级AMG").setType(CarType.BENZ).build();  
          
        cars.add(car1);  
        cars.add(car2);  
        cars.add(car3);  
          
        builder.addAllCars(cars);  
        ctx.getChannel().write(builder.build());
    }

    /**
     * 捕获异常
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) throws Exception {
        System.out.println("exceptionCaught");
        super.exceptionCaught(ctx, e);
    }

}
```

注：在客户端与服务创建连接后即向服务器端发送数据。

2）服务器端代码实现

在服务器端使用protobuf工具生成java代码，导入到eclipse中编写与客户端交互，反序列化客户端发送的数据。

```
package com.lyncc.netty.codec.protobuf.demo;

import java.net.InetSocketAddress;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.jboss.netty.bootstrap.ServerBootstrap;
import org.jboss.netty.channel.ChannelPipeline;
import org.jboss.netty.channel.ChannelPipelineFactory;
import org.jboss.netty.channel.Channels;
import org.jboss.netty.channel.socket.nio.NioServerSocketChannelFactory;
import org.jboss.netty.handler.codec.protobuf.ProtobufDecoder;
import org.jboss.netty.handler.codec.protobuf.ProtobufVarint32FrameDecoder;

public class PBServer {
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
                //protobuf对java类型的处理
                pipeline.addLast("decoder1", new ProtobufVarint32FrameDecoder());
                //服务器端对客户端发送的protobuf对象进行反序列化
                pipeline.addLast("decoder", new ProtobufDecoder(RichManProto.RichMan.getDefaultInstance()));
                //接收客户端消息处理类
                pipeline.addLast("ProtoBufServerHandler", new ProtoBufServerHandler());
                return pipeline;
            }
        });
        
        bootstrap.bind(new InetSocketAddress(10101));
        
        System.out.println("start!!!");
    }
}
```

注：第一个Decoder（ProtobufVarint32FrameDecoder）是将帧byte数据转化成message，第二步（ProtobufDecoder）就是将message转化成我们自定义的Rimanproto。

消息接收处理类

```
package com.lyncc.netty.codec.protobuf.demo;

import java.util.List;

import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.channel.ExceptionEvent;
import org.jboss.netty.channel.MessageEvent;
import org.jboss.netty.channel.SimpleChannelHandler;

import com.lyncc.netty.codec.protobuf.demo.RichManProto.RichMan.Car;

public class ProtoBufServerHandler extends SimpleChannelHandler {

    /**
     * 接收消息，telnet发送消息时触发
     */
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {
        RichManProto.RichMan req = (RichManProto.RichMan) e.getMessage();
        System.out.println(req.getName()+"他有"+req.getCarsCount()+"量车");  
        List<Car> lists = req.getCarsList();  
        if(null != lists) {  
              
            for(Car car : lists){  
                System.out.println(car.getName());  
            }  
        }
    }

    /**
     * 捕获异常，messageReceived方法抛出异常时，会触发该方法
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) throws Exception {
        System.out.println("exceptionCaught");
        super.exceptionCaught(ctx, e);
    }
}
```

