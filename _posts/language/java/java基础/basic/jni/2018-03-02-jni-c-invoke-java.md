---
title: C语言嵌入式JVM调用java
tags: [jni]
---

参考：http://blog.csdn.net/lin772662623/article/details/72859061

环境介绍，在windows平台下安装了MinGW-w64，从而提供了c/C++的编译运行环境。

1）编写Java类

新建HelloWorld.java文件，该文件提供被c调用的函数

```
package org.sjq.test.jni;

public class HelloWorld {
        public static int intMethod(int n){
                return n*n;
        }

        public static boolean booleanMethod(boolean bool) {
                return !bool;
        }
        public static void sayHello()
        {
                System.out.println("say Hello from C.");
        }
}
```

2）使用javac命令编译所编写的java类

```
javac -d . HelloWorld.java
```

3）编写c文件

新建hello.c文件，该文件将调用上面的java类。

编码步骤：

首先，包括准备本机应用程序以处理Java代码

然后，将JVM嵌入本机应用程序

最后，然后从该应用程序内找到并调用Java方法

```
#include <jni.h>
#include <string.h>

#ifdef _WIN32  
#define PATH_SEPARATOR ';'  
#else
#define PATH_SEPARATOR ':'  
#endif

int main(){
   
    JavaVMOption options[1];
    //表示JNI执行环境。
    JNIEnv *env;  
    //是指向JVM的指针，我们主要使用这个指针来创建、初始化和销毁JVM。
    JavaVM *jvm;
    //表示可以用来初始化JVM的各种JVM参数。
    JavaVMInitArgs vm_args;
    
    //创建JVM是否成功标识
    long status;
    //加载的java类句柄
    jclass cls;
    //获取的类方法句柄
    jmethodID mid;
    //获取方法返回int值句柄
    jint square;
    //获取方法返回boolean值句柄
    jboolean not;

    //向jvm参数中设置类路径
    options[0].optionString = "-Djava.class.path=.";
    memset(&vm_args, 0, sizeof(vm_args));
    vm_args.version = JNI_VERSION_1_2;
    vm_args.nOptions = 1;
    vm_args.options = options;

    //创建内嵌的JVM虚拟机
    status = JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args);

    if (status != JNI_ERR){
        //在类路径下查找加载的类
        cls = (*env)->FindClass(env, "org/sjq/test/jni/HelloWorld");

        //printf("cls=%d...\n",cls);

        if(cls !=0){
            //类中查找intMethod方法，类似java反射
            mid = (*env)->GetStaticMethodID(env, cls, "intMethod", "(I)I");
            
            if(mid !=0){
                //调用方法获取返回值，传递数值5，获取数值平方
                square = (*env)->CallStaticIntMethod(env, cls, mid, 5);
                printf("Result of intMethod: %d\n", square);
            }

            mid = (*env)->GetStaticMethodID(env, cls, "booleanMethod", "(Z)Z");
            if(mid !=0){
                not = (*env)->CallStaticBooleanMethod(env, cls, mid, 1);
                printf("Result of booleanMethod: %d\n", not);
            }

            mid = (*env)->GetStaticMethodID(env, cls, "sayHello", "()V");
            if(mid !=0){
                (*env)->CallStaticVoidMethod(env, cls, mid);
                printf("Result of voidMethod");
            }
        }

        (*jvm)->DestroyJavaVM(jvm);
        return 0;
    }
    else{
        return -1;
    }
}
```

a.引入jni.h头文件

jni.h文件包含在C代码中所需要的JNI的所有类型和函数定义。

b.JavaVMOption数组

声明所有希望在程序中使用的变量。JavaVMOption数组存放JVM的各种选项设置，要确保所声明的JavaVMOption数组足够大，以便能容纳您希望使用的所有选项。

注：在本例中，我们使用的唯一选项就是类路径选项。因为在本示例中，我们所有的文件都在同一目录中，所以将类路径设置成当前目录。

c.JNI_CreateJavaVM方法

创建JVM，处理完所有设置之后，现在就准备创建JVM了。先从调用方法JNI_CreateJavaVM开始。如果成功，则这个方法返回零，否则，则返回JNI_ERR，表示无法创建JVM。

d.装载类

查找并装入Java类，一旦创建了JVM之后，就可以准备开始在本机应用程序中运行Java代码。使用FindClass函数查找并装入Java类。cls变量存储执行FindClass函数后的结果。如果找到该类，则cls变量表示该Java类的句柄，如果不能找到该类，则cls将为零。

e.查找类方法

调用GetStaticMethodID函数在该类中查找某个方法。传递方法名，以及方法参数。如果找到了该方法，则mid变量表示该方法的句柄。如果不能找到该方法则mid将为零。

f.调用类方法

调用CallStaticIntMethod方法，实现java类方法的调用。传递参数cls（表示类）、mid（表示方法）以及用于该方法一个或多个参数。

4）编译并调用c文件

```
gcc -o hello hello.c -D_JNI_IMPLEMENTATION_ -I%JAVA_HOME%\include -I%JAVA_HOME%\include\win32 -L%JAVA_HOME%\jre\bin\server\ -ljvm
```

-I：指定第一个寻找头文件的目录

-L：指定第一个寻找库文件的目录

-l：表示在库文件目录中寻找指定的动态库文件

5）错误提示

提示无法启动此程序，因为计算机中丢失jvm.dll。尝试重新安装该程序以解决此问题。

![](/images/middleware/jni/c-jni-java-problem.png)

解决方法：

```
# 在path中添加jvm.dll所在的目录
%JAVA_HOME%\jre\bin\server;
```

6）运行结果与目录结构

运行结果

![](/images/middleware/jni/c-jni-java-output.png)

程序目录结构

![](/images/middleware/jni/c-jni-java.png)