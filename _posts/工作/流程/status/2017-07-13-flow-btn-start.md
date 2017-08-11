---
title: 流程中的启动按钮处理过程
tags: [bpmx]
---

流程第二个节点中点击同意按钮的处理过程。页面taskToStart.jsp

1）页面初始化操作

jsp页面会引入按钮所在的jsp（incToolBarNode.jsp）

```
<jsp:include page="incToolBarNode.jsp"></jsp:include>
```

在js中会调用initForm方法，该方法又会调用initBtnEvent()方法来初始化按钮。

2）incToolBarNode.jsp页面的同意按钮

```
<div class="group"><a id="btnAgree" class="link agree"><span></span>同意</a></div>
```

3）查看按钮初始化方法initBtnEvent方法

```
function initBtnEvent(){
    //0，弃权，1，同意，2反对。
    var objVoteAgree=$("#voteAgree");
    var objBack=$("#back");
    ....
    //反对
    if($("#btnNotAgree").length>0){
        $("#btnNotAgree").click(function(){
            setExtFormData();
            var isDirectComlete = getIsDirectComplete();
            operatorType=2;
            ////直接一票通过
            var tmp =isDirectComlete?'6':'2';
            objVoteAgree.val(tmp);
            objBack.val("0");
            completeTaskBefore();
        });
    }
}
```

注：这里可以看到，不同的按钮会对voteAgree设置不同的值。

4）ProcessRunService类getActionByCmdVoteAgree方法

```
private String getActionByCmdVoteAgree(Short sot) {
    if (sot == 1 || sot == 5)
        return BpmNodeSql.ACTION_AGREE;
    if (sot == 2 || sot == 6)
        return BpmNodeSql.ACTION_OPPOSITE;
    if (sot == 3)
        return BpmNodeSql.ACTION_REJECT;
    return null;

}
```