---
title: CXF客户端调用
tags: [webservice]
---

### jar包依赖

```
<!-- cxf 3.1.11版本-->
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-core</artifactId>
    <version>${cxf.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>${cxf.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>${cxf.version}</version>
</dependency>
```

### 类方式访问运行的webservice

```
public class CxfClient {
    public static void main(String[] args) {
        JaxWsDynamicClientFactory clientFactory=JaxWsDynamicClientFactory.newInstance();
        Client client = clientFactory.createClient("http://10.33.6.199:8989/WS_Server/Webservice?wsdl");  
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

### 配置方式调用结合spring

```
<jaxws:client id="webService" serviceClass="org.ipph.web.webservice.IWebService" address="http://localhost:7777/hello" />
```

测试类

```
@WebServlet(urlPatterns="/cxfClient")
public class CxfServletClient extends HttpServlet{
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        //调用ws
        ApplicationContext context= WebApplicationContextUtils.getWebApplicationContext(req.getServletContext());
        IWebService service=(IWebService) context.getBean("webService");
        service.sayHello("ssss");
    }
}
```