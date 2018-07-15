---
title: Java RMI服务
tags: [java_middleware]
---

Java RMI 指的是远程方法调用 (Remote Method Invocation)。它是一种机制，能够让在某个 Java 虚拟机上的对象调用另一个 Java 虚拟机中的对象上的方法。可以用此方法调用的任何对象必须实现该远程接口。

RMI是Java语言的远程调用，两端的程序语言必须是Java实现，对于不同语言间的通讯可以考虑用Web Service或者公用对象请求代理体系（CORBA）来实现。

注：缺点是只能基于Java语言，客户机与服务器紧耦合

RMI是J2EE的很多分布式技术的基础。

### RMI运行实例

参考：http://lavasoft.blog.51cto.com/62575/91679/

1）定义远程接口

定义一个远程接口，必须继承Remote接口，其中需要远程调用的方法必须抛出RemoteException异常

```
/**
* 定义一个远程接口，必须继承Remote接口，其中需要远程调用的方法必须抛出RemoteException异常
*/
public interface IHello extends Remote {

    /**
     * 简单的返回"Hello World！"字样
     * @return 返回"Hello World！"字样
     * @throws java.rmi.RemoteException
     */
    public String helloWorld() throws RemoteException;

    /**
     * 一个简单的业务方法，根据传入的人名返回相应的问候语
     * @param someBodyName  人名
     * @return 返回相应的问候语
     * @throws java.rmi.RemoteException
     */
    public String sayHelloToSomeBody(String someBodyName) throws RemoteException;
}
```

注：客户端也需要持有该即可，进行与服务器端通信。

2）实现接口类（服务器端代码）

继承UnicastRemoteObject，用于导出的远程对象和获得与该远程对象通信的存根。

```
/**
* 远程的接口的实现
*/
public class HelloImpl extends UnicastRemoteObject implements IHello {
    /**
     * 因为UnicastRemoteObject的构造方法抛出了RemoteException异常，因此这里默认的构造方法必须写，必须声明抛出RemoteException异常
     *
     * @throws RemoteException
     */
    public HelloImpl() throws RemoteException {
    }

    /**
     * 简单的返回"Hello World！"字样
     *
     * @return 返回"Hello World！"字样
     * @throws java.rmi.RemoteException
     */
    public String helloWorld() throws RemoteException {
        return "Hello World!";
    }

    /**
     * 一个简单的业务方法，根据传入的人名返回相应的问候语
     * @param someBodyName 人名
     * @return 返回相应的问候语
     * @throws java.rmi.RemoteException
     */
    public String sayHelloToSomeBody(String someBodyName) throws RemoteException {
        return "你好，" + someBodyName + "!";
    }
}
```

3）启动远程服务，并将服务器类注册到服务器中，供客户端调用

```
/**
* 创建RMI注册表，启动RMI服务，并将远程对象注册到RMI注册表中。
*/
public class HelloServer {
    public static void main(String args[]) {

        try {
            //创建一个远程对象
            IHello rhello = new HelloImpl();
            //本地主机上的远程对象注册表Registry的实例，并指定端口为8888，这一步必不可少（Java默认端口是1099），必不可缺的一步，缺少注册表创建，则无法绑定对象到远程注册表上
            LocateRegistry.createRegistry(8888);

            //把远程对象注册到RMI注册服务器上，并命名为RHello
            //绑定的URL标准格式为：rmi://host:port/name(其中协议名可以省略，下面两种写法都是正确的）
            Naming.bind("rmi://localhost:8888/RHello",rhello);

            System.out.println(">>>>>INFO:远程IHello对象绑定成功！");
        } catch (RemoteException e) {
            System.out.println("创建远程对象发生异常！");
            e.printStackTrace();
        } catch (AlreadyBoundException e) {
            System.out.println("发生重复绑定对象异常！");
            e.printStackTrace();
        } catch (MalformedURLException e) {
            System.out.println("发生URL畸形异常！");
            e.printStackTrace();
        }
    }
}
```

4）客户端调用

```
/**
* 客户端测试，在客户端调用远程对象上的远程方法，并返回结果。
*/
public class HelloClient {
    public static void main(String args[]){
        try {
            //在RMI服务注册表中查找名称为RHello的对象，并调用其上的方法
            IHello rhello =(IHello) Naming.lookup("rmi://localhost:8888/RHello");
            System.out.println(rhello.helloWorld());
            System.out.println(rhello.sayHelloToSomeBody("hello"));
        } catch (NotBoundException e) {
            e.printStackTrace();
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (RemoteException e) {
            e.printStackTrace();  
        }
    }
}
```

5）运行过程

运行HelloServer类启动rmi服务器

```
java org.ipph.example.HelloServer
```


![](/images/middleware/rmi/rmiServer.png)

重新打开一个cmd窗口，运行客户端

```
java org.ipph.example.HelloClient
```

![](/images/middleware/rmi/rmiclient.png)

远程对象的引用通常是通过RMI的注册表服务以及java.rmi.Naming接口获得的。远程对象需要导出（注册）相应的远程引用到注册表服务，之后注册表服务就可以监听并服务于客户端对远程对象引用的请求。标准的 Sun Java SDK提供了一个简单的RMI注册表服务程序，即rmiregistry用于监听特定的端口，等待远程对象的注册，以及客户端对这些远程对象引用的检索请求。在运行程序之前，首先要启动RMI的注册表服务（程序中通过LocateRegistry.createRegistry(8888);启动了RMI注册表服务）。这个过程很简单，只要直接运行rmiregistry命令即可。缺省的情况下，该服务将监听 1099 端口。如果需要指定其它的监听端口，可以在命令行指定希望监听的端口（如果你指定了其它端口，需要修改示例程序以适应环境）。

