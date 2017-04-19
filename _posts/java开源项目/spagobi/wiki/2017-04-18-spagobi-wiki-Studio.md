---
title: 开源spagobi--SpagoBI Studio设计器
tags: [spagobi]
---

SpagoBI Studio是基于Eclipse的开发环境。它允许开发人员设计和修改所有的分析文档，如报表，OLAP，仪表板，QBE和数据挖掘。该模块支持开发者设计、测试并直接部署分析文档到一个或多个 SpagoBI 服务器。这两个组件之间的交互可能要归功于SpagoBI SDK模块。

参考：http://wiki.spagobi.org/xwiki/bin/view/spagobi_studio/

### 1. settings 设置

1）运行：

解压压缩包后，运行SpagoBIStudio_5.2.0_win64目录下的SpagoBI.exe执行文件。

2）配置服务器连接（连接到spagobi-server服务器）

新建一个spagobi项目（工具栏中spagobi图表-new project），然后在项目的Resources目录下新建一个Server，对server进行本地服务连接配置。

![](/images/open/spagobi/wiki/connectionstudioserver.png)

3）下载template

![](/images/open/spagobi/wiki/studioDocuemntDownload.png)

注：从server上获取的文档模型位于功能文档树中。

4）部署template

新建一个xml文件，编辑文件内容，右击该文件--选择deploy

![](/images/open/spagobi/wiki/studioDocuemntDeploy.png)

注：会将文件部署显示到功能文档树中（server中查看）。

### 2. Geo Designer

使用Geo Designer插件来创建Geo模板文件。