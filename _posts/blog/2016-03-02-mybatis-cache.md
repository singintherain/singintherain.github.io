---
layout: post
title: MyBatis缓存机制
description: 简单介绍MyBatis的一级缓存和二级缓存
category: blog
---

一级缓存

基于PerpetualCache的HashMap本地缓存，其存储作用域为当前Session，
当Session flush或者close后，该Session的所有Cache都被清空
一级缓存默认都是开启的

二级缓存

与一级缓存机制相同，默认也是采用PerpetualCache，HashMap存储，不同在于其存储作用域为Mapper(Namespace)，并且可自定义存储源，如Ehcache（这是需要实现序列化方法）

避免二级缓存使用

二级缓存工作在namespace作用域下，使用Mybatis Generator生成的表，每个都有自己独立的namespace
例如users表的namespace为users，orders表的namespace为orders，如果在UserMapper中操作
users表一般是没有问题的，可以用到二级缓存；如果在OrdersMapper操作users表，则使用
的namespace为orders.如果这时在orders下有users的缓存，而用户在users namespace下把数据
做了更改，此时orders namespace下面的users缓存是过期的了，会出现数据不一致的现象。

多表操作肯定用不到二级缓存，因为多个表肯定不在同一个namespace下面


