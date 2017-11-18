---
title: CORBA开发实例
tags: [architecture]
---

参考：http://blog.csdn.net/drykilllogic/article/details/25971915

开发一个CORBA的Helloworld实例

### 使用idl语言开发idl文件

该文件中描述了接口的定义，使用IDL语言定义接口，接口文件为helloworld.idl

```
module corba{  
    interface HelloWorld{  
        string sayHello();  
    };  
};
```

module：对应了java中的package

interface：对应了java中的interface，HelloWorld即接口名称

sayHello：对应了java中interface声明的方法

string：对应了java中方法的返回值

### 由idl文件生成java类

将idl语言翻译成java语言，并生成java代码，jdk命令idlj（位于%JAVA_HOME%\bin下）

运行命令

```
idlj -td D:/ -fall helloworld.idl
```

注：-td表示生成的文件存放目录（todirectory），-fall，其中all只生成所有的代码（client，server端代码）。

将生成的代码拷贝到项目中

![](/images/middleware/corba/corba-gencode.png)

1）java是client需要的代码：

HelloWorldStub.java、HelloWorld.java、HelloWorldHelper.java、HelloWorldHolder.java、HelloWorldOperations

注：Sutb客户端的存根类，为客户端提供了CORBA功能；HelloWorld.java接口类，提供了标准的CORBA对象功能；Helper是一个辅助类，负责向CORBA流中写入或读取对象；Holder 这是一个final类，它持有一个public的Hello实例变量，用来操作CORBA输入输出流的参数；Operations类只包含我们定义的那个方法，不包含CORBA的任何东西。

2）java是server需要的代码：

HelloWorld.java、HelloWorldOperations.java、HelloWorldPOA

注：POA对象（Portable Object Adapter轻便对象适配器），提供了服务器端基本的CORBA功能

3）生成包替换命令

```
idlj -td D:/ -pkgTranslate corba org.ipph.web.corba.idl -fall helloworld.idl
```

注：会将corba模块对于的包替换为org.ipph.web.corba.idl包名


### server端代码

1）创建实现类

创建HelloWorldImpl类，继承生成的代码HelloWorldPOA。

```
public class HelloWorldImpl extends HelloWorldPOA{
    @Override
    public String sayHello() {
        return "helloworld";
    }
}
```

2）启动服务类

server启动的代码，这段代码主要功能是将接口实现注册到ORB中，并启动监听，等待client来调用

```
import org.omg.CORBA.ORB;
import org.omg.CORBA.ORBPackage.InvalidName;
import org.omg.CosNaming.NameComponent;
import org.omg.CosNaming.NamingContextExt;
import org.omg.CosNaming.NamingContextExtHelper;
import org.omg.CosNaming.NamingContextPackage.CannotProceed;
import org.omg.CosNaming.NamingContextPackage.NotFound;
import org.omg.PortableServer.POA;
import org.omg.PortableServer.POAHelper;
import org.omg.PortableServer.POAManagerPackage.AdapterInactive;
import org.omg.PortableServer.POAPackage.ServantNotActive;
import org.omg.PortableServer.POAPackage.WrongPolicy;

import corba.HelloWorld;
import corba.HelloWorldHelper;
public class HelloServer {  
    public static void main(String[] args) throws ServantNotActive, WrongPolicy, InvalidName, AdapterInactive, org.omg.CosNaming.NamingContextPackage.InvalidName, NotFound, CannotProceed {  
        //指定ORB的端口号 -ORBInitialPort 1050
        
        args = new String[2];  
        args[0] = "-ORBInitialPort";  
        args[1] = "1050";  

        //也可以将端口设置到属性对象上
        //props.put("org.omg.CORBA.ORBInitialPort", "1050");
           
        //创建一个ORB实例  
        ORB orb = ORB.init(args, null); //第二个参数可接收属性对象
           
        //拿到RootPOA的引用，并激活POAManager，相当于启动了server
        org.omg.CORBA.Object obj=orb.resolve_initial_references("RootPOA");  
        POA rootpoa = POAHelper.narrow(obj);
        rootpoa.the_POAManager().activate();  
           
        //创建一个HelloWorldImpl实例  
        HelloWorldImpl helloImpl = new HelloWorldImpl();  
          
        //从服务中得到对象的引用，并注册到服务中  
        org.omg.CORBA.Object ref = rootpoa.servant_to_reference(helloImpl);  
        HelloWorld href = HelloWorldHelper.narrow(ref);  
           
        //得到一个根名称的上下文  
        org.omg.CORBA.Object objRef = orb.resolve_initial_references("NameService");  
        NamingContextExt ncRef = NamingContextExtHelper.narrow(objRef);  
          
        //在命名上下文中绑定这个对象  
        String name = "Hello";  
        NameComponent path[] = ncRef.to_name(name);  
        ncRef.rebind(path, href);  
          
        //启动线程服务，等待客户端调用  
        orb.run();  
        
        System.out.println("server startup...");  
    }  
} 
```

### 客户端代码

```
package org.ipph.web.corba.client;

  
import org.omg.CORBA.ORB;  
import org.omg.CORBA.ORBPackage.InvalidName;  
import org.omg.CosNaming.NamingContextExt;  
import org.omg.CosNaming.NamingContextExtHelper;  
import org.omg.CosNaming.NamingContextPackage.CannotProceed;  
import org.omg.CosNaming.NamingContextPackage.NotFound;  

import corba.HelloWorld;
import corba.HelloWorldHelper;
  
  
public class HelloClient {  
    static HelloWorld helloWorldImpl;  
       
    static {  
        System.out.println("client开始连接server.......");  
           
        //初始化ip和端口号，-ORBInitialHost 127.0.0.1 -ORBInitialPort 1050  
        String args[] = new String[4];  
        args[0] = "-ORBInitialHost";  
        //server端的IP地址，在HelloServer中定义的  
        args[1] = "127.0.0.1";  
        args[2] = "-ORBInitialPort";  
        //server端的端口，在HelloServer中定义的  
        args[3] = "1050";  
           
        //创建一个ORB实例  
        ORB orb = ORB.init(args, null);  
           
        // 获取根名称上下文  
        org.omg.CORBA.Object objRef = null;  
        try {  
        objRef = orb.resolve_initial_references("NameService");  
        } catch (InvalidName e) {  
            e.printStackTrace();  
        }  
        NamingContextExt neRef = NamingContextExtHelper.narrow(objRef);  
           
        String name = "Hello";  
        try {  
              
            //通过ORB拿到了server实例化好的实现类  
            helloWorldImpl = HelloWorldHelper.narrow(neRef.resolve_str(name));  
        } catch (NotFound e) {  
            e.printStackTrace();  
        } catch (CannotProceed e) {  
            e.printStackTrace();  
        } catch (org.omg.CosNaming.NamingContextPackage.InvalidName e) {  
            e.printStackTrace();  
        }  
           
        System.out.println("client connected server.......");  
    }  
       
    public static void main(String args[]) throws Exception {  
        sayHello();  
    }  
       
    //调用实现类的方法  
    public static void sayHello() {  
        String str = helloWorldImpl.sayHello();  
        System.out.println(str);  
    }  
}  
```

注：server和client都创建了一个ORB的实例，因为在server和client都需要支持通信。

### 启动orbd服务

在%JAVA_HOME%\jre\bin下提供了一个orbd服务，orbd包含自启动服务、透明的命名服务、持久化命名服务和命名管理器的后台处理进程。

cmd中运行

```
orbd -ORBInitialPort 1050 -ORBInitialHost 127.0.0.1
```

注：这里的端口号要与server中指定的端口号相同

### 运行测试

先运行HelloServer服务，在运行HelloClient客户端。

### idlj生成的java文件注释乱码

