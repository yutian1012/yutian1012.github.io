---
title: 构造器
tags: [pattern]
---

### 类的实例化过程

静态变量和静态代码块是在类加载完成后就会执行和初始化的。然后是类的成员变量分配空间或执行相应的初始化（age默认赋值0，birthday会执行new Date初始化数据）。然后调用构造函数，构造函数可以对成员变量进行参数重新设置值。最后返回类的实例对象。

```
public class ConstructorTest {
    //类被加载时就会初始化
    static{
        System.out.println(name);
        System.out.println("静态代码块执行");
    }
    
    {
        System.out.println("动态代码块，成员变量初始化，按定义的先后顺序执行");
    }
    
    private int age;
    private Date birthday=new Date();
    
    public ConstructorTest(int age){
        if(null!=birthday){
            System.out.println("成员变量已默认初始化，动态代码块已执行");
            System.out.println(this.age);//默认值为0
            System.out.println(birthday);//默认值为初始设置值
        }
        System.out.println("构造函数执行");
        this.age=age;
        System.out.println(this.age);//构造函数显示的设置变量值
    }
    
    public static void main(String[] args) {
        ConstructorTest test=new ConstructorTest(22);
    }
}
```

### 子类的初始化过程

先执行父类的静态代码块，初始化父类的静态变量。然后执行子类静态代码块，初始化子类的静态变量。执行父类的成员变量的初始化操作，执行父类的构造函数。然后执行子类的成员变量初始化操作，执行子类的构造函数。最后，返回类的实例。

```
public class ConstructorTest {
    //类被加载时就会初始化
    private static final String name="sss";
    static{
        System.out.println(name);
        System.out.println("静态代码块执行");
    }
    
    {
        System.out.println("动态代码块，成员变量初始化，按定义的先后顺序执行");
    }
    
    private int age;
    private Date birthday=new Date();
    
    public ConstructorTest(int age){
        if(null!=birthday){
            System.out.println("成员变量已默认初始化，动态代码块已执行");
            System.out.println(this.age);//默认值为0
            System.out.println(birthday);//默认值为初始设置值
        }
        System.out.println("构造函数执行");
        this.age=age;
        System.out.println(this.age);//构造函数显示的设置变量值
    }
    
    public static void main(String[] args) {
        ConstructorSubTest test=new ConstructorSubTest(22);
    }
}

class ConstructorSubTest extends ConstructorTest{
    private String hello="sss";
    {
        System.out.println(hello);
        System.out.println("子类动态代码块，成员变量初始化，按定义的先后顺序执行");
    }
    static{
        System.out.println("子类静态代码块执行");
    }
    public ConstructorSubTest(int age) {
        super(age);
        System.out.println("子类构造函数执行");
    }
}
```

输出结果

```
静态代码块执行
子类静态代码块执行
sss
动态代码块--成员变量初始化，也会执行，按定义的先后顺序执行
成员变量已默认初始化
0
Sun Aug 13 14:23:53 CST 2017
构造函数执行
22
sss
子类动态代码块--成员变量初始化，也会执行，按定义的先后顺序执行
子类构造函数执行
```