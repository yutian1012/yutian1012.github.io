---
title: Java RMI服务
tags: [java_middleware]
---

Java RMI 指的是远程方法调用 (Remote Method Invocation)。它是一种机制，能够让在某个 Java 虚拟机上的对象调用另一个 Java 虚拟机中的对象上的方法。可以用此方法调用的任何对象必须实现该远程接口。

RMI是Java语言的远程调用，两端的程序语言必须是Java实现，对于不同语言间的通讯可以考虑用Web Service或者公用对象请求代理体系（CORBA）来实现。

RMI是J2EE的很多分布式技术的基础。

### 1. RMI运行实例

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

### 2. 原理分析

参考：http://www.blogjava.net/yruc/archive/2007/01/11/93215.html

1）对RPC的改进

远程过程调用（ Remote Procedure Call, RPC ）可以用于一个进程调用另一个进程（很可能在另一个远程主机上）中的过程，从而提供了过程的分布能力。Java的RMI则在RPC的基础上向前又迈进了一步，即提供分布式对象间的通讯，允许我们获得在远程进程中的对象（称为远程对象）的引用（称为远程引用），进而通过引用调用远程对象的方法，就好像该对象是与你的客户端代码同样运行在本地进程中一样。 RMI 使用了术语" 方法"（ Method ）强调了这种进步，即在分布式基础上，充分支持面向对象的特性。

2）RMI-IIOP

RMI 并不是 Java 中支持远程方法调用的唯一选择。在 RMI 基础上发展而来的 RMI-IIOP （ Java Remote Method Invocation over the Internet Inter-ORB Protocol ），不但继承了 RMI 的大部分优点，并且可以兼容于 CORBA 。 J2EE 和 EJB 都要求使用 RMI-IIOP 而不是 RMI 。RMI 使用 java.rmi 包，而 RMI-IIOP 则既使用 java.rmi 也使用扩展的 javax.rmi 包。

3）分布式对象

分布式对象（Distributed Object）, 指一个对象可以被远程系统所调用。对于 Java 而言，即对象不仅可以被同一虚拟机中的其他客户程序（Client）调用，也可以被运行于其他虚拟机中的客户程序调用，甚至可以通过网络被其他远程主机之上的客户程序调用。

4）分布式对象被调用的过程是这样的：

首先：客户程序调用一个被称为Stub（有时译作存根，为了不产生歧义，本文将使用其英文形式）的客户端代理对象。该代理对象负责对客户端隐藏网络通讯的细节。Stub知道如何通过网络套接字socket发送调用，包括如何将调用参数转换为适当的形式以便传输等。

然后：Stub通过网络将调用传递到服务器端，也就是分布对象一端的一个被称为Skeleton（骨架）的代理对象。同样，该代理对象负责对分布式对象隐藏网络通讯的细节。Skeleton 知道如何从网络套接字（Socket）中接受调用，包括如何将调用参数从网络传输形式转换为Java形式等。

最后：Skeleton将调用传递给分布式对象。分布式对象执行相应的调用，之后将返回值传递给Skeleton，进而传递到Stub，最终返回给客户程序。

这个场景基于一个基本的法则，即行为的定义和行为的具体实现相分离。客户端代理对象Stub和分布式对象都实现了相同的接口，该接口称为远程接口（Remote Interface）。正是该接口定义了行为，而分布式对象本身则提供具体的实现。对于Java RMI而言，我们用接口（interface）定义行为，用类（class）定义实现。 

![](/images/middleware/rmi/rmiinvoke.jpg)

### 3. 具体过程

1）客户程序调用Stub

Stub 负责将方法的参数转换为序列化（Serialized）形式，我们使用一个特殊的术语，即编列（Marshal）来指代这个过程。编列的目的是将这些参数转换为可移植的形式，从而可以通过网络传输到远程的服务对象一端。不幸的是，这个过程没有想象中那么简单。这里我们首先要理解一个经典的问题，即方法调用时，参数究竟是传值还是传引用呢？对于Java RMI 来说，存在四种情况，我们将分别加以说明。

情况一：

对于基本的原始类型（整型，字符型等等），将被自动的序列化，以传值的方式编列。

情况二：

对于 Java 的对象，如果该对象是可序列化的（实现了java.io.Serializable接口），则通过 Java 序列化机制自动地加以序列化，以传值的方式编列。对象之中包含的原始类型以及所有被该对象引用，且没有声明为transient的对象也将自动的序列化。当然，这些被引用的对象也必须是可序列化的。

情况三：

绝大多数内建的 Java 对象都是可序列化的。对于不可序列化的Java对象（java.io.File 最典型），或者对象中包含不可序列化对象，且没有声明为transient的其它对象的引用。则编列过程将向客户程序抛出异常，而宣告失败。

情况四：

客户程序可以调用远程对象，没有理由禁止调用参数本身也是远程对象（实现了 java.rmi.Remote 接口的类的实例）。此时，RMI采用一种模拟的传引用方式（当然不是传统意义的传引用，因为本地对内存的引用到了远程变得毫无意义），而不是将参数直接编列复制到远程。这种情况下，交互的双方发生的戏剧性变化值得我们注意。参数是远程对象，意味着该参数对象可以远程调用。当客户程序指定远程对象作为参数调用服务器端远程对象的方法时，RMI的运行时机制将向服务器端的远程对象发送作为参数的远程对象的一个 Stub 对象。这样服务器端的远程对象就可以回调（ Callback ）这个 Stub 对象的方法，进而调用在客户端的远程对象的对应方法。通过这种方法，服务器端的远程对象就可以修改作为参数的客户端远程对象的内部状态，这正是传统意义的传引用所具备的特性。是不是有点晕？ ( 那确实 ) 这里的关键是要明白，在分布式环境中，所谓服务器和客户端都是相对的。被请求的一方就是服务器，而发出请求的一方就是客户端。

2）客户端传输请求（透明的数据传输）

在调用参数的编列过程成功后，客户端的远程引用层从Stub那里获得了编列后的参数以及对服务器端远程对象的远程引用（参见java.rmi.server.RemoteRef API）。该层负责将客户程序的请求依据底层的RMI数据传输协议转换为传输层请求。在 RMI 中，有多种的可能的传输机制，比如点对点（ Point-to-Point ）以及广播（ Multicast ）等。不过，在当前的JMI版本中只支持点对点协议，即远程引用层将生成唯一的传输层请求，发往指定的唯一远程对象（参见 java.rmi.server.UnicastRemoteObject API ）。

3）服务器端接收请求

在服务器端，服务器端的远程引用层接收传输层请求，并将其转换为对远程对象的服务器端代理对象Skeleton的调用。Skeleton对象负责将请求转换为对实际的远程对象的方法调用。这是通过与编列过程相对的反编列（Unmarshal）过程实现的。所有序列化的参数被转换为Java对象形式，其中作为参数的远程对象（实际上发送的是远程引用）被转换为服务器端本地的Stub对象。

4）服务器端返回请求处理结果

如果方法调用有返回值或者抛出异常，则Skeleton负责编列返回值或者异常，通过服务器端的远程引用层，经传输层传递给客户端；相应地，客户端的远程引用层和Stub负责反编列并最终将结果返回给客户程序。

整个过程中，可能最让人迷惑的是远程引用层。这里只要明白，本地的Stub对象是如何产生的，就不难理解远程引用的意义所在了。远程引用中包含了其所指向的远程对象的信息，该远程引用将用于构造作为本地代理对象的Stub对象。构造后，Stub对象内部将维护该远程引用。真正在网络上传输的实际上就是这个远程引用，而不是Stub对象。

### 4. RMI对象

在RMI的基本架构之上，RMI提供服务与分布式应用程序的一些对象服务，包括对象的命名 /注册（Naming/Registry）服务，远程对象激活（Activation）服务以及分布式垃圾收集（DistributedGarbage Collection, DGC ）。

问题：客户端要调用远程对象，是通过其代理对象Stub完成的，那么Stub最早是从哪里得来的呢？

RMI 的命名 / 注册服务正是解决这一问题的。当服务器端想向客户端提供基于RMI服务时，它需要将一个或多个远程对象注册到本地的 RMI 注册表中（参见 java.rmi.registry.Registry API ）。每个对象在注册时都被指定一个将来用于客户程序引用该对象的名称。客户程序通过命名服务（参见 java.rmi.Naming API ），指定类似 URL 的对象名称就可以获得指向远程对象的远程引用。在 Naming 中的 lookup() 方法找到远程对象所在的主机后，它将检索该主机上的 RMI 注册表，并请求所需的远程对象。如果注册表发现被请求的远程对象，它将生成一个对该远程对象的远程引用，并将其返回给客户端，客户端则基于远程引用生成相应的 Stub对象，并将引用传递给调用者。之后，双方就可以按照我们前面讲过的方式进行交互了。

注意：RMI 命名服务提供的 Naming 类并不是你的唯一选择。 RMI 的注册表可以与其他命名服务绑定，比如 JNDI ，这样你就可以通过 JNDI 来访问 RMI 的注册表了。 

![](/images/middleware/rmi/rmiprotocol.JPG)
