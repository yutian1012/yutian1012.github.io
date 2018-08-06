---
title: java中的线程实现方式
tags: [concurrency]
---

线程的启动都是通过run方法来实现的。

1）重写Thread的run方法

```
Thread t1=new Thread(){
    @Override
    public void run(){
        System.out.println("hello world");
    }
};
t1.start();
```

2）实现一个Runnable接口

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