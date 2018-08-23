---
title: 接口的默认方法
tags: [jdk8]
---

Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用default关键字即可，这个特征又叫做扩展方法。

注：使用default方法允许给已存在的类型扩展方法。

当有和多类实现了同一个接口，如果想对该接口扩展方法，使不修改所有子类，而能够让子类自动获得到这个扩展方法。就可以使用jdk8提供的默认方法来实现。

1）实现代码

定义接口，即默认方法

```
package org.sjq.test.demo;

public interface IDefautlMethodDemo {
    double calculate(int a);

    //default方法可以在所有实现了该接口的方法中自动继承
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```

子类实现接口，并自定继承默认方法

```
package org.sjq.test.demo;

public class DefaultMethodDemo implements IDefautlMethodDemo{

    @Override
    public double calculate(int a) {
        return a*a;
    }
    public static void main(String[] args) {
        DefaultMethodDemo demo=new DefaultMethodDemo();
        System.out.println(demo.calculate(100));
        System.out.println(demo.sqrt(100));
    }
}
```