---
title: java序列化操作
tags: [basic]
---

序列化的实现，将需要被序列化的类实现Serializable接口，该接口没有需要实现的方法。Serializable只是为了标注该对象是可被序列化的，然后使用一个输出流，如FileOutputStream，来构造一个ObjectOutputStream对象，接着，使用ObjectOutputStream对象的writeObject方法就可以将参数为obj的对象写出(即保存其状态)，要恢复的话则用输入流。

### 三种方式实现序列化

假定一个Student类，它的对象需要序列化。

1）方法一（最简单的实现方式）

若Student类仅仅实现了Serializable接口，则可以按照以下方式进行序列化和反序列化.

ObjectOutputStream采用默认的序列化方式，对Student对象的非transient的实例变量进行序列化。

ObjcetInputStream采用默认的反序列化方式，对Student对象的非transient的实例变量进行反序列化。


2）方法二（能够灵活的指定序列化和反序列化的内容）

若Student类仅仅实现了Serializable接口，并且还定义了readObject和writeObject方法，则采用以下方式进行序列化与反序列化。

```
# 反序列化对象
readObject(ObjectInputStream in)
# 序列化对象
writeObject(ObjectOutputSteam out)，
```

ObjectOutputStream调用Student对象的writeObject的方法进行序列化。

ObjectInputStream会调用Student对象的readObject的方法进行反序列化。

3）方法三

若Student类实现了Externalnalizable接口，且Student类必须实现readExternal和writeExternal方法，则按照以下方式进行序列化与反序列化。

```
# 反序列化对象
readExternal(ObjectInput in)
# 序列化对象
writeExternal(ObjectOutput out)
```

ObjectOutputStream调用Student对象的writeExternal的方法进行序列化。

ObjectInputStream会调用Student对象的readExternal的方法进行反序列化。