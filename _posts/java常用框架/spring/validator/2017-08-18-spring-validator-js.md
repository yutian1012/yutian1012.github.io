---
title: spring的注解方式的校验生成前端js校验
tags: [spring]
---

系统中的校验信息有时即需要在前端配置，又需要在后台进行单独配置。spring结合hibernate的validate能够使用注解进行校验的配置。如果能够根据后端配置的校验注解信息自动生成相应的js校验语法，那么就实现了一致的校验方式。

参考：http://www.cnblogs.com/liukemng/p/4618851.html