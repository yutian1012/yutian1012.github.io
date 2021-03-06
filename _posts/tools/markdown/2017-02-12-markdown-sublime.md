---
title: 使用sublime编辑markdown
tags: [markdown]
---

Markdown is a markup language with a syntax closer to plain text.

### markdown语法

GFM 是 Github 拓展的基于 Markdown 的一种纯文本的书写格式,在 Markdown 中，连续的一行或多行就是一个段落。用空行来进行切段。

参考：http://www.cnblogs.com/hnrainll/p/3514637.html

### markdown编辑器，实现所见即所得。
#### 1. 常用软件：
Windows: MarkdownPad

Linux: ReText

sublime+markdown插件

#### 2. 使用sublime安装markdown插件
1）首先要先安装package   

control--sublime插件安装工具，然后利用该工具安装markdown相关的插件。

使用快捷键ctrl+shift+p打开package安装工具

![package control](/images/tools/markdown/package_control.png)

2）安装插件：在install package中输入markediting,markdown preview两个插件

安装插件1：输入markediting并回车，在sublime下方的状态栏中显示下载安装该插件

安装插件2：输入markdown preview并回车，在sublime下方的状态栏中显示下载安装该插件

注：插件安装过程。菜单：preferences--package control,输入install package--即选择安装插件命令，然后输入待安装的插件名，回车安装

安装完成后会在安装目录下存在相应的包文件

![](/images/tools/markdown/installedpackage.png)

参考：http://www.cnblogs.com/IPrograming/p/Sublime-markdown-editor.html 

参考2：http://jingyan.baidu.com/article/f006222838bac2fbd2f0c87d.html

3）重启sublime,查看是否插件安装成功

preferences--package control输入 list packages，查看已安装的插件

4）安装问题：sublime的package control无法安装插件

错误信息：Error parsing JSON from channel https://packagecontrol.io/channel_v3.json.

解决方法：打开底端的console（快捷键ctrl+~，或菜单栏-view--show console打开控制台），访问 https://packagecontrol.io/installation，在console中重新运行了一遍安装package control的代码。

注1：Package Control安装，如果Sublime Text 3还没有安装Package Control，请参考这里：([Package Control Installation](https://packagecontrol.io/installation)。)

注2：markdown编辑器插件安装。点击 Preferences --> 选择 Package Control: intall package，然后再插件库中分别选择和安装Markdown Editing和Markdown Preview即可。然后重启Sublime Text。

### 编辑并测试markdown
在sublime中使用快捷键方式创建并编辑文档

1）ctrl+N--新建文档，此时sublime并没有使用md的语法，ctrl+s保存文档，将文档存储为markdown文档（后缀名.md），sublime会立即使用markdown相应的语法来显示文档。

2）编辑并预览，sublime提供的markdown预览时在浏览器中显示的。

ctrl+shift+p调出package control命令框，输入markdown preview,选择preview in browser命令，即会在浏览器中显示。

注：在我的系统中没有直接在浏览器中显示，而是打开了一个txt编辑器，显示的是对应的html文本内容---原因是系统中设置的html文件的打开方式是使用txt编辑器。修改html文件默认打开方式为浏览器，即可直接在浏览器中查看。sublime是借助系统中html文档的打开方式来显示html内容的，将html文件的打开方式设置为浏览器，即可使用浏览器打开。

3）新建快捷键，在编辑完成.md文件后，直接使用快捷键在浏览器中预览

菜单：preferences--key bindings,会打开新的sublime窗口，左侧为：Default (windows).sublime-keymap-Default，右侧为：Default (windows).sublime-keymap-User
在右侧窗格中设置用户的快捷键：输入{"keys": ["alt+m"], "command": "markdown_preview", "args": { "target": "browser"} }（这是一个json类型的数组）

![](/images/tools/markdown/keymap.png)

4）设置完成快捷键后，保存完毕文档后，按alt+m即可立即在浏览器中显示。

注：创建一个以md为后缀的文件，使用sublime打开，既可以开始编辑Markdown文件了。