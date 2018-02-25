---
title: java注解常用方法
tags: [annotation]
---

参考：http://blog.csdn.net/bao19901210/article/details/17201173

1）isAnnotationPresent方法

```
boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
```

如果指定类型的注解存在于此元素上，则返回true，否则返回false。此方法主要是为了便于访问标记注解而设计的。

2）getAnnotation方法

```
<T extends Annotation> T getAnnotation(Class<T> annotationClass)
```

如果存在该元素的指定类型的注解，则返回这些注解，否则返回 null。

3）getAnnotations方法

```
Annotation[] getAnnotations()
```

返回此元素上存在的所有注解。如果此元素没有注解，则返回长度为零的数组。该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

4）getDeclaredAnnotations方法

```
Annotation[] getDeclaredAnnotations()
```

返回直接存在于此元素上的所有注解。与此接口中的其他方法不同，该方法将忽略继承的注解。如果没有注解直接存在于此元素上，则返回长度为零的一个数组。该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

5）getAnnotationsByType方法

```
public <A extends Annotation> A[] getAnnotationsByType(Class<A> annotationClass)
```

该方法用于获取指定类型的注解数组。
