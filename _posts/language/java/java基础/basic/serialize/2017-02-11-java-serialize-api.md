---
title: java序列化API
tags: [basic]
---

### JDK类库中序列化API--操作类

1）ObjectOutputStream

ObjectOutputStream表示对象输出流，writeObject方法进行对象序列化，把得到的字节序列写到一个目标输出流中。

```
# 类
java.io.ObjectOutputStream
# 将对象序列化到输出流中
writeObject(Object obj)
```

2）ObjectInputStream

ObjectInputStream表示对象输入流，readObject方法从输入流中读取字节序列，再把它们反序列化成为一个对象，并将其返回。

```
# 类
java.io.ObjectInputStream
# 将序列化的对象读入到内存
readObject()
```

3）Serializable接口或Externalizable接口

只有实现了Serializable或Externalizable接口的类的对象才能被序列化，否则抛出异常。