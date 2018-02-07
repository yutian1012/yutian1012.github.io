---
title: CXF安全认证
tags: [webservice]
---

采用基于用户名和密码认证的方式来实现客户端调用，通过添加拦截器实现。

CXF relies on WSS4J in large part to implement WS-Security

参考：http://blog.csdn.net/chrisjingu/article/details/52385632

参考：https://cxf.apache.org/docs/ws-security.html

### ws-security进行验证拦截

1）引入jar包

```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-ws-security</artifactId>
    <version>${cxf.version}</version>
</dependency>
```

2）创建验证类

```
public class ServerPasswordCallback implements CallbackHandler{
    @Override
    public void handle(Callback[] callbacks) throws IOException,
            UnsupportedCallbackException {
        WSPasswordCallback pc = (WSPasswordCallback) callbacks[0];
        if ("huwei".equals(pc.getIdentifier())) {
            pc.setPassword("hello");//设置带验证的密码
        }
    }
}
```

注：该类实现ws-security的CallbackHandler接口，并重写它的handle方法。该类就是身份验证的主要类，当客户端传过的用户名中为"huwei"时，该方法会将指定的密码告知ws-security的WSPasswordCallback 类，并让它后期去和客户端的密码进行验证，通过就放行，否则打回。在applicationContext.xml中该类会作为WSS4JInInterceptor拦截器的回调函数属性，进行配置。

3）配置安全拦截器（实现）

```
<bean id="serverPasswordCallback" class="org.ipph.web.webservice.cxf.security.ServerPasswordCallback"/>
            
<jaxws:endpoint implementor="#webServiceImpl" address="http://localhost:7777/hello" >
    <jaxws:inInterceptors>  
        <bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
        <bean class="org.apache.cxf.ws.security.wss4j.WSS4JInInterceptor">
            <constructor-arg>
                <map>
                    <entry key="action" value="UsernameToken"/>
                    <!-- 密码类型，PasswordText表示明文,密文是PasswordDigest -->    
                    <entry key="passwordType" value="PasswordText"/>
                    <entry key="passwordCallbackRef">
                     <!-- 回调函数引用 -->   
                        <ref bean="serverPasswordCallback"/>
                    </entry>
                </map>
            </constructor-arg>
        </bean>
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:endpoint>
```

4）代码方式发布实现

```
@WebService(
        name="",//默认为服务类名
        serviceName="",//默认是服务类名+Service
        targetNamespace="",//默认是包名倒写
        portName=""//默认为类名+Port
    )
public class JAXWSWebServiceSecurity {
    public String sayHello(String name){
        return name+" hello";
    }

    public static void main(String[] args){
        String url="http://localhost:7777/hello";
        //获取CXF发表的服务类，
        JaxWsServerFactoryBean bean=new JaxWsServerFactoryBean();
        //设置发布路径
        bean.setAddress(url);
        //设置实现类
        bean.setServiceBean(new JAXWSWebService());
        //给服务添加日期功能，添加请求日志
        bean.getInInterceptors().add(new LoggingInInterceptor());
        
        Map<String, Object> properties=new HashMap<String,Object>();
        properties.put(WSHandlerConstants.ACTION, WSHandlerConstants.USERNAME_TOKEN);
        properties.put(WSHandlerConstants.PASSWORD_TYPE, WSConstants.PASSWORD_TEXT);
        properties.put(WSHandlerConstants.PW_CALLBACK_REF, new ServerPasswordCallback());
        WSS4JInInterceptor wss4jInInterceptor=new WSS4JInInterceptor(properties);
        
        bean.getInInterceptors().add(wss4jInInterceptor);
        
        //添加回复/响应日志
        bean.getOutInterceptors().add(new LoggingOutInterceptor());
        //发表
        bean.create();
    }
}
```

5）客户端访问测试

使用SOAPUI访问出现授权访问错误

```
安全拦截后的异常：A security error was encountered when verifying the message
```

在左下角request properties面板中设置用户名和密码信息

![](/images/java_structure/webservice/soapui-security.png)

### 客户端调用

WSS4J同样可以用于CXF客户端调用。客户端需要使用一个out interceptors，在soapheader上加相应的认证内容。而服务器端则可以使用一个in interceptor来实现消息的认证。

The web service provider may not need both in and out WS-Security interceptors. For instance, if you are just requiring signatures on incoming messages, the web service provider will just need an incoming WSS4J interceptor and only the SOAP client will need an outgoing one.

1）配置客户端

```
<jaxws:client id="webService" serviceClass="org.ipph.web.webservice.IWebService" 
    address="http://localhost:7777/hello">
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.ws.security.wss4j.WSS4JOutInterceptor">
            <constructor-arg>
                <map>
                    <entry key="action" value="UsernameToken"></entry>
                    <entry key="user" value="huwei" />
                    <entry key="passwordType" value="PasswordText"/>
                    <entry key="passwordCallbackRef">
                        <ref bean="serverPasswordCallback"/>
                    </entry>
                </map>
            </constructor-arg>
        </bean>
    </jaxws:outInterceptors>
    <!-- <jaxws:properties>
        <entry>
            <
        </entry>
    </jaxws:properties> -->
    
</jaxws:client>
```

2）客户端代码实现

```
public class CxfClientSecurity {
    public static void main(String[] args) {
        JaxWsDynamicClientFactory clientFactory=JaxWsDynamicClientFactory.newInstance();
        Client client = clientFactory.createClient("http://localhost:7777/hello?wsdl");
        
        Map<String, Object> properties=new HashMap<String, Object>();
        properties.put(WSHandlerConstants.USER, "huwei");
        properties.put(WSHandlerConstants.ACTION, WSHandlerConstants.USERNAME_TOKEN);
        //设置密码
        properties.put(WSHandlerConstants.PW_CALLBACK_REF, new ServerPasswordCallback());
        //设置安全属性
        WSS4JOutInterceptor wss4jOutInterceptor=new WSS4JOutInterceptor(properties);
        
        client.getOutInterceptors().add(wss4jOutInterceptor);
        
        Object[] result;
        try {
            result = client.invoke("sayHello", "KEVIN");
            System.out.println(result[0]);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

3）客户端的ServerPasswordCallback类

```
public class ServerPasswordCallback implements CallbackHandler {

    public void handle(javax.security.auth.callback.Callback[] callbacks) {
        for (int i = 0; i < callbacks.length; i++) {  
            WSPasswordCallback pc = (WSPasswordCallback)callbacks[i];  
            pc.setPassword("hello");  
        }  
    }  

}
```

注：同服务端一样，ServerPasswordCallback也是实现了CallbackHandler接口，与服务器端不同的是客户端该类的用途只是为了设置密码，不需要做任何验证。