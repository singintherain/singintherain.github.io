---
layout: post
title: volatile VS atomic
description: 比较volatile和atomic原子变量
category: blog
---

volatile声明的变量，可以近似的保证多线程对该变量访问的一致性
atomic声明的变量为原子变量, 原子变量提供了WAS(原子性的比较并设置)操作原语来保证数据的一致性

volatile只所以说是近似的保证数据的一致性，是指当jvm发现存在对该变量的写操作时，所有
对该变量的读操作则堵塞。


