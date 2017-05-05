---
title: Axis框架
tags: [webservice]
---

```
<servlet>
    <servlet-name>AxisServlet</servlet-name>
    <servlet-class>org.apache.axis.transport.http.AxisServlet</servlet-class>
</servlet>
<servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>org.apache.axis.transport.http.AdminServlet</servlet-class>
    <load-on-startup>100</load-on-startup>
</servlet>
```