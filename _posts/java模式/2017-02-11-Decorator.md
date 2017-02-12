---
title: 装饰模式
tags: [pattern]
---

创建包装对象修饰扩展被包装对象的功能（经典实现：IO家族中BufferedXxx）

### 1. 什么是装饰模式
装饰模式又名包装（Wrapper）模式，装饰模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。

装饰模式通过创建一个包装对象，也就是装饰，来包裹真实的对象。装饰模式以对客户端透明的方式动态地给一个对象附加上更多的责任。换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不创造更多子类的情况下，将对象的功能加以扩展。

### 2. 装饰角色
1）抽象构件角色（Component）：给出一个抽象接口，以规范准备接收附加责任的对象。

2）具体构件角色（Concrete Component）：定义将要接收附加责任的类。

3）装饰角色（Decorator）：持有一个构件（Component）对象的引用，并定义一个与抽象构件接口一致的接口。

4）具体装饰角色（Concrete Decorator）：负责给构件对象“贴上”附加的责任。

### 3. Java IO中的装饰模式
1）抽象构件角色：InputStream,OutputStream类

2）具体构件角色：如FileInputStream，StringBufferInputStream等

3）装饰角色：是过滤流，如FilterInputStream和FilterOutputStream

4）具体装饰角色：FilterInputStream的派生类，如BufferedInputStream等
![](/images/java_pattern/decorator/javaio_decorator.png)

### 4. 装饰模式的特点
1）装饰对象和真实对象有相同的接口。这样客户端对象就可以以和真实对象相同的方式和装饰对象交互。

2）装饰对象包含一个真实对象的引用（reference）。

3）装饰对象接收所有来自客户端的请求，它把这些请求转发给真实的对象。

4）装饰对象可以在转发这些请求之前或之后附加一些功能。

5）这样就确保了在运行时，不用修改给定对象的结构就可以在外部增加附加的功能。

### 5. 实例
![](/images/java_pattern/decorator/decorator_demo.png)
1）抽象构件角色代码：
```
/**
 * 抽象构件角色，是一个接口
 */
public interface Component {
    public void doSomething();
}
```
2）具体构件角色代码：
```
/**
 * 具体构件角色实现了Component接口
 */
public class ConcreteComponent implements Component{
    public void doSomething() {
        System.out.println("功能A");
    }
}
```
3）装饰角色代码：
```
/**
 * 装饰角色，
 * 持有一个构件（Component）对象的引用
 * 并与抽象构件接口有一致的接口
 * 把请求转发给真实的对象执行
 */
public class Decorator implements Component {
    private Component component;
    public Decorator(Component component){
        this.component=component;
    }
    public void doSomething() {
        component.doSomething();
    }
}
```
4）具体装饰角色代码（定义2个实现类）：
```
/**
 * 具体装饰角色
 * 添加附加功能，并重写相应的component定义的接口方法
 */
public class ConcreteDecorator1 extends Decorator{
    public ConcreteDecorator1(Component component) {
        super(component);
    }
    @Override
    public void doSomething()
    {
        super.doSomething();
        this.doAnotherThing();
    }
    //辅助方法
    private void doAnotherThing()
    {
        System.out.println("功能B");
    }
}
/**
 * 具体装饰角色
 * 添加附加功能，并重写相应的component定义的接口方法
 */
public class ConcreteDecorator2 extends Decorator{
    public ConcreteDecorator2(Component component) {
        super(component);
    }
    @Override
    public void doSomething()
    {
        super.doSomething();
        this.doAnotherThing();
    }
    //辅助方法
    private void doAnotherThing()
    {
        System.out.println("功能C");
    }
}
```
5）客户端测试代码：
```
public class Client {
    public static void main(String[] args) {
        Component component = new ConcreteComponent();
        Component component1 = new ConcreteDecorator1(component);
        component1.doSomething();
        System.out.println("-----------");
        //可不断的扩充功能
        Component component2 = new ConcreteDecorator2(component1);
        component2.doSomething();
    }
}
```

参考：http://www.cnblogs.com/mengdd/archive/2013/02/12/2910302.html

参考：http://www.cnblogs.com/mengdd/category/426388.html