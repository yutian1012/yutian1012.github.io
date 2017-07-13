---
title: 流程中对groovy脚本的封装
tags: [bpmx]
---

### 定位使用

在BpmNodeUserCalculationScript类中

GroovyScriptEngine类实现了BeanPostProcessor接口，会在容器启动时将当前项目中的所有的service类和实现了IScript接口的类注册到脚本引擎中，在编写脚本时可以直接使用。