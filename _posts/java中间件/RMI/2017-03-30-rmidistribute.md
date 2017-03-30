---
title: Java RMI分布式应用
tags: [java_middleware]
---

### 1. 分布式对象应用：

1）定义

一个典型的客户端程序获取远程引用，指向一个或者多个服务端上的远程对象，然后调用这些远程对象所提供的方法。通常我们称这为分布式对象应用程序。

2）过程

首先，查找（定位）远程对象：

应用程序可以使用各种不同的机制取得远程对象的引用。比如应用程序可以通过 RMI 提供的简单的命名工具，RMI注册。或者应用程序可以传递和返回远程对象作为远程方法调用的一部分。

其次，和远程对象通信：

远程对象之间通信的细节由RMI处理，对程序员来说远程对象的通信和通常的Java方法调用没有区别。

注：因为RMI允许对象双向传递，因此它提供了加载对象类定义和传递对象数据的机制。

### RMI注册服务器

具体实例代码查看Java RMI服务博客信息。

1）通过命令行启动

```
rmiregistry 8888
或者：
start rmiregistry 8888
```

2）创建动态注册类

注册服务方式一

```
public static void main(String[] args) {
    try {
        IHello rhello = new HelloImpl();
        Naming.rebind("rmi://localhost:8888/RHello",rhello);
        System.out.println("服务器向命名表注册了1个远程服务对象！");
    } catch (Exception e) {
        e.printStackTrace();
    }
} 
```

注册服务方式二：

```
public static void main(String[] args) {
    try {
        IHello rhello = new HelloImpl();
        //初始化命名空间
        Context namingContext = new InitialContext();
        //将名称绑定到对象,即向命名空间注册已经实例化的远程服务对象
        namingContext.rebind("rmi://localhost:8888/RHello", rhello);
        //Naming.rebind("rmi://localhost:8888/RHello",rhello);
        System.out.println("服务器向命名表注册了1个远程服务对象！");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

运行注册

```
java org.ipph.example.HelloRegister
```

![](/images/middleware/rmi/rmiregister.png)

3）客户端调用

```
java org.ipph.example.HelloClient
```

### java.rmi.Naming和java.rmi.registry.LocateRegistry的区别 

参考：http://blog.csdn.net/xinghun_4/article/details/45772175

1）Naming类

从源码中可以看出，Naming类中的上述方法都是通过Registry类对"远程对象注册表"进行操作的。

如：Naming类的rebind方法

```
public static void rebind(String name, Remote obj)
        throws RemoteException, java.net.MalformedURLException
    {
        ParsedNamingURL parsed = parseURL(name);
        Registry registry = getRegistry(parsed);

        if (obj == null)
            throw new NullPointerException("cannot bind to null");

        registry.rebind(parsed.name, obj);
    }
```

注意: Naming类只是在“远程对象注册表”上进行存储和读取操作，该类并不能创建"远程对象注册表"。

2）LocateRegistry类

LocateRegistry 用于获取特定主机（包括本地主机）上的远程对象注册表的引用，或用于创建一个接受对特定端口调用的远程对象注册表。

客户端代码获取服务器端对象

```
IHello rhello =(IHello) Naming.lookup("rmi://localhost:8888/RHello");
```

查看Naming的lookup源码

```
public static Remote lookup(String name)
    throws NotBoundException,java.net.MalformedURLException,RemoteException
{
    ParsedNamingURL parsed = parseURL(name);
    Registry registry = getRegistry(parsed);

    if (parsed.name == null)
        return registry;
    return registry.lookup(parsed.name);
}

private static Registry getRegistry(ParsedNamingURL parsed)
    throws RemoteException
{
    return LocateRegistry.getRegistry(parsed.host, parsed.port);
}
```

实际上通过LocateRegistry类方法获取到的是Registry对象引用（一个 stub），然后我们可以通过该Registry对象引用对"远程对象注册表"进行操作的。更进一步，没有Naming类照样可以进行所有的操作。

3）不使用Naming类完成注册功能（服务对象注册）

```
public static void main(String[] args) {
    try {
        IHello rhello = new HelloImpl();
        //将名称绑定到对象,即向命名空间注册已经实例化的远程服务对象
        Registry registry=LocateRegistry.getRegistry("localhost",8888);  
        registry.rebind("RHello", rhello);
        System.out.println("服务器向命名表注册了1个远程服务对象！");
    } catch (Exception e) {
        e.printStackTrace();
    }
} 
```

4）不使用Naming类完成远程服务对象查找(客户端代码访问)

```
public static void main(String args[]){
    try {
        //在RMI服务注册表中查找名称为RHello的对象，并调用其上的方法
        //IHello rhello =(IHello) Naming.lookup("rmi://localhost:8888/RHello");
        Registry registry=LocateRegistry.getRegistry("localhost",8888);  
        IHello rhello=(IHello)registry.lookup("RHello");
            
        System.out.println(rhello.helloWorld());
        System.out.println(rhello.sayHelloToSomeBody("hello"));
    } catch (NotBoundException e) {
        e.printStackTrace();
    } catch (RemoteException e) {
        e.printStackTrace();  
    }
}
```

注：因此，Naming类只是封装了Registry接口方法，只需要一个URL就能对"远程对象注册表"进行相关操作，方便了服务对象的注册于查找。