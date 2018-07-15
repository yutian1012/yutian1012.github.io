---
title: log4j源码分析
tags: [log4j]
---

Log4J支持两种配置文件：properties文件和xml文件。Configurator解析配置文件，并将解析后的信息添加到LoggerRepository中。LogManager最终将LoggerRepository和Configurator整合在一起。

