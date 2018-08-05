---
title: spring BeanPostProcessor类实例（优化）
tags: [spring]
---

针对硬编码方式的版本实现调用，现使用BeanPostProcessor进行优化，使代码更简洁，便于维护。

1）新建枚举对象HelloVersion

```
package com.sjq.bean.postprocessor.after;

public enum HelloVersion {
    A,B;
}
```

注：这里定义两个版本，即A和B。

2）使用注解的方式实现不同版本方法是实现类的标识

HelloVersionInjected注解：（待注入的属性上）

针对注入对象声明时使用具有实现类，现修改为使用接口，并且如果类属性使用该注解修饰，则再注入对象实例时，需要根据当前环境设置的version版本信息，动态注入不同的实现类。

```
package com.sjq.bean.postprocessor.after;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.stereotype.Component;

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface HelloVersionInjected {
}
```

注：该注解相当于@Autowired，用于实现spring容器的对象依赖注入。

HelloVersionSwitch注解（不同版本实现的接口和接口方法上）

该注解可应用到具体的版本接口和方法上，实现了该接口的实现类，意味着会根据环境信息使用不同的实现类。

```
package com.sjq.bean.postprocessor.after;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.stereotype.Component;

@Target({ElementType.FIELD,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface HelloVersionSwitch {
    String value() default "";
}
```

3）修改IHelloService接口，添加HelloVersionSwitch注解

```
package com.sjq.bean.postprocessor.after;

@HelloVersionSwitch("hello.version")
public interface IHelloService {
    public void hello();
}
```

注：这里注解的值虽然是属性文件中的key，但并没有实际获取属性文件设置的值

4）实现类不需要进行任何修改

实现类：HelloServiceImplV1和HelloServiceImplV2

参考：2018-08-05-spring-bean-BeanPostProcessor-before-demo.md

5）修改调用类

```
package com.sjq.bean.postprocessor.after;
import javax.annotation.Resource;
import org.springframework.stereotype.Component;

@Component
public class HelloInvokeDemo {
    @Resource
    private IHelloService helloService;
    
    public void hello(){
        helloService.hello();
   }
}
```

6）实现BeanPostProcessor接口，实现自动识别注入实现类

```
package com.sjq.bean.postprocessor.after;

import java.lang.reflect.Field;
import java.util.Map;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanCreationException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class HelloVersionBeanPostProcessor implements BeanPostProcessor{
    
    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        Class<?> clazz = bean.getClass();
        Field[] fields = clazz.getDeclaredFields();
        for (Field f : fields) {
            // 初始化对象时，判断类的属性是否使用了HelloVersionInjected注解修饰
            if (f.isAnnotationPresent(HelloVersionInjected.class)) {
                if (!f.getType().isInterface()) {//判断对象的声明是否是接口
                    throw new BeanCreationException("HelloVersionInjected field must be declared as an interface:" 
                        + f.getName() + " @Class " + clazz.getName()); 
                } 
                try { 
                    //处理属性的注入
                    this.handleVerionInjected(f, bean, f.getType()); 
                } catch (IllegalAccessException e) {
                    throw new BeanCreationException("Exception thrown when handleAutowiredRouting", e);
                }
            } 
        } return bean;
    }
    /**
     * 注入类属性，这里通过代理实现属性的注入
     */
    private void handleVerionInjected(Field field, Object bean, Class<?> type) throws IllegalAccessException { 
        Map<String, ?> candidates = this.applicationContext.getBeansOfType(type);
        field.setAccessible(true); 
        if (candidates.size() == 1) { 
            field.set(bean, candidates.values().iterator().next());
        } else if (candidates.size() == 2) { 
            Object proxy = HelloVersionBeanProxyFactory.createProxy(type, candidates);
            field.set(bean, proxy); 
        } else { 
            throw new IllegalArgumentException("Find more than 2 beans for type: " + type);
        }
    }
}
```

7）创建代理类

代理类根据环境设置信息返回实现类的代理对象（对实现类封装了一层代理）

```
package com.sjq.bean.postprocessor.after;

import java.lang.reflect.Method;
import java.util.Map;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.aop.framework.ProxyFactory;
import org.springframework.util.StringUtils;

public class HelloVersionBeanProxyFactory {
    
    /**
     * 创建代理对象
     * @param targetClass
     * @param beans
     * @return
     */
    public static Object createProxy(Class<?> targetClass, Map<String, ?> beans) {
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setInterfaces(targetClass);
        proxyFactory.addAdvice(new HelloVersionMethodInterceptor(targetClass, beans)); 
        return proxyFactory.getProxy(); 
    } 
    
    /**
     * 实现代理对象，当调用代理对象的方法时，都会执行invoke方法
     * MethodInterceptor实现了Advice接口
     */
    static class HelloVersionMethodInterceptor implements MethodInterceptor {
        private String classSwitch; 
        private Object beanOfSwitchOn; //开关打开时使用V2实现类
        private Object beanOfSwitchOff; //开关关闭时使用V1实现类
        
        public HelloVersionMethodInterceptor(Class<?> targetClass, Map<String, ?> beans) {
            String interfaceName = StringUtils.uncapitalize(targetClass.getSimpleName()); 
            if(targetClass.isAnnotationPresent(HelloVersionSwitch.class)){ 
                this.classSwitch =((HelloVersionSwitch)targetClass.getAnnotation(HelloVersionSwitch.class)).value();
            } 
            this.beanOfSwitchOn = beans.get(this.buildBeanName(interfaceName, true));
            this.beanOfSwitchOff = beans.get(this.buildBeanName(interfaceName, false));
        } 
        /**
         * 获取类名称，实现类名称的命名与接口是有规则的
         * @param interfaceName
         * @param isSwitchOn
         * @return
         */
        private String buildBeanName(String interfaceName, boolean isSwitchOn) {
            String className=interfaceName;
            if(interfaceName.startsWith("i")) {
                className= StringUtils.uncapitalize(interfaceName.replaceFirst("i", ""));//首字母小写
            }
            
            return className + "Impl" + (isSwitchOn ? "V2" : "V1"); 
        }
        
        @Override 
        public Object invoke(MethodInvocation invocation) throws Throwable {
            Method method = invocation.getMethod(); 
            String switchName = this.classSwitch; //类上的注解value值
            if (method.isAnnotationPresent(HelloVersionSwitch.class)) { 
                switchName = method.getAnnotation(HelloVersionSwitch.class).value(); //获取方法上注解的value值
            } 
            if (StringUtils.isEmpty(switchName)) { 
                throw new IllegalStateException("HelloVersionSwitch's value is blank, method:" + method.getName());
            } 
            return invocation.getMethod().invoke(getTargetBean(switchName), invocation.getArguments()); 
        } 
        
        public Object getTargetBean(String switchName) { 
            boolean switchOn; 
            if (HelloVersion.A.equals(switchName)) { 
                switchOn = false; 
            } else if (HelloVersion.B.equals(switchName)) {
                switchOn = true; 
            } else { 
                switchOn = true;
            } 
            return switchOn ? beanOfSwitchOn : beanOfSwitchOff; 
        }

    }

}
```