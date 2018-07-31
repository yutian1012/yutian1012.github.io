---
title: java finally的深度理解
tags: [interview]
---

参考：https://www.cnblogs.com/lanxuezaipiao/p/3440471.html

注：在java-demo项目中运行测试

1）finally块的语句在try或catch中的return语句执行之后返回之前执行

即使try和catch中存在return语句，并且return语句已经执行了，finally语句也会执行

```
/**
 * try中return后，finally依然执行
 */
public void finalyAndReturn() {
    try {
        System.out.println("return语句执行后，fianlly依然执行");
        
        return;
        
    }catch(Exception e) {
        e.printStackTrace();
    }finally {
        System.out.println("fianlly语句执行");
    }
}
```

2）在finally语句中修改return对象的值

finally里的修改语句可能影响也可能不影响try或catch中return已经确定的返回值。

```
/**
 * 在finally中修改return的数值
 * @return
 */
public Integer finallyChangeData() {
    
    Integer num=new Integer(1);
    try {
        System.out.println("return语句执行后，fianlly依然执行");
        
        return num;
        
    }catch(Exception e) {
        e.printStackTrace();
    }finally {
        System.out.println("fianlly语句执行");
        num+=1;
    }
    
    return num;
}
```

注：针对原生类型及其包装类型的数据，finally中修改该值也不会影响return的数值，而针对对象类型数据，是会影响其值的（参考值传递和引用传递）。

3）finally中覆盖return语句

若finally里也有return语句则覆盖try或catch中的return语句直接返回。

```
/**
 * finally中执行return语句，会覆盖掉之前的return
 * @return
 */
@SuppressWarnings("finally")
public Integer finallyCoveryReturnData() {
    Integer num=new Integer(1);
    try {
        System.out.println("return语句执行后，fianlly依然执行");
        
        return num;
        
    }catch(Exception e) {
        e.printStackTrace();
    }finally {
        System.out.println("fianlly语句执行");
        num+=1;
        return num;
    }
}
```