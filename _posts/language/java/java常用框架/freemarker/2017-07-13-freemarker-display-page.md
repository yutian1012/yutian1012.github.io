---
title: freemarker回显页面
tags: [freemarker]
---

1）普通字段的回显

```
<input type="text" name="identityCard" value="${identityCard!}" style="width:70px"/>
```

注：将使用数据模型中的值替代输出

2）select数据回显

```
<select name="orderField">
    <option value="username" <#if (((orderField)!'') == 'username')>selected</#if>>用户名</option>
</select>
```

注：需要先进行空值的处理，然后利用if条件判断是否select的选项是否选中

3）checkbox数据回显

```
<input type='checkbox' name='asc' value='1' <#if (((asc)!'') == '1')>checked</#if>/>
```

注：处理过程与select的处理方式相同