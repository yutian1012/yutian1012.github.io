---
title: 持续集成jenkins的名词词汇
tags: [jenkins]
---

1）pipeline

中文： 管道; 输油管道; 渠道，传递途径。

Pipelines are made up of multiple steps that allow you to build, test and deploy applications. Jenkins Pipeline allows you to compose multiple steps in an easy way that can help you model any sort of automation process.

Pipeline，简单来说，就是一套运行于Jenkins上的工作流框架，将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂发布流程。

注：目的是将节点任务连接起来，自动的向下执行。

2）Pipeline as Code

Pipeline as Code describes a set of features that allow Jenkins users to define pipelined job processes with code, stored and versioned in a source repository. 

To use Pipeline as Code, projects must contain a file named Jenkinsfile in the repository root, which contains a "Pipeline script."

通过一个代码文件来预定义的执行jenkins的工作流。需要在版本库的根路径下定义一个Jenkinsfile文件，该文件定义了Pipeline执行脚本。

注：Jenkinsfile 是一个包含Jenkins Pipeline定义的文本文件，并被检入源代码控制。

3）Groovy DSL

DSL（Domain-Specific Languages），一个特定领域的语言（功能领域、业务领域）。

Groovy不是DSL，而是通用的编程语言。但Groovy却对编写出一个全新的DSL提供了良好的支持，这些支持都来自于Groovy自身语法的特性。
