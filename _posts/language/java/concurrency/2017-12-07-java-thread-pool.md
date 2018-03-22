---
title: java线程池
tags: [architecture]
---

线程池：在多线程并发的情况下，线程创建和销毁的代价是需要考虑的。希望将cpu更多的使用在业务处理上，而不是用来辅助上，因此引入了线程池。

线程池的主要作用是对线程进行重复利用。将使用完的线程放入池中，需要分配时就从线程池中获取。

### 简单线程池的实现

参考：https://www.cnblogs.com/wxwall/p/7050698.html

```
public class ThreadPool{
    private static ThreadPool instance=null;
    //空闲的线程队列
    private List<Worker> idleThreads;
    //已有的线程总数
    private int threadCounter;
    private boolean isShutDown=false;

    private ThreadPool(){
        this.idleTheads=new Vector(5);
        threadCounter=0;
    }

    public int getCreatedThredsCount(){
        return threadCounter;
    }
    //取得线程池实例
    public synchronzied static ThreadPool getInstance(){
        if(instance==null){
            instance=new ThreadPool();
        }
        return instance;
    }

    //将线程放入池中
    protected synchronized void repool(Worker repoolingThread){
        if(!isShutDown){
            idleThreads.add(repoolingThread);
        }else{
            repoolingThread.shutDown();//关闭线程
        }
    }
    //停止池中的所有线程
    public synchronized void shutdown(){
        isShutDown=true;
        for(int threadIndex=0;threadIndex<idleThreads.size();threadIndex++){
            Worker idleThread=(Worker)idleThreads.get(threadIndex);
            idleThread.shutDown();
        }
    }

    //执行任务
    public synchronized void start(Runnable target){
        Worker thread=null;
        //如果有空闲线程，则直接使用
        if(idleThreads.size()>0){
            int lastIndex=idleThreads.size()-1;
            thread=(Worker)idleThreads.get(lastIndex);
            idleThreads.remove(lastIndex);
            //立即执行这个任务
            thread.setTarget(target);
        }else{//没有空闲线程，则创建新的线程
            threadCounter++;
            //创建新线程
            thread=new Worker(target,"PThread #"+threadCounter,this);
            //启动这个线程
            thread.start();
        }
    }
}
```

工作线程

```
public class Worker extends Thread{
    //线程池
    private ThreadPool pool;
    //任务
    private Runnable target;
    private boolean isShutDown=false;
    private boolean isIdle=false;

    public Worker(Runnable target,String name,ThreadPool pool){
        super(name);
        this.pool=pool;
        this.target=target;
    }

    public Runnable getTarget(){
        return target;
    }

    public boolean isIdle(){
        return isIdle;
    }

    public void run(){
        while(!isShutDown){
            isIdle=false;
            if(target!=null){
                //运行任务
                target.run();
            }
            //任务结束
            isIdle=true;
            try{
                //该任务结束后，不关闭线程，而是放入线程池空闲队列
                pool.repool(this);
                synchronized(this){
                    //线程空闲，等待新的任务到来
                    wait();
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
            isIdle=false;
        }
    }

    public synchronized void setTarget(Runnable newTarget){
        target=newTarget;
        //设置了任务之后，通知run方法，开始执行任务
        notifyAll();
    }

    public synchronized void shutDown(){
        isShutDown=true;
        notifyAll();//结束线程
    }
}
```