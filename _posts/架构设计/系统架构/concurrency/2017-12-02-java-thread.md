---
title: java中的线程
tags: [architecture]
---

线程是进程内的执行单元。java中的线程会直接映射到操作系统的线程上。

1）java中线程状态的切换

![](/images/architecture/concurrency/thread-status.jpg)

2）线程的调用：

Thread的start方法是在当前系统上创建一个线程，并等待cpu的调度执行。

Thread的run方法不能真正开启一个线程，而是在原线程的基础上运行，因此不要使用run方法。

Thread的stop方法，不推荐使用，它会释放所有的monitor（锁）。很可能导致数据不一致的问题。

3）实现线程的两种方式

方式一：重写Thread的run方法

```
Thread t1=new Thread(){
    @Override
    public void run(){
        System.out.println("hello world");
    }
};
t1.start();
```

方式二：实现一个Runnable接口

```
public class Thread1 implements Runnable{
    @Override
    public void run(){
        System.out.println("hello world");
    }
}

//使用代码，为Thread赋值Runnable实例作为参数
new Thread(new Thread1()).start();
```

4) 线程的中断：

使用线程的中断代替stop方法，实现线程的停止。

Thread的interrupt方法，中断线程，该方法会将线程设置上中断标志。

Thread的isInterrupted方法，判断线程是否中断。

Thread的interrupted方法，判断是否被中断，并清除当前中断状态。

代码实例

```
t1.interrupt();//调用中断方法
...
public void run(){
    while(true){
        if(Thread.currentThread().isInterrupted()){
            System.out.println("interrupted");
            break;//退出while循环
        }
        ...
    }
}
```

注：如果在run方法中没有判断中断标志位，则interrupt方法将不会有任何作用。即要想真正的中断一个线程必须在线程执行中判断中断标志位（isInterrupted方法），然后退出线程的循环。这是一种最优雅的方式实现，而不是使用stop直接中断线程的执行。

5）线程的休眠

休眠中涉及到的中断操作，有时候线程在执行时，需要等待几秒钟，这时可以调用sleep方法。但是在等待过程中如果还希望能够中断该线程的执行，这里就必须捕获InterruptException异常。

``` 
public void run(){
    while(true){
        if(Thread.currentThread().isInterrupted()){
            System.out.println("interrupted");
            break;//退出while循环
        }
        try{
            Thread.sleep(2000);
        }catch(InterruptedException e){
            System.out.println("interrupted when sleeping");
            //如果一个线程抛出了该异常，那么线程的中断标志位就会被清空
            //这里需要重新进行设置中断标志位，
            //上面的isInterrupted方法才能检测到中断
            Thread.currentThread().interrupt();
        }
        ...
    }
}
```

6）线程的挂起和恢复

分别执行suspend和resume实现线程的挂起和恢复操作。另外，这两个方法是不推荐使用的（Deprecated）。

注：suspend不会释放锁

存在的问题，线程的resume发生在加锁资源后的suspend方法之前。如果线程1加锁进入了临界区，可能在线程1（或其他线程中）中调用了suspend方法（将线程挂起），同时在线程2中调用了resume方法想激活线程，但线程2的resume执行（t1.resume），不能保证一定在其他线程执行线程1的suspend（t1.suspend）之后调用。如果发生在线程1调用suspend之前，那么不会对线程产生任何影响。线程1再调用了suspend被挂起，就有可能永远被冻结（除非不断有线程调用线程1的resume方法，恢复线程，这是不可预知的）。因为线程1被冻结，导致线程1所占的资源无法释放，因此其他线程就无法获取线程1占用的资源，导致其他线程一直处于等待状态。

![](/images/architecture/concurrency/thread-suspend.png)

测试代码

```
public class BadSuspend{
    public static Object u=new Object();
    static ChangeObjectThread t1=new ChangeObjectThead("t1");
    static ChangeObjectThread t2=new ChangeObjectThead("t2");

    public static class ChangeObjectThread extends Thread{
        public ChangeObjectThread(String name){
            super.setName(name);//调用线程类的设置name的方法
        }
        @Override
        public void run(){
            synchronized(u){
                System.out.println("in"+getName());
                Thread.currentThread().suspend();
            }
        }
    }

    public static void main(String[] args) throws InterrupedException{
        t1.start();
        //主线程休眠，为了让t1能够有机会执行suspend，这样t1就不会被阻塞
        Thread.sleep(100);
        t2.start();
        t1.resume();//主线程中调用t1的resume方法
        //由于主线程并没有sleep，可能t2不会得到执行suspend的机会
        //即主线程先一步调用了t2的resume方法，t2可能永远被suspend冻结
        t2.resume();
        //主线程中等待t1和t2的执行完毕
        t1.join();
        t2.join();
    }
}
```

将线程dump出，查看堆栈信息，在cmd中运行java命令

```
jps //查看当前进程，确定进程号3308
jstack 3308 //导出该进程的执行线程信息
```

7）线程是有名字的

在线程被dump时，能够根据线程名来确定dump出的线程。

```
new Thead("name");
```

8）线程的join和yeild

yeild方法是指当前线程让出cpu，供大家一起争夺，当前线程也会参与竞争，而且有可能再次竞争到cpu继续执行。

join方法有时需要等待其他线程执行完成后，在继续执行本线程，因此会在该线程内调用其他线程的join方法。join方法也可接收一个等待时间，传递0表示无限期的等待。

join的本质，并且如果t1方法上调用了join方法，实际上会调用wait方法。而wait方法不是将t1设置等待，而是将当前线程设置为等待状态。当t1完成后，虚拟机会调用notifyAll方法，唤醒所有等待的该线程的线程。因此在t1上不要进行notify方法

注：不要在线程的实例上调用notify方法，当然可以在其他实例上调用，只要该实例不是线程实例。

调用t1的join方法，join方法的内部实现细节

```
while(isAlive()){
    wait(0)
}
//等待的线程执行完毕后会调用notifyAll()方法，通知等待的线程。
```

注：不要在线程实例上直接调用wait和notify方法。

9）守护线程

在后台默默地完成一些系统性的服务，如垃圾回收线程，JIT线程就可以理解为守护线程。

注：当一个java应用内，只要守护线程时，java虚拟机会自然退出。

```
public class DaemonDemo{
    public static class DaemonT extends Thread{
        @Overrid
        public void run(){
            while(true){
                System.out.println("I am alive");
                try{
                    Thread.sleep(1000);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args)throws InterruptedException{
        Thread t=new DeamonT();
        t.setDaemon(true);
        t.start();
    }
}

```

10）线程优先级

在线程上调用setPriority函数设置线程的优先级。

```
public final static int MIN_PRIORITY=1;
public final static int NORM_PRIORITY=5;
public final static int MAX_PRIORITY=10;
```

11）线程间的同步

在多线程执行过程中重要的领域是多线程之间如何通信。首先，如果一个线程调用了wait方法进行等待，别人怎么来通知唤醒我继续执行。其次，如果我与你之间存在资源竞争，我们之间怎么协调竞争呢。

java提供了两种机制，一种是通过synchronizd关键字，另外一种是使用Object.wait方法和Object的notify方法。

synchronized关键字的所有实现是在虚拟机内部完成的，包括进入临界区需要拿到锁，没拿到锁将线程挂起，或者在挂起之前可能会做一些优化（先自旋等待）等，这些过程都是在虚拟机内部实现的。

synchronized方法有三种用法：

方法一：指定加锁对象：对指定的对象加锁，进入同步代码前要获得给定对象的锁

方法二：直接作用于实例方法：相当于当前实例加锁，进入同步代码前要获得当前实例的锁

方式三：直接作用于静态方法：相当于对当前类加锁，进入同步代码前要获得当前类的锁

实例1：

```
public class AccountingSync implements Runnable{
    static AccountingSync instance =new AccountingSync();
    static int i=0;
    @Override
    public void run(){
        for(int j=0;j<10000000;j++){
            synchronized(instance){
                i++;
            }
        }
    }
    public static void main(String[] args)throws InterruptedException{
        Thread t1=new Thread(instance);
        Thread t2=new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);//正常执行输出20000000，否则小于20000000
    }
}
```

实例2

Object.wait表示线程等待在当前对象上，Object.notify表示通知等待在当前对象上的线程。

```
public class SimpleWN{
    final static Object object=new Object();
    public statci class T1 extends Thread{
        @Override
        public void run(){
            synchronized(object){
                System.out.println(System.currentTimeMillis()+"T1 start");
                try{
                    System.out.println(System.currentTimeMillis()+"T1 wait for object");
                    object.wait();//wait方法会释放获取的当前对象的锁
                }catch(InterruptedException e){
                    e.printStatckTrace();
                }
                System.out.println(System.currentTimeMillis()+"T1 end");
            }
        }
    }

    public statci class T2 extends Thread{
        @Override
        public void run(){
            synchronized(object){
                System.out.println(System.currentTimeMillis()+"T2 start");
                try{
                    System.out.println(System.currentTimeMillis()+"T2 notify for object");
                    object.notify();//wait方法会释放获取的当前对象的锁
                    
                    Thread.sleep(2000);
                }catch(InterruptedException e){
                    e.printStatckTrace();
                }
                System.out.println(System.currentTimeMillis()+"T2 end");
            }
        }
    }
}
```

注：wait和notify方法调用必须在获取对象锁后才能调用，即需要在synchronized块内。

注2：虽然在T2中调用了notify方法，并立即进行sleep，但t1也不能立即得到执行，因为在t1执行前必须要获取对象的锁，而此时的锁还在t2线程上。只要等到t2线程执行完成后，释放了object锁后，t1线程才有机会执行。