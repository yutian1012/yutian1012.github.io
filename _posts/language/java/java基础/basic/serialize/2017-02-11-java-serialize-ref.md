---
title: java序列化存储引用
tags: [basic]
---

序列化时写入同一个对象两次，只会存储一遍，另外再存储一个对象的引用。

Java序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用。

注：该存储规则极大的节省了存储空间。

1）序列化类

```
public class StudentSerialize {

    private String name;
    private int age;
    private Date birth;
    public StudentSerialize(String name) {
        this.name=name;
    }
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
    public Date getBirth() {
        return birth;
    }
    public void setBirth(Date birth) {
        this.birth = birth;
    }
    @Override
    public String toString() {
        return "StudentSerialize [name=" + name + ", age=" + age + ", birth="
                + birth + "]";
    }
}
```

2）测试类

```
public class SerializableMain {
    public static void main(String[] args)throws Exception {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("result.obj"));
        StudentSerialize t = new StudentSerialize("张三");
        //试图将对象两次写入文件
        out.writeObject(t);
        out.flush();
        System.out.println(new File("result.obj").length());//135
        out.writeObject(t);
        out.close();
        System.out.println(new File("result.obj").length());//140

        ObjectInputStream oin = new ObjectInputStream(new FileInputStream("result.obj"));
        //从文件依次读出两个文件
        StudentSerialize t1 = (StudentSerialize) oin.readObject();
        StudentSerialize t2 = (StudentSerialize) oin.readObject();
        oin.close();
                
        //判断两个引用是否指向同一个对象
        System.out.println(t1 == t2);//输入true
    }
}
```

3）分析

序列化对象后文件长度为135，再次序列化同一个对象，文件只增加了5个字节的长度。

5个字符长度的存储空间是新增引用和一些控制信息的空间。反序列化时，恢复引用关系，代码中的t1和t2指向唯一的对象，二者相等，输出true（表示的地址空间相同）。