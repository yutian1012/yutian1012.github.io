---
title: Servlet
tags: [web]
---

Servlet（Server Applet），全称Java Servlet，是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。

### 1. servlet的编写
继承HttpServlet，重写doGet和doPost方法处理请求

1）编写代码

```
public class HelloWorldServelt extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        process(req, resp);
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        process(req, resp);
    }
    //处理请求，返回相应
    protected void process(HttpServletRequest req,HttpServletResponse resp)throws ServletException,IOException
    {
        resp.setContentType("text/html");
        PrintWriter out=resp.getWriter();//从响应实例中获取输出流
        out.println("<html><head><title>My First Servlet</title></head>");
        out.println("<body>");
        out.println("<h1>hello world</h1>");
        out.println("</body></html>");
    }
}
```

注：继承HttpServlet，重写doGet和doPost方法处理请求。

2）在web.xml中配置

```
<servlet>
    <servlet-name>helloworld</servlet-name>
    <servlet-class>org.sjq.servelt.HelloWorldServelt</servlet-class>
</servlet>
<servlet-mapping>
    <!-- 这个名字必须与上面定义的相同-->
    <servlet-name>helloworld</servlet-name>
    <!-- 浏览器中的输入虚拟地址访问，名称任意 -->
    <url-pattern>/helloworld</url-pattern>
</servlet-mapping>
```

3）运行，servelt的运行不像java应用直接就可以运行，必须要借助servlet容器。

第一步：将web项目部署到tomcat中，启动tomcat，

第二步：访问，http://localhost:8080/java_web/helloworld

注：客户端的浏览器向服务器发送请求信息，服务器通过tomcat接收到客户端浏览器的请求信息，调用其相应的Servlet类完成操作，服务器将计算的结果信息返回给客户端。

### 2. Servelt接口（javax.servlet.Servlet）
1）httpServlet的继承体系

![](/images/java_net/servlet/httpServlet.png)

2）ervlet的生命周期

init()方法，负责初始化Servlet对象；

service()方法，负责响应客户的请求；

destroy()方法，当Servlet对象退出生命周期时，负责释放占用的资源。

3）客户端自定义Servelt

自定义的Servlet类，通常是通过继承HttpServlet抽象类，其他的一些方法都使用父类已经实现了的方法。我们不必在实现service()方法，因为在service()方法中根据请求的信息自动调用doGet()或doPost()方法。

4）HttpServlet的service方法的实现

```
protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                doGet(req, resp);
            }
            ...
        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } 
        ...
    }
```

注：截取了部分代码，通过req.getMethod()方法判断客户端请求的方法，并调用相应的doGet或doPost方法。

### 3. ServletRequest接口
1）封装了客户请求信息，如客户请求方式、参数名和参数值、客户端正在使用的协议、以及发出客户请求的远程主机信息等。ServletRequest接口还为Servlet提供了直接以二进制方式读取客户请求数据流的ServletInputStream。

2）ServletRequest的子类可以为Servlet提供更多的和特定协议相关的数据，例如：HttpServletRequest提供了读取Http Head信息的方法。

3）凡是涉及请求/响应对象，其真实对象都由有服务器为我们产生的，如doPost(HttpServletRequest request,HttpServletResponse response)，最底层传递的request和response两个对象都是由服务器产生的。

注：可以使用输出语句看出其真实的对象
System.out.println(response.getClass().getName());
输出内容：org.apache.catalina.connector.RequestFacade

4）ServletRequest接口的主要方法

getAttribute()，根据参数给定的属性名返回属性值

getContentType()，返回客户请求数据MIME类型

getInputStream()，返回以二进制方式直接读取客户请求数据的输入流

getParameter()，根据给定的参数名返回参数值

getRemoteAddr()，返回远程客户主机的IP地址

getRemoteHost()，返回远程客户主机名

getRemotePort()，返回远程客户主机的端口号。

### 4. ServletResponse接口
1）为Servlet提供了返回响应结果的方法。允许Servlet设置返回数据的长度和MIME类型，并且提供输出流ServletOutputStream。

2）ServletResponse子类可以提供更多和特定协议相关的方法。例如，HttpServletResponse提供设定Http Head信息的方法。

3）ServletResponse接口的主要方法

getOutputStream()，返回可以向客户端发送二进制数据的输出流对象ServletOutputStream

getWriter()，返回可以向客户端发送字符数据的PrintWriter对象

getCharacterEncoding()，返回Servlet发送的响应数据的字符编码

getContentType()，返回Servlet发送的响应数据的MIME类型

setContentType()，设置Servlet发送的响应数据的MIME类型。

### 5. Servlet的初始化阶段
即什么时候Servlet容器会装载Servlet？

1）Servlet容器启动时会自动装载某些Servlet

```
<servlet>
...
    <load-on-startup>1</load-on-startup>
</servlet>
```

2）在Servlet容器启动后，客户首次向Servlet发出请求

3）Servlet的类文件被更新后，重新装载Servlet

### 6. Servlet的响应客户请求阶段
对于到达Servlet容器的客户请求，Servlet容器创建特定于这个请求的ServeltRequest对象和ServletResponse对象，然后调用Servlet的service()方法。Service()方法从ServletRequest对象获得客户请求信息、处理该请求，并通过ServletResponse对象向客户返回响应结果。

注：对于每个Http请求，Servlet容器都会创建一个request对象和与之对应的Response对象

### 7. Servlet的终止阶段
当web应用被终止，或Servlet容器终止运行，或Servlet容器重新装载Servlet的新实例时，Servlet容器还会先调用Servlet的destroy()方法。在destroy()方法中，可以释放Servlet所占用的资源。

### 8. Servlet多线程问题
Servlet是单例模式的，一旦实例化之后，只能生成唯一的一个对象。不管多少人使用，都使用的是同一个对象，而变量又是成员变量，所以也是这个对象中唯一的变量，多少人修改，也是修改的这唯一一个变量。

注：避免在Servlet中使用成员变量，并且成员变量初始化后便比可以修改。或者将成员变量定义为方法的局部变量。

### 9. 使用JSP完成Servelt功能
1）在webapp目录下建立index.jsp文件，在文件中使用html元素输出helloworld

```
<html>
    <body>
        <h2>Hello World!!</h2>
    </body>
</html>
```

2）访问服务：http://localhost:8080/java_web

注：Servlet能完成的，JSP也能完成，两者可以相互代替。

注2：index.jsp不要放置到WEB-INF目录下，否则客户端请求不能直接访问。

注3：JSP在书写时与html非常类似，但底层也是使用Servlet来实现的。

### 10. JSP与Servlet的生成方式
1）Servlet首先被编译为class文件，然后由服务器调用

2）JSP首先被转化为Servlet（java文件），然后再被编译为class文件，最后由服务器调用。

注：在tomcat目录下的work子目录的localhost目录下有我们部署的项目，可以找到由JSP生成的java源文件。如：index.jsp.java 文件等。



