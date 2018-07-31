---
title: java finalize的深度理解
tags: [interview]
---

参考：https://blog.csdn.net/L_wwbs/article/details/70770447?locationNum=1&fps=1

参考：https://www.cnblogs.com/Smina/p/7189427.html

finalize()是Object中的方法，当垃圾回收器将要回收对象所占内存之前被调用。

1）虚拟机是如何判断一个对象需要被销毁

Java采用可达性分析算法来判定一个对象是否死期已到。Java中以一系列"GCRoots"对象作为起点，如果一个对象的引用链可以最终追溯到"GCRoots"对象，那么该对象就是存活的。否则，该对象就会被标记为待回收，等到GC回收内存时，将对象销毁释放内存。

2）finalize方法的执行

finalize的调用具有不确定性，只保证方法会调用，但不保证方法里的任务会被执行完（比如一个对象手脚不够利索，磨磨叽叽，就可能还在执行过程中，被杀死回收了）。

finalize方法可能会带来性能问题。因为JVM通常在单独的低优先级线程中完成finalize的执行。

注：Java语言规范并不保证finalize方法会被及时地执行、而且根本不会保证它们会被执行

3）finalize的作用

finalize的主要作用往往被认为是用来做最后的资源回收。由于该方法的执行具有不确定性，因此用来回收资源也不是一个好的方式，而且运行代价较高。

4）finalize与c++的析构函数

finalize()与C++中的析构函数不是对应的。C++中的析构函数调用的时机是确定的（对象离开作用域或delete掉），但Java中的finalize的调用具有不确定性。

4）观察finalize方法的执行

System.gc与System.runFinalization()方法会增加finalize方法执行的机会，但不可盲目依赖它们。

```
public class FinalizeDemo {
    private static FinalizeDemo finalizeObj;
    
    public static void main(String[] args) throws InterruptedException {
        finalizeObj=new FinalizeDemo();//创建对象
        //清空对象
        finalizeObj=null;
        //调用gc方法，准备回收实例化的finalizeDemo对象
        System.gc();
        Thread.sleep(500);
    }
    
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("execute finalize");
    }
}
```