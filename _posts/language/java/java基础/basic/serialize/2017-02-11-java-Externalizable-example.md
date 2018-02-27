---
title: java序列化与反序列化实例（Externalizable）
tags: [basic]
---

java.io.Externalizable的所有实现者必须提供读取和写出的实现。并且要有无参构造函数。

1）实例

序列化和反序列化将对象属性加解密。用户自定义的writeExternal和readExternal方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

2）实体类实现Externalizable接口

使用Externalizable接口需要实现的writeExternal和readExternal方法进行序列化

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

3）测试序列化

```
public class SerializableMain {
    public static void main(String[] args) {
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