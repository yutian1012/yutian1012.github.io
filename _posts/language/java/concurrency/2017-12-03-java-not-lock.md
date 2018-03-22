---
title: 无锁
tags: [architecture]
---

参考：https://www.cnblogs.com/756623607-zhang/p/6876060.html

1） 概念：

锁是一种悲观的策略。它总是假设每一次的临界区操作会产生冲突，因此，必须对每次操作都小心翼翼。如果有多个线程同时需要访问临界区资源，就宁可牺牲性能让线程进行等待，所以说锁会阻塞线程执行。

无锁是一种乐观的策略，它会假设对资源的访问是没有冲突的。既然没有冲突，自然不需要等待，所以所有的线程都可以在不停顿的状态下持续执行。那遇到冲突怎么办呢？无锁的策略使用一种叫做比较交换的技术（CAS Compare And Swap）来鉴别线程冲突，一旦检测到冲突产生，就重试当前操作直到没有冲突为止。

2）无锁的好处

第一，在高并发的情况下，它比有锁的程序拥有更好的性能；

第二，它天生就是死锁免疫的。

注：无锁的操作比阻塞的方式要好很多。使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程间频繁调度带来的开销，因此，它要比基于锁的方式拥有更优越的性能。

### 无锁原理

CAS（compare and swap）比较和交换。

CAS算法的过程是这样：它包含三个参数CAS(V,E,N)。V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS返回当前V的真实值。CAS操作是抱着乐观的态度进行的，它总是认为自己可以成功完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS操作即使没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。

简单地说，CAS需要你额外给出一个期望值，也就是你认为这个变量现在应该是什么样子的。如果变量不是你想象的那样，那说明它已经被别人修改过了。你就重新读取，再次尝试修改就好了。

在硬件层面，大部分的现代处理器都已经支持原子化的CAS指令。在JDK 5.0以后，虚拟机便可以使用这个指令来实现并发操作和并发数据结构，并且，这种操作在虚拟机中可以说是无处不在。

注：CAS是单条汇编指令，不要看成多条指令，因此具有原子性。

### 无锁类的使用

1）AtomicInteger类

为了让Java程序员能够受益于CAS等CPU指令，JDK并发包中有一个atomic包，里面实现了一些直接使用CAS操作的线程安全的类型。其中，最常用的一个类，应该就是AtomicInteger。你可以把它看做是一个整数。但是与Integer不同，它是可变的，并且是线程安全的。对其进行修改等任何操作，都是用CAS指令进行的。

```
private volatile int value;//它就代表了AtomicInteger的当前实际取值
//保存着value字段在AtomicInteger对象中的偏移量
private static final long valueOffset;

//获取比较并交换
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
//返回当前值，并自加方法，通过方法可以看出，程序不断循环，直到设置成功为止
public final int getAndIncrement(){
    for(;;){
        int current=get();
        int next=current+1;
        if(compareAndSet(current,next))
            return current;
    }
}
```


偏移量的概念：

valueOffset是指当前值相对于对象的偏移量。

类比C语言的结构体Struct

```
struct{
    int a;//占4位内存
    int b;//b相对于struct的偏移量就是4
}

//java中的偏移量的理解
//java类就相当于struct结构体，而当前值value的偏移量则是相当于该实例的偏移量地址
```

实例：

```
public class AtomicIntegerDemo{
    static AtomicInteger i=new AtomicInteger();
    public static class AddThread implements Runnable{
        public void run(){
            for(int k=0;i<10000;k++){
                i.incrementAndGet();
            }
        }
    }

    public static void main(String[] args){
        Thread[] ts=new Thead[10];
        for(int k=0;k<10;i++)   ts[k]=new Thread(new AddThread());

        for(int k=0;k<10;i++) ts[k].start();

        for(int k=0;k<10;i++) ts[k].join();

        System.out.println(i);//输出100000
    }
}
```

2）Unsafe类

Unsafe类非安全的操作类，类似C语言中指针的操作。是jdk内部使用的，一般不对外提供访问功能。

Unsafe类大多是围绕偏移量进行的操作。以AtomicInteger类的compareAndSet方法为例

```
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

注：unsafe类的compareAndSwapInt方法接收了AtomicInteger类的实例对象，valueOffset表示操作的值据当前对象的偏移量。该方法的目的是从对象偏移量处取得值与expect比较，如果与期望的值相同则更新，否则不更新。

注2：unsafe类中的方法使用CAS指令操作来完成的。

3）AtomicReference类

AtomicReference和AtomicInteger非常类似，不同之处就在于AtomicInteger是对整数的封装，而AtomicReference则对应普通的对象引用。也就是它可以保证你在修改对象引用时的线程安全性。

注：为了保证多线程将修改对象的引用提供的线程安全类

实例

```
public class AtomictReferenceTest{
    public final static AtomicReference<String> atomicStr=
        new AtomicReference<String>("abc");

    public static void main(String[] args){
        for(int i=0;i<10;i++){
            final int num=i;
            new Thread(){
                public void run(){
                    try{
                        Thread.sleep(Math.abs((int)(Math.random()*100)));
                    }catch(InterruptedException e){
                        e.printStackTrace();
                    }
                    if(atomicStr.compareAndSet("abc","def")){
                        System.out.println("Thread"+Thread.currentThread().getId()+"SUCCESS");
                    }else{
                        System.out.println("Thread"+Thread.currentThread().getId()+"FAILED");
                    }
                }
            }
        }
    }
}
```

注：开启了10个线程尝试把abc字符串修改为def，只会有一个线程能成功修改字符串的值，其他线程都不能成功。

4）AtomicStampedReference类

内部不仅维护了对象值，还维护了一个时间戳。当AtomicStampedReference对应的数值被修改时，除了更新数据本身外，还必须要更新时间戳。当AtomicStampedReference设置对象值时，对象值以及时间戳都必须满足期望值，写入才会成功。因此，即使对象值被反复读写，写回原值，只要时间戳发生变化，就能防止不恰当的写入。

注：AtomicStampedReference在AtomicReference的基础上新增了有关时间戳的信息

解决的问题是AtomicReference的引用类被不同的线程改变了多次，最终又改回来原来值，而使用它的线程可能不知道该值曾被修改过。这种情况下，针对简单的逻辑可能不存在问题，但对应有相关性复杂依赖或操作的引用，可能是致命的错误。

注：该类用于确保对于过程状态敏感的对象在多线程间操作不出问题。

![](/images/architecture/concurrency/atomicReference-problem.png)

实例：账号余额小于20时实现充值，不过只运行充值一次。开启3个充值线程，并且一直循环充值。使用消费线程进行余额减少，即使小于20，也只能充值一次。

```
public class AtomicStampedReferenceDemo {
    static AtomicStampedReference<Integer> money=new AtomicStampedReference<Integer>(19,0);
    public static void main(String[] args) {
        //模拟多个线程同时更新后台数据库，为用户充值
        for(int i = 0 ; i < 3 ; i++) {
            //先获取money对象的时间戳
            final int timestamp=money.getStamp();
             new Thread() {  
                public void run() { 
                    while(true){
                         Integer m=money.getReference();
                         if(m<20){
                             //增加余额时，需要传递比较的时间戳和一个操作后的新时间戳，虽然被线程消费后金额小于20，但不会重复充钱，只会成功充值一次
                             if(money.compareAndSet(m,m+20,timestamp,timestamp+1)){
                                 System.out.println("余额小于20元，充值成功，余额:"+money.getReference()+"元");
                             }
                         }
                   }
               } 
            }.start();
         }
        
        //用户消费线程，模拟消费行为
        new Thread() { 
             public void run() { 
                for(int i=0;i<100;i++){
                    while(true){
                        int timestamp=money.getStamp();
                        Integer m=money.getReference();
                        if(m>10){
                             System.out.println("大于10元");
                             if(money.compareAndSet(m, m-10,timestamp,timestamp+1)){
                                 System.out.println("成功消费10元，余额:"+money.getReference());
                                 break;
                             }
                        }else{
                            System.out.println("没有足够的金额");
                            break;
                        }
                    }
                    try {Thread.sleep(100);} catch (InterruptedException e) {}
                 }
             } 
        }.start(); 
    }
}
```

注：在操作后都需要设置一个新的stamp值。

5）AtomicIntegerArray类

与AtomicInteger相比，数组的实现不过是多了一个下标。

```
public final boolean compareAndSet(int i, int expect, int update) {
    return compareAndSetRaw(checkedByteOffset(i), expect, update);
}

private boolean compareAndSetRaw(long offset, int expect, int update) {
    //cas指令操作
    return unsafe.compareAndSwapInt(array, offset, expect, update);
}
```

6）AtomicIntegerFieldUpdate类

让普通变量也享原子操作。就比如原本有一个变量是int型，并且很多地方都应用了这个变量，但是在某个场景下，想让int型变成AtomicInteger，但是如果直接改类型，就要改其他地方的应用。AtomicIntegerFieldUpdater就是为了解决这样的问题产生的。

```
public class Test
{
    public static class V{
        int id;
        //只是普通变量，必须定义为volatile，不会影响其他使用使用此字段的类
        volatile int score;
        public int getScore()
        {
            return score;
        }
        public void setScore(int score)
        {
            this.score = score;
        }
 
    }
    //指定类的字段要享受原子操作
    public final static AtomicIntegerFieldUpdater<V> vv = 
    AtomicIntegerFieldUpdater.newUpdater(V.class, "score");

    //检查update是否工作正常，最终的getScore获取的值与allscore的值必须是相同的
    public static AtomicInteger allscore = new AtomicInteger(0);
 
    public static void main(String[] args) throws InterruptedException
    {
        //多个线程会同时访问这一个共享变量
        final V stu = new V();
        Thread[] t = new Thread[10000];
        for (int i = 0; i < 10000; i++)
        {
            t[i] = new Thread() {
                @Override
                public void run()
                {
                    if(Math.random()>0.4)
                    {
                        //对stu对象的score字段进行添加
                        vv.incrementAndGet(stu);
                        allscore.incrementAndGet();
                    }
                }
            };
            t[i].start();
        }
        for (int i = 0; i < 10000; i++)
        {
            t[i].join();
        }
        System.out.println("score="+stu.getScore());
        System.out.println("allscore="+allscore);
    }
}
```