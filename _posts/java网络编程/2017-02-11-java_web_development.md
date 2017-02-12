---
title: java web环境搭建
tags: [web]
---
windows下安装环境

### 1. 安装jdk环境
1）下载jdk安装程序，并安装

下载路径：http://www.oracle.com/technetwork/java/javase/downloads/index.html

2）设置环境变量（安装完毕后）

配置环境变量包括java_home，path和classpath三个部分。

JAVA_HOME：E:\work\Java\jdk1.7.0_40

PATH：E:\work\Java\jdk1.7.0_40\bin

CLASS_PATH：.;E:\work\Java\jdk1.7.0_40\jre\lib

3）测试命令：java -version

如果能正常输出java的版本信息，说明jdk安装成功！即java的开发环境配置成功。

### 2. 安装tomcat
1）下载tomcat的压缩包，解压到E:\work\apache-tomcat-7.0.42目录下

2）设置环境变量：ATALINA_BASE和CATALINA_HOME，设置目录为tomcat的解压目录E:\work\apache-tomcat-7.0.42。

3）测试是否能够启动

双击E:\work\apache-tomcat-7.0.42\bin\startup.bat文件，启动成功后在浏览器中访问http://localhost:8080。

### 3. eclipse开发工具
1）设置jdk安装目录：Windows---preferences---java---installed JREs

2）设置tomcat安装目录：Windows---preferences---Server---Runtime Environment

3）新建maven项目，select an Archetype选择maven-archetype-webapp

在pom.xml中添加依赖

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>compile</scope>
</dependency>
```

4）maven的web项目的目录结构

![](/images/java_net/web/maven_web_struct.png)

### 4. 开发第一个Servelt
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

### 5. 使用JSP完成Servelt功能
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

