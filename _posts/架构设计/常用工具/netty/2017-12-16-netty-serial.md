---
title: Netty serial 序列化知识
tags: [architecture]
---

### 序列化知识

1）将int转换成字节数组的方法

```
//大端字节序列（先写高位，再写低位）
//小端字节序列（先写低位，再写高位）
//这里采用大端序列--符合我们常用的思维
public static byte[] int2bytes(int i){
    byte[] bytes=new byte[4];
    bytes[0]=（byte）(i>> 3*8);//获取int的最高8位字节
    bytes[1]=（byte）(i>> 2*8);
    bytes[2]=（byte）(i>> 1*8);
    bytes[3]=（byte）(i>> 0*8);
    return bytes;
}

//反序列化字节数组成int
public static int bytes2int(byte[] bytes){
    return (bytes[0]<<3*8)| //xxxxxxxx 00000000 00000000 00000000
        (bytes[1]<<2*8)|    //00000000 xxxxxxxx 00000000 00000000
        (bytes[2]<<1*8)|    //00000000 00000000 xxxxxxxx 00000000
        (bytes[3]<<0*8);    //00000000 00000000 00000000 xxxxxxxx
}

//测试
public static void main(String[] args) throws IOException {
    int id = 101;
    int age = 21;
    
    //ByteArrayOutputStream方法的write(int)不能实现自动扩容，
    //101只会输出1个字节而不是4个字节
    ByteArrayOutputStream arrayOutputStream = new ByteArrayOutputStream();
    arrayOutputStream.write(int2bytes(id));
    arrayOutputStream.write(int2bytes(age));
    
    byte[] byteArray = arrayOutputStream.toByteArray();
    
    System.out.println(Arrays.toString(byteArray));
    
    //==============================================================
    ByteArrayInputStream arrayInputStream = new ByteArrayInputStream(byteArray);
    byte[] idBytes = new byte[4];
    arrayInputStream.read(idBytes);
    System.out.println("id:" + bytes2int(idBytes));
    
    byte[] ageBytes = new byte[4];
    arrayInputStream.read(ageBytes);
    System.out.println("age:" + bytes2int(ageBytes));   
}
```

注：ByteArrayOutputStream方法的write(int)不能实现自动扩容

2）将int转换成字节数组的简单方法

ByteBuffer（nio包中）类使用putInt会将int转换成对应的字节数组。

```
public static void main(String[] args) {
    int id = 101;
    int age = 21;
    
    ByteBuffer buffer = ByteBuffer.allocate(8);
    buffer.putInt(id);
    buffer.putInt(age);
    byte[] array = buffer.array();
    System.out.println(Arrays.toString(buffer.array()));
    
    //====================================================
    
    ByteBuffer buffer2 = ByteBuffer.wrap(array);
    System.out.println("id:"+buffer2.getInt());
    System.out.println("age:"+buffer2.getInt());
}
```

3）使用netty中的类实现字节自动扩容

ChannelBuffer类实现序列化和反序列化操作。

```
public static void main(String[] args) {

    ChannelBuffer buffer = ChannelBuffers.dynamicBuffer();
    buffer.writeInt(101);
    buffer.writeDouble(80.1);

    //序列化操作，将数据写入到byte[]数组中，根据写指针的大小构建byte数组大小
    byte[] bytes = new byte[buffer.writerIndex()];
    buffer.readBytes(bytes);
    
    System.out.println(Arrays.toString(bytes));
    
    //================================================
    ChannelBuffer wrappedBuffer = ChannelBuffers.wrappedBuffer(bytes);
    System.out.println(wrappedBuffer.readInt());
    System.out.println(wrappedBuffer.readDouble());
    
}
```

4）字符串类型的数据反序列化问题

```
public static void main(String[] args) {
    //================================================
    "abc".getBytes();//字符串的序列化
    String s=new String("abc".getBytes());//反序列化成字符串
}
```

如果给出了一个字节数组，数组中有各种类型的数据，那么如何确定反序列化的字符串长度呢？

在序列化字符串时，会在字符串前添加一个字符串的长度，那么在序列化时就能够知道获取的字符串对应的字节数组长度。

```
//定义字符编码
public static final Charset CHARSET = Charset.forName("UTF-8");
protected ChannelBuffer writeBuffer;

protected ChannelBuffer readBuffer;

//序列化字符串
public Serializer writeString(String value) {
    //针对空字符串，写长度为0
    if (value == null || value.isEmpty()) {
        writeShort((short) 0);
        return this;
    }

    //获取字符编码的字符串对应的字节数组
    byte data[] = value.getBytes(CHARSET);
    //输出字符串的长度
    short len = (short) data.length;
    writeBuffer.writeShort(len);
    writeBuffer.writeBytes(data);
    return this;
}

//反序列化字符串
public String readString() {
    //先获取字符串对应的字节数组的长度
    int size = readBuffer.readShort();
    if (size <= 0) {
        return "";
    }

    byte[] bytes = new byte[size];
    readBuffer.readBytes(bytes);

    return new String(bytes, CHARSET);
}
```

注：这里使用了netty的ChannelBuffer工具进行数据的读写

5）针对list集合的序列化和反序列化

```
//序列化集合数据
public <T> Serializer writeList(List<T> list) {
    //针对空集合的处理
    if (isEmpty(list)) {
        writeBuffer.writeShort((short) 0);
        return this;
    }

    //输出集合的长度，并遍历集合中的每一个对象调用相应的序列化方法
    writeBuffer.writeShort((short) list.size());
    for (T item : list) {
        writeObject(item);
    }
    return this;
}

//序列化Object对象
public Serializer writeObject(Object object) {
    
    //空对象输出0
    if(object == null){
        writeByte((byte)0);
    }else{//对Object使用instanceof进行类型的判断，并调用相应的方法
        if (object instanceof Integer) {
            writeInt((int) object);
            return this;
        }

        if (object instanceof Long) {
            writeLong((long) object);
            return this;
        }

        if (object instanceof Short) {
            writeShort((short) object);
            return this;
        }

        if (object instanceof Byte) {
            writeByte((byte) object);
            return this;
        }

        if (object instanceof String) {
            String value = (String) object;
            writeString(value);
            return this;
        }
        if (object instanceof Serializer) {
            writeByte((byte)1);
            Serializer value = (Serializer) object;
            value.writeToTargetBuff(writeBuffer);
            return this;
        }
        
        throw new RuntimeException("不可序列化的类型:" + object.getClass());
    }
    
    return this;
}

//反序列化集合
public <T> List<T> readList(Class<T> clz) {
    List<T> list = new ArrayList<>();
    //获取集合的长度
    int size = readBuffer.readShort();
    for (int i = 0; i < size; i++) {
        list.add(read(clz));
    }
    return list;
}
//根据class调用相应的方法读取数据
public <I> I read(Class<I> clz) {
    Object t = null;
    if ( clz == int.class || clz == Integer.class) {
        t = this.readInt();
    } else if (clz == byte.class || clz == Byte.class){
        t = this.readByte();
    } else if (clz == short.class || clz == Short.class){
        t = this.readShort();
    } else if (clz == long.class || clz == Long.class){
        t = this.readLong();
    } else if (clz == float.class || clz == Float.class){
        t = readFloat();
    } else if (clz == double.class || clz == Double.class){
        t = readDouble();
    } else if (clz == String.class ){
        t = readString();
    } else {
        throw new RuntimeException(String.format("不支持类型:[%s]", clz));
    }
    return (I) t;
}
```