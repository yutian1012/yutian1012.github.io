---
title: eclipse常用快捷键
tags: [eclipse,tools]
---

参考：https://www.cnblogs.com/iamfy/archive/2012/07/11/2586869.html

### 移动相关

1）代码块移动

上下移动（先选中代码块）

```
alt+上/下的方向键
```

左右移动（先选中代码块）

```
tab--向右移动
shift+tab--向左移动
```

2）返回上一次光标所在位置处

```
alt + 左右方向键
```

注：实现方法间的来回跳转，可以跨类文件跳转。

3）标签页切换

```
ctrl + shift +F6
```

### 查找类信息

1）查找类（源码中查找类）

```
shift+ctrl+r
```

注：在弹出的对话框中输入待查找到类。

![](/images/tools/eclipse/findresource.png)

2）查找类（包括jar包中的class文件）

```
shift+ctrl+t
```

注：适合操作jar包中的类信息

3）查找类中方法的快捷键

```
ctrl + o //快速outline
```

4）查看方法的子类实现

将光标定位到要查看的方法上（如一个抽象方法），查看实现类的方法实现

```
ctrl + T //查看方法的具体实现，tree继承关系
```

### 代码操作相关

1）快速生成Local变量（局部变量）

定位到需要生成变量的行，按住ctrl+2，出现选择提示框后，按L键

```
ctrl+2,L //按住ctrl+2，出现选择提示框后，按L键
```

### 运行代码相关

1）运行main方法

```
ctrl + F11
```