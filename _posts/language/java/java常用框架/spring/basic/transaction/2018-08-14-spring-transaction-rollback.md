---
title: spring事务回滚
tags: [spring]
---

1）事务类型

required：如果当前存在事务，则使用该事务，否则将新建一个事务

requires_new：如果当前存在事务，则挂起当前事务并且开启一个新事务继续执行。新事物执行完毕之后，然后再唤醒之前挂起的事务；如果当前不存在事务的话，开启一个新的事物。

1）父事务与子事务

```
//子事务
@Transactional(propagation=Propagation.REQUIRES_NEW)
public void child(){
    ...
}

@Transactional
public void parent(){
    ...
    child();//调用子方法
}
```

注：如果child抛出异常回滚，那么parent也会回滚。

2）异常抛出类型

抛出RuntimeException类型的异常，才会回滚。

```
int 1/0;//抛出异常实现回滚
```

3）parent中捕获child异常

```
//子事务
@Transactional(propagation=Propagation.REQUIRES_NEW)
public void child(){
    ...
    //插入数据
    ...
    //抛出异常
}

@Transactional
public void parent(){
    //插入数据
    ...
    try{
        child();//调用子方法
    }catch(Exception e){
        e.printStackTrace();
    }
    ...
}
```

期望的结果是：child正常回滚，而父类正常插入数据

实际结果是：child和parent方法都没有回滚，都将数据插入到了数据库中。

原因是jdk动态代理机制，@Transactional(propagation=Propagation.REQUIRES_NEW)不会起作用，而事务又有传播机制，因此，如果子方法中抛出了异常，而父方法不对其进行捕获，则父子方法都会回滚。同时，如果对其进行捕获，则造成子方法虽然抛出了异常，但并不能正常回滚。

4）spring 事务的底层实现

spring事务的实现是jdk的动态代理（或使用cglib实现）。在使用spring事务时一般通过aop来配置，底层则是jdk的动态代理机制。

为什么动态代理会导致spring回滚失效？？

```
//jdk动态代理是基于接口实现的，这里模拟一个接口类
public interface UserService{
    public void test();
    public void test1();
}
//接口实现类
public class UserServiceImpl implements UserService{
    public void test(){
        System.out.println("============test===========");
    }
    public void test1(){
        System.out.println("============test1===========");
    }
}
//创建代理类
public class MyInvocationHandler implements InvocationHandler{

    public Object target;//目标对象

    public MyInvocationHandler(Object target){
        this.target=target;
    }

    public Object invoke(Object proxy,Method method,Object[] args)throws Throwable{
        if(method.getName().startsWith("test")){
            System.out.println("代理所有方法名以test开头的方法，输出相关日志");
        }
        return method.invoke(target,args);//调用目标对象的方法
    }
}
```

调用代理实现

```
public static void main(String[] args){
    MyInvocationHandler handler=new MyInvocationHandler(new UserServiceImpl());

    UserService proxy=Proxy.newProxyInstance(this.getClassLoader(),new Class[]{UserService.class},handler);

    proxy.test();
    proxy.test1();
}
```

注：通过查看输出信息，可以发现test和est1方法调用时都会输出代理方法的相关日志。

改造方法，即在test方法中调用test1方法

```
public class UserServiceImpl implements UserService{
    public void test(){
        System.out.println("============test===========");
        this.test1();
    }
    public void test1(){
        System.out.println("============test1===========");
    }
}

//代理测试类调用test方法，
public static void main(String[] args){
    MyInvocationHandler handler=new MyInvocationHandler(new UserServiceImpl());

    UserService proxy=Proxy.newProxyInstance(this.getClassLoader(),new Class[]{UserService.class},handler);

    proxy.test();
}
```

注：从输出结果可以看出，只会打印出一行相关日志，即test1方法虽然被调用了，但不是由代理类来执行的，而是target类来执行的。

6）如何实现子类单独回滚呢？？

必须要获取AopProxy代理对象，通过代理调用子方法

```
<aop:aspectj-autoproxy expose-proxy="true"/>
```

要获取aop代理对象，必须先暴露aop到ThreadLocal中（有多种方式获取AOP代理对象？？）

```
//子事务
@Transactional(propagation=Propagation.REQUIRES_NEW)
public void child(){
    ...
    //插入数据
    ...
    //抛出异常
}

@Transactional
public void parent(){
    //插入数据
    ...
    try{
        //child();//调用子方法
        //通过AOP代理对象调用子方法
        ((UserService)AopContext.currentProxy()).child();
    }catch(Exception e){
        e.printStackTrace();
    }
    ...
}
```

注：只有使用Aop代理对象调用child方法，@Transactional(propagation=Propagation.REQUIRES_NEW)才会生效，创建一个新的事务，在新事务中执行child方法。如果child方法中抛出了异常，child方法就能正常回滚。

注2：通过ApplicationContext对象也能获取Aop代理对象。

分布式事务（可靠消息机制实现分布式事务，消息幂等策略，两阶段提交）