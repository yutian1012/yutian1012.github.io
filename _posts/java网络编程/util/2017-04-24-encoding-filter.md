---
title: web请求编码过滤
tags: [web-util]
---

### 1. web.xml中配置

```
<filter>
    <filter-name>EncodingFilter</filter-name>
    <filter-class>it.xxx.utilities.filters.EncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
```

### 2. 实现类

```
public class EncodingFilter implements Filter {

    
    private static transient Logger logger = Logger.getLogger(EncodingFilter.class);
     private FilterConfig filterConfig = null;
     
     public void init(FilterConfig filterConfig) {
            this.filterConfig = filterConfig;
          }
     
    
    public void destroy() {
        // do nothing
    }
    
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
            logger.debug("IN");
            
            String encoding = filterConfig.getInitParameter("encoding");
            if(encoding == null) encoding = "UTF-8";            
            request.setCharacterEncoding(encoding);
            
            chain.doFilter(request, response);
    }

}
```