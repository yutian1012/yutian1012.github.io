---
title: Netty 简易版（3.x版本）
tags: [architecture]
---

通过Netty源码抽取出一个netty的简易版本，编译了解netty的工作机制

### 从Netty中抽取简单实现

1）查看工作类结构

![](/images/architecture/netty/simple-netty.png)

2）服务器启动类

```
public class Start {

    public static void main(String[] args) { 
        //初始化线程管理类，内部管理boss和worker
        NioSelectorRunnablePool nioSelectorRunnablePool = new 
            NioSelectorRunnablePool(Executors.newCachedThreadPool(), Executors.newCachedThreadPool());
        
        //获取服务类，内部向boss中注册监听端口连接
        ServerBootstrap bootstrap = new 
            ServerBootstrap(nioSelectorRunnablePool);
        
        //绑定端口
        bootstrap.bind(new InetSocketAddress(10101));
        
        System.out.println("start");
    }
}
```

3）ServerBootstrap服务类

该类实现绑定端口，并向Boss中注册任务，启动SelectionKey.OP_ACCEPT客户端监听

```
public class ServerBootstrap {

private NioSelectorRunnablePool selectorRunnablePool;
    
    public ServerBootstrap(NioSelectorRunnablePool selectorRunnablePool) {
        this.selectorRunnablePool = selectorRunnablePool;
    }
    
    /**
     * 绑定端口，向Boss中注册任务，启动SelectionKey.OP_ACCEPT客户端监听
     * @param localAddress
     */
    public void bind(final SocketAddress localAddress){
        try {
            // 获得一个ServerSocket通道
            ServerSocketChannel serverChannel = ServerSocketChannel.open();
            // 设置通道为非阻塞
            serverChannel.configureBlocking(false);
            // 将该通道对应的ServerSocket绑定到port端口
            serverChannel.socket().bind(localAddress);
            
            //获取一个boss线程
            Boss nextBoss = selectorRunnablePool.nextBoss();
            //向boss注册一个ServerSocket通道
            nextBoss.registerAcceptChannelTask(serverChannel);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

4）线程管理类，管理Boss和Worker线程

Selector线程管理类，Boss和Worker都是selector（内部各有各的selector，控制对多个客户端的连接和交互）。

```
public class NioSelectorRunnablePool {
    /**
     * boss线程数组
     */
    private final AtomicInteger bossIndex = new AtomicInteger();
    private Boss[] bosses;

    /**
     * worker线程数组
     */
    private final AtomicInteger workerIndex = new AtomicInteger();
    private Worker[] workeres;

    
    public NioSelectorRunnablePool(Executor boss, Executor worker) {
        initBoss(boss, 1);
        initWorker(worker, Runtime.getRuntime().availableProcessors() * 2);
    }

    /**
     * 初始化boss线程，每个Boss线程的实现类为NioServerBoss
     * @param boss
     * @param count
     */
    private void initBoss(Executor boss, int count) {
        this.bosses = new NioServerBoss[count];
        for (int i = 0; i < bosses.length; i++) {
            bosses[i] = new NioServerBoss(boss, "boss thread " + (i+1), this);
        }

    }

    /**
     * 初始化worker线程，每个worker线程的实现类为NioServerWorker
     * @param worker
     * @param count
     */
    private void initWorker(Executor worker, int count) {
        this.workeres = new NioServerWorker[count];
        for (int i = 0; i < workeres.length; i++) {
            workeres[i] = new NioServerWorker(worker, "worker thread " + (i+1), this);
        }
    }

    /**
     * 获取一个worker，从取余操作中可以看出实现在多个Worker间合理分配客户端交互任务
     * @return
     */
    public Worker nextWorker() {
         return workeres[Math.abs(workerIndex.getAndIncrement() % workeres.length)];

    }

    /**
     * 获取一个boss
     * @return
     */
    public Boss nextBoss() {
         return bosses[Math.abs(bossIndex.getAndIncrement() % bosses.length)];
    }
}
```

5）Boss接口和实现类

Boss的任务队列是为了注册SelectionKey.OP_ACCEPT事件而存在（即接收客户端连接请求），任务队列定义在AbstractNioSelector中。

Boss实现客户端连接的建立

```
public interface Boss {
    
    /**
     * 加入一个新的ServerSocket
     * @param serverChannel
     */
    public void registerAcceptChannelTask(ServerSocketChannel serverChannel);
}
```

实现类NioServerBoss，继承了AbstractNioSelector抽象类

```
public class NioServerBoss extends AbstractNioSelector implements Boss{

    public NioServerBoss(Executor executor, String threadName, NioSelectorRunnablePool selectorRunnablePool) {
        super(executor, threadName, selectorRunnablePool);
    }
    
    /**
     * 处理客户端连接，循环在多个Worker间选择并执行任务
     */
    @Override
    protected void process(Selector selector) throws IOException {
        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        if (selectedKeys.isEmpty()) {
            return;
        }
        
        for (Iterator<SelectionKey> i = selectedKeys.iterator(); i.hasNext();) {
            SelectionKey key = i.next();
            i.remove();
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            // 新客户端
            SocketChannel channel = server.accept();
            // 设置为非阻塞
            channel.configureBlocking(false);
            // 获取一个worker，循环选择下一个worker，达到均分任务的目的
            Worker nextworker = getSelectorRunnablePool().nextWorker();
            // 注册新客户端接入任务
            nextworker.registerNewChannelTask(channel);
            
            System.out.println("新客户端链接");
        }
    }
    
    /**
     * 向任务队列中注册热任务，在任务中执行SelectionKey.OP_ACCEPT事件注册
     */
    public void registerAcceptChannelTask(final ServerSocketChannel serverChannel){
         final Selector selector = this.selector;
         registerTask(new Runnable() {
            @Override
            public void run() {
                try {
                    //注册serverChannel到selector
                    serverChannel.register(selector, SelectionKey.OP_ACCEPT);
                } catch (ClosedChannelException e) {
                    e.printStackTrace();
                }
            }
        });
    }
    
    @Override
    protected int select(Selector selector) throws IOException {
        return selector.select();
    }

}
```

6）worker接口和实现类

Worker的任务队列是为注册SelectionKey.OP_READ事件而存在（即与客户端进行数据交互），任务队列定义在AbstractNioSelector中。

worker实现客户端数据的交互

```
public interface Worker {
    
    /**
     * 加入一个新的客户端会话
     * @param channel
     */
    public void registerNewChannelTask(SocketChannel channel);

}
```

实现类NioServerWorker类，继承了AbstractNioSelector抽象类

```
public class NioServerWorker extends AbstractNioSelector implements Worker{

    public NioServerWorker(Executor executor, String threadName, NioSelectorRunnablePool selectorRunnablePool) {
        super(executor, threadName, selectorRunnablePool);
    }
    /**
     * 执行客户端数据交互
     */
    @Override
    protected void process(Selector selector) throws IOException {
        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        if (selectedKeys.isEmpty()) {
            return;
        }
        Iterator<SelectionKey> ite = this.selector.selectedKeys().iterator();
        while (ite.hasNext()) {
            SelectionKey key = (SelectionKey) ite.next();
            // 移除，防止重复处理
            ite.remove();
            
            // 得到事件发生的Socket通道
            SocketChannel channel = (SocketChannel) key.channel();
            
            // 数据总长度
            int ret = 0;
            boolean failure = true;
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            //读取数据
            try {
                ret = channel.read(buffer);
                failure = false;
            } catch (Exception e) {
                // ignore
            }
            //判断是否连接已断开
            if (ret <= 0 || failure) {
                key.cancel();
                System.out.println("客户端断开连接");
            }else{
                 System.out.println("收到数据:" + new String(buffer.array()));
                 
                //回写数据
                ByteBuffer outBuffer = ByteBuffer.wrap("收到\n".getBytes());
                channel.write(outBuffer);// 将消息回送给客户端
            }
        }
    }

    /**
     * 加入一个新的socket客户端
     * 向任务队列中注册任务，任务中注册SelectionKey.OP_READ事件
     */
    public void registerNewChannelTask(final SocketChannel channel){
         final Selector selector = this.selector;
         registerTask(new Runnable() {
            @Override
            public void run() {
                try {
                    //将客户端注册到selector中
                    channel.register(selector, SelectionKey.OP_READ);
                } catch (ClosedChannelException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    @Override
    protected int select(Selector selector) throws IOException {
        return selector.select(500);
    }
}
```

7）AbstractNioSelector抽象类

抽象selector线程类

```
public abstract class AbstractNioSelector implements Runnable {

    /**
     * 线程池
     */
    private final Executor executor;

    /**
     * 选择器
     */
    protected Selector selector;

    /**
     * 选择器wakenUp状态标记
     */
    protected final AtomicBoolean wakenUp = new AtomicBoolean();

    /**
     * 任务队列
     */
    private final Queue<Runnable> taskQueue = new ConcurrentLinkedQueue<Runnable>();

    /**
     * 线程名称
     */
    private String threadName;
    
    /**
     * 线程管理对象
     */
    protected NioSelectorRunnablePool selectorRunnablePool;

    AbstractNioSelector(Executor executor, String threadName, NioSelectorRunnablePool selectorRunnablePool) {
        this.executor = executor;
        this.threadName = threadName;
        this.selectorRunnablePool = selectorRunnablePool;
        openSelector();
    }

    /**
     * 获取selector并启动线程
     */
    private void openSelector() {
        try {
            this.selector = Selector.open();
        } catch (IOException e) {
            throw new RuntimeException("Failed to create a selector.");
        }
        executor.execute(this);
    }

    @Override
    public void run() {
        
        Thread.currentThread().setName(this.threadName);

        while (true) {
            try {
                //先设置false，防止并发，保证一个一个的执行，该方法其实是一个自旋锁
                wakenUp.set(false);
                
                //两种唤醒方式：
                //方式一：下方的registerTask中的selector.wakeup方法唤醒，
                //     针对NioServerBoss类，只会在ServerBootstrap启动时注册一次registerTask，因此NioServerBoss类的taskQueue中只会出现一次任务，以后一直都是空的
                //     针对NioServerWorker类，会在每次创建客户端连接时都会调用registerTask，然后唤醒执行channel.register(selector, SelectionKey.OP_READ);
                //方式二：selector的等待事件发生，则唤醒
                //     针对NioServerBoss类，有客户端发起连接请求时唤醒
                //     针对NioServerWorker类，客户端发送消息时唤醒，NioServerWorker会同时负责多个客户端channel的交互
                select(selector);

                processTaskQueue();
                
                //模板方法
                process(selector);
            } catch (Exception e) {
                // ignore
            }
        }

    }

    /**
     * 注册一个任务并激活selector
     * 
     * @param task
     */
    protected final void registerTask(Runnable task) {
        taskQueue.add(task);

        Selector selector = this.selector;

        if (selector != null) {
            if (wakenUp.compareAndSet(false, true)) {
                //唤醒，在该类的run方法（即上面的方法）中调用select会处于等待状态，这里唤醒，继续执行run方法中的processTaskQueue方法，
                //从task队列中取出task，并执行run方法
                //针对boss的task任务，为serverChannel.register(selector, SelectionKey.OP_ACCEPT);
                //针对work的task任务，为channel.register(selector, SelectionKey.OP_READ);
                selector.wakeup();
            }
        } else {
            taskQueue.remove(task);
        }
    }

    /**
     * 执行队列里的任务
     */
    private void processTaskQueue() {
        for (;;) {
            final Runnable task = taskQueue.poll();
            if (task == null) {
                break;
            }
            task.run();
        }
    }
    
    /**
     * 获取线程管理对象
     * @return
     */
    public NioSelectorRunnablePool getSelectorRunnablePool() {
        return selectorRunnablePool;
    }

    /**
     * select抽象方法
     * 
     * @param selector
     * @return
     * @throws IOException
     */
    protected abstract int select(Selector selector) throws IOException;

    /**
     * selector的业务处理
     * 
     * @param selector
     * @throws IOException
     */
    protected abstract void process(Selector selector) throws IOException;
}
```

### 执行过程分析

1）运行Start类，启动服务

初始化线程管理类：NioSelectorRunnablePool nioSelectorRunnablePool。初始化Boss线程和Worker线程。调用AbstractNioSelector类的初始化方法。

AbstractNioSelector类的构造函数，并在线程池中运行线程

```
AbstractNioSelector(Executor executor, String threadName, 
    NioSelectorRunnablePool selectorRunnablePool) {
    this.executor = executor;
    this.threadName = threadName;
    this.selectorRunnablePool = selectorRunnablePool;
    openSelector();
}
```

AbstractNioSelector类的openSelector方法

```
/**
 * 获取selector并启动线程
 */
private void openSelector() {
    try {
        this.selector = Selector.open();
    } catch (IOException e) {
        throw new RuntimeException("Failed to create a selector.");
    }
    //在线程池中运行线程
    executor.execute(this);
}
```

查看线程池中运行线程的代码，因为AbstractNioSelector类实现了runable接口，因此查看AbstractNioSelector类的run方法。该方法中会调用selector的select方法，等待相应的事件发生（Boss线程等待客户端连接，Worker线程等待）。

```
@Override
public void run() {
    
    Thread.currentThread().setName(this.threadName);

    while (true) {
        try {
            //先设置false，防止并发，保证一个一个的执行，该方法其实是一个自旋锁
            wakenUp.set(false);
            
            /*两种唤醒方式：
            //方式一：下方的registerTask中的selector.wakeup方法唤醒，
            //     针对NioServerBoss类，只会在ServerBootstrap启动时注册一次registerTask，因此NioServerBoss类的taskQueue中只会出现一次任务，以后一直都是空的
            //     针对NioServerWorker类，会在每次创建客户端连接时都会调用registerTask，然后唤醒执行channel.register(selector, SelectionKey.OP_READ);
            //方式二：selector的等待事件发生，则唤醒
            //     针对NioServerBoss类，有客户端发起连接请求时唤醒
            //     针对NioServerWorker类，客户端发送消息时唤醒，NioServerWorker会同时负责多个客户端channel的交互*/

            //会调用selector.select()
            //Boss类等待客户端连接，
            //Worker类等待客户端交互
            select(selector);

            processTaskQueue();
            
            //模板方法
            process(selector);
        } catch (Exception e) {
            // ignore
        }
    }

}
```

2）运行ServerBootstrap绑定服务

ServerBootstrap绑定端口，向Boss线程中注册任务，会线程池中唤醒等待的NioServerBoss线程，执行ServerSocketChannel接收客户端连接的监听

```
public void bind(final SocketAddress localAddress){
    try {
        // 获得一个ServerSocket通道
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        // 设置通道为非阻塞
        serverChannel.configureBlocking(false);
        // 将该通道对应的ServerSocket绑定到port端口
        serverChannel.socket().bind(localAddress);
        
        //获取一个boss线程
        Boss nextBoss = selectorRunnablePool.nextBoss();
        //向boss注册一个ServerSocket通道
        nextBoss.registerAcceptChannelTask(serverChannel);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

查看NioServerBoss类的registerAcceptChannelTask方法的实现，注册 SelectionKey.OP_ACCEPT事件，等待客户端连接。

```
public void registerAcceptChannelTask(final ServerSocketChannel serverChannel){
     final Selector selector = this.selector;
     registerTask(new Runnable() {
        @Override
        public void run() {
            try {
                //注册serverChannel到selector
                serverChannel.register(selector, SelectionKey.OP_ACCEPT);
            } catch (ClosedChannelException e) {
                e.printStackTrace();
            }
        }
    });
}
```

3）当客户端此时连接服务器

NioServerBoss类的selector.select方法会被唤醒并返回，执行后续方法，可查看AbstractNioSelector的run方法（线程池中运行的等待阻塞方法被唤醒）。processTaskQueue从Boss线程任务队列中获取任务并执行（客户端连接并不会注册任务，因此队列中没有任务）。紧接着执行process方法，查看NioServerBoss类的process方法，用于建立客户端连接SocketChannel，并交由Worker类来管理客户端交互

```
/**
 * 处理客户端连接，循环在多个Worker间选择并执行任务
 */
@Override
protected void process(Selector selector) throws IOException {
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    if (selectedKeys.isEmpty()) {
        return;
    }
    
    for (Iterator<SelectionKey> i = selectedKeys.iterator(); i.hasNext();) {
        SelectionKey key = i.next();
        i.remove();
        ServerSocketChannel server = (ServerSocketChannel) key.channel();
        // 新客户端
        SocketChannel channel = server.accept();
        // 设置为非阻塞
        channel.configureBlocking(false);
        // 获取一个worker，循环选择下一个worker，达到均分任务的目的
        Worker nextworker = getSelectorRunnablePool().nextWorker();
        // 注册新客户端接入任务
        nextworker.registerNewChannelTask(channel);
        
        System.out.println("新客户端链接");
    }
}
```

注：nextworker.registerNewChannelTask(channel);完成向worker的selector中注册channel，并绑定SelectionKey.OP_READ事件，即等待客户端与服务器的交互操作。

NioServerWorker类的registerNewChannelTask方法

```
public void registerNewChannelTask(final SocketChannel channel){
     final Selector selector = this.selector;
     registerTask(new Runnable() {
        @Override
        public void run() {
            try {
                //将客户端注册到selector中
                channel.register(selector, SelectionKey.OP_READ);
            } catch (ClosedChannelException e) {
                e.printStackTrace();
            }
        }
    });
}
```

注：匿名的Runnable实现类会在processTaskQueue中执行run方法。

4）客户端发送数据

worker的selector在注册了channel并绑定了SelectionKey.OP_READ事件后，客户端发过来数据，会唤醒线程池中work相关的selector的select方法，继续执行。由于是由客户端消息发送过来的事件所唤醒，因此worker的taskqueue（任务队列中并没有相关的任务），而只会执行NioServerWorker的process方法，实现与客户端的交互

```
@Override
protected void process(Selector selector) throws IOException {
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    if (selectedKeys.isEmpty()) {
        return;
    }
    Iterator<SelectionKey> ite = this.selector.selectedKeys().iterator();
    while (ite.hasNext()) {
        SelectionKey key = (SelectionKey) ite.next();
        // 移除，防止重复处理
        ite.remove();
        
        // 得到事件发生的Socket通道
        SocketChannel channel = (SocketChannel) key.channel();
        
        // 数据总长度
        int ret = 0;
        boolean failure = true;
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //读取数据
        try {
            ret = channel.read(buffer);
            failure = false;
        } catch (Exception e) {
            // ignore
        }
        //判断是否连接已断开
        if (ret <= 0 || failure) {
            key.cancel();
            System.out.println("客户端断开连接");
        }else{
             System.out.println("收到数据:" + new String(buffer.array()));
             
            //回写数据
            ByteBuffer outBuffer = ByteBuffer.wrap("收到\n".getBytes());
            channel.write(outBuffer);// 将消息回送给客户端
        }
    }
}
```

注：Boss的任务队列是为了注册SelectionKey.OP_ACCEPT事件而存在（即接收客户端连接请求），Worker的任务队列是为注册SelectionKey.OP_READ事件而存在（即与客户端进行数据交互）。