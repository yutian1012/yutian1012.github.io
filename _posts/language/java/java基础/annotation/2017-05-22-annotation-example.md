---
title: spring中Component注解实例分析
tags: [annotation]
---

### 注解实例

1）查看spring的Component注解定义

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {
    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     * @return the suggested component name, if any
     */
    String value() default "";
}
```

2）Retention注解

该注解接收一个RetentionPolicy的枚举值，表示注解的保留位置。通俗点说，就是不同RetentionPolicy类型的Annotation的作用域不同。

例如：@Override标志就是一个Annotation。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查！编译器处理完后，@Override就没有任何作用了。查看@Override的注解定义，可以发现@Retention(RetentionPolicy.SOURCE)。

```
# 注解仅存在于源码中，在class字节码文件中不包含
@Retention(RetentionPolicy.SOURCE)
# 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得，
@Retention(RetentionPolicy.CLASS)
# 注解会在class字节码文件中存在，在运行时可以通过反射获取到
@Retention(RetentionPolicy.RUNTIME) 
```

RetentionPolicy表明注解的生命周期，包括三个值：

```
package java.lang.annotation;

public enum RetentionPolicy {
    SOURCE, /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了*/

    CLASS,  /* 编译器将Annotation存储于类对应的.class文件中。默认行为 */

    RUNTIME /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
}
```

注：一个Annotation对应一个RetentionPolicy（属于一对一的关系）

3）Target注解

该注解指定定义注解的作用目标，接收ElementType类型的枚举数组。若一个Annotation对象是METHOD类型，则该Annotation只能用来修饰方法。

```
@Target(ElementType.TYPE)   //接口、类、枚举、注解
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包

# 对应多个ElementType，需要用大括号将多个值括起来
@Target({ElementType.FIELD,ElementType.METHOD})
```

ElementType表明注解的目标元素类型，是一个枚举对象

```
package java.lang.annotation;

public enum ElementType {
    TYPE,               /* 类、接口（包括注释类型）或枚举声明  */
    FIELD,              /* 字段声明（包括枚举常量）  */
    METHOD,             /* 方法声明  */
    PARAMETER,          /* 参数声明  */
    CONSTRUCTOR,        /* 构造方法声明  */
    LOCAL_VARIABLE,     /* 局部变量声明  */
    ANNOTATION_TYPE,    /* 注释类型声明  */
    PACKAGE             /* 包声明  */
}
```

注：一个Annotation对应多个ElementType（属于一对多的关系）

4）Document

类和方法的Annotation在缺省情况下是不出现在javadoc中的。如果使用@Documented修饰该Annotation，则表示它可以出现在javadoc中。定义Annotation时，@Documented可有可无；若没有定义，则Annotation不会出现在javadoc中。

5）Inherited

说明子类可以继承父类中的该注解

6）注解体

```
# 意味着，Component使用时能指定参数
String value() default "";
```