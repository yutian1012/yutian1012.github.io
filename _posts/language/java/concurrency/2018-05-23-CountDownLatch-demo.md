---
title: java并发编程CountDownLatch实例
tags: [java]
---

实现一个并发实例，通过多线程开启并发访问，当countDownLatch为0时同时发起请求。

CountDownLatch是一个线程协调机制，当计数为0后，才

1）代码实例

```
public class CountDownLatchDemo {
    
    private int threadNum=50;
    
    private CountDownLatch cdl=new CountDownLatch(threadNum);
    
    public void request() throws InterruptedException {
        for(int i=0;i<threadNum;i++) {
            
            System.out.println("准备线程"+i+"执行请求...");
            
            Thread.sleep(100);
            
            new Thread(new UserRequest(i)).start();
            
            cdl.countDown();
        }
        
    }

    class UserRequest implements Runnable{
        
        private int threadNum;
        
        public UserRequest(int threadNum) {
            this.threadNum=threadNum;
        }

        @Override
        public void run() {
            //所有子线程等待，直到CountDownLatch实例减为0后再执行
            try {
                cdl.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            
            //子线程发起请求操作
            try {
                System.out.println("线程"+threadNum+"发起请求...");
                //模拟请求处理
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程"+threadNum+"执行完毕...");
        }
    }
    //测试并发方法的执行
    public static void main(String[] args) {
        CountDownLatchDemo demo=new CountDownLatchDemo();
        try {
            demo.request();
            //主线程挂起，等待子线程执行完毕
            Thread.currentThread().join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("执行完毕");
    }
}
```