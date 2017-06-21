---
title: 自定义异常类的使用
tags: [basic]
---

自定义异常类能够更清晰有效的定义系统异常（对异常进行细化），可以针对特定类型的异常捕获，进行特定的异常处理（系统更有针对性的处理异常）。同时也便于查找和定位错误。

### 自定义异常类

```
public class CpcAppNoException extends Throwable {
    
    private int errorCode;
    
    public CpcAppNoException(String message) {
        super(message);
    }
    
    public CpcAppNoException(String message,int errorCode){
        super(message);
        this.errorCode=errorCode;
    }
    

    public int getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(int errorCode) {
        this.errorCode = errorCode;
    }
    
}
```

注：一般继承Exception类即可。

### 捕获自定义异常

```
public class ExceptionTest {
    @Test
    public void testException(){
        try{
            sayHello();
        }catch(CpcAppNoException cpcEx){
            System.out.println(cpcEx.getMessage());
            System.out.println(cpcEx.getErrorCode());
            cpcEx.printStackTrace();
        }
        //...可
    }
    
    public void sayHello() throws CpcAppNoException{
        //...业务代码

        //不满足条件时，手动抛出异常
        throw new CpcAppNoException("解析错误",1);
    }
}
```

