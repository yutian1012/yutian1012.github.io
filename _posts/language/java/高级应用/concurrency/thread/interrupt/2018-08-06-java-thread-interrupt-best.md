---
title: java中的线程中断的最佳实践
tags: [concurrency]
---

参考：https://www.cnblogs.com/skywang12345/p/3479949.html

要想真正的中断一个线程必须在线程执行中判断中断标志位（interrupted方法），然后退出线程的循环。这是一种最优雅的方式实现，而不是使用stop直接中断线程的执行。

注：单独调用线程的interrupt方法只会对线程设置中断标志位，并不会影响正在运行的线程。

1）终止处于运行状态的线程

通过标记方式终止处于运行状态的线程，即判断中断标志位，决定是否退出run方法的执行。

```
package org.sjq.thread.demo.interrupt;

public class ThreadInterruptDemo2 implements Runnable {

    @Override
    public void run() {
        while(true) {
            System.out.println("helloworld");
            if(Thread.interrupted()) {//线程自身判断中断标志位
                break;
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(new ThreadInterruptDemo2(),"thread1");
        t.start();
        Thread.sleep(1000);
        //调用线程的中断
        t.interrupt();
        //其他线程判断某线程的中断标志位，多次运行会发现一个问题，有时候返回false有时候返回true
        System.out.println(t.getName()+" is interrupted "+t.isInterrupted());
    }
}
```

2）中止处于阻塞状态的线程

当线程由于被调用了sleep，wait，join等方法而进入阻塞状态。若此时调用线程的interrupt将线程的中断标记设为true。由于处于阻塞状态，中断标记会被清除，同时产生一个InterruptedException异常。将InterruptedException放在适当的为止就能终止线程。

```
package org.sjq.thread.demo.interrupt;

public class ThreadInterruptDemo3 implements Runnable {

    @Override
    public void run() {
        try {// 通常将catch异常的代码放在while循环外层，这样InterruptedException通过异常直接就跳出来循环体
            while(true) {
                System.out.println("helloworld");
                Thread.sleep(5000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(new ThreadInterruptDemo3(),"thread1");
        t.start();
        Thread.sleep(1000);
        //调用线程的中断
        t.interrupt();
        // 阻塞状态的线程无法被设置中断标志位
        System.out.println(t.getName()+" is interrupted "+t.isInterrupted());
    }
}
```

注：对InterruptedException的捕获务一般放在while循环体的外面，这样，在产生异常时就退出了while循环。否则，InterruptedException在while循环体之内，就需要额外的添加退出处理。