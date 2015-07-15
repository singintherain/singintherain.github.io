---
layout: post
title: 构建webdriver集群
description: 使用集群执行大批量的测试用例，提高测试执行效率
category: blog
---

## 准备

下载selenium-server-standalone

一个Grid集群有一个Hub和多个Node组成；

启动Hub的命令如下：

```
java -jar selenium-server-standalone-2.44.0.jar -role hub
```

启动Node的命令如下：

```
java -jar selenium-server-standalone-2.44.0.jar -role node  -hub http://localhost:4444/grid/register
```

## 多线程执行测试任务

只有多线程执行任务时，才能发挥Grid集群的效率。执行策略：

* 多线程执行每个任务
* 每个任务有一个webDriver对象
* 每个webDriver对象需要都建立远程Hub的连接

## 执行场景

1. 1个hub, 2个node，大小为3的线程池，任务6个

每次最多可以打开3个浏览器，这个是由线程池的大小决定的

2. 调大线程池为5

最多同时打开5个浏览器

3. node默认配置

在启动node时，每个node默认可以打开11个浏览器，包括5个chrome，5个firefox，1个IE

## 错误

1. 线程池大小为7，node个数为1，任务数为10

当node打开5个浏览器时，停止主进程，此时hub一直提示两条：

has no free slots

出现这种情况的原因是，首先打开的浏览器已经占满了node所有的slot，而且所有的浏览器没有
正常关闭，这是slot一直没有释放。hub这里一下就接到了7个任务，但是它只能分配5个任务，
造成有2个任务一直未执行，hub会循环的从node中查询是否有可用的slot。

* 如果手动关闭2个浏览器，node并不会delete这个sessioin(也就是浏览器)，此时如果重新启动进程，
则又发送了7个任务给hub，hub此时会有9个任务一直处于has no slot的状态；

* 重启node，则hub会重新执行任务，但是每个任务都不能正确执行，因为此时主进程已经关闭，node
仅仅会打开一个浏览器，其余的action都无法执行的。


node启动的时候一定要制定timeout值，hub当检测到一个node在timeout的时间内没有request发送时，会主动release，
就不会造成上述的一直has no slot；默认情况下是300秒。

