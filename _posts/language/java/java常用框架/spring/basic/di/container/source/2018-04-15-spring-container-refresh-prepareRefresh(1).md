---
title: spring容器prepareRefresh方法
tags: [spring]
---

该方法是refresh方法中的一个环节，用于初始化前的准备工作，例如对系统属性或者环境进行准备及验证。

1）该方法位于AbstractApplicantContext类中

```
protected void prepareRefresh() {
    ....

    // Initialize any placeholder property sources in the context environment
    //初始化资源占位符需要的资源文件信息，
    initPropertySources();

    // Validate that all properties marked as required are resolvable
    // see ConfigurablePropertyResolver#setRequiredProperties
    //从环境变量中判断验证必填属性
    getEnvironment().validateRequiredProperties();
}
```