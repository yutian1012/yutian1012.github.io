---
title: Java实现RPC框架
tags: [java]
---

参考：http://www.cnblogs.com/codingexperience/p/5930752.html

### 实现技术方案

下面使用比较原始的方案实现RPC框架，采用Socket通信、动态代理与反射与Java原生的序列化。

### RPC框架架构

RPC架构分为三部分：

1）服务提供者，运行在服务器端，提供服务接口定义与服务实现类。

2）服务中心，运行在服务器端，负责将本地服务发布成远程服务，管理远程服务，提供给服务消费者使用。

3）服务消费者，运行在客户端，通过远程代理对象调用远程服务。

### 具体实现

1）服务提供者接口定义与实现，代码如下：（客户端和服务端都引用该接口）

```
/**
 * 服务提供者接口定义与实现
 */
public interface IHelloService {
    String sayHi(String name);
}
```

2）实现类（服务提供者）

```
/**
 * 服务提供者接口定义与实现
 */
public class HelloServiceImpl implements IHelloService{
    @Override
    public String sayHi(String name) {
        return "Hi, " + name;
    }
}
```

3）服务中心接口（用于注册服务提供者、接收请求socket通信，返回信息）

```
/**
 * 服务中心
 */
public interface Server {
    public void stop();
 
    public void start() throws IOException;
 
    public void register(Class<?> serviceInterface, Class<?> impl);
 
    public boolean isRunning();
 
    public int getPort();
}
```

4）服务中心实现类

```
/**
 * 服务中心
 */
public class ServiceCenter implements Server{
    //运行线程池
    private static ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
     
    private static final HashMap<String, Class<?>> serviceRegistry = new HashMap<String, Class<?>>();
 
    private static boolean isRunning = false;
 
    private int port;
 
    public ServiceCenter(int port) {
        this.port = port;
    }
    /**
     * 关闭服务
     */
    public void stop() {
        isRunning = false;
        executor.shutdown();
    }
    /**
     * 启动服务
     */
    public void start() throws IOException {
        ServerSocket server = new ServerSocket();//启动ServerSocket通信服务
        server.bind(new InetSocketAddress(port));
        System.out.println("start server");
        try {
            while (true) {
                // 1.监听客户端的TCP连接，接到TCP连接后将其封装成task，由线程池执行
                //server.accept方法会阻塞，直到接收到客户端的连接
                executor.execute(new ServiceTask(server.accept()));
            }
        } finally {
            server.close();
        }
    }
    /**
     * 注册服务，将实现放置到map集合中存放
     */
    public void register(Class<?> serviceInterface, Class<?> impl) {
        serviceRegistry.put(serviceInterface.getName(), impl);
    }
 
    public boolean isRunning() {
        return isRunning;
    }
 
    public int getPort() {
        return port;
    }
    /**
     * 远程方法调用，通过接收客户端请求数据，获取服务提供者并调用相应实现方法返回数据
     */
    private static class ServiceTask implements Runnable {
        Socket clent = null;
 
        public ServiceTask(Socket client) {
            this.clent = client;
        }
 
        public void run() {
            ObjectInputStream input = null;
            ObjectOutputStream output = null;
            try {
                // 2.将客户端发送的码流反序列化成对象，反射调用服务实现者，获取执行结果
                input = new ObjectInputStream(clent.getInputStream());
                String serviceName = input.readUTF();
                String methodName = input.readUTF();
                Class<?>[] parameterTypes = (Class<?>[]) input.readObject();
                Object[] arguments = (Object[]) input.readObject();
                Class<?> serviceClass = serviceRegistry.get(serviceName);
                if (serviceClass == null) {
                    throw new ClassNotFoundException(serviceName + " not found");
                }
                Method method = serviceClass.getMethod(methodName, parameterTypes);
                Object result = method.invoke(serviceClass.newInstance(), arguments);
 
                // 3.将执行结果反序列化，通过socket发送给客户端
                output = new ObjectOutputStream(clent.getOutputStream());
                output.writeObject(result);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (output != null) {
                    try {
                        output.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (input != null) {
                    try {
                        input.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (clent != null) {
                    try {
                        clent.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
 
        }
    }
}
```

5）客户端服务代理对象

```
/**
 * 客户端远程代理对象
 */
public class RPCClient<T> {
    @SuppressWarnings("unchecked")
    public static <T> T getRemoteProxyObj(final Class<?> serviceInterface, final InetSocketAddress addr) {
        // 1.将本地的接口调用转换成JDK的动态代理，在动态代理中实现接口的远程调用
        return (T) Proxy.newProxyInstance(serviceInterface.getClassLoader(), new Class<?>[]{serviceInterface},
            new InvocationHandler() {
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    Socket socket = null;
                    ObjectOutputStream output = null;
                    ObjectInputStream input = null;
                    try {
                        // 2.创建Socket客户端，根据指定地址连接远程服务提供者
                            socket = new Socket();
                            socket.connect(addr);
 
                            // 3.将远程服务调用所需的接口类、方法名、参数列表等编码后发送给服务提供者
                            output = new ObjectOutputStream(socket.getOutputStream());
                            output.writeUTF(serviceInterface.getName());
                            output.writeUTF(method.getName());
                            output.writeObject(method.getParameterTypes());
                            output.writeObject(args);
 
                            // 4.同步阻塞等待服务器返回应答，获取应答后返回
                        input = new ObjectInputStream(socket.getInputStream());
                        return input.readObject();
                    } finally {
                        if (socket != null) socket.close();
                        if (output != null) output.close();
                        if (input != null) input.close();
                    }
                }
        });
    }
}
```

6）测试类

```
public class RPCTest {
    public static void main(String[] args) throws IOException {
        //服务注册中心
        new Thread(new Runnable() {
            public void run() {
                try {
                    Server serviceServer = new ServiceCenter(8098);
                    //注册相关的服务实现
                    serviceServer.register(IHelloService.class, HelloServiceImpl.class);
                    serviceServer.start();//启动soket服务
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        //客户端，获取接口代理对象，调用接口方法。
        //底层是通过socket通信的，而对客户端是透明的
        IHelloService service = RPCClient.getRemoteProxyObj(IHelloService.class, new InetSocketAddress("localhost", 8098));

        System.out.println(service.sayHi("test"));
    }
}
```

运行结果

```
start server
Hi, test
```

注：客户端调用时，屏蔽了底层的实现，运行的对象实际是在服务器端进行的，将计算结果回传给客户端。RPC本质为消息处理模型，RPC屏蔽了底层不同主机间的通信细节，让进程调用远程的服务就像是本地的服务一样。

### 改进的地方

这里实现的简单RPC框架是使用Java语言开发，与Java语言高度耦合，并且通信方式采用的Socket是基于BIO实现的，IO效率不高，还有Java原生的序列化机制占内存太多，运行效率也不高。可以考虑从下面几种方法改进。

1）可以采用基于JSON数据传输的RPC框架；

2）可以使用NIO或直接使用Netty替代BIO实现；

3）使用开源的序列化机制，如Hadoop Avro与Google protobuf等；

4）服务注册可以使用Zookeeper进行管理，能够让应用更加稳定。
