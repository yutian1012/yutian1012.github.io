---
title: 自定义注解
tags: [annotation]
---

使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。

### 最简单的注解定义

1）自定义最简单的注解：

```
public @interface MyAnnotation {

}
```

注：保留策略默认为@Retention(RetentionPolicy.CLASS)，作用目标默认为所有元素。

如果注释类型声明中不存在 Target 元注释，则声明的类型可以用在任一程序元素上。如果存在这样的元注释，则编译器强制实施指定的使用限制。

2）使用自定义注解：

```
public class AnnotationTest2 {

    @MyAnnotation
    public void execute(){
        System.out.println("method");
    }
}
```

### 注解中添加变量

1）定义注解修改

```
public @interface MyAnnotation {

    String value1();
}
```

2）使用自定义注解：

```
public class AnnotationTest2 {

    @MyAnnotation(value1="abc")
    public void execute(){
        System.out.println("method");
    }
}
```

注：当注解中使用的属性名为value时，对其赋值时可以不指定属性的名称而直接写上属性值，除了value意外的变量名都需要使用name=value的方式赋值。

3）添加默认值：

```
public @interface MyAnnotation {

    String value1() default "abc";
}
```

### 注解中设置多变量使并用枚举：

1）修改注解定义

```
public @interface MyAnnotation {

    String value1() default "abc";
    MyEnum value2() default MyEnum.Sunny;
}
enum MyEnum{
    Sunny,Rainy
}
```

2）使用自定义注解：

```
public class AnnotationTest2 {

    @MyAnnotation(value1="a", value2=MyEnum.Sunny)
    public void execute(){
        System.out.println("method");
    }
}
```

### 注解中设置数组变量：

1）修改注解定义

```
public @interface MyAnnotation {

    String[] value1() default "abc";
}
```

2）使用自定义注解：

```
public class AnnotationTest2 {

    @MyAnnotation(value1={"a","b"})
    public void execute(){
        System.out.println("method");
    }
}
```