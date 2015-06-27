---
layout: post
title: 依赖倒置原则
description: 敏捷开发原则之依赖倒置原则：决不能让国家的重大利益依赖于那些会动摇人类意志的众多可能性
category: blog
---

## why

所有结构良好的面向对象设计的系统都是分层次的，每个层次通过一个定义良好的、受控的接口向外提供了一组内聚的服务。

But，怎么样组织服务之间的依赖关系是一个简单而又复杂的问题，依赖倒置原则为此类问题解决方案的制定提供了一个准则。

简单的方案设计如下：
![Simple Design](/images/design_pattern/simple_design.png)

**缺陷**

* 依赖的传递性导致系统非常的脆弱，当有某个模块的服务发生改变时，会影响在调用链上得所有服务。
 - 每次低层的业务模块代码修改，都有可能需要花费很大的代价去逐次修改所有的调用者.
* 很难修改，影响太大，由于无法预测每次变更会导致什么样的问题，使得系统无法维护.
* 服务或者模块之间严重耦合，导致PolicyLayer和MerchanismLayer服务不能被重用.

## what

* 高层模块不能依赖于低层模块，二者都应该只依赖于抽象
* 抽象不能依赖于实现，实现应该依赖于抽象

实现依赖于抽象，它需要实现抽象定义的功能方法，所以称之为依赖倒置

遵循该原则的架构模型如下：

![abstract_layer](/images/design_pattern/abstract_layer.png)

## how

* 任何类不能从派生之一个具体的类
* 任何变量不能持有一个指向具体类的引用
* 任何方法不能复写基类已经实现的方法(在里氏替换原则中也适用)

## Example

考虑一个简单的示例：从读取键盘输入数据并输出到打印机

**Design1**

![first copy design](/images/design_pattern/copySystem.png)

业务功能涉及三个子系统或者模块，Copy负责调用Read Keyboard和Write Printer。代码如下：

```
Class Keyboard
{
  public int read(){...}
}

class Printer
{
  public void write(){...}
}

Class Copy
{
  private Keyboard keyBoard;
  private Printer printer;
  public Copy(Keyboard keyBoard, Printer printer){...}

  pubic void run()
  {
    while(keyBoard.read){
      printer.write();
    }
  }
}
```

很明显，Copy这个业务模块不能在输入不是键盘的业务中重用，而且不能将输出打印到磁盘。

**Design2**

依赖于抽象，而不依赖于实现，针对业务做更高一级的抽象

![abstarct_logical](/images/design_pattern/dependency_abstract.png)

代码如下：

```
interface Reader
{
  public int read();
}

interface Writer
{
  public void write();
}

Class Copy
{
  private Reader reader;
  private Writer writer;
  public Copy(Reader reader, Writer writer){...}

  pubic void run()
  {
    while(reader.read){
      writer.write();
    }
  }
}

Class Keyboard implements Reader
{
  public int read(){...}
}

class Printer implements Writer
{
  public void write(){...}
}

```

## Conlusion

写代码要采用从上往下的模式，当需要使用某个服务的时候，先定义服务的抽象层，不要直接去写实现.

做好上层设计，即业务建模。

## FAQ

并不是所有的类在设计的时候都需要先抽象，如何把控。

实际上，在提出这个问题的时候，就已经是从下向上编程了，不要孤立的去考虑一个类的设计，要充分考虑业务的上下文场景。
