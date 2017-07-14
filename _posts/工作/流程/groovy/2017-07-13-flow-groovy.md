---
title: 流程中对groovy脚本的封装
tags: [bpmx]
---

### 定位使用

在BpmNodeUserCalculationScript类中getExecutor方法会加载BpmNodeUser中设置的脚步（人员选择脚本），然后绑定流程中设置的变量以及相关的流程信息，调用GroovyScriptEngine类的executeObject方法。

BpmNodeUserCalculationScript类是计算任意的具体策略类，实现了IBpmNodeUserCalculation接口。

```
@Override
public List<SysUser> getExecutor(BpmNodeUser bpmNodeUser, CalcVars vars) {
    Long prevUserId = vars.getPrevExecUserId();
    Long startUserId = vars.getStartUserId();
    
    String script = bpmNodeUser.getCmpNames();//脚本
    List<SysUser> users = new ArrayList<SysUser>();//返回执行人集合对象

    //groovy设置变量--通过将流程中的变量信息设置到该对象中
    Map<String, Object> grooVars = new HashMap<String, Object>();
    
    ProcessCmd cmd=TaskThreadService.getProcessCmd();
    
    String actInstId="";
    if(StringUtil.isNotEmpty(vars.getActInstId()) ){
        actInstId=vars.getActInstId();
        if (cmd!=null) {
            cmd.addTransientVar("actInstId", actInstId);
        }
    }
    grooVars.put("actInstId", actInstId);
    if(vars.getVars().size()>0){
        grooVars.putAll(vars.getVars());
    }
    grooVars.put(BpmConst.PrevUser, prevUserId);
    grooVars.put(BpmConst.StartUser, startUserId.toString());
    
    //groovyScriptEngine封装了groovy.lang.GroovyShell
    //返回的结果是用户的主键字符串集合
    Object result = groovyScriptEngine.executeObject(script, grooVars);
    if (result == null) {
        return users;
    }
    Set<String> set = (Set<String>) result;
    for (Iterator<String> it = set.iterator(); it.hasNext();) {
        String userId = it.next();
        SysUser sysUser = sysUserService.getById(Long.parseLong(userId));
        users.add(sysUser);
    }
    return users;
}
```

### GroovyScriptEngine封装类

该类是用来封装脚本执行的过程，并提供一些接口方法供客户端调用。脚本的执行需要GroovyShell（groovy.lang.GroovyShell）对象，传递待执行的脚本，以及执行脚本需要的参数。

该类提供了executeBoolean，executeString，executeInt等方法，用于获取执行脚本返回的对象值（这里只列出了部分方法实现）。

```
public class GroovyScriptEngine implements BeanPostProcessor {
    private GroovyBinding binding = new GroovyBinding();

    /**
     * 设置执行参数。
     * @param shell
     * @param vars
     */
    private void setParameters(GroovyShell shell, Map<String, Object> vars) {
        if(vars==null) return;
        Set<Map.Entry<String, Object>> set = vars.entrySet();
        for (Iterator<Map.Entry<String, Object>> it = set.iterator(); it.hasNext();) {
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
    
    ...

    /**
     * 执行脚本返回对象数据。
     * @param script
     * @return
     * @throws Exception 
     */
    public Object executeObject(String script,Map<String, Object> vars) {
        GroovyShell shell = new GroovyShell(binding);
        setParameters(shell,vars);
        
        //格式化脚本，防止脚本注入攻击
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

    ...
}
```

注：通过查看executeObject方法，传递进来的脚本参数通过shell.setVariable方式设置到脚本执行的shell中，并且每次都会new出一个新的GroovyShell对象，因此可以有效的防止多线程问题。

### GroovyBinding封装

GroovyBinding（com.hotent.core.engine.GroovyBinding）对象是一个自定义对象，并非是Groovy提供的。该类继承了Binding（groovy.lang.Binding）对象，并覆盖了相应的方法，从而在获取getVariable或getProperty实现了自定义的方法

```
public class GroovyBinding extends Binding{
      private Map variables;
      private static ThreadLocal<Map<String,Object>> localVars=new ThreadLocal<Map<String,Object>>();
      
      private static Map<String,Object> propertyMap=new HashMap<String, Object>();

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

      ...

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

注：该类定义了存放变量的集合，包括线程本地变量（为了解决多线程问题），propertyMap集合主要存放spring中的bean对象。

### GroovyScriptEngine使用spring容器中的bean

GroovyScriptEngine类实现了BeanPostProcessor接口，会在容器启动时将当前项目中的所有的service类和实现了IScript接口的类注册到脚本引擎中，在编写脚本时可以直接使用。

重写postProcessAfterInitialization方法，将spring的bean设置到脚本的Binding对象中

```
/**
 * 给groovy脚本引擎注入 service对象引用和实现IScript对象的引用。
 */
public Object postProcessAfterInitialization(Object bean, String beanName)
        throws BeansException {
    boolean rtn = BeanUtils.isInherit(bean.getClass(), BaseService.class);
    boolean isImplScript = BeanUtils.isInherit(bean.getClass(),IScript.class);
    if (rtn || isImplScript) {
        binding.setProperty(beanName, bean);
    }
    return bean;
}
```

注：当spring容器中的bean初始化完成后，都会执行该方法判断bean对象是否继承了BaseService或者IScript类，如果是则将其设置到Groovy引擎的Binding中，供引擎使用。