---
title: spring BeanPostProcessor类实例（背景）
tags: [spring]
---

参考：https://www.jianshu.com/p/1417eefd2ab1

实例：使用BeanPostProcessor实现依据开关动态注入相应的依赖对象。

1）背景

系统开发的版本决定了最终调用的不同版本方法。

2）接口类

```
package com.sjq.bean.postprocessor.before;

public interface IHelloService {
    public void hello();
}
```

不同版本的实现类：HelloServiceImplV1类

```
package com.sjq.bean.postprocessor.before;
import org.springframework.stereotype.Service;
@Service
public class HelloServiceImplV1 implements IHelloService{
    @Override
    public void hello() {
        System.out.println("【HelloServiceImplV1】 version 1");
    }
}
```

HelloServiceImplV2类：

```
package com.sjq.bean.postprocessor.before;
import org.springframework.stereotype.Service;
@Service
public class HelloServiceImplV2 implements IHelloService{
    @Override
    public void hello() {
        System.out.println("【HelloServiceImplV2】 version 2");
    }
}
```

3）调用类

实例注入与调用类，该类根据设置的version实现不同类的调用

```
package com.sjq.bean.postprocessor.before;

import javax.annotation.Resource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
@Component
public class HelloInvokeDemo {
    @Resource
    private HelloServiceImplV1 helloServiceImplV1;
    @Resource
    private HelloServiceImplV2 helloServiceImplV2;
    
    @Value("${hello.version}")
    private String version;
    
    public void hello(){
        if("A".equals(version)){
             helloServiceImplV1.hello();
        }else{
             helloServiceImplV2.hello();
        }
   }
}
```

4）配置信息

配置文件applicationContext_PostProcessor.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
            http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
            
    <context:component-scan base-package="com.sjq.bean.postprocessor.before"></context:component-scan>
    
    <!-- spring的属性加载器，加载properties文件中的属性 -->  
    <bean id="propertyConfigurer"  
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="location" value="classpath:app.properties" />  
    </bean>
    
</beans>
```

属性文件：app.properties

```
hello.version=A
```

5）启动类ApplicationPostProcessorRun

```
package com.sjq.bean.postprocessor.before;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ApplicationPostProcessorRun {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext_PostProcessor.xml"); 
        //调用类
        HelloInvokeDemo invoke=context.getBean(HelloInvokeDemo.class);
        invoke.hello();
    }
}
```

6）存在的问题：

首先HelloInvokeDemo类声明的对象是具有的实现类，而不是使用接口来声明对象

```
@Resource
private IHelloService helloService;
```

注：最好的方式是使用接口声明，实际注入时可注入不同的实现类。

其次很多代码都需要进行if逻辑判断，这种写死的方式实现版本实现的调用是非常繁琐的，同时也不便于维护。