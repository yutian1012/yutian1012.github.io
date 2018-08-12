---
title: jvm类加载器(面试)
tags: [jvm]
---

参考：https://blog.csdn.net/zhangliangzi/article/details/51338291

1）JVM运行流程

java源程序（.java）-->java字节码文件(.class)-->JVM（加载到内存）

注：JVM会根据不同平台解释成不同的本地机器指令。

2）JVM基本结构

类加载器，执行引擎，运行时数据区，本地接口。

class file-->classloader-->运行时数据区-->执行引擎-->本地库接口

3）类加载过程

类的装载过程：加载，连接（验证，准备，解析），初始化，使用，卸载。

注：前三步是由classloader来完成的。

加载：将class文件的二进制流加载到方法区中（类结构信息和静态常量信息）

初始化：执行类的构造器，为类的静态变量赋予正确的初始值。类的构造器包含：静态变量和静态代码块，构造器的执行顺序是按class文件的书写顺序来执行的，构造器执行完成会产生一个类对象，保存到堆中。

```
public class Demo{
    public static int tmp=1;
    static{
        tmp=2;
    }
    public static void main(String[] args){
        tmp=3
    }
}
```

问题：当类加载完成后，为什么要先执行static修饰的变量和代码块，而不是直接执行main方法。

类加载完成后，类的构造器会执行相关静态变量的初始化。

问题：构造器与构造方法的区别

构造器是用来构造类的，与构造方法不同，构造方法是用来构造对象的。

4）类加载器双亲委派模型

类加载默认都有一个父类加载器，bootstrap是顶级类加载器。

classloader在试图加载一个类时会先将任务委托给父类加载器（可能父类加载器已经加载过来这个类），每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载其中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的Class），子类加载器才会尝试自己去加载。

注：避免类的重复加载，如果没有双亲委派加载机制，我们可以随意定义String系统类，而使用我们自定义的类加载来加载，从而改变了String类的底层实现。

5）ClassLoader的种类

JDK默认的类加载器：

BootStrapClassLoader，java程序要运行，JVM必须要先启动，而这个启动过程使用的加载器就是bootstrap加载器。该加载器用于加载rt.jar文件。

Extension Classloader（extends ClassLoader），加载%JAVA_HOME%/lib/ext/*.jar

App Classloader（extends ClassLoader），加载classpath下的jar

自定义类加载器（extends ClassLoader），完成自定义加载类

```
public class Demo2{
    public static void main(String[] args){
        ClassLoader loader=Demo2.class.getClassLoader();
        while(loader!=null){
            System.out.println(loader);
            lodar=loader.getParent();
        }
    }
}
```

4）ClassLoader源码解析

rt.jar包中java.lang.Classloader源码，查看loadClass方法的实现，findClass目的是实现自己来加载类。

方法执行过程：传入类的名称到loadClass中，调用findLoadedClass方法（判断该类加载器是否已经加载了），如果没有加载，则调用父了的loadClass（递归操作），如果父类没有加载到，退回到该类加载，则调用findClass方法自己进行加载。

5）从源码分析实现自定义类加载器

自定义类加载器不建议复写loadClass方法，尽量复写findClass方法

```
public class MyClassLoader extends ClassLoader{
    private String path;//加载类的路径
    private String name;//类加载器名称
    public MyClassLoader(String path,String name){
        super();//让系统类加载器作为该类的父类加载器
        this.path=path;
        this.name=name;
    }

    public MyClassLoader(ClassLoader parent,String path,String name){
        super(parent);//指定类加载器作为父类加载器
        this.path=path;
        this.name=name;
    }
    //加载自己定义的类，通过自定义类加载器
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException{
        //读取Class文件
        byte[] data=readClassFileToByteArr(name);
        return this.defineClass(name,data,0,data.length);
    }

    //获取类的字节码文件的字节数组
    private byte[] readClassFileToByteArr(String name){
        InputStream is=null;
        byte[] returnData=null;
        name=name.replaceAll("\\.","/");
        String filePath=this.path+name+".class";
        File file=new file(filePath);

        //通过输入流
        ByteArrayOutputStream os=new ByteArrayOutputStream();
        try{
            is=new FileInputStream(file);
            int tmp=0;
            while((tem=is.read())!=-1){
                os.write(tmp);
            }
            returnData=os.toByteArray();
        }catch(Exception e){
            
        }finally{
            //关闭流
            ...
        }
        return returnData;    
    }
}

// 假设在F盘创建一个Demo.class文件

//测试类
public class TestDemo{
    public static void main(String[] args){
        // 第一个参数为类加载器名称，第二个参数为类加载路径
        MyClassLoader laoder=new MyClassLoader("xxxx","F:/");
        Class<?> c=loader.loadClass("Demo");//传递报名+类名
        c.newInstance();
    }
}
```

问题：如果使用自定义类加载器加载

方式一：将类不要放到App classLoader能够加载到的目录下，

方式二：设置自定义类加载器的父加载器为null，即为bootStrap加载器。

方式三：自定义类加载器字节实现loadClass，不再委托父类加载，自己加载就好。

问题：Tomcat自定义的ClassLoader的实现