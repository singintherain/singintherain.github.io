---
layout: post
title: ZooKeeper学习
description: 浅析ZooKeeper原理和使用方法
category: blog
---

![image_of_bird](images/638283-6182c311077445cf.jpg)

ZK对节点的操作create/delete/set/get都提供了同步和异步两种方式

如何完成Request/Response/Watcher的绑定的
客户端的Watcher对象没有序列化
Watcher在服务器端是一次性的，触发一次就失效了

临时节点：客户端创建临时节点成功，退出后临时节点会被删除
永久节点：客户端创建临时节点成功，退出后临时节点不会被删除, 一直存在

使用临时节点的特性，处理master选举业务场景

使用zk做数据持久化，数据量不能太大，例如一些进程或者机器状态信息

客户端对某个节点注册Watcher事件，只需要通知服务端这段信息：
客户端A针对节点Path注册Watcher事件，不用标明针对这个Path注册的是哪种类型的Watcher
服务端保存了一个节点->注册事件的列表，当节点信息发生变更时（不包括子节点信息变更，
子节点信息Watcher事件保存在另外一个集合里），通知给对应的客户端发生了哪种类型的
Watcher事件。
这就造成，一旦节点信息发生变更，就会通知注册客户端，不管节点发生了什么事件

KeeperState      | EventType              | 触发条件                                         | 说明
-----------      | -----------            | -----------                                      | -----------
SyncConnected(3) | Node(-1)               | 客户端与服务端成功建立会话                       | 此时客户端与服务端处于连接状态
                 | NodeCreated(1)         | Watcher监听的对应节点建立                        |
                 | NodeDeleted(2)         | Watcher监听的对应节点被删除                      |
                 | NodeDataChanged(3)     | Watcher监听的对应节点的数据内容发生变更          |
                 | NodeChildrenChanged(4) | Watcher监听的对应节点子节点数据内容发生变更      |
 Disconnected(0) | None(-1)               | 客户端与服务器断开连接                           | 客户端和服务器处于断开连接状态
 Expired(-112)   | Norne(-1)              | 会话超时                                         | 此时客户端会话失效，客户端也会收到SessionExpiredException
 AuthFailed(4)   | None(-1)               | 使用错误的schema做权限检查，或者SASL权限检查失败 | 通常会收到AuthFailedException



