---
title: java虚拟机线程相关的概念
tags: [architecture]
---

### 相关概念

1 原子性

原子性是指一个操作是不可中断的，即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

另外，在32位的机器上读64位的Long，并不是一个原子的操作。

2 有序性

1）有序性指，程序的执行顺序与程序的书写顺序未必是一样的，编译后的程序代码可能会出现语句位置颠倒的情况。

注：顺序性是不可预知的，有可能编译顺序和书写顺序一样，也有可能不一样（JVM内部的优化策略会对其产生影响，而这种影响往往是不可预知的）。

实例：

```
class OrderExample{
    int a=0;
    boolean flag=false;
    public void writer(){
        a=1;
        flag=true;
    }
    public void reader(){
        if(flag){
            int i=a+1;
        }
    }
}
```

实际执行时flag=true可能会在a=1之前先被执行到。而在多线程中执行时就会出现问题。线程A执行writer方法，先将flag=true执行，这时线程B执行reader方法，判断flag为true，就进行执行判断体，这时默认a=1，而实际上a=1还没有在线程A的writer方法中执行。OrderExample实例做为线程间共享的资源被多线程访问时，就容易出现逻辑上的错误。

2）有序性出现乱序的原理

在一条指令的执行可以分为很多步骤（被编译为汇编的指令）：

取指 IF；译码和取寄存器操作数 ID；执行或者有效地址计算 EX；存储器访问 MEM；写回 WB；

3）指令重排可以使流水线更加顺畅

```
//执行指令
a=b+c
d=e-f

//汇编
LW Rb,b        --  IF ID EX MEM WB
LW Rc,c        --     IF ID EX  MEM WB
ADD Ra,Rb,Rc   --        IF ID  X   EX MEM WB
SW a,Ra        --           IF  X   ID EX  MEM WB
LW Re,e        --               X   IF ID  EX  MEM WB
LW Rf,f        --                      IF  ID  EX  MEM WB
SUB Rd,Re,Rf   --                          IF  ID  X   EX MEM WB
SW d,Rd        --                              IF  X   ID EX  MEM WB

//指令重排优化减少耽搁的时钟周期
LW Rb,b        --  IF ID EX MEM WB
LW Rc,c        --     IF ID EX  MEM WB
LW Re,e        --        IF ID  EX  MEM WB
ADD Ra,Rb,Rc   --           IF  ID  EX  MEM WB
LW Rf,f        --               IF  ID  EX  MEM WB
SW a,Ra        --                   IF  ID  EX  MEM WB
SUB Rd,Re,Rf   --                       IF  ID  EX  MEM WB
SW d,Rd        --                           IF  ID  EX  MEM WB
```

注1：第三条指令执行时，由于c还没有从MEM中读入，因此指令流水线必须耽搁一个时钟周期，这里使用X表示该时钟周期不执行。但是第三条可以不用等到c写入到寄存器中后才开始执行，计算机中使用旁路技术就可以取到数据c，因此这里只耽误一个时钟周期就好。

注2：通过指令重排的方式消除了耽搁不执行的时钟周期，从而提高了效率。

注3：指令重排会导致指令的执行顺序与书写顺序会有不同，目的是使指令运行的流水线更加顺畅。

3 可见性

1）可见性是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。

针对多核cpu，每个cpu都有自己独立的寄存器，独立的高速缓存，在不同线程运行过程中，一个值的存放就有可能不能被其他线程/其他cpu访问到。

注：JVM中编译器优化会对可见性产生影响。同时硬件的优化也有可能对可见性造成影响。可见性问题是可能会因为各种优化策略导致的。

2）java volatile汇编代码研究：

参考：http://hushi55.github.io/2015/01/05/volatile-assembly

查看汇编命令，在生成的class文件上运行

```
java -server -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly  classfile
```

注：server模式上的优化要明显比client的优化强很多。如果要看到相应的优化策略后的代码重排现象，可以在java命令中添加-server参数。

4 Happen-Before规则

程序顺序原则：一个线程内保证语义的串行性，即在单线程中，不管是否发生指令重排，都会保证语义的正确性。

volatile规则：volatile变量的写，先发生于读，这保证了volatile变量的可见性

锁规则：解锁（unlock）必然发生在随后的加锁（lock）前

传递性：A先于B，B先于C，那么A必然先于C

线程的start方法先于它的每一个动作

线程的所有操作先于线程的终结（Thread.join）

线程的中断（interrupt）先于被中断线程的代码

对象的构造函数执行结束先于finalize方法
