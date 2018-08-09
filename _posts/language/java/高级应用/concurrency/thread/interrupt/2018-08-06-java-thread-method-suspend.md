---
title: java中线程的suspend方法
tags: [concurrency]
---

参考：https://blog.csdn.net/xingjiarong/article/details/47984659

suspend被弃用的原因是因为它会造成死锁。suspend方法和stop方法不一样，它不会破换对象和强制释放锁，相反它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。

suspend方法是将线程阻塞挂起，而不能控制线程获取锁的释放操作。被阻塞的线程需要调用resume方法才能唤醒继续执行。

1）实例代码

```
package org.sjq.thread.demo.suspend;

public class ThreadSuspendDemo implements Runnable{

    private Object lock=null;
    
    public ThreadSuspendDemo(Object lock) {
        this.lock=lock;
    }
    
    @Override
    public void run() {
        synchronized (lock) {
            System.out.println(Thread.currentThread().getName()+":helloworld");
            try {
                Thread.sleep(5000);
                System.out.println(Thread.currentThread().getName()+":ending...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Object lock=new Object();
        
        Thread t=new Thread(new ThreadSuspendDemo(lock));
        Thread t2=new Thread(new ThreadSuspendDemo(lock));
        
        t.start();
        Thread.sleep(1000);
        //对t执行suspend方法，t2仍然不能执行
        t.suspend();
        t2.start();
        Thread.sleep(1000);
        t.resume();//唤醒t再次执行
    }
    
}
```