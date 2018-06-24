---
title: vue动态添加数据行
tags: [vue]
---

场景：现实与vue实现一个动态添加行数据。系统中有多个部门，每个部门对应不同数量的员工，当选择一个部门后，将该部门的员工信息显示到下方的div中，每行数据一个div，同时针对员工的属性提供相关的文本框来编辑员工信息。

注：员工姓名不能编辑，但员工的家庭住址和电话是可以编辑的。

参考：https://blog.csdn.net/zhuming3834/article/details/70170305

1）创建json数据用来模拟部门员工数据（真实情况是从数据库中查询）

创建department.json文件，文件中包含部门员工数据信息。

```
var departmentJson={
    "developDept":
    [
        {
            "name":"zhangsan1",
            "address":"zhangsan1住址",
            "phone":"zhangsan112345678"
        },
        {
            "name":"lisi1",
            "address":"lisi1住址",
            "phone":"lisi112345678"
        },
        {
            "name":"wangwu1",
            "address":"wangwu1住址",
            "phone":"wangwu112345678"
        }
    ],
    "resoucesDept":
    [
        {
            "name":"zhangsan2",
            "address":"zhangsan2住址",
            "phone":"zhangsan212345678"
        },
        {
            "name":"lisi2",
            "address":"lisi2住址",
            "phone":"lisi212345678"
        },
        {
            "name":"wangwu2",
            "address":"wangwu2住址",
            "phone":"wangwu212345678"
        }
    ]
}
```

2）创建html

通过select选择不同的部门，div中显示不同部门的员工信息。

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <script src="https://cdn.bootcss.com/vue/2.5.16/vue.min.js"></script>
</head>
<body>
    <div id="app">
        <div>
            选择部门：
            <select v-model="department" v-on:change="deptChange">
                <optgroup>
                    <option value="developDept">开发部</option>
                    <option value="resoucesDept">人力资源部</option>
                </optgroup>
            </select>
        </div>
        <div class="line" v-for="(item,index) in dataModel">
            <span>{{ dataModel[index].name }}</span>
            <input type="text" v-model="dataModel[index].address">
            <input type="text" v-model="dataModel[index].phone" />         
        </div>
    </div>
    
</body>
<!-- 引入相关的数据信息 -->
<script type="text/javascript" src="department.json"></script>
<script type="text/javascript">
    var app = new Vue({
        el: "#app",
        data: {
            department : '',//默认值为空
            dataModel: [] // 创建一个空的数组
        },
        methods: {
            deptChange: function (event){
                //首先清空数组
                this.dataModel.splice(0,this.dataModel.length);
                //从数据库中加载department下的员工信息
                for(var i=0;i<departmentJson[this.department].length;i++){
                    this.dataModel.push(departmentJson[this.department][i]);
                }
                //直接赋值的方式无法显示出信息来
                /*dataModel=departmentJson[this.department];//将员工信息绑定到dataModel控件上*/
            }
        }
    })
</script>
</html>
```