---
title: spring属性文件的多重方式获取
tags: [spring]
---

### 多重方式获取属性文件

实现：即配置PropertyPlaceholderConfigurer同时又能从代码中获取属性文件的值

1）配置文件

```
<!--加载配置文件-->
<bean id="configproperties" 
     class="org.springframework.beans.factory.config.PropertiesFactoryBean">
      <property name="location" value="classpath:/conf/app.properties"/>
</bean>
 <bean id="propertyConfigurer"  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
     <property name="properties" ref="configproperties"/>
</bean>
```

注：配置了PropertyPlaceholderConfigurer能够保证xml和注解中使用属性文件

2）配置PropertiesFactoryBean会返回Properties实例

自定义属性获取类

```
/**
 * 系统参数读取。
 * @author ray
 *
 */
public class AppConfigUtil implements ApplicationContextAware{
    private static ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext)
            throws BeansException {
        this.applicationContext=applicationContext;
    }
    /**
     * 从配置文件中读取配置属性
     * @param propertyKey 属性key
     * @return
     */
    public static String get(String propertyKey){
        Properties properties=(Properties)applicationContext.getBean("configproperties");
        return properties.getProperty(propertyKey);
    }
    
    /**
     * 从参数文件中读取整形的配置参数。
     * @param propertyKey
     * @return
     */
    public static int getInt(String propertyKey){
        String val=get(propertyKey);
        return Integer.parseInt(val); 
    }
    
    /**
     * 从参数文件中读取整形的配置参数，可以设置默认值。
     * @param propertyKey   键值
     * @param defaultValue  默认值
     * @return
     */
    public static int getInt(String propertyKey,int defaultValue){
        String val=get(propertyKey);
        if(StringUtil.isEmpty(val)) return defaultValue;
        return Integer.parseInt(val);
    }
    
    /**
     * 从配置文件中读取配置属性
     * @param propertyKey 属性key
     * @param defultValue 如果为空，填充默认值。
     * @return
     */
    public static String get(String propertyKey,String defultValue){
        String val= get(propertyKey);
        if(StringUtil.isEmpty(val)) return defultValue;
        return val;
    }
    
    /**
     * 获取布尔值属性。
     * @param propertyKey 属性key
     * @return
     */
    public static boolean getBoolean(String propertyKey){
        String val= get(propertyKey);
        if(null==val) return false;
        return val.equalsIgnoreCase("true");
    }
}
```

注：通过容器的getBean获取属性文件，并提供相应的获取属性值方法完成在类中获取属性

3）使用

配置文件中设置工具类AppConfigUtil

```
<bean id="appConfigUtil" class="xxx.AppConfigUtil"/>
```

使用代码

```
String test=AppConfigUtil.get("cpquery.interface.user.clientsecret");
```