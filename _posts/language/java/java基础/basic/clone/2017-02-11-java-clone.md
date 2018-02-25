---
title: 对象深浅复制
tags: [basic]
---

*Java中要想自定义类的对象可以被复制，自定义类就必须实现Cloneable接口，并重写clone()方法。*

### 对象的深拷贝和浅拷贝

参考：https://www.cnblogs.com/acode/p/6306887.html

1）什么是浅复制/拷贝

浅复制：将一个对象复制后，基本数据类型的变量都会重新创建，而引用类型，指向的还是原对象所指向的。

注：浅拷贝不会将对象类型的成员进行复制，而只是简单的利用引用指针指向同一个内存地址下的对象。

2）什么是深复制/拷贝

深拷贝是相对于浅拷贝来讲的，深复制将一个对象复制后，不论是基本数据类型还有引用类型，都是重新创建的。

注：要求对象类型的成员必须同时满足实现了Cloneable接口，并提供了相应的clone方法。

### 深拷贝实例

1）Professor教授类

每个学生类中都包含一个教授类。为了在clone学生类不影响教授对象的值，即深拷贝时，教授对象也进行拷贝。因此Professor类要实现Cloneable接口。因为该类中并不含任何其他符合类型的成员变量，因此，clone方法只需要简单调用super.clone方法即可。

```
public class Professor implements Cloneable {

    private String name;

    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "Professor [name=" + name + ", age=" + age + "]";
    }

    public Object clone() throws CloneNotSupportedException{
        return super.clone();
    }
}
```

2）Student学生类

该类包含一个Professor类作为其成员变量，因此在深拷贝时，需要在clone方法中调用professor类的clone方法，实现将成员变量Professor进行拷贝并赋值给新到拷贝对象上。

```
public class Student implements Cloneable {
    
    private String name;
    
    private int age;
    
    private Professor professor;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Professor getProfessor() {
        return professor;
    }

    public void setProfessor(Professor professor) {
        this.professor = professor;
    }

    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + ", professor="
                + professor + "]";
    }
    
    public Object clone() throws CloneNotSupportedException{
        Student newStudent = (Student) super.clone();
        newStudent.professor = (Professor) professor.clone();
        return newStudent;
    }

}
```

3）测试类

```
public class ShadowCopy {
    public static void main(String[] args) {
        Professor p1 = new Professor();
        p1.setName("Professor Zhang");
        p1.setAge(30);

        Student s1 = new Student();
        s1.setName("xiao ming");
        s1.setAge(18);
        s1.setProfessor(p1);

        System.out.println(s1);

        try {
            Student s2 = (Student) s1.clone();
            s2.setName("xiao hong");
            s2.setAge(17);
            Professor p2 = s2.getProfessor();
            p2.setName("Professor Li");
            p2.setAge(45);
            s2.setProfessor(p2);
            System.out.println("复制后的：s1 = " + s1);
            System.out.println("复制后的：s2 = " + s2);
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

    }
}
```