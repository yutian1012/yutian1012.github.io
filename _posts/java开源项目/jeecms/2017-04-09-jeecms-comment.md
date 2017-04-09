---
title: 开源的门户网站jeecms--评论
tags: [jeecms]
---

新闻页面的评论功能

1）新闻内容模板页

模板页：WebRoot\WEB-INF\t\cms\www\default\content\news2.html

```
[#if channel.commentControl!=2]
    [#include "inc_comment_input.html"/]
    [#include "inc_comment_list.html"/]
[/#if]
```

channel.commentControl需要在后台新闻栏目中设置是否关闭评论，关闭评论值为2.

![](/images/open/jeecms/jeecms-comment.png)

2）输入评论

```
 [#include "inc_comment_input.html"/]
```

查看模板

文件目录：\WebRoot\WEB-INF\t\cms\www\default\content\inc_comment_input.html

```
<form id="commentForm" action="${base}/comment.jspx" method="post">
    <textarea name="text" placeholder="发表你此时的观点和想法 ……" class="comments-text" id="comments-text"></textarea>
    [#if user??&&user.group.needCaptcha||!user??]
        <div class="plfr1 clearfix">
            <div style="float:left;">
                <input name="captcha" type="text" placeholder="验证码" id="captcha" vld="{required:true}" class="plcode"/>
            </div>
            <div style="float:left;">
                <img id="commentCaptcha" src="${base}/captcha.svl" onclick="this.src='${base}/captcha.svl?d='+new Date()"/>
            </div>
            <div class="down clearfix">
                <input type="hidden" name="contentId" value="${content.id}"/>
                <input type="hidden" name="sessionId" id="sessionId" />
                <input type="submit" value="提交评论" class="submit-on">
            </div>
        </div>
    [/#if]
</form>
```

3）提交评论功能

提交地址：/jeecms/comment.jspx

处理类，jeecms-servlet-front-action.xml文件中定义

```
<bean id="commentAct" class="com.jeecms.cms.action.front.CommentAct"/>
```

处理方法

```
@RequestMapping(value = "/comment*.jspx", method = RequestMethod.GET)
 public String page(Integer contentId, Integer pageNo,
        HttpServletRequest request, HttpServletResponse response,
        ModelMap model) {
    CmsSite site = CmsUtils.getSite(request);
    Content content = contentMng.findById(contentId);
    if (content == null) {
        return FrontUtils.showMessage(request, model,
            "comment.contentNotFound");
    }
    if (content.getChannel().getCommentControl() == ChannelExt.COMMENT_OFF) {
        return FrontUtils.showMessage(request, model, "comment.closed");
    }
    // 将request中所有参数保存至model中。
    model.putAll(RequestUtils.getQueryParams(request));
    FrontUtils.frontData(request, model, site);
    FrontUtils.frontPageData(request, model);
    model.addAttribute("content", content);
    return FrontUtils.getTplPath(request, site.getSolutionPath(),
            TPLDIR_SPECIAL, COMMENT_PAGE);
}
```

注：提交评论后，会将数据存放到jc_comment表中，并在后台通过审核后才能查看到。

注2：审核评论，维护--互动模块--评论管理，审核后才能查看。