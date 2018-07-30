---
title: java相关
tags: [interview]
---

1）类的实例化过程： 

a.父类中的static代码块和static变量信息，当前类的static 

b.顺序执行父类的普通代码块

c.父类的构造函数

d.子类普通代码块

e.子类（当前类）的构造函数，按顺序执行

f.子类方法的执行

注：静态优先，其次代码块，最后构造函数。

2）实例代码

```
//父类
public class JavaInstanceSuperDemo {
    private static String sId="hello";
    
    static {
        System.out.println("super static block invoked!");
    }
    
    {
        System.out.println("super plain block invoked!");
    }
    
    public JavaInstanceSuperDemo() {
        System.out.println("super constructor invoked!");
    }
}
//子类
public class JavaInstanceDemo extends JavaInstanceSuperDemo{
    private static String id="hello";
    
    static {
        System.out.println("sub static block invoked!");
    }
    
    {
        System.out.println("sub plain block invoked!");
    }
    
    public JavaInstanceDemo() {
        System.out.println("sub constructor invoked!");
    }
    
    public static void main(String[] args) {
        JavaInstanceDemo instance=new JavaInstanceDemo();
    }
}
```

3）输出

```
//输出信息
super static block invoked!
sub static block invoked!
super plain block invoked!
super constructor invoked!
sub plain block invoked!
sub constructor invoked!
```