---
title: java序列化1
tags: [basic]
---

java序列化操作

### 1. Java序列化与反序列化
Java序列化是指把Java对象转换为字节序列的过程；而Java反序列化是指把字节序列恢复为Java对象的过程。

注：序列化真正需要保存的只是对象属性的值，和对象的类型。而不会去序列化什么所谓的方法。只要你的JVM classloader可以load到这个类，那么类方法指令自然就可以获得。

### 2. 为什么需要序列化与反序列化（有什么用）
1)实现了数据的持久化，通过序列化可以把数据永久地保存到硬盘上（通常存放在文件里），或数据库中。

2)利用序列化实现远程通信，即在网络上传送对象的字节序列

3)当你想通过RMI（远程方法调用）传输对象的时候。

注：只有序列化的对象才可以存储在存储设备上。为了对象的序列化而需要继承的接口也只是一个象征性的接口而已。也就是说继承这个接口说明这个对象可以被序列化了，没有其他的目的。之所以需要对象序列化，是因为有时候对象需要在网络上传输，传输的时候需要这种序列化处理，从服务器硬盘上把序列化的对象取出，然后通过网络传到客户端，再由客户端把序列化的对象读入内存，执行相应的处理。

### 3. 如何实现序列化和反序列化
1）JDK类库中序列化API--操作类

java.io.ObjectOutputStream：表示对象输出流，它的writeObject(Object obj)方法可以对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中。

java.io.ObjectInputStream：表示对象输入流，它的readObject()方法源输入流中读取字节序列，再把它们反序列化成为一个对象，并将其返回。

2）实现序列化的要求--被序列化类的要求

只有实现了Serializable或Externalizable接口的类的对象才能被序列化，否则抛出异常。

3）实现Java对象序列化与反序列化的方法--调用方法实现序列化

假定一个Student类，它的对象需要序列化，可以有如下**三种方法**：

方法一：若Student类仅仅实现了Serializable接口，则可以按照以下方式进行序列化和反序列化
ObjectOutputStream采用默认的序列化方式，对Student对象的非transient的实例变量进行序列化。
ObjcetInputStream采用默认的反序列化方式，对对Student对象的非transient的实例变量进行反序列化。

方法二：若Student类仅仅实现了Serializable接口，并且还定义了readObject(ObjectInputStream in)和writeObject(ObjectOutputSteam out)，则采用以下方式进行序列化与反序列化。

ObjectOutputStream调用Student对象的writeObject(ObjectOutputStream out)的方法进行序列化。

ObjectInputStream会调用Student对象的readObject(ObjectInputStream in)的方法进行反序列化。

方法三：若Student类实现了Externalnalizable接口，且Student类必须实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法，则按照以下方式进行序列化与反序列化。

ObjectOutputStream调用Student对象的writeExternal(ObjectOutput out))的方法进行序列化。

ObjectInputStream会调用Student对象的readExternal(ObjectInput in)的方法进行反序列化。

4）总结：

序列化的实现：将需要被序列化的类实现Serializable接口，该接口没有需要实现的方法，implements Serializable只是为了标注该对象是可被序列化的，然后使用一个输出流(如：FileOutputStream)来构造一个 ObjectOutputStream(对象流)对象，接着，使用ObjectOutputStream对象的writeObject(Object obj)方法就可以将参数为obj的对象写出(即保存其状态)，要恢复的话则用输入流。

### 4. JDK类库中序列化的步骤
步骤一：创建一个对象输出流，它可以包装一个其它类型的目标输出流，如文件输出流：
```
ObjectOutputStream out = new ObjectOutputStream(new fileOutputStream("D:\\objectfile.obj"));
```
步骤二：通过对象输出流的writeObject()方法写对象：
```
out.writeObject(“Hello”);
out.writeObject(new Date());
```

### 5. JDK类库中反序列化的步骤
步骤一：创建一个对象输入流，它可以包装一个其它类型输入流，如文件输入流：
```
ObjectInputStream in = new ObjectInputStream(new fileInputStream("D:\\objectfile.obj"));
```
步骤二：通过对象输出流的readObject()方法读取对象：
```
String obj1 = (String)in.readObject();
Date obj2 = (Date)in.readObject();
```
说明：为了正确读取数据，完成反序列化，必须保证向对象输出流写对象的顺序与从对象输入流中读对象的顺序一致。

参考：http://blog.csdn.net/wangloveall/article/details/7992448/

### 6. java api中的常用类实现了serializable接口
java中常见的几个类（如：Interger、String等），都实现了serializable接口。

### 7. 序列化类的所有子类
序列化类的所有子类本身都是可序列化的。

### 8. 序列化(Serializable)和外部化(Externalizable )的区别
1）两者的关系
```
public interface Externalizable extends Serializable {
        void readExternal(ObjectInput in);
        void writeExternal(ObjectOutput out);
}
```
2）区别

通过Serializable接口对对象序列化的支持是内建于核心 API 的，但是java.io.Externalizable的所有实现者必须提供读取和写出的实现。Java 已经具有了对序列化的内建支持，也就是说只要实现java.io.Serializable接口，Java 就会试图存储和重组你的对象。如果使用外部化，你就可以完全由自己完成读取和写出的工作。

3）优缺点

Serializable接口，优点：内建支持，易于实现。缺点：占用空间过大，由于额外的开销导致速度变比较慢。

Externalizable接口，优点：开销较少（程序员决定存储什么），可观的速度提升。缺点：虚拟机不提供任何帮助，也就是说所有的工作都落到了开发人员的肩上。

### 9. 应用场景
1）比如有一个类记录用户信息设置等，当你程序退出后下次再运行要保留上次的信息设置，那你就可以把这个类作为配置文件存在磁盘上，每次运行的时候再读取。

2）将二级缓存中的内容持久化保存下来，便于恢复缓存的信息，hibernate的缓存机制通过使用序列化。

3) RMI 技术是完全基于 Java 序列化技术的，服务器端接口调用所需要的参数对象来至于客户端，它们通过网络相互传输。

### 问题：父类的成员变量会被子类序列化到文件中吗？
如果父类实现了Serializable接口，父类的成员变量会被序列化，反之不会。

父类没有实现Serializable即可，而子类要想反序列化成功，父类必须提供默认无参构造函数。
