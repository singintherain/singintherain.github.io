---
layout: post
title: Builder构造器设计模式
description: 介绍内部类、Builder构造器、不可变对象的概念
category: blog
---

## 内部类

内部类最大的好处是将逻辑相关的类组织在一起，在项目组织中，尽量不要将业务逻辑相关的类的定义
散落在不同的地方，否则会降低代码的可维护性。

定义，如示例, 定义一个集合的迭代器

业务分析：

* 针对集合对象操作
* 每次迭代时需要记录迭代的位置，因此集合和迭代器有着1:1的关系

代码实现：

```
public class Sequence {
    private Object[] items;
    private int next = 0;

    public Sequence(int size) {
        items = new Object[size];
    }

    public void add(Object o) {
        items[next++] = o;
    }

    private class SequenceSelector {
        private int i = 0;
        public boolean end() { return i == items.length;  }
        public Object current() { return items[i];  }
        public void next() { if(i < items.length ) i++;  }
    }

    public SequenceSelector selector() {
        return new SequenceSelector();
    }

    public static void main(String[] args) {
        Sequence sequence = new Sequence(5);
        for( int i = 0; i < 5; i++ ) {
            sequence.add(Integer.toString(i));

        }

        SequenceSelector selector = sequence.selector();

        while(!selector.end()) {
            System.out.print(selector.current());
            selector.next();
        }
    }
}
```

通过示例，注意以下几点：

* 内部类的普通方法可以直接访问外部类的方法和字段，无论是私有的还是公有的
* 必须通过外部类对象才能获取内部类对象，虽然没有显示的引用关系，但是内部类对象
会隐式的获取一个外部类对象的引用(`此时的外部类并不是static的`); 如果是分开定义的
两个类，则需要手动的维护对象之间的引用关系；这里放给编译器来做，很明显，内部类
和外部类有着很强的关联关系
* 你可以通过在外部类定义方法来获取内部类对象，也可以在直接实例化内部类对象
* 内部类不能有static属性, 所有的static属性必须放在外部类；这个强制的要求可以帮助理解
内部类和外部类的关系，在具体场景使用中应该思考这个约束条件

```
Outer outer = new Outer();
Outer.Inner inner = outer.new Inter();
```

内部类的访问权限控制和普通类的访问权限控制是一样的，当使用private或者protected
修饰内部类时，不能在外部通过外部类对象的方式来实例化内部类对象

### 匿名内部类

这里有个使用非常普遍的内部类，即匿名内部类, 定义如下：

```
public class AnonymizeClass {
    private abstract class Student {
        private int age;
        public Student(int age) {
            this.age = age;
        }
        abstract public void hello();
    }

    public Student student() {
        return new Student(3) {
            public void hello() {
                System.out.print("hello");
            }
        };
    }

    public static void main(String[] args) {
        AnonymizeClass anonymizeClass = new AnonymizeClass();
        AnonymizeClass.Student student = anonymizeClass.student();
    }
}

```

如果现在匿名内部类中使用外部定义的参数，则必须要求外部参数是final修饰的, 但是如果是
构造器中使用，这没有这个约束
在匿名内部类定义的时候想执行实例初始化的逻辑操作时，可以如下操作：

```
        return new Student(3) {
            { println('initialize'); }
            public void hello() {
                System.out.print("hello");
            }
        };
```

### 嵌套类

如果想消除内部类对象和外部类对象的隐式引用关系，可以将内部类声明为static

* 嵌套类对象实例化时不需要有外部类对象
* 嵌套类对象不能非静态的外部类对象, 由于嵌套类完全看不到外部类对象，使得它就像一个
独立的static方法，它必须遵守static的所有约束

### 为什么使用内部类

* 每个内部类都能独立地继承自一个接口的实现，所以无论外部类是否已经继承了某个实现，
对内部类都没有影响
* 可以处理多继承问题，因为内部类可以看到外部类的所有属性和方法，而外部类又能独立的
继承实现，因此内部类相当于可以扩展外部类的功能，如下：

```
class D {
    private int message;
}
abstract E {
    abstract void fun();
}
class Z extends D {
    { message = "outer message"; }
    class F extends E {
        void fun() {
          System.out.println(message);
        }
    }
  E makeE { return new E() {}; }
}

这里Z.F对象可以使用D类中定义的各种操作方法
```

- 闭包和回调

闭包是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。
闭包一般用来回调，回调的价值在于在运行时动态决定需要调用什么样的方法。
内部类是面向对象的闭包
有关闭包和回调，我会在之后开专题博客介绍。

## Builder构造器

对象实例化有几种方式

1. 静态工厂和构造器方式

当有大量的多参数时，不易扩展

2. JavaBeans模式

即维护一个默认构造方法，给所有的属性定义getter和setter方法，看样子可以解决问题，
但是带来了对象不一致的风险；
对象实例化赋值操作分布在不同的步骤中，在构造过程中JavaBean可能处于不一致的状态，
尤其在多线程中，使得需要额外付出努力保证它的线程安全。

3. Builder模式

目标：使用一种简单易读的方式构造出一个不可变的对象, 示例如下：

```
public class Student {
    private final int age;
    private final String name;

    private Student(Builder builder) {
        this.age = builder.age;
        this.name = builder.name;
    }

    public static class Builder {
        private int age = 0;
        private String name = "";

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Student build() {
            return new Student(this);
        }
    }

    public static void main(String[] args) {
        Student student = new Student.Builder().age(3).name("kitty").build();
    }
}
```

示例中使用嵌套类实现了基本你的Builder模式，实例化出来的对象是不可改变的，因为变量
都是final类型的

## 不可变对象

这是我在学习scala函数式编程的时候开始注意的一个概念，它一再强调不可变对象的重要性，
尤其是在并发编程领域（这个比较好理解)；并发领域最难控制的就是数据共享，如果所有的
对象都是不可变的，那就没有复杂的同步和锁问题；
但是如果所有的对象都是不可变的，则就意味着在不同的状态都需要更多的对象来维护，这个
性能问题如何解决，对象的数量应该会急剧膨胀吧。

