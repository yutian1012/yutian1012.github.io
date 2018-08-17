---
title: mysql统计使用case
tags: [database]
---

参考：http://www.cnblogs.com/printN/p/6724732.html

1）将一个字段的统计放置在一行中

需求：统计不同公司申请的专利总量，及各个专利类型的量。

![](/images/database/mysql/statistic/data1.png)

```
select applyer,count(0) total,
   sum(case when S.patentType = '发明专利' then 1 else 0 end) as 'FM',
   sum(case when S.patentType = '实用新型'then 1 else 0 end) as 'SY',
   sum(case when S.patentType = '外观设计'then 1 else 0 end) as 'WG'
   
 from z_patent S where S.authdate > '2016-01-01' and S.authdate < '2017-01-01' 
 GROUP BY S.applyer  order by total DESC
```

注：专利类型分为发明专利，实用新型，外观设计

2）如果有多个条件属于同一类

```
select year(AppDate) year,
    SUM(case WHEN apptype='fmzl' then 1 ELSE 0 END) as '发明',
    SUM(case WHEN apptype='syxx' then 1 ELSE 0 END) as '实用新型',
    SUM(case WHEN apptype='wgzl' then 1 ELSE 0 END) as '外观专利',
    SUM(case WHEN apptype='pct'  then 1 WHEN apptype='blgy' then 1 ELSE  0 END) as '国外专利',
    count(1) as '总量'
from t_patent_app 
where AuthDate IS NULL and AppType is not null and appdate is not NULL 
group by year(AppDate)
ORDER BY AppDate DESC
```

注：case可接多个when条件。