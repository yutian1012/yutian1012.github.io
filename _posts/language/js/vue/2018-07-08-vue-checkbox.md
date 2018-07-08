---
title: 获取复选框的值
tags: [vue]
---

1）实例代码

```
<tr v-for="(batch,index) in batchList" v-cloak>
    <th scope="row" >
        <input type="checkbox" name="batchId" :value="batch.id" v-model="batchId" class="form-control"></th>
</tr>
```

注1：获取for循环变量的值使用:value，而不能使用value="{{batch.id}}"

注2：将checkbox绑定到了batchId变量上，当checkbox选择，会向该数值中添加value值

2）获取value数组值

```
var app = new Vue({
  el: '#app',
  data:{
    batchList:[],//数据集合
    batchId:[]//记录选中的批次
  },
  methods:{
    listen:function(page){
        this.currentpage=page;
        loadData(page,this.pageSize);
    },
    test:function(event){//执行选中批次
        if(this.batchId.length==0){//获取复选框选中的值
            toastr.error('未选择xxx！', '操作失败');
            return false;
        }
        ...
    }
  }
});
```