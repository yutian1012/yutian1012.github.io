---
title: java中的线程中断interrupt方法
tags: [concurrency]
---

参考：https://www.cnblogs.com/skywang12345/p/3479949.html

1）线程中断方式

stop方法（不推荐使用），Thread的stop方法，不推荐使用，它会释放所有的monitor（锁）。很可能导致数据不一致的问题。

interrupt方法，使用线程的中断代替stop方法，实现线程的停止。Thread的interrupt方法，中断线程，该方法会将线程设置上中断标志。

2）相关方法

Thread的isInterrupted方法，判断线程是否中断。

Thread的interrupted方法，判断是否被中断，并清除当前中断状态。

3）interrupt方法设置中断标志

```
package org.sjq.thread.demo.interrupt;

public class ThreadInterruptDemo implements Runnable {

    @Override
    public void run() {
        while(true) {
            System.out.println("helloworld");
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(new ThreadInterruptDemo());
        t.start();
        Thread.sleep(1000);
        t.interrupt();//调用线程的中断，查看是否能够退出run方法的循环
    }
}
```

在main方法中调用了线程的interrupt方法，线程并没有收到任何影响。

结论：如果在run方法中没有判断中断标志位，对中断标志为进行逻辑判断出来，则interrupt方法将不会有任何作用。

4）判断中断标志位，决定是否退出run方法的执行

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

在run方法中判断中断标志位，如果当前线程被设置了中断标志位，则程序应在符合业务逻辑的前提下优雅的退出run方法（线程中断的最佳实践）。

5）interrupted和isInterrupted的区别

interrupted和isInterrupted都能够用于检测对象的“中断标记”。区别是，interrupted方法除了返回中断标记之外，它还会清除中断标记（即将中断标记设为false）。而isInterrupted方法仅仅返回中断标记。