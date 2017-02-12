---
title: 建造者模式
tags: [pattern]
---

一步一步创建一个复杂的对象（经典实现：MyBatis中的SQLSession就是结合了Configure，executor等对象，以此来实现SQLSession的复杂功能）

### 1. 什么是建造者模式
是将一个复杂的对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。
建造者模式的本质和建造楼房是一致的：即流程不变，但每个流程实现的具体细节则是经常变化的。建造者模式的好处就是保证了流程不会变化，流程即不会增加、也不会遗漏或者产生流程次序错误，这是非常重要的。

### 2. 建造者模式的构成
1）builder：给出一个抽象接口，以规范产品对象的各个组成成分的建造。这个接口规定要实现复杂对象的哪些部分的创建，并不涉及具体的对象部件的创建。--提供规范

2）ConcreteBuilder：实现Builder接口，针对不同的商业逻辑，具体化复杂对象的各部分的创建。 在建造过程完成后，提供产品的实例。--包含Product的实例

3）Director：调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建。--指定创建流程

4）Product：要创建的复杂对象。--最终产品对象

5）设计图
![](/images/java_pattern/builder/builder.jpg)

### 3. 实例
在游戏开发中建造小人是经常的事了，要求是：小人必须包括，头，身体，手和脚。现在系统要包括的分为胖人和瘦人。

1）Product类--最终产品人
```
/**
 * 具体待构建对象
 */
public class Product {
    private String partHead;
    private String partBody;
    private String partHand;
    private String partFeet;
    public String getPartHead() {
        return partHead;
    }
    public void setPartHead(String partHead) {
        this.partHead = partHead;
    }
    public String getPartBody() {
        return partBody;
    }
    public void setPartBody(String partBody) {
        this.partBody = partBody;
    }
    public String getPartHand() {
        return partHand;
    }
    public void setPartHand(String partHand) {
        this.partHand = partHand;
    }
    public String getPartFeet() {
        return partFeet;
    }
    public void setPartFeet(String partFeet) {
        this.partFeet = partFeet;
    }
    public void show(){
        System.out.println(partHead+":"+partBody+":"+partHand+":"+partFeet);
    }
}
```
2）builder接口
```
/**
 * 规定出待构建对象的模块--起到规范作用
 */
public interface Builder {
    void buildHead();
    void buildBody();
    void buildHand();
    void buildFeet();
    Product getProduct();
}
```
3）builder接口实现类
```
/**
 * 具体构建类，实现Builder接口
 * 胖人对象
 */
public class FatPersonBuilder implements Builder{
    
    private Product product=new Product();
    
    public void buildHead() {
        product.setPartHead("胖人头");
    }

    public void buildBody() {
        product.setPartBody("胖人身体");
    }

    public void buildHand() {
        product.setPartHand("胖人手");
    }

    public void buildFeet() {
        product.setPartFeet("胖人脚");
    }

    public Product getProduct() {
        return product;
    }
}
/**
 * 具体构建类，实现Builder接口
 * 瘦人对象
 */
public class ThinPersonBuilder implements Builder{
    
    private Product product=new Product();
    
    public void buildHead() {
        product.setPartHead("瘦人头");
    }

    public void buildBody() {
        product.setPartBody("瘦人身体");
    }

    public void buildHand() {
        product.setPartHand("瘦人手");
    }

    public void buildFeet() {
        product.setPartFeet("瘦人脚");
    }

    public Product getProduct() {
        return product;
    }

}
```
4）Director类
```
/**
 * 规定创建流程和顺序
 */
public class Director {
    public void constructProduct(Builder builder){
        builder.buildHead();
        builder.buildBody();
        builder.buildHand();
        builder.buildFeet();
    }
}
```

5）客户端调用
```
public class Client {
    public static void main(String[] args) {
        Director director=new Director();
        FatPersonBuilder fatPersonBuilder=new FatPersonBuilder();
        director.constructProduct(fatPersonBuilder);
        Product product=fatPersonBuilder.getProduct();
        product.show();
    }
}
```

注：通过建造者模式，使得建造过程通过Director类的Construct函数固定了，即建造过程不会变。但是具体的头，身体，手脚这些身体的各个部分会变化，基类Builder中将各种Build函数定义为抽象方法，必须在子类中实现。这样不仅仅使得建造小人的过程不变，而且很利于系统的扩展，一旦出现其他种类的人根本不需要改动之前的FatPersonBuider，ThinPersonBuilder，Director，Product等类，只需要新添加新的类。

### 4. 使用场景
1）当创建复杂对象的算法独立于该对象的组成部分以及它们的装配方式时。

2）当构造过程必须允许被构造的对象有不同表示时（ 相同的方法，不同的执行顺序，产生不同的结果时--新建新的Director类）

参考：http://www.cnblogs.com/BeyondAnyTime/archive/2012/07/19/2599980.html

参考2：http://blog.csdn.net/hello_haozi/article/details/38819935
