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

1）Condition相当于在某个锁上进行wait和notify。

在Object的wait和notify方法调用时都要先获取锁（synchronized关键字），同样在使用Condition时也需要获取锁（ReentrantLock）。

注：Condition与ReentrantLock相结合使用，wait和notify与synchronized关键字结合使用。

2）常用API：await等待，sinal唤醒等方法

3）实例

```
public class ReentrantLockCondition implements Runnable{
    public static ReentrantLock lock=new ReentrantLock();
    //在锁的基础上创建condition
    public static Condition condition=lock.newCondition();
    @Override
    public void run(){
        try{
            lock.lock();
            //等待，会释放锁
            condition.await();
            System.out.println("Thread is going on");    
        }catch(InterruptedException e){
            e.printStackTrace();
        }finally{
            lock.unlock();
        }
    }

    public static void main(String[] args)throws InterruptedException{
        ReentrantLockCondition rc=new ReentrantLockCondition();
        Thread t1=new Thread(rc);
        t1.start();
        Thread.sleep(2000);
        lock.lock();
        //唤醒等待线程
        condition.singal();
        lock.unlock();
    }
}
```

注：condition的await和signal方法的前后由lock和unlock包围。

注2：condition.await方法会释放锁，

### Semaphore类（信号量）

1）概念：类似操作系统中的信号量。可以设置允许多少个线程进入临界区，超过信号量值的线程就必须等待。可以理解为是共享锁。

2）主要API：acquire方法获取信号量，release释放信号量。

3）实例：

```
public class SemapDemo implements Runnable{
    final Semaphore semp=new Semaphore(5);
    @Override
    public void run(){
        try{
            semp.acquire();//semp.acquire(2);//可获取多个信号量
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getId()+":done!");
            
        }catch(InterruptedException e){
            e.printStackTrace();
        }finally{
            semp.release();//semp.release(2);//释放时也要多个释放
        }
    }

    public static void main(String[] args){
        //使用线程池启动多个线程
        ExecutorService exec=Executors.newFiexedThreadPool(20);
        final SemapDemo demo=new SemapDemo();
        for(int i=0;i<20;i++){
            exec.submit(demo);
        }
    } 
}
```

### ReadWriteLock类

1）概念：ReentrantLock锁不会对锁进行区分，都必须先获取锁才能继续执行。而读写锁可以使并行度更高。读锁可以同时多个线程并行运行，而写锁与读锁/写锁互斥。

注：读/读不互斥（读读之间不阻塞），读/写互斥（读阻塞写，写也阻塞读），写/写互斥

2）常用API

```
ReadWriteLock lock = new ReentrantReadWriteLock()
//获取读锁，并加锁，防止写进入
lock.readLock().lock();  
//获取读锁，释放读锁,一般在finally中执行
lock.readLock().unlock();
//获取写锁，并加锁，防止读/写进入
lock.writeLock().lock(); 
//获取写锁，释放写锁，一般在finally中执行
lock.writeLock().unlock();
```

3）实例：

```
public class ReentrantReadWriteLockTest {

    static class MyObject {  
        private String value;
        private ReadWriteLock lock = new ReentrantReadWriteLock();  
        
        //读数据
        public void get() {
            //获取读锁
            lock.readLock().lock();  
            System.out.println(Thread.currentThread().getName() + "准备读数据!!");  
            try {  
                Thread.sleep(new Random().nextInt(1000));  
                System.out.println(Thread.currentThread().getName() + "读数据为:" + this.value);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            } finally {
                //释放读锁
                lock.readLock().unlock();  
            }  
        }  
        //写数据
        public void put(String value) {  
            //获取写锁
            lock.writeLock().lock();  
            System.out.println(Thread.currentThread().getName() + "准备写数据");  
            try {  
                Thread.sleep(new Random().nextInt(1000));  
                this.value = value;
                System.out.println(Thread.currentThread().getName() + "写数据为" + this.value);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            } finally {  
                //释放写锁
                lock.writeLock().unlock();  
            }  
        }  
    }  
      
    public static void main(String[] args) throws InterruptedException {  
        //在多线程将共享的实例变量
        final MyObject myObject = new MyObject();  
        //定义线程池
        ExecutorService executor = Executors.newCachedThreadPool();  
        
        //开启3个写线程
        for (int i = 0; i < 3; i++) {  
            executor.execute(new Runnable() {  
                @Override  
                public void run() {  
                    for (int j = 0; j < 5; j++)
                        myObject.put(new Random().nextInt(1000)+"");  
                }  
            });  
        }  
        //开启3个读线程
        for (int i = 0; i < 3; i++) {  
            executor.execute(new Runnable() {  
                @Override  
                public void run() {
                    for (int j = 0; j < 5; j++)  
                        myObject.get();  
                }
            });  
        }  
          
        executor.shutdown();  
    }  
  
} 
```

### CountDownLatch类

1）概念：类似于倒数的计时器，如最终的操作需要分10步，这10步中的每一步都可以启动一个线程进行操作，等到这10步操作都做完了，才可以开始最终的操作了。开始时CountDownLatch初值为10，每一步的线程操作完成后，CountDownLatch都会减一，直到CountDownLatch为0时才能继续执行最终操作。

2）常用API：

```
CountDownLatch end=new CountDownLatch(10);
end.countDown();//减一
end.await();//线程等待CountDownLatch减0
```

3）实例：

```
public class CountDownLatchDemo implements Runnable{
    static final CountDownLatch end=new CountDownLatch(10);
    static final CountDownLatchDemo demo=new CountDownLatchDemo();
    @Override
    public void run(){
        try{
            Thread.sleep(new Random().nextInt(10)*1000);
            System.out.println("check complete");
            end.countDown();//完成一个线程countDown减一
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
        ExecutorService exec=Executors.newFixedThreadPool(10);
        for(int i=0;i<10;i++){
            exec.submit(demo);
        }
        //等待检查
        end.await();
        System.out.println("主线程开始执行");
        exec.shutdown();
    }
}
```

4）使用场景

有时主线程将任务分为多个子任务，每个子任务交由独立的线程来执行，只有所以的子任务都执行完成后，主线程才能继续执行，对任务进行后续的完善工作。

### CyclicBarrier类

1）概念：cyclic意味着循环，类似于循环的CountDownLatch。循环栅栏线程控制，可以设置到达条件后执行后续任务，然后再次循环使用cyclicBarrier实例。

2）常用API：new CyclicBarrier(int,Runnable)，第二个参数是当指定的线程都到达后执行Runnable任务，而这些线程还可以继续使用cyclicBarrier进行再一次的循环。

3）实例：

```
public class CyclicBarrierDemo{
    //单独运行的线程，引用CyclicBarrier栅栏控制线程的协作
    public static class Soldier implements Runnable{
        private String soldier;
        private final CyclicBarrier cyclic;

        Soldier(CyclicBarrier cyclic,String soldierName){
            this.cyclic=cyclic;
            this.soldier=soldierName;
        }

        public void run(){
            try{
                //等待所有士兵（线程）到齐，到齐后会执行cyclic中的后续任务，
                cyclic.await();
                //cyclic中定义的后续任务完成后，继续执行各个线程的任务
                System.out.println(soldier+":开始做任务");
                doWork();
                //再次等待所有士兵（线程）完成工作
                cyclic.await();
            }catch(InterruptedException e){
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
        }
        //模拟耗时任务的执行
        void doWork(){
            try{
                Thread.sleep(Math.abs(new Random().nextInt()%10000));
            }catch(InterruptedException e){
                e.printStackTrace();
            }
            System.out.println(soldier+":任务完成");
        }
    }

    //定义栅栏控制线程（CyclicBarrier实例）的后续任务
    //可以用该任务来定义多个线程的协作状态（第一步完成，第二步完成...）
    public static class BarrierRun implements Runnable{
        boolean flag;
        int N;
        public BarrierRun(boolean flag,int N){
            this.flag=flag;
            this.N=N;
        }
        public void run(){
            if(flag){
                System.out.println("司令:[士兵"+N+"个，任务完成!]");
            }else{
                System.out.println("司令:[士兵"+N+"个，集合完毕!]");
                flag=true;
            }
        }
    }

    public static void main(String args[]){
        final int N=10;//相当于10个士兵
        Thread[] allSoldier=new Thread[N];
        boolean flag=false;
        //10个士兵到齐后执行BarrierRun任务
        CyclicBarrier cyclic=new CyclicBarrier(N,new BarrierRun(flag,N));
        //设置屏障点，主要是为了执行这个方法
        System.out.println("集合队伍！");
        for(int i=0;i<N;i++){
            System.out.println("士兵"+i+" 报道!");
            allSoldier[i]=new Thread(new Soldier(cyclic,"士兵"+i));
            allSoldier[i].start();
            /*if(i==5){
                //中断线程，该线程会抛出InterruptedException异常
                //由于并存的10个线程中存在一个无法完成的线程，
                //因此其他线程会抛出BrokenBarrierException异常
                allSoldier[0].interrupt();
            }*/
        }
    }
}
```