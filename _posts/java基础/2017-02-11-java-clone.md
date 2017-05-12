---
title: 对象深浅复制
tags: [basic]
---

对象的深拷贝和浅拷贝

参考：http://blog.csdn.net/zhangjg_blog/article/details/18369201

### 1. 什么是浅复制/拷贝

浅复制：将一个对象复制后，基本数据类型的变量都会重新创建，而引用类型，指向的还是原对象所指向的。

### 2. 什么是深复制/拷贝

深复制：将一个对象复制后，不论是基本数据类型还有引用类型，都是重新创建的。

### 3. 思考

clone方法拷贝对象中的字符串时是浅拷贝，即两个对象都指向同一个string对象。在使用时，如果改变字符串内容，则会创建新的字符串，这是字符串自身的性质。看上去像是深拷贝，其实不是。通过打印字符串地址可以证明。对于int类型的数据使用的是直接拷贝值。但integer类型呢？基本数据类型的包装类在拷贝时应该是浅拷贝，这样在修改一个对象的值时，是会影响其他对象的。

```
public class Student implements Cloneable{
    public String name;
    public int age;
    public Integer grade;
    public Student(String name,int age,Integer grade){
        this.name=name;
        this.age=age;
        this.grade=grade;
    }
    
    public static void main(String[] args) throws CloneNotSupportedException {
        Student stu=new Student("dddd", 12, 3);
        Student stu2=(Student) stu.clone();
        
        //判断深浅克隆
        System.out.println(stu.name==stu2.name);//true
        System.out.println(stu.grade==stu2.grade);//true

        //改变值查看
        stu.grade=5;
        System.out.println(stu2.grade);//3
        System.out.println(stu.grade==stu2.grade);//false
    }
}
```

注：证明String类和包装了的clone是浅拷贝，但在使用时却不需要担心。

注2：stu对象的grade（Integer包装类型）成员变量对象在赋值为5时，会将5包装成一个新的Integer对象，并赋值给stu对象的成员变量grade，而不会影响原来的stu2对象中grade的指向。

结论：String，基本数据类型以及基本数据类型的包装类型在使用拷贝时不需要关心其拷贝方式，拷贝对象和原对象在使用时不会相互影响。


