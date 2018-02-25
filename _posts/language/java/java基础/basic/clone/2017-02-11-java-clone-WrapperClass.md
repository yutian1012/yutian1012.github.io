---
title: 对象拷贝中包装类型和String类型
tags: [basic]
---

结论：String，基本数据类型以及基本数据类型的包装类型在使用拷贝时不需要关心其拷贝方式，拷贝对象和原对象在使用时不会相互影响。

1）字符串类型的拷贝问题

clone方法拷贝对象中的字符串时是浅拷贝，即两个对象都指向同一个string对象。在使用时，如果改变字符串内容，则会创建新的字符串，这是字符串自身的性质。看上去像是深拷贝，其实不是。

注：通过打印字符串地址可以证明。

2）java原生类型与包装类型的拷贝问题

对于int类型的数据使用的是直接拷贝值。但integer类型呢？基本数据类型的包装类在拷贝时是浅拷贝（只拷贝引用地址），但是当改变包装类型的值时，会重新创建一个新的包装类型对象。因此，表面上看来没有影响到原对象的包装类型变量的值。

这种现象可以称为延迟复制。但不是真正的复制，而是在使用时重新创建了新的对象。


这样在修改一个对象的值时，是会影响被克隆对象的成员对象值。

3）代码示例

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
        
        //判断深浅克隆，地址值都相同，因此可以判断出是浅拷贝
        System.out.println(stu.name==stu2.name);//true
        System.out.println(stu.grade==stu2.grade);//true

        //改变值查看，这里5会自动装箱，即new Integer(5)
        //将新创建对象复制给stu的grade成员变量上，
        //但并没有改变stu2的grade对象的指向
        stu.grade=5;
        System.out.println(stu2.grade);//3
        System.out.println(stu.grade==stu2.grade);//false 
    }
}
```

注：证明String类和包装了的clone是浅拷贝，但在使用时却不需要担心。

注2：stu对象的grade（Integer包装类型）成员变量对象在赋值为5时，会将5包装成一个新的Integer对象，并赋值给stu对象的成员变量grade，而不会影响原来的stu2对象中grade的指向。