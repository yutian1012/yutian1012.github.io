---
title: java序列化与反序列化实例（Serializable）
tags: [basic]
---

### 对象序列化

1）实体类代码：

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

2）将对象序列化到文件中并从文件中反序列化获取对象

```
public class SerializableMain {
    public static void main(String[] args) {
        StudentSerialize student=new StudentSerialize();
        student.setName("张三");
        student.setAge(13);
        student.setBirth(new Date());
        
        SerializableMain main=new SerializableMain();
        //序列化对象到文件中
        main.serialzeObject(student);
        
        //从文件中反序列化到对象中
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

3）总结：序列化不会处理类中的方法

将对象直接序列化到了txt文本中，可以直接打开查看。

文件中显示的内容包含类信息，类中属性的类型，变量名，属性值等，不包含任何方法。而且如果在类中添加新的方法，再次序列化输出，序列化文件大小不变，从而也可以证明类方法不会被序列化到文件中。