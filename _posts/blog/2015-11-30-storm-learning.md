---
layout: post
title: storm学习记录
description: 简述storm的工作模式
category: blog
---

实时处理系统(s4, storm)对比直接使用MQ来做好处在哪里？
它替你做了：集群控制、任务分配、任务分发、监控等
而MQ只是个消息中间件，它负责系统间的解耦，一人生产，多人消费

## 工作流程

将自己定义的业务代码(打包成jar包)和topology发布给Nimbus,Nimbus负责将代码发布给
集群并分配worker去执行自定义的topology

## 基本概念

- topology 拓扑图

- 流式传输方式

## 关键类

1. Fields

2. Values

如何做到的失败重新执行




