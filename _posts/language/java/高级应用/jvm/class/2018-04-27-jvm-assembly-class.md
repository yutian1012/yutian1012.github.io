---
title: jvm反汇编执行
tags: [jvm]
---

参考：https://blog.csdn.net/u012807459/article/details/52327603

为了更好地理解Java代码，内部具体是怎么运行，常常会通过反汇编来查看汇编代码。

1）实例代码

```
public class Hello {
    
    public void test(){
        int b=1;
        int a = b + 1;
        System.out.println("a="+a);
    }

    public static void main(String[] args) {
        Hello hello=new Hello();
        hello.test();
    }
}
```

2）编译并输出执行过程

```
javac Hello.java
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly Hello
```

注：使用PrintAssembly前需要先UnlockDiagnosticVMOptions参数来激活相应汇编设置。

错误提示

```
Could not load hsdis-amd64.dll; library not loadable; PrintAssembly is disabled
```

下载依赖的dll文件，也可以通过openjdk源码进行编译生成dll文件。将hsdis-amd64.dll文件放置到jdk1.8.0_131\jre\bin\server（这里放置到了jdk中的jre目录下）。

再次运行输出汇编信息

```
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly Hello > hello.txt
```

注：这种方式输出的内容太多，不便于查看。

```
java -XX:+UnlockDiagnosticVMOptions -XX:CompileCommand=print,*Hello.test Hello > hello.txt
```

注：这里只输出了方法名，并没有输出汇编信息