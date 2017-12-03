---
title: java并发API
tags: [architecture]
---

### ReentrantLock类

该类相当于synchronized关键字的增强版（1.4版本之前），目前版本由于对synchronized关键字进行了优化，两者在性能方法已经不相上下了。

1）可重入

单线程可以重复进入。

实例

```
public class ReenterLock implements Runnable{
    public static ReentrantLock lock=new ReentrantLock();
    public static int i=0;
    @Override
    public void run(){
        for(int j=0;j<1000000;j++){
            lock.lock();
            try{
                i++;
            }finally{//lock一定要在finally中释放锁
                lock.unlock();
            }

            //synchronized(this){i++}
        }
    }

    public static void main(String[] args){
        ReenterLock reenter=new ReenterLock();
        Thread t1=new Thread(reenter);
        Thread t2=new Thread(reenter);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```

注：lock和unlock必须成对存在，lock几次就要释放几次。

2）可中断

在加锁的同时能够响应中断的响应。使用lockInterruptibly方法实现

实例，完成任务需要两把锁，获取第一把锁后，需要再获取第二把锁才能正常工作，如果使用lock方法，可能会出现死锁的情况，而且无法通过中断的方式解开死锁的进程。而使用lockInterruptibly就可在发生死锁后，通过中断线程解开死锁。

```
public class ReenterLockInt implements Runnable{
    public static ReentrantLock lock1=new ReentrantLock();
    public static ReentrantLock lock2=new ReentrantLock();

    int lock;//实例变量
    public ReenterLockInt(int lock){
        this.lock=lock;
    }

    @Override
    public void run(){
        try{
            if(lock==1){
                lock1.lockInterruptibly();
                try{
                    Thread.sleep(500);
                }catch(InterruptedException e){}
                lock2.lockInterruptibly();
            }else{
                lock2.lockInterruptibly();
                try{
                    Thread.sleep(500);
                }catch(InterruptedException e){}
                lock1.lockInterruptibly();
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }finally{
            if(lock1.isHeldByCurrentThread()){
                lock1.unlock();//解锁
            }
            if(lock2.isHeldByCurrentThread()){
                lock2.unlock();
            }
            System.out.println(Thread.currentThread().getId()+":线程退出");
        }
    }
    public static void main(String[] args){
        ReenterLockInt r1=new ReenterLockInt();
        ReenterLockInt r2=new ReenterLockInt();
        Thread t1=new Thread(r1);
        Thread t2=new Thread(r2);
        t1.start();
        t2.start();
        Thread.sleep(1000);
        //中断其中一个线程
        DeadlockChecker.check();
    }
}

//检测死锁，并中断相应的死锁线程
public class DeadlockChecker{
    private final static ThreadMXBean mbean=
         ManagementFactory.getThreadMXBean();
    final static Runnable deadLockCheck=new Runnable(){
        @Override
        publi void run(){
            while(true){
                long[] deadlockedThreadIds=mbean.findDeadlockedThreads();
                if(deadlockedThreadIds!=null){
                    ThreadInfo[] threadInfos=mbean.getThreadInfo(deadlockedThreadIds);
                    for(Thread t:Thread.getAllStackTraces().keySet()){
                        for(int i=0;i<thradInfos.length;i++){
                            if(t.getId()==threadInfos[i].getThreadId()){
                                t.interrupt();//中断死锁的线程
                            }
                        }
                    }
                }
                try{
                    Thread.sleep(5000);
                }catch(InterruptedException e){}
            }
        }
    }

    public static void check(){
        Thread t=new Thread(deadLockCheck);
        //做为守护线程一直开启
        t.setDaemon(true);
        t.start();
    }
}
```

3）可限时

指定获取锁的时间，是一种防止死锁和限制长期等待的策略。

实例

```
public class TimeLock implements Runnable{
    public static ReentrantLock lock=new ReentrantLock();
    @Override
    public void run(){
        try{
            //指定5秒内获取锁，如果超过5秒则放弃申请
            if(lock.tryLock(5,TimeUtnit.SECONDS)){
                Thread.sleep(6000);
            }else{
                System.out.println("get lock failed");
            }
        }catch(InterruptedException e){}
        finally{
            if(lock.isHeldByCurrentThread()){
                lock.unlock();    
            }
        }
    }

    public static void main(String[] args){
        TimeLock timeLock=new TimeLock();
        Thread t1=new Thread(timeLock);
        Thread t2=new Thread(timeLock);
        t1.start();
        t2.start();
    }
}
```

4）公平锁

指先来先得，先来的线程必然先获得锁。公平锁的效率比一般锁效率要低，使用时将ReentrantLock的参数设置为true，则表示创建一个公平锁。

```
ReentrantLock lock=new ReentrantLock(true);
```

### Condition类

