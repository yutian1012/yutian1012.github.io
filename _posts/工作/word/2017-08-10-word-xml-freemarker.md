---
title: word存储成xml--制作freemarker模板文件
tags: [word]
---

### 步骤

1）准备工作

首先以word文档的形式打开，并在想要插入数值的地方使用$代替，将文档另存为xml。

2）使用xmlspay工具打开并编辑文档

该工具能够以缩进等方式对xml文档进行格式化，便于我们查找待替换的值，然后使用freemarker占位符的形式替换之前设置$符号的地方。

3）使用freemarker引擎输出模板

### 涉及到的freemarker语法

1）插入值

```
${wfPatentSupportList[0].patentNo!""}
```

注：从list集合中获取第一个元素，并访问patentNo属性，如果该属性值空，则使用默认值空字符串代替输出。

2）日期类型的值

```
<#if wfPatentSupportList[0].appDate??>
    ${wfPatentSupportList[0].appDate?("yyyy-MM-dd")}
</#if>
```

3）循环变量集合

```
<#if orgList?? && (orgList?size > 0) >
    <#list orgList as org>${org.name}</#list>
</#if>
```

注：判断集合是否为空，可通过size函数判断。

4）集合索引值

```
<#if orgList?? && (orgList?size > 0) >
    <#list orgList as org>
        ${org.name}
        <!--判断是否是第一个元素-->
        <#if org_index==0>...</#if>
        <!--判断是否是最后一个元素-->
        <#if !org_has_next>...</#if>
    </#list>
</#if>
```

注：对象索引信息可通过下划线的方式来获取。

5）金额格式化

```
<#-- 如果小数点后不足两位，用 0 代替 -->
${num?string('0.00')}

<#-- 如果小数点后多余两位，就只保留两位，否则输出实际值 -->
${num?string('#.##')}
输出为：1239765.46

<#-- 整数部分每三位用 , 分割，并且保证小数点后保留两位，不足用 0 代替 -->
${num?string(',###.00')}
输出为：1,239,765.46

<#-- 整数部分每三位用,分割，并且小数点后多余两位就只保留两位，不足两位就取实际位数，可以不不包含小数点 -->
${num?string(',###.##')}
输出为：1,239,765.46

<#-- 整数部分如果不足三位（000），前面用0补齐，否则取实际的整数位 -->
${num?string('000.00')}
输出为：012.70

<#-- 整数取实际的位数 -->
${num?string('###.00')} 等价于 ${num?string('#.00')}
输出为：12.70
```