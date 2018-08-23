---
title: Lambda表达式的底层实现
tags: [jdk]
---

参考：https://blog.csdn.net/peiyuWang_2015/article/details/72630444

通过实例分析lambda底层的实现

1）实例代码

静态方法中使用lambda表达式：(a,b)-> a+b，可以先简单的理解为，使用lambda构造了匿名内部类的方式实现方法调用。

```
package org.sjq.test.demo.lambda;

@FunctionalInterface
public interface ILambdaCaculator {
    int result(int a,int b);
}
```

相关调用实现

```
package org.sjq.test.demo.lambda;

public class LambdaCaculatorDemo {
    public static void main(String[] args) {
        //在静态方法中使用lambda表达式：(a,b)-> a+b，可以先简单的理解为，使用lambda构造了匿名内部类的方式实现方法调用
        System.out.println("add :"+LambdaCaculatorUse((a,b)-> a+b,12,14));
    }
    public static int LambdaCaculatorUse(ILambdaCaculator lambda,int a,int b){
        return lambda.result(a, b);
    }
}
```

注：这里并没有类实现了ILambdaCaculator接口，而是使用lambda表达式进行的实现。

2）分析代码的执行过程

运行代码查看能否正确输出？可正确输出26

对LambdaCaculatorDemo类进行反编译分析：

```
# 定位到target目录下的LambdaCaculatorDemo类目录，cmd中运行，并重定向到文件中
javap -p LambdaCaculatorDemo.class > E:/lambdaCaculatorDemo.txt
```

输出结果

```
Compiled from "LambdaCaculatorDemo.java"
public class org.sjq.test.demo.lambda.LambdaCaculatorDemo {
  public org.sjq.test.demo.lambda.LambdaCaculatorDemo();
  public static void main(java.lang.String[]);
  public static int LambdaCaculatorUse(org.sjq.test.demo.lambda.ILambdaCaculator, int, int);

  private static int lambda$0(int, int);
}
```

注：查看代码可以看到这里多出来一个lambda$0方法，并且该方法是private static进行修饰的。

3）进一步分析

lambda$0方法有什么用呢？带着问题分析是一个比较好的思考角度

猜测一下上面生成代码的作用。lambda$0接收两个参数，与我们定义lambada表达式使用的参数列(a,b)是一致的，那么方法体则很可能是a+b

```
# lambda表达式
(a,b)-> a+b
```

使用反编译工具，进一步查看代码实现

```
javap -p -c LambdaCaculatorDemo.class > E:/lambdaCaculatorDemo.txt
```

lambda$0方法体的实现

```
private static int lambda$0(int, int);
    Code:
       0: iload_0
       1: iload_1
       2: iadd
       3: ireturn
```

注：方法中可以看到，加载两个变量，对变量进行加法计算，返回值。

4）分析main方法的实现

main方法上面反编译的结果，其他方法的反编译代码比较容易辨识，这里不在列出。从12开始到21是调用逻辑代码LambdaCaculatorUse((a,b)-> a+b,12,14)

```
public static void main(java.lang.String[]);
    Code:
       0: getstatic     #16                 // Field java/lang/System.out:Ljava/io/PrintStream;
       3: new           #22                 // class java/lang/StringBuilder
       6: dup
       7: ldc           #24                 // String add :
       9: invokespecial #26                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V

      12: invokedynamic #32,  0             // InvokeDynamic #0:result:()Lorg/sjq/test/demo/lambda/ILambdaCaculator;
      17: bipush        12  //将立即数12压入栈
      19: bipush        14  //将立即数14压入栈
      21: invokestatic  #33                 // Method LambdaCaculatorUse:(Lorg/sjq/test/demo/lambda/ILambdaCaculator;II)I

      24: invokevirtual #37                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      27: invokevirtual #41                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      30: invokevirtual #45                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      33: return
```

注：可以看到 12: invokedynamic #32,  0，涉及到ILambdaCaculator接口。