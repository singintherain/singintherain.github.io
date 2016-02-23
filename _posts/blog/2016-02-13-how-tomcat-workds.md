---
layout: post
title: How Tomcat Workds
description: 刨析tomcat的工作原理
category: blog
---

区分SpringMVC中的Fitler和Tomcat的容器嵌套技术

Tomcat中的Pipelining Tasks流水线任务技术和Netty的Pipeline用法

在Tomcat中Servlet之间可以嵌套

## 类加载器

在Tomcat中，每个Servlet都需要有自己的加载器，因为每个Servlet加载时只允许加载其对应的
WEB-INF/目录及其子目录下面的类以及部署在WEB-INF/lib目录下面的类库，每个Servlet的可访问
类路径都是独立的

