---
title: freemarker实际使用
tags: [freemarker]
---

1) 输出list集合内容

```
//判断集合不空，并且集合大于0
<#if (xxList??) && (xxList?size >0)>
    <#list xxxList as xxxItem>
        <tr>
            <td title="序号">${xxxItem_index}</td>
            <td title="名称">${xxxItem.name}</td>
            ...
        </tr>
    </#list>
</#if>
```

