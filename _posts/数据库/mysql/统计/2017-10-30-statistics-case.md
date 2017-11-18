---
title: mysql统计使用case
tags: [database]
---

参考：http://www.cnblogs.com/printN/p/6724732.html

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