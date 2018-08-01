---
title: json-lib处理字段属性
tags: [json]
---

1）问题引入

最近项目中使用json-lib对json字符串进行序列化及反序列化，在开发的时候，经常会用到一个方法toBean方法，该方法会很容易地将Json字符串转换为Java实体类。前提条件是，json字符串的key值与Java实体类的属性必须一致。

```
JsonObject.toBean(jsonObject, Object.class)
```

很多情况是不需要进行字段设置。如json中有一个字段name，而实体类中并不存在。如果直接转换会抛出异常。

```
net.sf.json.JSONException: java.lang.NoSuchMethodException: Unknown property 'name' on class 'class ...'
```

2）解决方法

通过JsonConfig类设置属性赋值策略，利用spring提供的ReflectionUtils实现反射字段是赋值

```
import org.springframework.util.ReflectionUtils;

/**
 * 获取商标详细信息
 * @param json
 * @return
 */
public static TradeMarkDetailModel getTradeMarkDetail(JSONObject json){
    if(null!=json){
        if("000000".equals(json.getString("errorCode"))){
            JSONObject data=json.getJSONObject("context");
            if(null!=data){
                
                JSONUtils.getMorpherRegistry().registerMorpher(new DateMorpher((new String[] { "yyyyMMdd" })));
                Map<String,  Class<?>> map=new HashMap<String,  Class<?>>();
                map.put("tradeMark", TradeMarkModel.class);
                map.put("applicants", TradeMarkApplicantModel.class);
                JsonConfig config=new JsonConfig();
                config.setPropertySetStrategy(new PropertySetStrategy() {
                    @Override
                    public void setProperty(Object obj, String field, Object value)
                            throws JSONException {
                        Field f=ReflectionUtils.findField(obj.getClass(), field);
                        if(null!=f){
                            f.setAccessible(true);
                            ReflectionUtils.setField(f, obj, value);
                        }
                    }
                });
                config.setRootClass(TradeMarkDetailModel.class);
                config.setClassMap(map);
                
                JSONArray array=data.getJSONArray("records");
                if(null!=array&&array.size()>0){
                    JSONObject model=array.getJSONObject(0);
                    /*return (TradeMarkDetailModel) JSONObject.toBean(model,TradeMarkDetailModel.class,map);*/
                    
                    return (TradeMarkDetailModel) JSONObject.toBean(model,config);
                }
            }
        }
    }
    return null;
}
```
