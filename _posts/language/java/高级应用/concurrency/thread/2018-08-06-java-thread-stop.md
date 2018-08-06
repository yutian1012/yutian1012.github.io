---
title: java中线程的stop方法
tags: [concurrency]
---

参考：https://blog.csdn.net/jiangwei0910410003/article/details/19900007

1）调用stop方法的问题

调用Thread.stop()方法是不安全的，这是因为当调用Thread.stop()方法时，会发生下面两件事

a.即刻抛出ThreadDeath异常：

在线程的run()方法内，任何一点都有可能抛出ThreadDeathError，包括在catch或finally语句中。

b.释放线程占用的锁

会释放该线程所持有的所有的锁，而这种释放是不可控制的，非预期的。

2）ThreadDeath错误

ThreadDeath的继承体系：

```
java.lang.Object
   java.lang.Throwable
       java.lang.Error
           java.lang.ThreadDeath
```

注：可以看出抛出的并非异常，而是Error，因此程序中很难对Error进行处理，即很容易忽略了这部分逻辑的处理。

3）拋出ThreadDeath错误

需要使用Throwable来捕获异常，否则看不到错误的抛出。

```
package org.sjq.thread.demo.stop;

public class ThreadStopThreadDeathDemo implements Runnable{

    @Override
    public void run() {
        try {
            System.out.println("hellowlord");
            Thread.sleep(3000);
            System.out.println("shutdown");
        } catch (Throwable ex) {
            System.out.println("抛出异常的线程："+Thread.currentThread().getName());
            ex.printStackTrace();
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(new ThreadStopThreadDeathDemo(),"ThreadStopThreadDeathDemo");
        t.start();
        Thread.sleep(1000);//休眠1秒
        t.stop();
    }
}
```

输出内容：

```
hellowlord
抛出异常的线程：ThreadStopThreadDeathDemo
java.lang.ThreadDeath
    at java.lang.Thread.stop(Unknown Source)
    at org.sjq.thread.demo.stop.ThreadStopThreadDeathDemo.main(ThreadStopThreadDeathDemo.java:21)
```

从输出内容可以看出，调用了线程的stop方法后，线程抛出ThreadDeath异常，线程退出，并且下方的输出没有得到执行。

4）释放资源

```
package org.sjq.thread.demo.stop;

public class ThreadStopLockFreeDemo implements Runnable{
    
    private Object lockObj;
    
    public ThreadStopLockFreeDemo(Object lock) {
        this.lockObj=lock;
    }

    @Override
    public void run() {
        synchronized (lockObj) {
            try {
                System.out.println(Thread.currentThread().getName()+"acquired lock...");
                Thread.sleep(5000);//休眠5秒
                System.out.println(Thread.currentThread().getName()+"quiting...");
            } catch (Throwable e) {
                 System.out.println("抛出异常的线程："+Thread.currentThread().getName());
                 e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Object lock=new Object();//定义锁对象
        Thread t1=new Thread(new ThreadStopLockFreeDemo(lock),"ThreadStopLockFreeDemo1");
        Thread t2=new Thread(new ThreadStopLockFreeDemo(lock),"ThreadStopLockFreeDemo2");
        t1.start();
        Thread.sleep(1000);
        t2.start();
        Thread.sleep(1000);
        t1.stop();
    }
}
```

输出内容：

```
ThreadStopLockFreeDemo1acquired lock...
抛出异常的线程：ThreadStopLockFreeDemo1
java.lang.ThreadDeath
    at java.lang.Thread.stop(Unknown Source)
    at org.sjq.thread.demo.stop.ThreadStopLockFreeDemo.main(ThreadStopLockFreeDemo.java:40)
ThreadStopLockFreeDemo2acquired lock...
ThreadStopLockFreeDemo2quiting...
```

从输出内容可以看出，线程调用stop方法后，会抛出异常并释放掉该线程所拥有的全部锁。

5）释放锁理解

在编写程序时，一般都只会对因此信息进行捕获，而不会去主动捕获Error信息。因为，这部分信息是不可预知的。因此，上面的程序中不捕获Throwable，则不会在catch中对线程异常进行进一步处理，就可能出现逻辑和数据一致性问题。

下面的逻辑中使用了2个锁，当我们捕获了Throwable异常，第二个锁不会释放，即该线程仍然没有真正的关闭。而如果我们不捕获Throwable异常，则会释放全部的锁，线程退出。

```
package org.sjq.thread.demo.stop;

public class ThreadStopLockFreeDemo2 implements Runnable{
    
    private Object lockObj1;
    private Object lockObj2;
    
    public ThreadStopLockFreeDemo2(Object lock1,Object lock2) {
        this.lockObj1=lock2;
        this.lockObj2=lock2;
    }
    
    @Override
    public void run() {
        synchronized (lockObj1) {
            synchronized (lockObj2) {
                try {
                    System.out.println(Thread.currentThread().getName()+"acquired lock...");
                    Thread.sleep(5000);//休眠5秒
                    System.out.println(Thread.currentThread().getName()+"quiting...");
                } catch (InterruptedException e) {//catch (Throwable e) {//想不起来try这个错误，就会将锁都释放掉
                     System.out.println("抛出异常的线程："+Thread.currentThread().getName());
                     e.printStackTrace();
                }
            }
            try {
                System.out.println("lockObj2 not free...");
                Thread.sleep(2000);
                System.out.println("lockObj2 freeing...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Object lock=new Object();//定义锁对象
        Object lock2=new Object();
        Thread t1=new Thread(new ThreadStopLockFreeDemo2(lock,lock2),"ThreadStopLockFreeDemo1");
        Thread t2=new Thread(new ThreadStopLockFreeDemo2(lock,lock2),"ThreadStopLockFreeDemo2");
        t1.start();
        Thread.sleep(1000);
        t2.start();
        Thread.sleep(1000);
        t1.stop();
    }

}
```