---
title: 数据结构
tags: [java]
---

1）数据结构

数据的组织形式，能够提高程序的执行效率。包括逻辑关系和物理关系。逻辑关系指人为的用于分析数据关系的抽象，物理关系表示内存数据的存放位置关系。

逻辑关系：集合（在一个范围内有多个数据，数据之间没有关系）；线性（1对1的关系）；树形（1对多）；图：（多对多）

物理关系（内存存储）：顺序存储（数组）；链式存储（链表）；hash存储；

2）大数运算

使用数组来实现，算法描述

![](/images/algorithm/bigdata.png)

3）排序

a. 冒泡排序

b. 选择排序

c. 插入排序

d. 快速排序

e. 堆排序

4）查找

a. 二分查找

b. 分块查找

c. 散列表

5）常见类的实现

ArrayList和LinkList原理，代码实现，性能区别。

栈和队列的代码实现。

6）递归算法

文件夹遍历，八皇后，汉诺塔，寻址，快速排序

7）树算法

二叉树，堆排序，中序排序，红黑树

8）ArrayList的实现（自定义实现）

```
public class MyArrayList{

    Object[] objs=new Object[4];

    int size=0;

    public int getSize(){
        return size;
    }
    //添加元素
    public void add(Object value){
        //是否能够容纳
        if(size>=objs.length){
            Object[] temp=new Object[objs.length*2];//扩展空间
            for(int i=0;i<objs.length;i++){
                temp[i]=objs[i];
            }
            objs=temp;//重新定位对象
        }
        objs[size]=value;
        size++;
    }
    public void set(int index,Object value){
        //判断下标是否越界
        if(index<0||index>=size){
            throws new Exception("超出范围");
        }
        objs[index]=value;
    }
    public Object get(int index){
        //判断下标是否越界
        if(index<0||index>=size){
            throws new Exception("超出范围");
        }
        return objs[index];
    }
    public void clear(){
        size=0;//只要使用户无法访问到即可
    }
    public void removeAt(int index){
        //判断下标是否越界
        if(index<0||index>=size){
            throws new Exception("超出范围");
        }
        //需要将后面的位置向前移动
        for(int i=index+1;i<size;i++){
            obj[i-1]=obj[i];
        }
        size--;
    }
}
```

注：这里不考虑线程安全问题。