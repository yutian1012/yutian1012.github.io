---
title: 模板方法模式
tags: [pattern]
---

父类定义公共方法，不同子类重写父类抽象方法，得到不同结果（经典实现：Tomcat生命周期中的init，SpringIOC上层类加载具体子类指定的配置文件）

### 1. 什么是模板方法模式
一个抽象类中，有一个主方法，再定义1...n个方法，可以是抽象的，也可以是实际的方法，定义一个类，继承该抽象类，重写抽象方法，通过调用抽象类，实现对子类的调用。

### 2. 类设计图

![](/images/java_pattern/template/template.jpg)

### 3. 实现
1）抽象模板类

```
/**
 * 抽象模板类
 * 定义了计算过程，先准备待计算的数据，然后将计算值交由具体的实现类计算出结果
 * 模板类定义了完成工作的流程，只把最核心的实现交由子类来处理，准备工作和后续工作都可以自己实现，特殊的方法也可以由子类来实现
 * 类似于写信：定义了收件人位置，主题位置，寄件人位置等，子类只需要在相应的位置上填写自己的实现即可。
 */
public abstract class AbstractCalculator {
    /*主方法，实现对本类其它方法的调用*/  
    public final int calculate(String exp,String opt){  
        int array[] = split(exp,opt);  
        return calculate(array[0],array[1]);  
    }  
      
    /*被子类重写的方法*/  
    abstract public int calculate(int num1,int num2);  
    
    public int[] split(String exp,String opt){  
        String array[] = exp.split(opt);  
        int arrayInt[] = new int[2];  
        arrayInt[0] = Integer.parseInt(array[0]);  
        arrayInt[1] = Integer.parseInt(array[1]);  
        return arrayInt;  
    } 
}
```

2）子类实现抽象模板

```
/**
 * 具体实现类，实现了具体的加法操作
 */
public class Plus extends AbstractCalculator{
    @Override
    public int calculate(int num1, int num2) {
        return num1+num2;
    }
}
```

3）客户端调用

```
public class Client {
    public static void main(String[] args) {  
        String exp = "8+8";  
        AbstractCalculator cal = new Plus();  
        int result = cal.calculate(exp, "\\+");  
        System.out.println(result);  
    }
}
```

### 4. 适用场景
1）对一些复杂的算法进行分割，将其算法中固定不变的部分设计为模板方法和父类具体方法，而一些可以改变的细节由其子类来实现。即：一次性实现一个算法的不变部分，并将可变的行为留给子类来实现。

2）各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。

3）需要通过子类来决定父类算法中某个步骤是否执行，实现子类对父类的反向控制。
