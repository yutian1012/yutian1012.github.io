---
title: github开源项目云收藏（左侧菜单）
tags: [project]
---

通过查看left.html页面，可以看到该页面中使用了${user}来获取登录用户的信息。如果在home页面不传递用户信息，是无法正确的展示该页面的。

1）配置左侧树信息

修改home页面的处理类，修改IndexController类的home方法，跳转home页面时，将当前用户传递到model的属性中，页面就可以获取相关的信息。

```
@RequestMapping(value="/",method=RequestMethod.GET)
public String home(Model model) {
    model.addAttribute("user",getUser());
    return "home";
}
```

注：传递user对象后，left.html就可以正常显示了。

1）左侧树连接

a.首页（home页面）

当前登录用户的home页面，首页内容会显示自己的所有收藏信息

b.未读列表(collect对象)

属于用户的一个特殊的收藏夹

c.收藏夹（favorites）

用户创建的收藏夹，存储在favorites表中

d.发现

显示系统中用户的最近收藏信息。

e.导入收藏夹

以火狐为例，通过浏览器菜单：书签--管理所有书签--导入和备份--导出书签到html，将浏览器的收藏信息导出成html文件，然后再倒入到系统中，系统会自动生成一个收藏夹，收藏夹的名称为导入自浏览器。

f.导出收藏夹

可以选择用户的相关收藏夹进行导出，可以将导出的html文件再导入到浏览器的收藏夹中。