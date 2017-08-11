---
title: word存储成xml后的结构信息--段落和字体设置
tags: [word]
---

参考：http://blog.csdn.net/u011580143/article/details/49835957

### 段落字体

1）设置内容

标签：w:t，t表示text，该标签一般使用在w:r标签内

```
<#-- 文字内容 -->
<w:t>这是文字</w:t>

<#-- xml:space="preserve"从字面上理解是保持空格 -->
<w:t xml:space="preserve">       </w:t>
```

2）文本样式

标签：w:r，表示一个样式串，指明它包括的文本的显示样式，如文本属性加粗、下划线、倾斜的分割，内含w:rsidRPr属性。

标签：w:rpr，w:r标签内的标签，对文本属性进行修饰

```
<#-- 下面是一段粗体 --> 
<w:r>
    <w:rPr> 
        <#-- 字体信息 -->
        <w:rFonts w:ascii="楷体_GB2312"/>
        <#-- 字体大小 -->
        <w:szCs w:val="21"/>
        <#-- 粗体 -->
        <w:b w:val="on"/>
    </w:rPr> 
    <w:t> 这是粗体</w:t>
</w:r>
```

注：其中w:b表示该格式串种的文本为粗体

3）段落

标签：w:p，表示段落和html中的p标签类似，一般是w:r的外层。

```
<w:p>
    <w:pPr>
        <w:rPr>
            <#-- 这句话表示段落对齐方式 -->
            <w:jc w:val="center"/>   
             <#-- 设置行距，要进行运算，要用数字除以240，如此处为600/240=2.5倍行距 -->             
            <w:spacing  w:line="600" w:lineRule="auto"/> 
        </w:rPr>
    </w:pPr>

    <w:r w:rsidRPr="00F2468A">
        <w:rPr>
            <#-- 粗体 -->
            <w:b w:val="on"/>
            <w:sz w:val="40"/>
            <w:rFronts w:ascii="宋体" w:hAnsi="宋体" w:hint="esatAsia"/>
        </w:rPr>
        
        <#-- ${wordtext1}是freemarker变量，一般是字符串，用于输出，这个可以参考freemarker文档 -->
        <w:t>${wordtext1}</w:t>
    </w:r>
</w:p> 
```

注：标签w:pPr用来设置段落样式（段落属性），pr是Property的缩写。文本格式使用w:rPr标签来设置。