---
title: word存储成xml后的结构信息--页面设置
tags: [word]
---

参考：http://blog.csdn.net/u011580143/article/details/49835957

1）页面标签

标签：w:body，设置页面信息

```
<#-- 设置了页的宽，高，和页的各边距。各项的值均是英寸乘1440得出 -->
<w:body> 
    <#-- 页面区域的属性 -->
    <w:sectPr>
        <#-- 页面尺寸设置 -->
        <w:pgSz w:w="12240" w:h="15840"/>
        <#-- 页面margin设置 -->
        <w:pgMar w:top="1440" w:right="1800" w:bottom="1440" w:left="1800" w:header="720" w:footer="720" w:gutter="0"/>
    </w:sectPr>
</w:body> 
```

2）页面属性

标签：w:sectPr，section property页面区域的属性

标签：w:pgSz，page size页面尺寸

标签：w:pgMar，page margin表示页面边框。

3）文档属性

标签：w:docPr，document property文档属性设置

```
<w:docPr>
    <w:view w:val="print"/>
    <#-- 视图比例 -->
    <w:zoom w:percent="100"/>
</w:docPr>
```