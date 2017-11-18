---
title: 反射中专利类型的判断
tags: [basic]
---

参考：https://www.cnblogs.com/toSeeMyDream/p/5680007.html

反射获取父类中的属性和方法

```
/**
     * 不断遍历父类，直到找到属性
     * @param object
     * @param fieldName
     * @return
     */
    private static Field getDeclaredField(Class<?> clazz, String fieldName){  
        Field field = null ;  
        for(; clazz != Object.class ; clazz = clazz.getSuperclass()) {  
            try {  
                field = clazz.getDeclaredField(fieldName) ;  
                return field ;  
            } catch (Exception e) {  
                //这里甚么都不能抛出去。  
                //如果这里的异常打印或者往外抛，则就不会进入                  
            }   
        }  
      
        return null;  
    }
    /**
     * 不断遍历父类，直到找到方法
     * @param object
     * @param methodName
     * @param parameterTypes
     * @return
     */
    public static Method getDeclaredMethod(Class<?> clazz, String methodName, Class<?> ... parameterTypes){  
        Method method = null ;  
        for(; clazz != Object.class ; clazz = clazz.getSuperclass()) {
            try {  
                method = clazz.getDeclaredMethod(methodName, parameterTypes) ;  
                return method ;  
            } catch (Exception e) {  
            }  
        }  
        return null;  
    }  
```