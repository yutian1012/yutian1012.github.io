---
title: 原型模型
tags: [pattern]
---

复制对象，包括深度复制和浅度复制，深度复制重建引用对象，浅度复制不创建（经典实现：java序列化）

### 1. 什么是原型模型
将一个对象作为原型，对其进行复制、克隆，产生一个和原对象类似的新对象。
原型模式是一种比较简单的模式，也非常容易理解，实现一个接口，重写一个方法即完成了原型模式。

注：原型模式主要用于对象的复制

### 2. Prototype类需要具备以下两个条件：
1）实现Cloneable接口。在java语言有一个Cloneable接口，它的作用只有一个，就是在运行时通知虚拟机可以安全地在实现了此接口的类上使用clone方法。在java虚拟机中，只有实现了这个接口的类才可以被拷贝，否则在运行时会抛出CloneNotSupportedException异常。

2）重写Object类中的clone方法。Java中，所有类的父类都是Object类，Object类中有一个clone方法，作用是返回对象的一个拷贝，但是其作用域protected类型的，一般的类无法调用，因此，Prototype类需要将clone方法的作用域修改为public类型。

### 3. 实例代码
1）原型代码
```
public class Prototype implements Cloneable, Serializable {  
      
    private static final long serialVersionUID = 1L;  
    private String string;  
  
    private SerializableObject obj;  
    /* 浅复制 */  
    public Object clone() throws CloneNotSupportedException {  
        Prototype proto = (Prototype) super.clone();  
        return proto;  
    }  
    /* 深复制 */  
    public Object deepClone(){
        Object obj=null;
        ObjectOutputStream oos=null;
        ObjectInputStream ois=null;
        try{
            /* 写入当前对象的二进制流 */  
            ByteArrayOutputStream bos = new ByteArrayOutputStream();  
            oos = new ObjectOutputStream(bos);  
            oos.writeObject(this);  
            /* 读出二进制流产生的新对象 */  
            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());  
            ois = new ObjectInputStream(bis); 
            
            obj=ois.readObject();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=oos){
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(null!=ois){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return obj;
    }  
  
    public String getString() {  
        return string;  
    }  
  
    public void setString(String string) {  
        this.string = string;  
    }  
  
    public SerializableObject getObj() {  
        return obj;  
    }  
  
    public void setObj(SerializableObject obj) {  
        this.obj = obj;  
    }  
  
}  
  
class SerializableObject implements Serializable {  
    private static final long serialVersionUID = 1L;
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    
}
```
2）客户端代码
```
public class Client {
    public static void main(String[] args) {
        Prototype prototype=new Prototype();
        SerializableObject serObj=new SerializableObject();
        serObj.setName("zhangsan");
        prototype.setString("hello");
        prototype.setObj(serObj);
        
        Prototype t=(Prototype) prototype.deepClone();
        t.setString("world");
        t.getObj().setName("lisi");
        
        //输出对象值
        System.out.println(prototype.getString()+":"+prototype.getObj().getName());
        System.out.println(t.getString()+":"+t.getObj().getName());
    }
}
```
注：这里深度拷贝的实现方式是通过对象序列化实现的，要求类属性需要实现Serializable接口。

### 4. 原理模型的优点
1）使用原型模式创建对象比直接new一个对象在性能上要好的多，因为Object类的clone方法是一个本地方法，它直接操作内存中的二进制流，特别是复制大对象时，性能的差别非常明显。

2）使用原型模式的另一个好处是简化对象的创建，使得创建对象就像我们在编辑文档时的复制粘贴一样简单。

### 5. 原型模型的适应场景
在需要重复地创建相似对象时可以考虑使用原型模式。比如需要在一个循环体内创建对象，假如对象创建过程比较复杂或者循环次数很多的话，使用原型模式不但可以简化创建过程，而且可以使系统的整体性能提高很多。

### 6. 注意事项
1）使用原型模式复制对象不会调用类的构造方法。因为对象的复制是通过调用Object类的clone方法来完成的，它直接在内存中复制数据，因此不会调用到类的构造方法。不但构造方法中的代码不会执行，甚至连访问权限都对原型模式无效。

2）与单例模式的冲突。单例模式中，只要将构造方法的访问权限设置为private型，就可以实现单例。但是clone方法直接无视构造方法的权限，所以，单例模式与原型模式是冲突的，在使用时要特别注意。

参考：http://blog.csdn.net/zhengzhb/article/details/7393528