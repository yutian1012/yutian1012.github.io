---
title: XSS攻击，跨站脚本攻击(Cross Site Scripting)
tags: [web-util]
---

### 1. 封装request

```
public class XSSRequestWrapper extends HttpServletRequestWrapper {
    
    private static transient Logger logger = Logger.getLogger(XSSRequestWrapper.class);
    
    public XSSRequestWrapper(HttpServletRequest servletRequest) {
        super(servletRequest);
    }

    @Override
    public String[] getParameterValues(String parameter) {
        String[] values = super.getParameterValues(parameter);

        if (values == null) {
            return null;
        }

        int count = values.length;
        String[] encodedValues = new String[count];
        for (int i = 0; i < count; i++) {
            encodedValues[i] = stripXSS(values[i]);
        }

        return encodedValues;
    }

    @Override
    public String getParameter(String parameter) {
        String value = super.getParameter(parameter);

        return stripXSS(value);
    }

    @Override
    public String getHeader(String name) {
        String value = super.getHeader(name);
        return stripXSS(value);
    }

    protected static String stripXSS(String value){
        logger.debug("IN");
        String initialValue = value;
        if (value != null) {
            // NOTE: It's highly recommended to use the ESAPI library and uncomment the following line to
            // avoid encoded attacks.
            // value = ESAPI.encoder().canonicalize(value);

            // Avoid null characters
            value = value.replaceAll("", "");
             
            // Avoid anything between script tags
            Pattern scriptPattern = Pattern.compile("<script>(.*?)</script>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            
            scriptPattern = Pattern.compile("&lt;script&gt;(.*?)&lt;/script&gt;", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid anything in a src='...' type of expression
            scriptPattern = Pattern.compile("src[\r\n]*=[\r\n]*\\\'(.*?)\\\'", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            scriptPattern = Pattern.compile("src[\r\n]*=[\r\n]*\\\"(.*?)\\\"", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            // Remove any lonesome </script> tag
            scriptPattern = Pattern.compile("</script>", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            
            scriptPattern = Pattern.compile("&lt;/script&gt;", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");

            // Remove any lonesome <script ...> tag
            scriptPattern = Pattern.compile("<script(.*?)>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            
            scriptPattern = Pattern.compile("&lt;script(.*?)&gt;", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid eval(...) expressions
            scriptPattern = Pattern.compile("eval\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid expression(...) expressions
            scriptPattern = Pattern.compile("expression\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid javascript:... expressions
            scriptPattern = Pattern.compile("javascript:", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid vbscript:... expressions
            scriptPattern = Pattern.compile("vbscript:", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");

            // Avoid onload= expressions
            scriptPattern = Pattern.compile("onload(.*?)=", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
                        
            // Avoid anything between form tags
            Pattern formPattern = Pattern.compile("<form(.*?)</form>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = formPattern.matcher(value).replaceAll("");
            
            // Avoid anything between a tags
            Pattern aPattern = Pattern.compile("<a(.*?)</a>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = aPattern.matcher(value).replaceAll("");
            
            aPattern = Pattern.compile("<a(.*?/)>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = aPattern.matcher(value).replaceAll("");
            
            aPattern = Pattern.compile("&lt;a(.*?)&lt;/a&gt;", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = aPattern.matcher(value).replaceAll("");
            
            // Example value ="<object data=\"javascript:alert('XSS')\"></object>"
            // Avoid anything between script tags
            Pattern objectPattern = Pattern.compile("<object(.*?)</object>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = objectPattern.matcher(value).replaceAll("");
            
            objectPattern = Pattern.compile("&lt;object(.*?)&lt;/object&gt;", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = objectPattern.matcher(value).replaceAll("");
            
            // Remove any lonesome </object> tag
            objectPattern = Pattern.compile("</object>", Pattern.CASE_INSENSITIVE);
            value = objectPattern.matcher(value).replaceAll("");
            
            objectPattern = Pattern.compile("&lt;/object&gt;", Pattern.CASE_INSENSITIVE);
            value = objectPattern.matcher(value).replaceAll("");
            
            // Remove any lonesome <object ...> tag
            objectPattern = Pattern.compile("<object(.*?/)>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = objectPattern.matcher(value).replaceAll("");
            
            objectPattern = Pattern.compile("&lt;object(.*?/)&gt;", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = objectPattern.matcher(value).replaceAll("");

            
            if(!value.equalsIgnoreCase(initialValue)){
                logger.warn("Message: detected a web attack through injection");
            }

        }

        logger.debug("IN");
        return value;
    }
}
```

### 2. filter调用

```
/**
 * XSS攻击：跨站脚本攻击(Cross Site Scripting)
 *
 */
public class AntiInjectionFilter implements Filter {

    private static transient Logger logger = Logger.getLogger(AntiInjectionFilter.class);
    
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    public void destroy() {
    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
        logger.debug("IN");
        chain.doFilter(new XSSRequestWrapper((HttpServletRequest) request), response);
        logger.debug("OUT");
    }

}
```