---
layout: post
title: Scala学习手记
description: 联系之前学习Ruby和Java的经验，在scala学习中对比各个语言的语法差异
category: blog
---

scala提供了丰富的对集合操作的函数，而且针对函数式编程在语言层面提供了丰富的定义

但是更重要的是, 需要考虑

`业务逻辑如何进行函数式编程，函数式编程是对OO编程的补充还是颠覆`

## 基本语法

* if else

该语句有返回值，类似于Ruby，不同于Java

* 不需要分号作为语句结束符，这个还是比较爽的

* {}块表达式和赋值

{}内部自成作用域，即内部定义的常量或者变量对外不可见
{}的返回值为最后一个表达式的值

和Ruby一样

* 输入输出

支持C语法的printf，可以直接println，在console中可以readLine读取一行

不支持p函数

* 循环

支持while do语句，不支持Java语法的for(;;)，但是支持for(i <- 1 to n)，这种表达式
更接近于自然描述语言; Ruby中根本不推荐使用for循环，示例：

```Ruby
循环3次
3.times do 
  print "ho"
end
循环9次，带参数
0.upto(9) do |x|
  print x, ''
end
集合遍历，推荐使用each
[1,1,1,1].each { |val| p val; }
如果想控制循环次数，则可以使用while去做，for做不到
```

言归正传，在scala中1 to n是一个表达式，返回的是一个Range，而`i <- 表达式`，是指让
i循环遍历表达式的所有值; 另i没有val和var的类型声明，它的作用域只局限于循环体

`无break和continue定义，这个略坑`

* 多重循环

for(;;)这种是定义多重循环，里面可以有多个分号分割符，每一个分号代表新增一层循环

* yield

yield比较特殊，远没有Ruby的强大，在Ruby中yield是实现block闭包的先决条件。
在scala中仅仅用例作为for推倒式，将结果数据保存在一个集合中

```
for( i <- "hello" ) yield (i + 1).toChar
返回值是：String: ifmmp
这里返回值是个String类型的数据，不是集合。。。Why
```

* 函数

scala支持函数，函数不同于方法，可以参考c++中函数的定义，在Java、Ruby都没有函数定义，
但是它的函数的语法比较坑，竟然需要写等号
函数允许不声明返回值类型，但是如果涉及到递归调用时则需要指定函数返回值类型，因为
它无法自动推倒出函数的类型来坐递归掉调用
不指定return，函数的返回值是最后执行的一条表达式的值

* 带名参数

即传递的参数如果有名称，则不需要和函数定义的参数顺序保持一致

```
def fun1(str : String, left : String = "[]", right: String = ">>") = {
    str + left + right
}

fun1(left = "<<<", str = "abc")
```

* 过程

函数没有=号定义时，就认为是个过程，即没有返回值，他是函数
def fun1() : Unit = {}的`简写`
Unit为空类型，是void.class

```
def box(s: String) {
    println(String)
}
```

## 集合

* Seq

有先后顺序的序列

- IndexedSeq

可以通过下标访问任意元素

-- Vector

是ArrayBuffer的不可变版本，一个带有下标的序列; 低层使用树形结构保存数据，每个节点
可以有不超过32个子节点。访问速度比链表要快很多。

-- Range

整数序列，不存储所有的值，只保存起始值、结束值和增值。

- List

不可变链表，推荐使用递归的方式遍历链表

- LinkedList

可变链表

- Stream

- Stack

- Queue

- Priority Queue

- LinkedList

- Double LinkedList

* Set

无先后顺序、不可重复的集合, 低层是使用hash保存数据，因此查询数据非常快，o(1);

LinkedHashSet
SortedSet

并、交、差、子集

union |
intersect &
diff $~
subsetOf

* Map

一组键值对偶
可以使用 key -> value的方式定义一个键值对
内部数据元素的顺序无关

eq
equal
==

## 类

```
class Point(xc: Int, yc: Int) {
    var x: Int = xc
    var y: Int = yc
    def move(dx: Int, dy: Int) {
        x = x + dx
        y = y + dy
    }
    override def toString(): String = "(" + x + ", " + y + ")";
}
```

这里需要注意几点

* 主构造方法的定义
* 属性xc,yc在整个类体中都是可见的
* 重写的方法必须使用override声明
* 看似Point没有集成任何类，toString应该不会重写别的类的方法，事实上这里有个Predef

类自定义是会自动给非private的var属性，生成类似的getter setter方法，自动给非private
的val属性生成getter方法，但是和Bean的getter ,setter规则不同；使用时，可以object.var
获取值，object.var = anotherValue赋值。

`貌似没有protected访问权限`

属性在没有定义是var还是val时默认为val，在主构造器中

## Object

Object可以理解为单例类, 常用来作为工厂，而且是线程安全的

它可以单独定义，此时类似于枚举
也可以和所控制的类同时定义, 此时如果名称和所操作类一样，则必须处于同一个文件中，
这是被称为伴生类，它可以访问实体类中所有访问权限的属性.

一般用伴生类保存全局使用的变量和方法

## 函数

和Lambda表达式相比较，scala提供了轻量级的匿名函数, 匿名函数在执行时最终会转化为
某个类调用某个方法的形式。

函数定义的几种形式:

```
def fun1(x: Int) = x + 3
def fun1 = ( x: Int ) => x + 3
def fun1 = { x: Int => x + 3 }
def fun1 = new Function1[Int, Int] {
    def apply(v1: Int): Int = v1 + 1
}
```

第一种方式比较适用普通场景下的函数定义
第二种和第三种方式在函数作为参数传递或者作为其他函数的返回值场景下比较常用定义函数

函数作为参数时的声明：

```
def fun1(f: Int => Boolean)
```

```
scala中的block，无参数无返回值的函数
{ println("hello"); var i = 0; i = i + 1; }
这段函数可以直接传递给
def runInThread(block: () => Unit) {
    new Thread {
      override def run() { block() }
    }.start()
}

runInThread{ println("hello"); var i = 0; i = i + 1; }
```


## case class

使用时不需要new，只提供主构造器，属性是public的，可以认为是普通类的简写方式
它可以定义方法
这样的类和纯Bean很像, 使用起来比较方便;编译器重载了equals(==)方法，只比较内部的
数据是否相同，toString方法同样也不重载

case class Tiger(name: String, location: String)

## 偏函数

一种更灵活的函数调用方式，它把对函数的调用前置条件加入到函数的定义时刻
使用这可以借助orElse和andThen来组织函数的调用方式

```
val list = List(4, 6, 7, 8, 9, 13, 14)
val partialFunction1: PartialFunction[Int, Int] = {
    case x: Int if x % 2 == 0 => x * 3
}
val partialFunction2: PartialFunction[Int, Int] = {
    case y: Int if y % 2 != 0 => y * 4
}
val result = list.collect(partialFunction1 orElse partialFunction2)
```

fun1 orElse fun2 andThen fun3

## 正则匹配

value match {
  case 
}

支持正则匹配的功能更强大的判断语句

```
复杂的条件转移语句在设计模式中经常被诟病，这里思考scala强化条件判断语句功能的意义
```

`Scala是弱类型语言，那么存在编译器可能发现不了的错误, 如某个变量并不存在对应的方法,
这样就得类似于Ruby的Ducky编程了`

`scala的隐式参数转换和unapply特性，就TM的扯淡了`

* 守卫

可以在case中增加条件，如

```
ch match {
  case '+' => sign = 1
  case '-' if Charater.isDigit(ch) => digit
  case _ => ...
}
```

* 值匹配

可以匹配任何类型的值，而不仅仅是数字，如

```
color match {
  case Color.RED =>
  case Color.BLUE => 
}
```

* 变量匹配

如果case关键字后面跟着的是一个变量名，则匹配的表达式会赋值给那个变量
case后面的变量必须以小写字母开头，大小字母开头的会认为是常量；
如果有一个`小写字母`开头的`常量`，则需要将它包在反引号中;

* 类型匹配

```
obj match {
  case x: Int => x
  case s: String => ...
  case _: BigInt => ...
  case _ => ...
}
```
* 匹配数组

```
arr match {
  case Array(0) => "0"
  case Array(x, y) => x + " " + y
  case _ => "something else"
}
```

内部使用的是提取器功能, 带有从对象中提取值的unapply和unapplySeq方法的对象，
例如，上面的case Array(x, y),是借助的Array的伴生对象定义了一个unapplySeq方法，
该方法被调用时，以`被执行匹配动作的表达式为参数`, 例如Array.unapplySeq(arr)生成一个序列的值，第一个值赋给x,第二个值赋给y

* 样例类

case class

样例类是一个特殊的类，他们经过优化被用于模式匹配;
样例类在匹配的时候，如果匹配到指定的类型，则会将属性值绑定到变量上；

例如：

```
amt match {
  case Dollar(v) => "$" + v
  case Currency(_, u) => "Oh noes, I got " + u
  case Nothing => ""
}
```

样例类在声明的时候有以下事情发生：

- 为构造器中的每个参数都成为val常量（除非它被显示的声明为var，一般不建议这么做)
- 在伴生对象中提供了apply方法，使得不用使用关键字new就可以构造出相应的对象，如Dollar(24)
- 提供unapply方法，让模式匹配可以工作
- 自动生成toString, equals,hashCode和copy方法

* 密封类sealed class

为了将所有的同类型的样例类放在一块儿定义，可以将所有的样例类的公共父类定义为密封类，
密封类的所有子类都必须与该密封类相同的文件中定义

样例类可以用来模拟枚举

```
sealed abstract class Color
case object Red extends Color
case object Yellow extends Color
case object Blue extends Color

color match {
  case Red => ""
}
```

函数中只有模式匹配，例如

```
def receive = {
  case "test" => log.info("receive test")
  case _      => log.info("receive unknown message")
}
```

* 偏函数

被包在花括号内的一组case语句是一个偏函数，一个并非对所有输入值都有定义的函数，
它是PartialFunction[A, B]的一个实例，A是参数类型，B是返回类型。

## 注意

### main方法的定义

main方法必须定义在一个object中才会自动执行，定义在class中没有效果

main方法可以定义在trait中，但是该train只有被object类混入才有意义

### trait VS abstract class

trait和abstract class类似，可以使用匿名对象，但是trait支持多继承

### 变量函数

无参数，无返回值的函数可以简化使用

```
object App {
  def startUp {
    println("app startUp")

  }
}
trait Component {
  val server = {
    App.startUp
  }
}
object ParamFunc {
  def main(args: Array[String]) {
    new Component {}.server
  }
}
```

### val var 数据

这些数据应该是无类型的，针对无类型的对象，如果操作? 例如：

val duck = fun()
fun()函数在定义时声明的返回值类型为Duck，那么在后续的对duck的使用中，可以直接
duck.guagua()调用吗，guagua()为Duck类的方法






