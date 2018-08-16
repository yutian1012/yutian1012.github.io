---
title: jvm java虚拟机栈-线程状态的查看
tags: [jvm]
---

1）实例代码

```
package org.sjq.thread.demo.stack;

public class ThreadRunnableStatus implements Runnable{

    @Override
    public void run() {
        while(true) {
            //保证线程一直处于运行状态
        }
    }
    
    public static void main(String[] args) {
        Thread t1=new Thread(new ThreadRunnableStatus());
        t1.start();
        printThreadInfo(t1);//打印堆栈信息
    }

    private static void printThreadInfo(Thread t) {
        System.out.println(t.getId());
        System.out.println(t.getName());
        System.out.println(t.getPriority());
        for(StackTraceElement e:t.getStackTrace()) {
            System.out.print(e.getClassName()+" "+e.getMethodName() +" "+e.getLineNumber());
            System.out.print("<--");
        }
        System.out.println();
        System.out.println(t.getState());
    }
}
```

2）查看堆栈打印信息

a. 运行实例

b. 定位进程号

```
tasklist | findstr java
```

注：一般运行的java程序，在windows中显示的是javaw。

c. 执行jstack命令输出信息到文件

```
jstack.exe 4872 > e:/jstack_runnable.txt
```

注：通过上一步查询，获取的进程ID号为4872

3）分析堆栈数据

```
# 其他线程数据暂时忽略
...
"ThreadRunnableStatus" #10 prio=5 os_prio=0 tid=0x000000001629a000 nid=0x96c runnable [0x0000000016e1f000]
   java.lang.Thread.State: RUNNABLE
    at org.sjq.thread.demo.stack.ThreadRunnableStatus.run(ThreadRunnableStatus.java:7)
    at java.lang.Thread.run(Unknown Source)

```

格式输出信息

```
# 线程名                ID   线程优先级    操作系统优先级   java线程的identifier
"ThreadRunnableStatus" #10    prio=5      os_prio=0     tid=0x000000001629a000 

# native线程的identifier   状态         线程栈起始地址
    nid=0x96c             runnable   [0x0000000016e1f000]

#  线程状态
   java.lang.Thread.State: RUNNABLE

#   线程调用栈
    at org.sjq.thread.demo.stack.ThreadRunnableStatus.run(ThreadRunnableStatus.java:7)
    at java.lang.Thread.run(Unknown Source)
```