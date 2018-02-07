---
title: java序列化2
tags: [basic]
---

java序列化实例

### 1. 查看序列化保存的内容
待序列化类代码：

```
public class StudentSerialize implements Serializable{

    private static final long serialVersionUID = 1L;
    
    private String name;
    private int age;
    private Date birth;
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

将对象序列化到文件中

```
public class SerializableMain {
    public static void main(String[] args) {
        StudentSerialize student=new StudentSerialize();
        student.setName("张三");
        student.setAge(13);
        student.setBirth(new Date());
        
        SerializableMain main=new SerializableMain();
        main.serialzeObject(student);
        
        StudentSerialize stu=main.getObjectFromSerialize();
        System.out.println(stu);
    }
    /**
     * 将对象序列化输出到文件中
     * @param obj
     */
    private void serialzeObject(Object obj){
        String filePath="E:/serialObj.txt";
        ObjectOutputStream oos=null;
        try{
            oos=new ObjectOutputStream(new FileOutputStream(filePath));
            oos.writeObject(obj);
        }catch(Exception e){
            e.printStackTrace();
        }
        finally{
            if(null!=oos){
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 获取序列化对象
     * @param clazz
     * @return
     */
    @SuppressWarnings("unchecked")
    private <T> T getObjectFromSerialize(){
        String filePath="E:/serialObj.txt";
        ObjectInputStream ois=null;
        try{
            ois=new ObjectInputStream(new FileInputStream(filePath));
            Object obj=ois.readObject();
            if(null!=obj){
                return (T)obj;
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=ois){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
```

注：上面的对象直接输出到了txt文本中，因此可以直接打开查看。文件中显示的内容包含类信息，类中属性的类型，变量名，属性值等，不包含任何方法。而且如果在类中添加新的方法，再次序列化输出，序列化文件大小不变，从而也可以证明类方法不会被序列化到文件中。

### 2. 序列化与反序列化条件
虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 private static final long serialVersionUID = 1L）。

虽然两个类的功能代码完全一致，但是序列化ID不同，他们无法相互序列化和反序列化。

注：如果没有特殊需求，就是用默认的 1L 就可以，这样可以确保代码一致时反序列化成功。那么随机生成的序列化 ID 有什么作用呢，有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。

### 3. 序列化并不保存静态变量
序列化保存的是对象的状态，静态变量属于类的状态.

### 4. 父类的序列化
要想将父类对象也序列化，就需要让父类也实现Serializable 接口。如果父类不实现的话的，就需要有默认的无参的构造函数。 

在父类没有实现 Serializable 接口时，虚拟机是不会序列化父对象的，而一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。所以反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。

因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。如果你考虑到这种序列化的情况，在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如 int 型的默认是 0，string 型的默认是 null。

注：根据父类对象序列化的规则，我们可以将不需要被序列化的字段抽取出来放到父类中，子类实现 Serializable 接口，父类不实现，根据父类序列化规则，父类的字段数据将不被序列化。

### 5. transient关键字修饰的变量
Transient（瞬时的） 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

### 6. 序列化和反序列化将对象属性加解密
在序列化过程中，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化，如果没有这样的方法，则默认调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法。

用户自定义的 writeObject 和 readObject 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

实例代码：

```
public class PassSerialize implements Externalizable{
    private String pass;

    public String getPass() {
        return pass;
    }

    public void setPass(String pass) {
        this.pass = pass;
    }
    /**
     * 序列化输出时调用
     */
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(this.password(this.pass));//对象字段加密,将待序列化的内容输出
    }
    /**
     * 反序列化输入时调用
     */
    public void readExternal(ObjectInput in) throws IOException,
            ClassNotFoundException {
        String pass=(String) in.readObject();
        this.setPass(unpassword(pass));//this指反序列化后的对象
    }
    /**
     * 模仿加密操作
     * @return
     */
    private String password(String pass){
        String pa=pass+"ddd";
        return pa;
    }
    /**
     * 模仿解码
     * @param pass
     * @return
     */
    private String unpassword(String pass){
        String pa=pass.substring(0,pass.length()-3);
        return pa;
    }
}
```

测试代码：

```
public class SerializableMain {
    public static void main(String[] args) {
        test2();
    }
    
    public static void test2(){
        PassSerialize pass=new PassSerialize();
        pass.setPass("Helloworld");
        
        SerializableMain main=new SerializableMain();
        main.serialzeObject(pass);
        PassSerialize pass2=main.getObjectFromSerialize();
        System.out.println(pass2.getPass());
        
    }
    
    /**
     * 将对象序列化输出到文件中
     * @param obj
     */
    private void serialzeObject(Object obj){
        String filePath="E:/serialObj.txt";
        ObjectOutputStream oos=null;
        try{
            oos=new ObjectOutputStream(new FileOutputStream(filePath));
            oos.writeObject(obj);
        }catch(Exception e){
            e.printStackTrace();
        }
        finally{
            if(null!=oos){
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 获取序列化对象
     * @param clazz
     * @return
     */
    @SuppressWarnings("unchecked")
    private <T> T getObjectFromSerialize(){
        String filePath="E:/serialObj.txt";
        ObjectInputStream ois=null;
        try{
            ois=new ObjectInputStream(new FileInputStream(filePath));
            Object obj=ois.readObject();
            if(null!=obj){
                return (T)obj;
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=ois){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
```

注：out.writeObject时，都会调用被实例化对象的WriteExtenal()方法，而在WriteExtenal（）方法中我们需要将当前对象的值写入到流中；而每次ois.readObject()时，调用的是默认的构造函数，如果我们不在 readExternal（）方法中初始化对象成员变量，否则成员变量为null或初值

注2：write流中输出的对象顺序与reader流中获取对象的顺序要对应

在PassSerialize类中添加一个带参数的构造方法，而不提供默认构造函数,反序列化时抛出异常：no valid constructor.

```
public PassSerialize(String pass){
    this.pass=pass;
}
```

注：如果一个类要使用Externalizable实现序列化时，在此类中必须存在一个无参构造方法，因为在反序列化时会默认调用无参构造实例化对象，如果没有此无参构造，则运行时将会出现异常，这一点的实现机制与Serializable接口是不同的。

### 7. 问题：不能在writeExternal中将this对象写出
修改writeExternal方法为：

```
public void writeExternal(ObjectOutput out) throws IOException {
    this.setPass(this.password(this.pass));
    out.writeObject(this);//
}
```

修改readExternal方法为：

```
public void readExternal(ObjectInput in) throws IOException,
        ClassNotFoundException {
    PassSerialize ser=(PassSerialize) in.readObject();
    if(null!=ser){
        this.setPass(unpassword(ser.getPass()));
    }
}
```

注：序列化结果中没有pass字段的值，反序列化后自然无法获取pass字段值。out.writeObject(this)写入了5个字节的内容（对象存储空间的引用）。

注2：在Java语言中，当创建一个对象后，Java虚拟机就会为其分配一个指向对象本身的指针，这个指针就是"this"。这里可以理解this是对象的一个特殊的变量值。
![](/images/java_basic/object/this_super.png)

### 8. 序列化时写入同一个对象两次
代码：

```
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("result.obj"));
Test test = new Test();
//试图将对象两次写入文件
out.writeObject(test);
out.flush();
System.out.println(new File("result.obj").length());
out.writeObject(test);
out.close();
System.out.println(new File("result.obj").length());

ObjectInputStream oin = new ObjectInputStream(new FileInputStream("result.obj"));
//从文件依次读出两个文件
Test t1 = (Test) oin.readObject();
Test t2 = (Test) oin.readObject();
oin.close();
        
//判断两个引用是否指向同一个对象
System.out.println(t1 == t2);//输入true
```

注：Java 序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用，上面增加的 5 字节的存储空间就是新增引用和一些控制信息的空间。反序列化时，恢复引用关系，代码中的 t1 和 t2 指向唯一的对象，二者相等，输出true。该存储规则极大的节省了存储空间。

### 9. 父类实现了Serializable接口，子类序列化
父类实现了Serializable即可，相当于子类也实现了序列化接口。从类的继承体系上就能看到。


参考：http://www.cnblogs.com/wxgblogs/p/5849951.html




