---
title: word存储成xml后的结构信息--表格
tags: [word]
---

### 表格标签

标签：w:tbl，定义表格。

标签：w:tblPr，定义表格属性

标签：w:tblGrid，定义表格的单元格信息

```
<w:tbl>
    <#-- 表格属性 -->
    <w:tblPr>
        <w:tblW w:w="0" w:type="auto"/>
        <w:tblInd w:w="737" w:type="dxa"/>
        <#-- 表格边框 -->
        <w:tblBorders>
            <w:top w:val="single" w:sz="4" w:space="0" w:color="auto"/>
            <w:left w:val="single" w:sz="4" w:space="0" w:color="auto"/>
            <w:bottom w:val="single" w:sz="4" w:space="0" w:color="auto"/>
            <w:right w:val="single" w:sz="4" w:space="0" w:color="auto"/>
            <w:insideH w:val="single" w:sz="4" w:space="0" w:color="auto"/>
            <w:insideV w:val="single" w:sz="4" w:space="0" w:color="auto"/>
        </w:tblBorders>
        <w:tblLayout w:type="fixed"/>
        <w:tblLook w:val="04A0"/>
    </w:tblPr>

    <#--表单元格 -->
    <w:tblGrid>
        <w:gridCol w:w="2635"/>
        <w:gridCol w:w="3372"/>
        <w:gridCol w:w="2074"/>
    </w:tblGrid>

    <#-- 表格行属性设置 -->
    <w:tr w:rsidR="00F2468A" w:rsidRPr="00F2468A" w:rsidTr="00D074CF">
        <#-- 设置行高属性 -->
        <w:trPr>
            <w:trHeight w:val="566"/>
        </w:trPr>
        <#-- 设置表格列 -->
        <w:tc>
            <w:tcPr>
                <w:tcW w:w="2000" w:type="dxa"/>
            </w:tcPr>

            <w:p w:rsidR="00F2468A" w:rsidRPr="00F2468A" w:rsidRDefault="00F2468A" w:rsidP="00E9655B">

                <#-- 表格中段落属性 -->
                <w:pPr>
                    <w:jc w:val="center"/>
                    <w:rPr>
                        <w:rFonts w:ascii="宋体" w:hAnsi="宋体"/>
                        <w:szCs w:val="21"/>
                    </w:rPr>
                </w:pPr>

                <#-- 表格列内容属性 -->
                <w:r w:rsidRPr="00F2468A">
                    <w:rPr>
                        <w:rFonts w:ascii="宋体" w:hAnsi="宋体"/>
                        <w:szCs w:val="21"/>
                    </w:rPr>

                    <w:t>第一列</w:t>
                </w:r>
            </w:p>
        </w:tc>

        <#-- 其他列... -->
    </w:tr>
</w:tbl>
```