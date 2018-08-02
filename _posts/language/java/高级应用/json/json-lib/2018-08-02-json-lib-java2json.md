---
title: json-lib处理java对象转json日期字段处理
tags: [json]
---

参考：https://www.cnblogs.com/natsu72/p/7809893.html

1）代码

```
public class DateJsonValueProcessor implements JsonValueProcessor {
    
    private static Map<String,String> datePattern=new HashMap<String, String>();
    
    static{
        datePattern.put("\\d{2}/\\d{2}/\\d{2} \\d{2}:\\d{2}:\\d{2}", "yyyy/MM/dd HH:mm:ss");
        datePattern.put("\\d{8}", "yyyyMMdd");
    }
    
    @Override
    public Object processArrayValue(Object arg0, JsonConfig arg1) {
        return null;
    }
    /**
    * Processes the value an returns a suitable JSON value.
    *
    * @param key the name of the property
    * @param value the value of the property
    * @return a valid JSON value that represents the input property
    * @throws JSONException if an error occurs during transformation
    */
    @Override
    public Object processObjectValue(String arg0, Object value, JsonConfig arg2) {
        if (value instanceof Date) {
            Pattern pattern=null;
            Iterator<String> p=datePattern.keySet().iterator();
            while(p.hasNext()){
                pattern=Pattern.compile(p.next());
                if(pattern.matcher((String)value).matches()){
                    return DateFormatUtil.format((Date)value,datePattern.get(p));
                }
            }
        }
        return null;
    }

}
```

2）配置到JsonConfig中生效

