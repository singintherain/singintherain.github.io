---
layout: post
title: String VS StringBuiler VS StringBuffer
description: 作为最常见的类型String，虽然一直在用，但是这里还是有很多暗坑，需要踩一踩
category: blog
---

在写这个博客的时候，我对String和StringBuilder、StringBuffer的了解仅限于JavaDoc里面的介绍；即String一般直接使用，StringBuilder和StringBuffer是一样的，但是StringBuiler是线程安全的，然后..nothing..

## String

* String对象是一个不可变对象, 对象一经创建就不能被更改
* 它保存在内存的常量区，所有保存在常量区的内容都是线程安全的

  示例：
  ```
  String one = "moon";
  String two = new String("moon");
  String literal = "moon";

  System.out.println(one == two);      # false
  System.out.println(one.equals(two)); # true
  System.out.println(one == literal);  # true
  ```
  当java编译器碰到一个string字面量时，它会先在查看string contant pool中是否有改字面量，如果有则变量会直接指向改字面量；如果没有则在常量池中新建字面量，并且变量指向该字面量；考虑下面的两短解释

  ```
  When the compiler encounters a String literal, it checks the pool to see if an identical String already exists. If a match is found, the reference to the new literal is directed to the existing String, and no new String literal object is created.
  ```

  ```
  In this case, we actually end up with a slightly different behavior because of the keyword "new." In such a case, references to String literals are still put into the constant table (the String Literal Pool), but, when you come to the keyword "new," the JVM is obliged to create a new String object at run-time, rather than using the one from the constant table.
  ```

  ok，那现在考虑第二个赋值语句做了什么:
  1. 查询常量池中是否有"moon"
  2. 发现有new关键字，则需要在堆中创建一个String对象

具体的对象内存对象图，如下所示：
![Simple Design](/images/java/string_detail.png)

  参考图示，个人理解String对象中保存的char[]数组最终也是指向了字面量,但是two中是保存String对象的地址而非字面量的地址，one中保存的是字面量的地址。

## StringBuilder

## StringBuffer

