---
title: 查询表单的回车处理
tags: [bpmx]
---

问题描述：在查询表单中输入文字后，回车会将日期字段设置值，造成查询结果与期望不符。

1）页面日期字段：

![](/images/work/bpmx/js/searchformdate.png)

```
<div>
<span class="label">申请时间:</span>  
<input type="text" id="Q_beginapplyDate_DL" name="Q_beginapplyDate_DL" class="inputText Wdate" onfocus="WdatePicker()" value="${param['Q_beginapplyDate_DL']}" />
<input type="text" id="Q_endapplyDate_DG" name="Q_endapplyDate_DG" class="inputText Wdate" onfocus="WdatePicker()" value="${param['Q_endapplyDate_DG']}" />

<a class="link search"  onclick="WeekBt('Q_beginapplyDate_DL','Q_endapplyDate_DG')">本周</a>&nbsp;&nbsp;
<a class="link search"  onclick="MonthBt('Q_beginapplyDate_DL','Q_endapplyDate_DG')">本月</a>&nbsp;&nbsp;
<a class="link search"  onclick="QuarterBt('Q_beginapplyDate_DL','Q_endapplyDate_DG')">本季</a>&nbsp;&nbsp;
<a class="link search"  onclick="YearBt('Q_beginapplyDate_DL','Q_endapplyDate_DG')">本年 </a>&nbsp;&nbsp;
<a class="link del" onclick="clearDate('Q_beginapplyDate_DL','Q_endapplyDate_DG')">清空</a>
</div>
```

2）js/hotent/displaytag.js文件

```
/**
 * 处理回车查询
 */
function handleSearchKeyPress(){
    $(".panel-search :input").keypress(function(e) {
        if( e.keyCode == 13){//回车
            $("a.link.search").click();
        }else if(e.keyCode == 27){//ESC
            var searchForm = $("#searchForm");
            if(searchForm)
                searchForm[0].reset();
        }       
    })
}
```

注：可以看到日期空间中选择本周、本年等字段符合jquery的筛选条件，因此自动触发了click事件。

3）解决方法：修改js，修改筛选条件为查询按钮

```
function handleSearchKeyPress(){
    $(".panel-search :input").keypress(function(e) {
        if( e.keyCode == 13){//回车
            //要求所有页面的查询按钮的id必须为btnSearch
            $("#btnSearch").click();
        }else if(e.keyCode == 27){//ESC
            var searchForm = $("#searchForm");
            if(searchForm)
                searchForm[0].reset();
        }       
    })
}
```