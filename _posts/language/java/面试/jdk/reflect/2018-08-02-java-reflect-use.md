---
title: 反射的使用
tags: [interview]
---

1）获得 Class 对象

使用 Class类的 forName() 静态方法：

```
public static Class<?> forName(String className)
```

注：在JDBC开发中常用此方法加载数据库驱动:

直接获取某一个对象的class

```
Class<?> klass = int.class;
Class<?> classInt = Integer.TYPE;
```

调用某个对象的getClass()方法

```
StringBuilder str = new StringBuilder("123");
Class<?> klass = str.getClass();
```

2）判断是否为某个类的实例

用instanceof关键字来判断是否为某个类的实例

```
String hello="sss";
if(hello instanceof String){
}
```

借助反射中Class对象的isInstance方法来判断是否为某个类的实例（它是一个Native方法）

```
String hello="sss";
if(String.class.isInstance(hello)){
    System.out.println(hello);
}
```

3）创建实例

使用Class对象的 newInstance方法来创建对象对应类的实例

```
String hello=String.class.newInstance();
```

注：会抛出InstantiationException, IllegalAccessException异常信息

调用Constructor对象的newInstance方法来创建实例

```
Constructor<String> constructor = String.class.getConstructor(String.class);
String obj = constructor.newInstance("helloworld"); 
System.out.println(obj);
```

4）或者字段信息

```
getFiled: 访问公有的成员变量
getDeclaredField：所有已声明的成员变量。但不能得到其父类的成员变量
getFileds和getDeclaredFields
```