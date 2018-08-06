---
title: java中的线程中断
tags: [concurrency]
---

1）stop方法

Thread的stop方法，不推荐使用，它会释放所有的monitor（锁）。很可能导致数据不一致的问题。

2）interrupt方法

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