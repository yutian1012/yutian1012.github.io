---
title: github开源项目云收藏（javascript）
tags: [project]
---

1）连接页面的跳转

如查看left.html页面定义的功能列表

```
<li class="" id="home">
  <a title="首页"  href="javascript:void(0);"  
    onclick="locationUrl('/standard/my/0','home')">
     <em class="icon-home"></em>
     <span>首页</span>
  </a>
</li>
```

注：定义了连接的onclick事件，locationUrl方法的第一个参数为跳转访问的连接，第二个参数表示当前激活的li标签，主要是为了应用激活样式。

查看locationUrl代码，该代码定义在comm.js文件中

```
//locationUrl主要是调用goUrl方法
var xmlhttp = new getXMLObject();
function goUrl(url,params) {
    fixUrl(url,params);
    if(xmlhttp) {
        xmlhttp.open("POST",url,true);
        //配置方法的回调函数
        xmlhttp.onreadystatechange = handleServerResponse;
        xmlhttp.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded;charset=UTF-8');
        xmlhttp.send(params);
    }
}
//获取XMLHttpRequest实现js与后台数据交换
function getXMLObject() {
    var xmlHttp = false;
    try {
        xmlHttp = new ActiveXObject("Msxml2.XMLHTTP") // For Old Microsoft
                                                        // Browsers
    } catch (e) {
        try {
            xmlHttp = new ActiveXObject("Microsoft.XMLHTTP") // For Microsoft
                                                                // IE 6.0+
        } catch (e2) {
            xmlHttp = false // No Browser accepts the XMLHTTP Object then false
        }
    }
    if (!xmlHttp && typeof XMLHttpRequest != 'undefined') {
        xmlHttp = new XMLHttpRequest(); // For Mozilla, Opera Browsers
    }
    return xmlHttp; // Mandatory Statement returning the ajax object created
}
//查看设置的回调，如果返回的数据不是错误页面，则直接显示到content页面中
function handleServerResponse() {
    if (xmlhttp.readyState == 4) {
        //document.getElementById("mainSection").innerHTML =xmlhttp.responseText;
        var text = xmlhttp.responseText;
        if(text.indexOf("<title>Favorites error Page</title>") >= 0){
            window.location.href="/error.html";
        }else{
            $("#content").html(xmlhttp.responseText);
        }
    }
}
```

注：连接跳转方式，直接将从后台获取的html页面，设置到content控件中显示。

2）页面中保存修改等提交操作

这里查看收藏夹的创建过程来查看其操作过程，查看newfavorites.html页面，可以发现数据的提交是使用ajax方式提交到后台的。并从后台接受Response的json数据，从而判断操作是否成功。

a.页面表单

```
<section>
 <div class="content-wrapper">
    <h3>新建收藏夹</h3>
    <div class="row">
       <div class="col-md-3">
          <form id="form" data-parsley-validate="true" onsubmit="return false">
             <div class="form-group">
                <input id="name" type="text" class="form-control" placeholder="收藏夹名" data-parsley-error-message="请输入收藏夹名" required="required"/>
             </div>
             <button id="addFavorites" class="btn btn-block btn-primary mt-lg">提交</button>
          </form>
       </div>
    </div>
 </div>
</section>
```

b.绑定提交按钮，ajax提交数据到后台，使用toastr弹框提示操作结果

```
<script type='text/javascript'>
 $(function(){
     toastr.options = {
        'closeButton': true,
        'positionClass': 'toast-top-center',
        'timeOut': '5000',
     };
     //ajax方式绑定表单的提交事件
     $("#addFavorites").click(function(){
         var ok = $('#form').parsley().isValid({force: true});
         if(ok){
             $.ajax({
                    async: false,
                    type: 'POST',
                    dataType: 'json',
                    data:'name=' + $("#name").val(),
                    url: '/favorites/add',
                    error : function(XMLHttpRequest, textStatus, errorThrown) {
                        console.log(XMLHttpRequest);
                        console.log(textStatus);
                        console.log(errorThrown);
                    },
                    success: function(response){
                        if(response.rspCode == '000000'){
                             loadFavorites();
                             toastr.success('收藏夹创建成功！', '操作成功');
                        }else{
                            toastr.error(response.rspMsg, '操作失败');
                     }
                    }
                });
         }
     });
 });
 </script>
```

3）使用vue实现数据提交

查看login.html用户登录页面实现代码，该页面提交表单使用的是vue框架实现的。

a.提交表单代码

```
<div class="panel-body" id="loginPage">
   <p class="text-center pv">请登录</p>
   <form id="form" data-parsley-validate="true" onsubmit="return false">

      <div class="form-group has-feedback">
         <input type="text" placeholder="邮箱地址或登录名称" v-model="username" class="form-control" data-parsley-error-message="请输入邮箱地址或登录名称" required="required" />
         <span class="fa fa-envelope form-control-feedback text-muted"></span>
      </div>

      <div class="form-group has-feedback">
         <input type="password" placeholder="密码" v-model="password" class="form-control" data-parsley-error-message="请输入密码" required="required" />
         <span class="fa fa-lock form-control-feedback text-muted"></span>
      </div>

      <div class="clearfix">
         <div class="pull-right">
            <a href="/forgotPassword" class="text-muted">忘记密码？</a>
         </div>
      </div>

      <button v-on:click="login" class="btn btn-block btn-primary mt-lg">登录</button>
   </form>
</div>
```

b.js代码实现数据提交

```
<script th:src="@{/webjars/jquery/2.0.0/jquery.min.js}"></script>
<script th:src="@{/webjars/vue/1.0.24/vue.min.js}"></script>  
<script th:src="@{/webjars/vue-resource/0.7.0/dist/vue-resource.min.js}"></script>
<script th:src="@{/vendor/parsleyjs/dist/parsley.min.js}"></script> 
<script type='text/javascript'>
Vue.http.options.emulateJSON = true; 
var loginPage = new Vue({//绑定页面div的id为loginPage元素
    el: '#loginPage',
    data: { //定义变量与页面的input域双向绑定
        'username': '',
        'password': ''
    },
    methods: {
        login: function(event){ //定义登录按钮事件，页面使用v-on绑定了click事件
            //使用parsley插件进行表单校验
            var ok = $('#form').parsley().isValid({force: true});
            if(!ok){
                return;
            }
            var datas={
                //封装待提交数据，页面控件的值与vue变量进行了双向绑定，
                //因此，this.username的值就是页面控件输入的值
                     userName: this.username,
                     passWord: this.password
                     };
            //请求登录操作
            this.$http.post('/user/login',datas).then(function(response){
                //后端定义response对象并返回json数据
                 if(response.data.rspCode == '000000'){
                        //返回跳转连接，这里登录后跳转到用户首页
                         window.open(response.data.url, '_self');
                 }else{
                      $("#errorMsg").html(response.data.rspMsg);
                      $("#errorMsg").show();
                 }
             }, function(response){
                 console.log('error');
             })
        }
    }
})
</script>
```