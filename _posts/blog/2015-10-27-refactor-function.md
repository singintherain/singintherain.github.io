---
layout: post
title: 重构之重新组织函数
description: 重构-改善既有代码的设计
category: blog
---

## 提炼函数

### 目的

* 减小函数的粒度，增加函数的复用性
* 避免过长的函数，增强函数的表述性

### 方法

#### 思考方法命名

如果不能找到一个合适的名字，就不要单独拆出方法

#### 局部变量问题

* 无局部变量

```
void printOwing() {
  Enumeration e = order.elements();
  double outstanding = 0.0;

  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    outstanding += each.getAmount();
  }

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
}
```

```
void printOwing() {
  Enumeration e = order.elements();
  double outstanding = 0.0;

  printBanner();

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    outstanding += each.getAmount();
  }

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
}

void printBanner() {
  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")
}
```

* 有局部变量，仅用来输出

```
void printOwing() {
  Enumeration e = order.elements();
  double outstanding = 0.0;

  printBanner();

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    outstanding += each.getAmount();
  }

  printDetail(outstanding);
}

void printBanner() {
  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")
}

void printDetail(double outstanding) {

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);

}
```

* 有局部变量，但是变量被修改，只有一种处理

```
void printOwing() {
  Enumeration e = order.elements();
  double outstanding = getOutStanding();

  printBanner();
  printDetail(outstanding);
}

void printBanner() {
  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")
}

void printDetail(double outstanding) {

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);

}

float getOutStanding() {
  double outstanding = 0.0;

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    outstanding += each.getAmount();
  }

  return outstanding;
}
```

* 有局部变量，但是变量被修改，只有多种处理


```
void printOwing(double previousAmount) {
  Enumeration e = order.elements();
  double outstanding = previousAmount * 1.2;

  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    outstanding += each.getAmount();
  }

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
}
```

```
void printOwing(double previousAmount) {
  Enumeration e = order.elements();
  double outstanding = previousAmount * 1.2;
  
  outstanding = getOutStanding(outstanding);

  printBanner();
  printDetail(outstanding);
}

void printBanner() {
  // print banner
  System.out.println("***************")
  System.out.println("*****Customer owes******")
  System.out.println("***************")
}

void printDetail(double outstanding) {

  // print details
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);

}

float getOutStanding(float initialValue) {
  double result = initialValue;

  // caculate outstanding
  while(e.hasMoreElements()) {
    Order each = (Order) e.nextElement();
    result += each.getAmount();
  }

  return result;
}

```

## 函数内联

### 动机

简短的函数能够明显的表达动作意图，但是如果函数过于简单，而且不会在多处使用，则
直接写内容就可以，不需要单独抽离为一个函数.如下：

```
int getRating() {
  return (moreThanFiveLateDeliveries() ? 2 : 1)
}

boolean moreThanFiveLateDeliveries() {
  return numberOfLateDeliveries < 5;
}
```

## 临时变量内联

临时变量只被简单的使用了一次，则不需要定义，例如：

```
double basePrice = anOrder.basePrice();

return basePrice;

可直接写为

return anOrder.basePrice();
```

## 以查询取代临时变量

变量赋值有着复杂的计算逻辑，为了增加计算逻辑的可维护性，可将该逻辑抽离成方法

`这里先不要担心性能问题`

```
double basePrice = quantity * itemPrice;

if(basePrice < 100) {
   return basePrice * 0.95;
} else {
   return basePrice * 0.98;
}
```

```
if(basePrice() < 100) {
   return basePrice() * 0.95;
} else {
   return basePrice() * 0.98;
}

double basePrice() {
    return quantity * itemPrice;
}

```

## 引入解释性变量

上面的方法，好不容易将临时变量删除，这里又提出一个引入变量的方法。
这种方法不常使用，如果方法名本身表述性很强，而且没有什么性能问题，则不需要这么做。

```
if((platform.toUpperCase().indexOf("MAC") > -1 ) &&
   (browser.toUpperCase().indexOf("IE") > -1) &&
   (wasInitialized() && resize > 0)) {
   .....
}

final boolean isMacOs = (platform.toUpperCase().indexOf("MAC") > -1 );
final boolean isIEBrowser = (browser.toUpperCase().indexOf("IE") > -1);
final boolean wasResized = (resize > 0);

if(isMacOs && isIEBrowser && wasResized && wasInitialized()) {
....
}
```

## 解剖临时变量

临时变量不应该多次赋值去表达不同的意思

```
double temp = 2 * (height + width);
System.out.println(temp);

temp = height * width;
System.out.println(temp);
```

```
double perimeter = 2 * (height + width);
System.out.println("周长:" + perimeter);

double area = height * width;
System.out.println("面积:" + area);
```

## 移除对参数的赋值

函数的参数如果是值传递，则函数内部对其修改并返回没有意义
函数的参数如果是引用传递，则可以在函数内容对其修改

## 以函数对象取代函数

函数耦合过渡，将不属于该对象处理的业务耦合了进来，例如下单的计算价格的方法需要有
单独的价格计算器取处理，而非仅仅在一个Order的calculatePrice()就解决了.

## 替换算法

将某个算法表述的更清晰，例如

```
String foundPerson(String[] people) {
  for(int i = 0; i < people.length; i++) {
    if(people[i].equals("Don")) {
      return "Don";
    }
    if(people[i].equals("John")) {
      return "Don";
    }
    if(people[i].equals("Kent")) {
      return "Don";
    }
  }

  return "";
}
```

```
String foundPerson(String[] people) {
  List candidates = Array.asList(new String[] {"Don", "John", "Kent"});

  for(int i = 0; i < people.length; i++) {
    if(candidates.contains(people[i]))
      return people[i];
  }

  return "";
}

```

