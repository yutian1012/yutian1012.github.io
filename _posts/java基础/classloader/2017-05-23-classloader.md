---
title: classloader类
tags: [basic]
---

### 查找某个类是否被虚拟机加载

这里以oracle驱动的加载为例来进行测试。

```
public class SPITest {
    public static void main(String[] args) throws Exception {
        ServiceLoader<Driver> serviceLoader=ServiceLoader.load(Driver.class);
        Iterator<Driver> drivers=serviceLoader.iterator();
        while(drivers.hasNext()){
            System.out.println(drivers.next());
        }
        
        isContainClass("java.lang.System");
        isContainClass("oracle.jdbc.OracleDriver");
    }
    /**
     * findLoadedClass位于ClassLoader类中，不过是protected方法，因此，只能通过反射来查看
     * @param className
     * @return
     */
    public static boolean isContainClass(String className)throws Exception{
        Class<?> cl = Class.forName("java.lang.ClassLoader", false, Thread.currentThread().getContextClassLoader());
        Method method = cl.getDeclaredMethod("findLoadedClass", new Class[] { String.class });
        method.setAccessible(true);
 
        if (method.invoke(Thread.currentThread().getContextClassLoader(),className) != null) {
            System.out.println(className+"已经加载!");
            return true;
        } else {
            System.out.println(className+"尚未加载!");
            return false;
        }
    }
}
```

注：一般我们使用Class.forName方法加载类，即要求JVM查找并加载指定的类，也就是说JVM会执行该类的静态代码段。因此Class.forName方法一般写在静态代码块中。