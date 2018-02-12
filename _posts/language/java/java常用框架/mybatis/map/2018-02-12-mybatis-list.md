---
title: mybatis集合
tags: [mybatis]
---

1）map文件中判断集合是否空

```
<if test="list !=null and list.size()>0">
    <foreach collection="list" item="item" separator="," open="(" close=")">
        #{item}
    </foreach>
</if>
```

2）返回主键或某个字段的list集合

```
<select id="getAppNoList" resultType="java.lang.String">
    SELECT DISTINCT APPNUMBER FROM Z_PATENT
</select>
```

3）返回map对象

```
# 处理方法
public Map<String,Date> getFeeMap(){
    final Map<String,Date> result=new HashMap<String, Date>();
    //获取sqlSessionTemplate对象，是spring整合mybatis提供的对象
    getSqlSessionTemplate().select("getFeeMap", new ResultHandler() {
        @Override
        public void handleResult(ResultContext ResultContext) {
            if(ResultContext.getResultCount()>0){
                Map map = (Map) ResultContext.getResultObject();
                result.put((String)map.get("APPNO"),(Date) map.get("PAYDATE"));
            }
        }
    });
    return result;
}

# sql查询语句
<select id="getFeeMap" resultType="java.util.Map">
    select APP_NUMBER AS APPNO,MAX(PAY_DATE) AS PAYDATE 
    FROM T_IMPORT_FEE GROUP BY APP_NUMBER
</select>
```