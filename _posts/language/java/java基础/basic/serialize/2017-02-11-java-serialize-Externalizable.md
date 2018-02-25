---
title: Serializable和Externalizable接口
tags: [basic]
---

1）两者的关系

```
public interface Externalizable extends Serializable {
    void readExternal(ObjectInput in);
    void writeExternal(ObjectOutput out);
}
```

2）区别

通过Serializable接口实现对象序列化的支持是内建于核心API的，但是java.io.Externalizable的所有实现者必须提供读取和写出的实现。Java已经具有了对序列化的内建支持，也就是说只要实现java.io.Serializable接口，Java就会试图存储和重组你的对象。如果使用外部化，你就可以完全由自己完成读取和写出的工作。

注：Externalizable需要自己实现序列化的读写操作。

3）优缺点

a.Serializable接口

优点：内建支持，易于实现。

缺点：占用空间过大，由于额外的开销导致速度变比较慢。

b.Externalizable接口

优点：开销较少（程序员决定存储什么），可观的速度提升。

缺点：虚拟机不提供任何帮助，也就是说所有的工作都落到了开发人员的肩上。