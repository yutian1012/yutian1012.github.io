---
title: java调用脚本的封装
tags: [groovy]
---

1）GroovyScriptEngineWrapper类

该类封装了GroovyShell执行脚本的执行过程，对外提供相应的接口，用户只需要设置相应的脚步，以及脚本执行时需要的变量，就可以获取相应的返回结果。

```
package groovy.wrapper;

import groovy.lang.GroovyShell;

import java.util.Iterator;
import java.util.Map;
import java.util.Set;

public class GroovyScriptEngineWrapper {
    private GroovyBindingWrapper binding = new GroovyBindingWrapper();
    
    /**
     * 执行groovy代码无返回。
     * 
     * @param script
     * @throws Exception 
     */
    public void execute(String script,Map<String, Object> vars)  {
        executeObject(script,vars);
    }

    /**
     * 设置执行参数。
     * @param shell
     * @param vars
     */
    private void setParameters(GroovyShell shell, Map<String, Object> vars) {
        if(vars==null) return;
        Set<Map.Entry<String, Object>> set = vars.entrySet();
        Iterator<Map.Entry<String, Object>> it = set.iterator();
        while(it.hasNext()){
            Map.Entry<String, Object> entry = (Map.Entry<String, Object>) it.next();
            shell.setVariable(entry.getKey(), entry.getValue());
        }
    }

    /**
     * 执行groovy代码返回布尔型。
     * @param script
     * @return
     * @throws Exception 
     */
    public boolean executeBoolean(String script,Map<String, Object> vars) {
        Boolean rtn = (Boolean)  executeObject(script,vars);
        return rtn;
    }

    /**
     * 执行脚本返回字符串数据。
     * 
     * @param script
     * @return
     * @throws Exception 
     */
    public String executeString(String script,Map<String, Object> vars) {
        String str = (String) executeObject(script,vars);
        return str;
    }

    /**
     * 执行脚本返回整形数据。
     * @param script
     * @return
     * @throws Exception 
     */
    public int executeInt(String script,Map<String, Object> vars)  {
        Integer rtn = (Integer) executeObject(script,vars);
        return rtn;
    }

    /**
     * 执行脚本返回浮点型数据。
     * @param script
     * @return
     * @throws Exception 
     */
    public float executeFloat(String script,Map<String, Object> vars) {
        Float rtn =(Float) executeObject(script,vars);
        return rtn;
    }
    
    /**
     * 执行脚本返回对象数据。
     * 
     * @param script
     * @return
     * @throws Exception 
     */
    public Object executeObject(String script,Map<String, Object> vars) {
        GroovyShell shell = new GroovyShell(binding);
        setParameters(shell,vars);
        
        //防止脚本注入
        script=script.replace("&apos;", "'")
        .replace("&quot;", "\"")
        .replace("&gt;", ">")
        .replace("&lt;", "<")
        .replace("&nuot;", "\n")
        .replace("&amp;", "&");
        
        Object rtn =  shell.evaluate(script);
        binding.clearVariables();
        return rtn;
    }

    public Object getVariable(String key){
        return binding.getVariable(key);
    }
    /**
     * 向bindings中设置bean对象
     * @param properyName
     * @param bean
     */
    public void setProperty(String properyName,Object bean){
        binding.setProperty(properyName, bean);
    }
}
```

2）GroovyBindingWrapper类

该类继承了Binding，并覆盖了相应的方法，用于设置脚本执行时的变量信息，并为解决多线程问题提供了方法。

```
package groovy.wrapper;

import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

import groovy.lang.Binding;

public class GroovyBindingWrapper extends Binding{
      private static ThreadLocal<Map<String,Object>> localVars=new ThreadLocal<Map<String,Object>>();
      
      private static Map<String,Object> propertyMap=new HashMap<String, Object>();

      public GroovyBindingWrapper()
      {
      }

      public GroovyBindingWrapper(Map<String,Object> variables)
      {
          localVars.set(variables);
        //this.localVars = variables;
      }

      public GroovyBindingWrapper(String[] args)
      {
        this();
        setVariable("args", args);
      }

      public Object getVariable(String name)
      {
          Map<String,Object> map=localVars.get();
          Object result=null;
          if(map!=null && map.containsKey(name)){
             result=map.get(name);
          }
          else{
              result=propertyMap.get(name);
          }
        
        return result;
      }

      public void setVariable(String name, Object value)
      {
          if(localVars.get()==null){
              Map<String,Object> vars=  new LinkedHashMap<String,Object>();
              vars.put(name, value);
              localVars.set(vars);
          }
          else{
              localVars.get().put(name, value);
          }
      }

      public Map<String,Object> getVariables() {
          if(localVars.get()==null){
              return new LinkedHashMap<String,Object>();
          }
          
        return localVars.get();
      }
      
      public void clearVariables(){
          localVars.remove();
      }

      public Object getProperty(String property)
      {
        return propertyMap.get(property);
      }

      public void setProperty(String property, Object newValue)
      {
          propertyMap.put(property, newValue);
      }
}
```

3）ScriptImpl类

该类是具体脚本执行方法的实现类。

```
package groovy.wrapper;
/**
 * 提供脚本的实现，并在脚本中调用相应的实现
 */
public class ScriptImpl {
    /**
     * 返回字符串
     * @param name
     * @return
     */
    public String sayHello(String name){
        return "hello "+name;
    }
    /**
     * 返回计算结果的数值
     * @param num1
     * @param num2
     * @return
     */
    public int caculateAdd(int num1,int num2){
        return num1+num2;
    }
}
```

4）测试类


```
package groovy.wrapper;

import java.util.HashMap;
import java.util.Map;

import org.junit.Test;

public class GroovyScriptEngineWrapperTest {

    @Test
    public void testScript() {
        //第一步构造出脚本执行引擎包装类
        GroovyScriptEngineWrapper  groovyScriptEngine=new GroovyScriptEngineWrapper();

        //第二步构造出具体的脚步方法实现类
        ScriptImpl script=new ScriptImpl();
        
        //第三步将脚本实现类设置到脚本引擎变量中
        groovyScriptEngine.setProperty("script", script);
        
        //第四步编写用户的执行脚本
        String spt="script.sayHello(name)";
        
        //第五步设置脚本变量
        Map<String,Object> vars=new HashMap<String,Object>();
        vars.put("name", "zhangsan");
        
        //第六步执行脚本引擎提供的接口，并获取返回值
        String hell=groovyScriptEngine.executeString(spt, vars);
        
        assert "hello zhangsan".equals(hell);
    }

}
```