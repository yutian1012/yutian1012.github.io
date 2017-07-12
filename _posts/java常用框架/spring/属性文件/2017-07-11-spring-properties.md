---
title: spring属性文件的配置与使用
tags: [spring]
---

参考：http://blog.csdn.net/bnna8356586/article/details/51406459

### spring属性文件

1）spring配置文件中配置

```
<!-- spring的属性加载器，加载properties文件中的属性 -->  
<bean id="propertyConfigurer"  
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
    <property name="location" value="classpath:config.properties" />  
</bean> 
```

注：config.properties指定属性文件位置

2）属性文件的使用--xml

```
<bean id="mailSender"  
    class="org.springframework.mail.javamail.JavaMailSenderImpl">  
    <property name="host">  
        <value>${email.host}</value>  
    </property>  
    <property name="port">  
        <value>${email.port}</value>  
    </property>  
    <property name="username">  
        <value>${email.username}</value>  
    </property>  
    <property name="password">  
        <value>${email.password}</value>  
    </property>  
    <property name="javaMailProperties">  
        <props>  
            <prop key="mail.smtp.auth">true</prop>  
            <prop key="sendFrom">${email.sendFrom}</prop>  
        </props>  
    </property>  
</bean> 
```

注：配置bean时，从属性文件中获取属性值

3）属性文件的使用--注解

```
@Component("configInfo")  
public class ConfigInfo {  
    @Value("${file.savePath}")  
    private String fileSavePath;  
  
    @Value("${file.backupPath}")  
    private String fileBakPath;  
          
    public String getFileSavePath() {  
        return fileSavePath;  
    }  
  
    public String getFileBakPath() {  
        return fileBakPath;  
    }      
}
```

注：使用@Value注解可以获取属性文件中的属性值