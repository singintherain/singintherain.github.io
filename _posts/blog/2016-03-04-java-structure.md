---
layout: post
title: Java数据结构
description: 总结Java数据结构
category: blog
---

## 基础容器

List

ArrayList: 随机存取，插入、删除比较耗时

LinkedList: 顺序存取，插入、删除比ArrayList高效
删除对象时，使用对象的equals方法匹配
提供了类似于Stack栈的操作方法
提供个了类似于Queue的操作方法

Set
不保存重复数据
HashSet: 为快速查找定义的Set，存入的元素必须定义hashCode(), 不维护元素的顺序
LinkdedHashSet: 使用链表维护了元素的插入顺序, 遍历时元素按照插入的顺序显示, 元素必须实现hashCode方法
SortedSet -> TreeSet: 使用红黑树保存元素, 可以提取有序序列, 元素必须实现Comparable接口

Map
键值对
SortedMap -> TreeMap：有序的, 红黑树
HashMap: 散列表, hash(Key)冲突时，直接拉链
LinkedHashMap

Queue: 队列
LinkedList
PriorityQueue: 通过Comparable控制优先级次序

## 并发容器

### ConcurrentHashMap

分段锁

### CopyOnWriteArrayList

写入时复制，容器的安全性在于其低层维护了一个数组，每次增加或者删除一个元素时，都会
在既有数组的基础上复制一个新的数组，然后将新数组替换掉旧数组
即数据一旦发布就是不可变的，要想变就是用新的数组替换掉它
集合的迭代器在迭代阶段不会判断数组的长度是否发生变更，也就不会抛出
ConcurrentModificationException
在对数组的迭代期间没有必要加额外的锁保持同步
在对数组迭代时，如果数组同时做了修改，迭代器仍然是对旧的低层数组做的遍历，而不会感知到新数组

### BlockingQueue

可阻塞的插入和获取操作

